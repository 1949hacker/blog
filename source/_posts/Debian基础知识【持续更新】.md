---
title: Debian基础知识【持续更新】
categories: [知识库]
date: 2023-03-01 15:45:44
tags:
    - Linux
    - Debian
---

# 本文将持续更新Debian系统的各类基础知识，欢迎您持续关注，有任何问题可以在本页末尾评论或通过展示的联系方式联系我

**根据红帽的公告，CentOS将不再适合作为生产环境的稳定系统使用，为此我个人建议您尝试著名且优秀的Debian系统**

<!-- more -->

>2020年12月08日，CentOS官方宣布了停止维护CentOS Linux 8的计划，并推出了CentOS Stream项目。具体信息，请阅读CentOS官方公告。其具体规划如下：
>
> - CentOS Linux 8作为RHEL 8的复刻版本，生命周期缩短，于2021年12月31日停止更新并停止维护（EOL）。
> - 基于以上官方变更计划，CentOS Linux 8用户将无法获得包括问题修复和功能更新在内的任何软件维护和支持。CentOS官方建议停止维护后：对于开发或测试环境，可以将环境迁移至CentOS Stream版本； 对于生产环境或部署关键业务的系统，建议使用稳定的Red Hat Enterprise Linux。对此，用户需评估以下问题：
> - CentOS Stream是一个滚动发行的版本，仅为RHEL前置测试版，运用于生产环境时，可能存在一定风险。
> [亚马逊云科技上在CentOS在停止维护后的几种选择](https://aws.amazon.com/cn/blogs/china/aws-choices-for-centos-after-stopping-maintenance/)

---

## Debian 更换国内源

在使用过阿里云、网易163、清华、中科大源之后，我个人建议更换为中科大源，Debian更换源的方式非常简单，在此非常感谢[sNullp先生](https://github.com/snullp)的[源配置生成器](https://mirrors.ustc.edu.cn/repogen/)!

在使用该配置生成器之前，你需要先确认你的Debian版本，使用`cat /etc/os-release`命令，在输出的结果中找到`VERSION_CODENAME=bullseye`，位于`=`后面的就是你的版本代号，如图：

![20230301160807](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301160807.png)

然后打开[源配置生成器](https://mirrors.ustc.edu.cn/repogen/)，找到Debian并选择你对应的版本号，如图：

![20230301161044](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301161044.png)

其中，HTTPS/HTTP不必赘述，就是字母意思，而IPv4也很好理解，就是选择IPv4或IPv6地址访问，默认HTTPS和IPv4即可

复制红框中的内容后，回到你的Debian系统，使用`vim /etc/apt/sources.list`命令编辑apt源配置文件，如果提示`-bash: /usr/bin/vim: No such file or directory`则是因为你没有安装`vim`编辑器，我强烈建议你安装`vim`编辑器，如果暂时无法安装，也可使用`nano`编辑器，对应命令为`nano /etc/apt/sources.list`，使用编辑器打开`sources.list`文件后，如果你是`vim`编辑器，则可以将光标移到首行，然后按`d`再按`Shift+G`即可直接清空内容，然后粘贴你在源配置生成器复制的红框中内容即可，如图：

![20230301161642](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301161642.png)

`vim`编辑器在编辑完成后按`ESC`再输入`:wq`回车即可退出保存，编辑器的操作方式在此不过多赘述。

完成源配置文件的编辑后，还需要使用`apt update`命令更新源，至此，Debian更换中科大源的教程结束。

## Debian iSCSI挂载卷

首先，你需要安装`open-iscsi`，使用`apt install -y open-iscsi`进行安装，然后运行`iscsiadm -m discovery -t st -p 服务器地址`探测服务器发布的卷，如图：

![20230301162849](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301162849.png)

随后使用`iscsiadm -m node -T iqn开头的对应地址 -p 服务器地址 -l`即可完成挂载，如图：

![20230301164715](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301164715.png)

随后使用`lsblk`命令发现新的磁盘，如图：

![20230301164844](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301164844.png)

扩展内容：

```shell
# 自动挂载
iscsiadm -m node -T iqn地址:目标 -p 服务器地址:端口 --op update -n node.startup -v automatic

# 解除挂载
iscsiadm -m node -T iqn地址:目标 -p 服务器地址:端口 -u

# 解除所有连接
iscsiadm -m node --logoutall=all

# 查看所有iscsi连接
iscsiadm -m session
```

## Debian 格式化并挂载磁盘以及lvm逻辑卷的使用

紧跟上文，iscsi挂载后的卷无法直接使用，需要像磁盘一样进行分区及挂载到系统目录

安装parted：`apt install -y parted`

使用`parted /dev/设备`命令进入磁盘分区管理，随后使用`mklable gpt`将磁盘设置为优秀的`GPT`分区表，然后使用`mkpart 名称 文件系统 起始扇区 结束地址`创建分区，如图：

![20230301165916](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301165916.png)

其中，2048s是为了将扇区进行对齐，有利于提升性能，而100%则是指定结束地址为最后，100%可以替换为明确的扇区、容量，如9999s（扇区）或100G（容量），同理，起始地址也是如此，如果你需要创建多个分区，则之后的分区起始地址应紧随上一个分区的结束地址。

分区创建完毕后，输入`q`回车即可退出parted，将分区格式化之后即可挂载使用

**但，我在此强烈建议你使用lvm逻辑卷，因为lvm优点有便于容量调整、创建跨区卷等**

lvm卷的使用步骤有创建物理卷，创建卷组，创建逻辑卷

```shell
# 创建物理卷
pvcreate /dev/设备

# 创建卷组
vgcreate 卷组名 物理卷

# 创建逻辑卷
lvcreate -l +100%FREE -n 逻辑卷名 所属卷组名
```

示例如图：

![20230301170847](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301170847.png)

lvm的逻辑卷地址位于`/dev/mapper/`下，命名格式为`卷组名-逻辑卷名`，将逻辑卷格式化后即可挂载使用，如图：

![20230301171003](https://cdn.jsdelivr.net/gh/1949hacker/picgo/20230301171003.png)

至此，Debian 磁盘分区及lvm逻辑卷教程完毕，如有疑问，欢迎咨询！