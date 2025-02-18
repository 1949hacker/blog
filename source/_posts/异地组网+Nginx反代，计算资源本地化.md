---
title: 异地组网+Nginx反代 | 计算资源本地化
comments: true
categories: [知识库]
date: 2025-02-17 16:59:52
keywords:
	- wireguard
	- 局域网
	- 云计算
	- 计算资源本地化
	- 异地组网
	- Nginx
	- 反向代理
tags:
	- wireguard
	- 局域网
	- 云计算
	- 计算资源本地化
	- 异地组网
	- Nginx
	- 反向代理
---

> 本次部署的目的是利用本地高性能的计算资源和云服务器畅通无阻的公网，实现无公网环境也可正常部署业务。

## 本文所用到的技术如下

|名称|作用|软件|
|---|---|---|
|单向异地组网|将云服务器和本地服务器组成局域网，因本地服务器没有公网，所以由云服务器建立UDP连接实现组网|WireGuard|
|反代|将对云服务器发起的访问根据配置的业务转发到本地服务器进行处理|Nginx|
|网络转发|配合wireguard实现对网络流量的路由和转发，确保组网的通畅|iptables|

<!-- more -->

## 单向异地组网

> 2025.2.18补充：组网成功后大概十几分钟，被运营商墙了，太狗了，因为wireguard走的是UDP协议，被运营商检测到了。目前有两个思路，找运营商加个公网IP或者想其他办法组网
> 2025.2.18 10:55记录思路：使用udp2raw将UDP流量封装到TCP流量中，并伪装成HTTPS
> 2025.2.18 10:57补充，尝试udp2raw能打通后先用着，如果再次被封则换为xtls-rprx-vision透明代理UDP流量，实现最强HTTPS伪装

| 特性 | `udp2raw` | `xtls-rprx-vision` |
|------|----------|----------------|
| **工作原理** | 把 UDP 伪装成 TCP 或 ICMP，进行 NAT 穿透 | 通过 Xray Vision 协议将流量伪装成 HTTPS/TLS |
| **支持协议** | 任何 UDP 流量（WireGuard、IPSec、OpenVPN等） | 任何 TCP/UDP 流量（支持 UDP over TCP） |
| **流量伪装** | 可伪装为 TCP（HTTPS、SSH）或 ICMP | 伪装成合法 HTTPS 流量 |
| **加密方式** | XOR、AES（轻量级加密，防止 DPI 识别） | XTLS（高级 TLS 伪装，难以被识别） |
| **端口** | 可伪装成 443（HTTPS）等，但仍是伪 TCP | 直接监听 443 端口，真实 HTTPS 伪装 |
| **数据包特征** | 模拟 TCP，但仍然是 UDP 数据包封装 | 真实 HTTPS 流量，运营商难以检测 |
| **防 DPI** | 基本防御，部分 DPI 仍能检测出伪 TCP | 最高级别防御，难以被识别 |
| **延迟** | 低，基本无影响 | 略高，取决于 TLS 处理开销 |
| **适用场景** | 适用于 UDP 被封但 TCP 可用的环境 | 适用于 DPI 严格的环境，如中国大陆或企业网络 |
| **是否需要证书** | 不需要 | 需要 TLS 证书（Let's Encrypt 等） |
| **稳定性** | 高，几乎无性能损耗 | 高，但 TLS 可能增加 CPU 负担 |
| **适合长期使用** | 可以，但可能被进一步分析 | 更适合长期使用，难以封锁 |

### 服务器部分

```shell
# 安装wireguard
apt install wireguard -y

# 生成私钥公钥到/etc/wireguard/目录下
wg genkey | tee /etc/wireguard/server_private.key | wg pubkey | tee /etc/wireguard/server_public.key
# 查看私钥公钥，稍后需要写入到配置文件中
cat /etc/wireguard/server_private.key
cat /etc/wireguard/server_public.key
```

在`/etc/wireguard/`下创建一个配置文件，例如`wg0.conf`，按如下所述写入配置：

```conf
[Interface]
Address = 10.0.0.1/24 # 服务器的局域网IP，/24是前缀，对应255.255.255.0
ListenPort = 26666 # 端口协议是UDP，端口可以自定义
PrivateKey = 云服务器私钥 # 刚刚生成的服务器私钥
DNS=8.8.8.8
MTU=1420

# 这部分是配置的iptables的转发，其中eth0替换为云服务器公网的网卡名称
# 原理：
# 允许wg0的流量通过iptables转发到eth0
# iptables -A FORWARD -i wg0 -j ACCEPT
# 将eth0上的流量进行源地址伪装，让对方以为来自局域网的流量是从公网IP发出的
# iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# 端口启动时执行，其中-A是添加
PostUp   = iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
# 端口关闭时执行，其中-D是删除
PostDown = iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = 内网服务器公钥 # 稍后在内网服务器生成的公钥
# 允许内网服务器连接，IP是10.0.0.2，前缀32对应255.255.255.255
# 作用是仅允许这一个IP连接。可以根据需要自行组合IP段和前缀
AllowedIPs = 10.0.0.2/32
```

```shell
# 编辑完配置文件后启动服务
# 其中wg-quick是wireguard的启动工具，@连接wg0，是配置文件的名称
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

### 内网服务器部分

```shell
# 与服务器一致，生成公钥私钥，只是名称是client开头
wg genkey | tee /etc/wireguard/client_private.key | wg pubkey | tee /etc/wireguard/client_public.key
# 查看公钥私钥，准备写入配置文件
cat /etc/wireguard/client_private.key
cat /etc/wireguard/client_public.key

```

在`/etc/wireguard/`下创建一个配置文件，例如`wg0.conf`，按如下所述写入配置：

```conf
[Interface]
Address = 10.0.0.2/24 # 10.0.0.2则是内网服务器在组网内的IP地址
PrivateKey = 内网服务器私钥

[Peer]
PublicKey = 云服务器公钥
AllowedIPs = 10.0.0.1/32 # 前缀32限制只允许云服务器IP连入
Endpoint = 云服务器IP:26666 # 指向云服务器IP和端口
PersistentKeepalive = 25
```

```shell
# 编辑完配置文件后启动服务
# 其中wg-quick是wireguard的启动工具，@连接wg0，是配置文件的名称
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```

### 共同配置部分

在内网和云服务器均开启转发

```shell
echo "net.ipv4.ip_forward=1" | sudo tee -a /etc/sysctl.conf
sysctl -p
```

## 反向代理
