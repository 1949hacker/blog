---
title: 关于磁盘残留lvm的解决办法
comments: true
categories: [知识库]
date: 2024-11-28 10:28:48
keywords:
	- Linux
	- lvm
	- 磁盘
	- 解决办法
	- dmsetup
tags:
	- Linux
	- lvm
	- 磁盘
	- 解决办法
	- dmsetup
---

很多时候从其他机器拿过来的盘通常是没有经过格式化的，这样就会出现一些残留的lvm信息，导致无法正常使用。

出现该残留lvm的原因是因为设备映射器device mapper正在管理该磁盘，所以wipefs -a -f和dd if=/dev/zero of=/dev/disk命令无法清除该lvm

```shell
# lsblk显示如下
nvme0n1					disk
└storage-data			lvm

# ls -l /dev/mapper查询device mapper管理的磁盘信息
lrwxrwxrwx 1 root root 		7 Nov 28 18:04 storage-data ->../dm-0

#dmsetup remove删除
dmsetup remove /dev/mapper/storage-data
```
