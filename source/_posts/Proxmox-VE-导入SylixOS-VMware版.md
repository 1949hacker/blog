---
title: Proxmox-VE 导入SylixOS VMware版
date: 2023-02-28 16:31:43
keywords: Proxmox-VE,SylixOS,VMware
description: Proxmox-VE 导入SylixOS VMware版
tags:
    - Proxmox-VE
    - SylixOS
categories: [知识库]
---

将SylixOS传到服务器

解压SylixOS VMware版，找到其中的

```shell
x86_boot.vmdk
x86_main.vmdk
```

<!-- more -->

将这两个文件上传到服务器，使用qemu-img命令将vmdk虚拟机磁盘转为qcow2格式

```shell
qemu-img convert -f vmdk -O qcow2 x86_boot.vmdk x86_main.qcow2
qemu-img convert -f vmdk -O qcow2 x86_boot.vmdk x86_boot.qcow2
```

切换到虚拟化服务器web界面，创建新虚拟机

1. 操作系统类型：Linux kernel 2.4，不使用任何光盘介质
    ![20230224155828](https://img.1949hacker.cn/20230224155828.png)

2. 创建IDE硬盘，硬盘大小无所谓
    ![20230224155929](https://img.1949hacker.cn/20230224155929.png)

3. 配置CPU，内核数量根据自己需求而定，类型为host
   ![20230224160025](https://img.1949hacker.cn/20230224160025.png)

4. 修改网络模型，如果网卡不可用，尝试修改为其他模型
    ![20230224160056](https://img.1949hacker.cn/20230224160056.png)

创建虚拟机的步骤完毕，现在连接到服务器的shell，可以使用web shell或另行连接

```shell
# 进入到虚拟机配置文件目录
cd /etc/pve/qemu-server/

# 编辑刚刚创建的ID为125的虚拟机所属配置文件
vim 125.conf
```

原配置文件如下：

![20230224160513](https://img.1949hacker.cn/20230224160513.png)

修改第6行ide0开头的内容如下

```shell
ide0: nfshare:125/x86_boot.qcow2
ide1: nfshare:125/x86_main.qcow2

# 注意！
# 其中，ide0 ide1务必不要重复，以免冲突
# 其中，nfshare为我虚拟化服务器所在的存储名称，实际名称以你为准！
```

修改完毕后，将刚刚转化完毕的两个qcow2文件移动到虚拟机磁盘映像所在目录

**先删除现有的文件**

![20230224160856](https://img.1949hacker.cn/20230224160856.png)

随后打开web界面，启动虚拟机即可！