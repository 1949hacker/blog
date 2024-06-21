---
title: 本站上线私有apt源仓库及自制实用deb包
categories: [资源]
comments: true
date: 2023-10-18 09:50:41
sticky: 100
keywords:
    - aptdownloader
    - yumdownloader
    - apt包下载工具
tags:
    - apt
    - deb
    - aptdownloader
    - apt源
---
# 本站apt源添加命令：

```shell
curl -o hackerbs.asc https://apt.ygeit.cn/hackerbs.asc && mv hackerbs.asc /etc/apt/trusted.gpg.d/
echo "deb https://apt.ygeit.cn bookworm main" >> /etc/apt/sources.list
apt update
```

以下是工具介绍

<!-- more -->

## aptdownloader

如果你用过yumdownloader从yum源下载rpm包，那么看到名字你应该就知道这是一个什么工具了

[github仓库地址:https://github.com/1949hacker/deb](https://github.com/1949hacker/deb)

### 使用说明

`aptdownloader <package name>`

该命令会下载指定的包及其依赖到当前目录中，多个包名用空格分隔，示例：

`aptdownloader docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`

**注意事项：该工具下载的包及其依赖是基于当前系统的，所以你要离线导入deb包的目标系统也必须是相同系统才可！**

### 工具原理

原命令`apt download $(apt-rdepends -p <package name> | grep -v "^ ")`

使用python sys传参，subprocess执行命令，简化了原命令的操作方式