---
title: Proxmox-VE批量加集群、批量克隆、批量设置并同步时间、批量清除集群残留、批量创建网卡等一站式解决方案！（本教程采用YgeCloud实测）
categories: [伟大的知识库]
comments: true
date: 2024-02-06 05:32:15
keywords:
    - Proxmox-VE
    - YgeCloud
    - 昱格
tags:
    - Proxmox-VE
    - YgeCloud
    - 昱格
    - 批量
---

---
**本文章所有命令均在YgeCloud进行实测并列出了各种BUG，本文涉及的命令均为Linux KVM QEMU虚拟化通用命令，如命令有不兼容*十分拜托您务必联系我修正文章！十分感谢！***
---

**以下命令中`10.0.1.$i`是因为本次批量部署节点均在10.0.1.0/24段，需自行修改为你节点所在网段，如`192.168.100.$i`**

**批量创建网卡适用于如本次部署一样的情况，如外网网线统一连接到enp6s0网口，然后桥接到vmbr1给虚拟机连接使用，请根据你的情况自行修改**

## 批量加集群

<!-- more -->

*更新：利用expect实现初次连接自动登陆或自动上传ssh key*

```shell
# 初次连接自动登陆
expect -c 'spawn ssh -o StrictHostKeyChecking=no root@10.0.1.$i; expect "(yes/no)" { send "yes\n"; exp_continue } "assword:" { send "密码\n" }; interact'

# 初次连接自动上传ssh key，便于后续免密连接批量执行命令
expect -c 'spawn ssh-copy-id -o StrictHostKeyChecking=no root@10.0.1.$i; expect "(yes/no)" { send "yes\n"; exp_continue } "assword:" { send "密码\n" }; interact'

# 批量做免密连接
for i in {开始节点如104..结束节点如150}; do expect -c "spawn ssh-copy-id -o StrictHostKeyChecking=no root@10.0.1.$i; expect \"(yes/no)\" { send \"yes\n\"; exp_continue } \"assword:\" { send \"密码\n\" }; interact" ; sleep 1 ; done
```

```shell
# 从创建集群的主节点批量远程自动加集群
for i in {开始节点如104..结束节点如150}; do ssh -t root@10.0.1.$i "echo -e \"密码\\nyes\" | pvecm add 主节点完整IP如10.0.1.101 && while true; do if pvecm status | grep -q $i; then echo -e \"successfully\"; break; else echo -e \"Please wait a moment.\"; sleep 5; fi; done"; done

# 自动加集群示例
for i in {126..150}; do ssh -t root@10.0.1.$i "echo -e \"密码\\nyes\" | pvecm add 10.0.1.125 && while true; do if pvecm status | grep -q $i; then echo -e \"successfully\"; break; else echo -e \"Please wait a moment.\"; sleep 5; fi; done"; done

# 批量自动创建vmbr1网卡
for i in {127..150}; do ssh -t root@10.0.1.$i 'echo -e "auto vmbr1\niface vmbr1 inet manual\n\tbridge-ports enp6s0\n\tbridge-stp off\n\tbridge-fd 0" >> /etc/network/interfaces && systemctl restart networking'; done

# 从开始节点循环执行到结束节点
# 自动移除yge-data
# 扩容yge-root
# 刷新yge-root分区容量
# 修改时间服务器为10.0.1.110节点
# 重启chrony.service
# 强制同步时间
# 自动加入集群
# 搜索当前ip比对pvecm status输出结果
# 未输出当前节点ip则持续提示等待
# 输出当前节点ip则表示加入集群成功，输出successfully
# 间隔5秒执行
# 批量创建vmbr1网卡，桥接enp6s0
# -- 命令为单行命令，严禁换行，”一条龙命令“ --
# 因YgeCloud无sshpass工具，且新节点与主节点无免密登陆
# 所以该命令需要手动输入ssh密码

for i in {开始节点如104..结束节点如150}; do ssh -t root@10.0.1.$i "lvremove -f /dev/yge/data && lvextend -l +100%FREE /dev/yge/root && resize2fs /dev/yge/root && sed -i 's/pool 2.debian.pool.ntp.org/server 10.0.1.110/' /etc/chrony/chrony.conf && systemctl restart chrony && chronyc makestep && sleep 3 && date && echo -e \"密码\\nyes\" | pvecm add 主节点完整IP如10.0.1.101 && while true; do if pvecm status | grep -q $i; then echo -e \"successfully\"; break; else echo -e \"Please wait a moment.\"; sleep 5; fi; done && echo -e \"auto vmbr1\niface vmbr1 inet manual\n\tbridge-ports enp6s0\n\tbridge-stp off\n\tbridge-fd 0\" >> /etc/network/interfaces && systemctl restart networking"; done
```

