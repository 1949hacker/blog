---
title: Debian x86_64平台搭建arm docker环境
categories: [知识库]
date: 2023-09-12 21:51:36
keywords: Docker
tags:
    - Docker
    - ARM
---

在Debian x86_64平台搭建Docker arm环境的方法

<!-- more -->

```shell
# 安装QEMU用户模式工具：QEMU将用于在x86_64主机上模拟ARMv8架构
apt install qemu binfmt-support qemu-user-static

# 启用QEMU用户模式支持
update-binfmts --enable qemu-arm
update-binfmts --enable qemu-aarch64
```

使用以上命令便可搭建Docker的arm环境，再使用如下命令即可测试环境是否可以正常使用

```shell
# 拉取debian armv8镜像
docker pull arm64v8/debian

# 运行该镜像，使用了--rm参数，会在退出时自动删除容器
# 执行成功便会自动进入容器shell，使用exit即可退出
docker run -it --rm arm64v8/debian /bin/bash
```