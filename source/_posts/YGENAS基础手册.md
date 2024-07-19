---
title: YGENAS基础手册
comments: true
categories: [知识库]
date: 2024-07-16 11:00:29
keywords:
    - YGENAS
    - 昱格NAS
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
tags:
    - YGENAS
    - 昱格NAS
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
---

# 本文为昱格NAS（YGENAS）产品基础版使用手册

本文主要讲解昱格NAS的常见配置项，如SMB共享、NFS共享、用户创建、权限配置等

如您需要技术支持，可联系对应的销售，或拨打400-028-0061并告知来自本站

<!-- more -->

## 初始化配置

### 插入硬盘并创建存储池

在配置存储池之前，简单介绍下最常见的配置方式，对于YGENAS-R-H12，也就是12盘及以下的产品，可以将最多12个盘作为一组RAIDZ，超过12盘的，如YGENAS-R-H16，则建议将盘分为多组，建议为8个一组RAIDZ，24盘则是`3*8盘`RAIDZ，36盘则可以酌情调整为`4*9盘`RAIDZ，如果一个RAIDZ中的盘太多，则会导致数据块被分的太散，会略微影响每个盘数据读写的速度，最关键的是单个RAIDZ中数据盘数量越多，后期故障重构时间就越久，重构期间掉盘的风险也就越大。

|产品|RAIDZ盘数|RAIDZ数量|
|---|---|---|
|YGENAS-V-H5|5|1|
|YGENAS-V-H8|8|1|
|YGENAS-R-H12|12|1|
|YGENAS-R-H16|8|2|
|YGENAS-R-H24|8|3|
|YGENAS-R-H36|9|4|

#### 创建存储池

## YGENAS发布SMB共享
