---
title: Linux技巧(随时更新)
categories: [知识库,技术笔记,软件]
tags:
  - Linux
date: 2022-11-17 16:15:48
---

>若您有任何技术问题，可以通过本站展示的联系方式咨询我

<!--more-->

## Linux查看硬盘SN

```shell
# 将#替换为你对应的硬盘号 如sda
lsblk --nodeps -no serial /dev/sd*
```

## Linux接触文件、目录占用

```shell
# 使用该命令查看占用文件、文件夹的程序id
fuser -cu /你要查询的文件或目录
# 示例
fuser -cu docker-application/
# 我的输出结果
/mnt/TEST/docker-application: 16676m(root) 16702m(root)
# 其中16676m(root)和16702m(root)便是表明该目录由id为16676和16702的进程占用
# 使用kill id杀死进程后便可删除或使用mount -f强制卸载
```
