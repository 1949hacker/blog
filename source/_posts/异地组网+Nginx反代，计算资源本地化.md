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
# 将 51820 端口的 UDP 流量转发到内网服务器 192.168.1.200 的 51820 端口
# [其中内网服务器192.168.1.200的IP根据自己的情况替换，内网和云服务器的端口也都是可以替换的]
# iptables -t nat -A PREROUTING -p udp --dport 51820 -j DNAT --to-destination 192.168.1.200:51820
# 允许转发到 192.168.1.200:51820 的 UDP 流量
# iptables -A FORWARD -p udp -d 192.168.1.200 --dport 51820 -j ACCEPT

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
