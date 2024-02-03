---
title: apt等各种源列表（持续更新）
categories: [知识库]
date: 2024-01-28 19:12:01
keywords: apt源
tags:
    - apt源
    - docker源
---

## 清华大学debian apt源（最新debian12 bookworm版）

```shell
deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://mirrors.tuna.tsinghua.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
deb-src https://mirrors.tuna.tsinghua.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
```

## 清华大学docker ce源（安装docker的源，一键自动安装）

```shell
curl -fsSL https://get.docker.com -o get-docker.sh && sudo DOWNLOAD_URL=https://mirrors.tuna.tsinghua.edu.cn/docker-ce sh get-docker.sh
```

## 清华大学docker hub源

```shell
vim /etc/docker/daemon.json

# 添加以下内容
{
  "registry-mirrors": ["https://docker.mirrors.tuna.tsinghua.edu.cn/"]
}

# 用root用户或具有sudo权限的用户加sudo后运行
# 重启docker
systemctl restart docker
```