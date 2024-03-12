---
title: apt等各种源列表（持续更新）
categories: [知识库]
date: 2024-01-28 19:12:01
keywords: apt源
tags:
    - apt源
    - docker源
---

# 操作系统源

**使用源时务必注意版本**

若你的系统不是最新版，可以使用*snullp*大佬开发的[配置生成器](https://mirrors.ustc.edu.cn/repogen/)

## 中科大debian apt源（最新debian12 bookworm版）

**我个人觉得中科大源更好些，且在中科大下载iso也比清华快些，如需使用清华源，将ustc替换为tuna.tsinghua即可，其他配置同理**

```shell
deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
```

<!-- more -->

## 中科大ubuntu apt源（最新ubuntu24.04 noble源）

```shell
deb https://mirrors.ustc.edu.cn/ubuntu/ noble main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-security main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-security main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-updates main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-updates main restricted universe multiverse

deb https://mirrors.ustc.edu.cn/ubuntu/ noble-backports main restricted universe multiverse
deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-backports main restricted universe multiverse

## Not recommended
# deb https://mirrors.ustc.edu.cn/ubuntu/ noble-proposed main restricted universe multiverse
# deb-src https://mirrors.ustc.edu.cn/ubuntu/ noble-proposed main restricted universe multiverse
```

## 中科大docker ce源（安装docker的源，一键自动安装）

```shell
curl -fsSL https://get.docker.com -o get-docker.sh && sudo DOWNLOAD_URL=https://mirrors.ustc.edu.cn/docker-ce sh get-docker.sh
```

# 软件源

## 中科大docker hub源

```shell
vim /etc/docker/daemon.json

# 添加以下内容
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}

# 用root用户或具有sudo权限的用户加sudo后运行
# 重启docker
systemctl restart docker
```

## 中科大npm源（反代的https://registry.npmjs.org/）

```shell
# 编辑~/.npmrc
registry=https://npmreg.proxy.ustclug.org/

# 临时使用中科大源安装软件包
npm --registry https://npmreg.proxy.ustclug.org/ install 包名
```

## 清华大学PyPI源

中科大PyPI源公告：
由于 PyPI 源日益增长的空间与 mirror 磁盘空间非常有限的矛盾和用户报告的 PyPI 源的诸多问题，以及考虑到 PyPI 源的资源占用对其他镜像服务质量的影响，我们暂时移除了对 PyPI 的本地镜像。即日起至新的 PyPI 源镜像方案实施前，本站 PyPI 源的 HTTP 协议访问将重定向到 TUNA PyPI 源；PyPI 源的 RSYNC 同步方式停止提供。

```shell
# 临时使用
pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple package

# 使用清华镜像站来升级 pip
pip install -i https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple pip -U
pip config set global.index-url https://mirrors.tuna.tsinghua.edu.cn/pypi/web/simple
```

## 中科大Qt镜像

[从中科大镜像下载Qt在线安装器](https://mirrors.ustc.edu.cn/qtproject/official_releases/online_installers/)

使用以下两种方式之一在安装器中配置使用科大源下载 Qt：

1. （推荐）新版本的安装器（4.0.1-1 后）支持 --mirror 命令行参数。在命令行中执行安装器，添加 --mirror https://mirrors.ustc.edu.cn/qtproject 参数。例如 Windows 下执行当前目录的安装器的命令为 .\qt-unified-windows-x86-online.exe --mirror https://mirrors.ustc.edu.cn/qtproject；

2. 或在启动安装器后在设置中禁用默认源，添加新源 http://mirrors.ustc.edu.cn/qtproject/online/qtsdkrepository/linux_x64/root/qt/ （其他版本注意更改地址）。

## github release镜像（仅部分仓库）

[清华大学](https://mirrors.tuna.tsinghua.edu.cn/github-release/)

[中科大](https://mirrors.ustc.edu.cn/github-release/)