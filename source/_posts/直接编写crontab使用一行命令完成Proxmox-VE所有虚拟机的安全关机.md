---
title: 直接编写crontab使用一行命令完成Proxmox-VE所有虚拟机的安全关机和依次开机
comments: true
categories: [知识库]
date: 2024-09-12 17:14:48
keywords:
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
tags:
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
---

```shell
# 依次关机
i=$(qm list)&&vmid=$(echo "$i" | awk '{if (NR>1) {print $1}}')&&for i in $vmid;do qm shutdown $i;done

#依次开机，间隔一分钟
i=$(qm list)&&vmid=$(echo "$i" | awk '{if (NR>1) {print $1}}')&&for i in $vmid;do qm start $i&&sleep 60s;done
```
