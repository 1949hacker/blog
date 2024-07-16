---
title: YgeCloud创建虚拟机教程
comments: true
categories: [知识库]
date: 2024-07-12 11:28:50
keywords:
    - 昱格虚拟化平台
    - YgeCloud
    - Proxmox-VE
    - 虚拟机
    - Windows
    - Linux
tags:
    - YgeCloud
    - Proxmox-VE
    - 虚拟机
    - Windows
    - Linux
    - 昱格虚拟化平台
---

昱格服务器虚拟化系统创建虚拟机教程

![YgeCloud](https://www.ygeit.com/upload/1/cms/content/editor/1643091301539.png)

<!-- more -->

# 安装前准备

YgeCloud安装后默认有`local`和`local-lvm`两个存储，`local-lvm`可以忽略掉，通常情况下都会外挂其他存储或是切换为目录模式挂载，目录模式支持的内容有：

|类型|作用|
|---|---|
|磁盘映像|用于存放虚拟机磁盘，要将虚拟机创建到该存储则必选该项|
|ISO镜像|勾选后可以将iso上传到此存储|
|VZDump备份文件|勾选后支持将虚拟机备份到该存储|
|片段|

## Windows安装前准备

Windows因为没有kvm驱动，安装前还需获取最新的kvm驱动iso，选择最新的`virtio-win-x.x.xxx.iso`下载即可，然后上传到YgeCloud

[下载驱动](https://fedorapeople.org/groups/virt/virtio-win/direct-downloads/archive-virtio/?C=M;O=D)

# YgeCloud创建Debian虚拟机

**本教材也适用于OpenEuler、Arch Linux、Ubuntu等其他Linux发行版**