## 批量从模板恢复虚拟机，比克隆更骚的办法

**vzdump生成的备份恢复时是根据名称匹配的，如果修改了命名会导致恢复失败，使用qmrestore恢复前一定要还原名称**

**如需临时命名以便区分备份文件，强烈建议添加前缀，不要修改原本的文件名！**

**-storage指向的目标存储不能为共享！否则会出现cfs无法锁定报错（local在集群中错误的设置为了共享发现的）**

```shell
# 加了集群之后可以免密了！！！
# ssh连接到对应的子节点
# 执行qmrestore命令
# /mnt/pve/backup/dump是YGENAS发布的NFS在集群存储添加为NFS挂载的位置
# 从 [你的备份文件名称] 备份文件批量恢复虚拟机并将虚拟机ID设置为当前节点ID，避免ID重复
# --store 指定恢复到local存储
# --uniaue使恢复的虚拟机重设网卡MAC以防MAC冲突
# 恢复完成后使用qm set VMID --name string命令重设虚拟机名为对应当前节点IP
for i in {125..150}; do ssh -t root@10.0.1.$i "qmrestore /mnt/pve/backup/dump/vzdump-qemu-2002-2024_02_04-16_44.vma.lzo $i -storage local -unique && qm set $i --name g$i";done

# 并发恢复，将输出重定向到对应的$i.log，避免ssh -t等待
for i in {125..150}; do nohup ssh -t root@10.0.1.$i "qmrestore /mnt/pve/backup/dump/vzdump-qemu-2002-2024_02_04-16_44.vma.lzo $i -storage local -unique && qm set $i --name g$i" > $i.log 2>&1 & done

# 间隔1秒持续显示恢复进度
while true; do bash -c 'clear;for i in {125..150}; do echo $i ; tail -n 1 "$i.log"; done'; sleep 1; done

# 排除rescan volumes...从而筛选出恢复失败的节点
while true; do clear; for i in {125..150}; do tail -n 1 "$i.log" | grep -q "update" || { echo "$i error"; }; done; sleep 1; done

# 批量删除克隆错误的虚拟机
for i in {125..150};do nohup ssh -t root@10.0.1.$i "qm destroy $i" > del$i.log 2>&1 & done
```

## 加错集群处理办法

### 主节点批量移除失效节点

```shell
# 删除[104到124]的所有节点，yge是因为我的节点名前缀为yge，可以根据你的hostname来
# 我的hostname输出为yge104
for i in {104..124}; do pvecm delnode yge$i; done
```

### 在主节点删除后子节点会残留集群信息无法加入，需清除残留信息

```shell
# 一条龙清理
# 从127开始清理到150，需在主节点执行，误加集群后主节点可以免密登陆子节点
# 先在主节点创建cleanup.sh脚本并chmod +x cleanup.sh赋予可执行权限
for i in {127..150}; do scp cleanup.sh root@10.0.1.$i:/tmp/ && ssh -t root@10.0.1.$i "bash /tmp/cleanup.sh" && ssh -t root@10.0.1.$i "rm /tmp/cleanup.sh"; done
```

**因ssh -t无法处理!，所以迫不得已使用脚本执行**

```shell
#!/bin/bash

systemctl stop pve-cluster
systemctl stop corosync
pmxcfs -l
rm /etc/pve/corosync.conf
rm -rf /etc/corosync/*
rm -rf /etc/pve/nodes/!(hostname)
killall pmxcfs
systemctl start pve-cluster
```

## 基于免密后批量配置crontab

```shell
for i in {101..103};do ssh root@10.0.1.$i -t "crontab -e";done
```