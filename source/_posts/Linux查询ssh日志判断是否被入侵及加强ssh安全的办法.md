---
title: Linux查询ssh日志判断是否被入侵及加强ssh安全的办法
categories: [知识库]
comments: true
date: 2024-03-05 17:14:44
keywords:
    - Linux
    - ssh
    - 网络安全
tags:
    - Linux
    - ssh
    - 网络安全
    - ssh爆破
---

## 查询ssh登陆日志的办法


`journalctl _COMM=sshd`可以查看sshd所有日志

```shell
# 查询ssh登陆失败总次数
journalctl _COMM=sshd | grep Failed |wc -l

#查询ssh登陆成功总次数
journalctl _COMM=sshd | grep Accepted |wc -l

#查询被多少不同IP爆破了
journalctl _COMM=sshd | grep "Failed" | awk '{print $(NF-3)}' | sort -u | wc -l

# 列出爆破你的IP
journalctl _COMM=sshd | grep "Failed" | awk '{print $(NF-3)}' | sort -u

# 列出成功登陆的IP
journalctl _COMM=sshd | grep -E "Accepted" | awk '{match($0, /([0-9]+\.){3}[0-9]+/); print substr($0, RSTART, RLENGTH)}' | sort -u
```

<!-- more -->

## 加强ssh安全的办法

1. 禁用root密码登陆

编辑`/etc/ssh/sshd_config`文件，默认会有`PermitRootLogin prohibit-password`
保持默认的`prohibit-password`可使root用户密码登陆时一直提示密码错误，而不是直接提示禁止密码登陆，起到迷惑入侵者的作用

还可配置`PasswordAuthentication no`使普通用户也无法通过密码登陆并提示需要publickey

2. 修改ssh服务端口

编辑`/etc/ssh/sshd_config`文件，将默认的`# Port 22`修改为`Port 20022`或其他符合规定的端口
修改端口后ssh连接时需要使用`-p 20022`指定端口
使用scp上传文件时也需要在scp命令后紧跟着使用`-P 20022`指定端口

3. ssh连接时使用指定的key进行验证

ssh客户端在连接服务器时可以使用`-i`参数使用指定的id_rsa私钥进行连接，需要注意，id_rsa私钥文件权限需为600，也就是仅所有者可读写
使用root指定私钥访问20022端口的服务器示例：
```shell
ssh -i ./myKeys/id_rsa -p 20022 root@192.168.2.254
```

修改`sshd_config`文件后需要重启ssh服务，debian通常使用`systemctl restart ssh`即可