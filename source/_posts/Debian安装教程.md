---
title: Debian安装教程
comments: true
categories: [知识库]
date: 2024-06-03 14:29:16
keywords:
    - Debian
    - Linux安装
tags:
    - Debian
    - Debian安装
    - Linux安装
    - Debian服务器
    - Linux
---

## debian-live-12.5.0-amd64-standard安装教程

该系统下载地址：

[中科大镜像站下载](https://iso.mirrors.ustc.edu.cn/debian-cd/current-live/amd64/iso-hybrid/debian-live-12.5.0-amd64-standard.iso)

[网盘ISO文件夹中下载](https://disk.ygeit.cn/s/RpAtcoJcbpYtaoa)

> nano打开文件便可直接编辑，保存则需要先使用Ctrl+o保存然后使用Ctrl+x退出

<!-- more -->

### 进入安装程序

选择`Start installer`

![20240603143147](https://img.hackerbs.com//20240603143147.png)

### 选择语言

**这一步切记，只选择English，不要选择其他，否则在命令行模式会导致无法正常显示字体，出现方块状乱码**

![20240603143249](https://img.hackerbs.com//20240603143249.png)

### 选择地区

选择other -> Asia -> China

![20240603143317](https://img.hackerbs.com//20240603143317.png)

![20240603143329](https://img.hackerbs.com//20240603143329.png)

![20240603143338](https://img.hackerbs.com//20240603143338.png)

### 本地化配置

这一步和上一步都相当于Windows的地区/语言，跟着教程选择默认的即可，除了English、en_UF.UTF-8之外的任何，除非你是老手，否则新手一律选择默认，避免字体显示错误

![20240603143448](https://img.hackerbs.com//20240603143448.png)

### 配置键盘

默认 American English

![20240603143508](https://img.hackerbs.com//20240603143508.png)

### 配置网络

如果有DHCP，这一步会直接跳过，没有DHCP需要手动配置的参考下文手动配置IP

![20240603143600](https://img.hackerbs.com//20240603143600.png)

#### 手动配置IP

若没有DHCP或没有网线,这一步会提示网络连接失败

图中提示的内容大概意思是`您的网络没有DHCP服务器或DHCP响应过慢`

![20240603143716](https://img.hackerbs.com//20240603143716.png)

选择`Continue`，选择`Configure network manually`手动配置网络

![20240603143838](https://img.hackerbs.com//20240603143838.png)

填写IP地址和子网掩码前缀，在Linux建议更多是使用前缀，而不是输一堆255

255.255.255.0 = 11111111.11111111.11111111.0 = 24 （前缀就是有多少个1就是多少，所以是24）

255.255.252.0 = 11111111.11111111.11111100.0 = 22

![20240603145231](https://img.hackerbs.com//20240603145231.png)

填写网关

![20240603145247](https://img.hackerbs.com//20240603145247.png)

填写DNS

国内腾讯`119.29.29.29`，境外谷歌`8.8.8.8`，这两个DNS足够解析全世界的域名并兼顾速度了

![20240603154540](https://img.hackerbs.com//20240603154540.png)

### 填写主机名

也就是hostname，没啥可说的，默认或者按需修改

![20240603154644](https://img.hackerbs.com//20240603154644.png)

### 填写域名

留空或者按需填写

![20240603154725](https://img.hackerbs.com//20240603154725.png)

### 设置root用户密码

设强点，别老用弱密码

![20240603154847](https://img.hackerbs.com//20240603154847.png)

### 创建新用户

这一步无法跳过，如果没有特别需要，直接随便建，进了系统删了即可

这一步`Full name for the new user:`是显示名称，和登陆无关，可以理解为QQ名

![20240603154927](https://img.hackerbs.com//20240603154927.png)

这一步才是用户名，要遵循规范，可以理解为QQ号，是登陆系统的账号

![20240603155108](https://img.hackerbs.com//20240603155108.png)

设密码

![20240603155132](https://img.hackerbs.com//20240603155132.png)

### 格式化磁盘

如果你玩不来（你都看我教程了肯定是玩不来），那就记住了，选`Guided - use entire disk and set up LVM`，**必须必须必须使用LVM逻辑卷**，不然后期调整磁盘你就整个傻住了（恕我直言，不用lvm逻辑卷的全是瓜娃子！）

![20240603160221](https://img.hackerbs.com//20240603160221.png)

选择硬盘

![20240603160247](https://img.hackerbs.com//20240603160247.png)

### 选择分区方式

新手，听我的，直接第一个`All files in  one partition`，将所有文件放到一个分区就行，别觉得分开分区有什么卵用，就算是生产环境都基本用不着

![20240603160451](https://img.hackerbs.com//20240603160451.png)

### 确认是否分区

选Yes然后Enter就完事儿了

![20240603160521](https://img.hackerbs.com//20240603160521.png)

默认会选择全部空间，或者你也可以输个max

![20240603160550](https://img.hackerbs.com//20240603160550.png)

选择`Finish partitionging and write changes to disk`然后`Continue`

![20240603160627](https://img.hackerbs.com//20240603160627.png)

选择Yes然后Enter

![20240603160645](https://img.hackerbs.com//20240603160645.png)

### 等待系统安装

![20240603160756](https://img.hackerbs.com//20240603160756.png)

### 是否启用网络镜像

没网的直接NO，有网的Yes然后Enter

![20240603160857](https://img.hackerbs.com//20240603160857.png)

选择China

![20240603160918](https://img.hackerbs.com//20240603160918.png)

建议`mirrors.ustc.edu.cn`，中科大万岁

![20240603160943](https://img.hackerbs.com//20240603160943.png)

不需要代理，直接Enter下一步

![20240603161008](https://img.hackerbs.com//20240603161008.png)

开始配置apt，也就是网络镜像

![20240603161318](https://img.hackerbs.com//20240603161318.png)

#### 提示镜像故障

这一步就是网络问题了，可以考虑直接先不用网络镜像

![20240603161048](https://img.hackerbs.com//20240603161048.png)

### 安装完毕

安装完毕，Continue重启

![20240603161655](https://img.hackerbs.com//20240603161655.png)

### 删除安装过程中不得不创建的用户

登陆

![20240603161753](https://img.hackerbs.com//20240603161753.png)

使用`userdel -r 用户名`删除创建的用户

![20240603161857](https://img.hackerbs.com//20240603161857.png)

### 额外内容

#### 手动配置网络

使用`ip addr`查看网卡名称

本次演示的只有一张网卡，是enp6s18，其中`state UP`表示该网卡已经插好网线

![20240603162420](https://img.hackerbs.com//20240603162420.png)

使用`nano /etc/network/interfaces`编辑网络配置文件

使用`nano /etc/resolv.conf`编辑DNS配置文件

|配置项|作用|注意事项|
|---|---|---|
|auto enp6s18|网卡开机自启||
|iface enp6s18 inet static|指定配置哪张网卡|static表示静态IP,dhcp则是动态|
|address|IP地址/前缀||
|gateway|网关地址||
|dns-search|对应安装过程中的domain域名||
|nameserver|DNS服务器|一行一个，nameserver 119.29.29.29|

网卡配置文件示例

![20240603162336](https://img.hackerbs.com//20240603162336.png)

DNS配置文件示例

![20240603163137](https://img.hackerbs.com//20240603163137.png)

编辑完成后使用`systemctl restart networking.service`重启网络

![20240603163210](https://img.hackerbs.com//20240603163210.png)

#### 手动配置apt源

**如果安装的时候没有使用网络镜像，且需要配置apt源的话，进行该步骤，否则忽略**

直接使用`rm -f /etc/apt/sources.list`删除源配置文件

编辑`nano /etc/apt/sources.list`源配置文件，先手动添加第一行

`deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware`

![20240603163523](https://img.hackerbs.com//20240603163523.png)

编辑后使用`apt update`更新源

![20240603163751](https://img.hackerbs.com//20240603163751.png)

#### 安装OpenSSH-server并配置root密码登陆

使用`apt install openssh-server -f`安装OpenSSH server

![20240603163908](https://img.hackerbs.com//20240603163908.png)

`nano /etc/ssh/sshd_config`编辑ssh配置文件，在原本的`#PermitRootLogin prohibit-password`下方添加`PermitRootLogin yes`

上方的`#Port 22`取消注释修改为其他端口即可实现修改ssh端口功能，如`Port 10022`

![20240603164013](https://img.hackerbs.com//20240603164013.png)

编辑后使用`systemctl restart ssh`重启ssh服务，之后便可使用ssh连接服务器了

#### 配置完全apt国内镜像

继上文配置ssh后，便可使用ssh远程访问服务器，这样复制粘贴便方便多了

在这里推荐snullp大佬开发的[配置生成器](https://mirrors.ustc.edu.cn/repogen/)，注意根据你的Debian版本选择不同的内容，本教程使用的是bookworm，apt完全配置内容如下，其他配置内容可以查看本站的[apt等各种源列表（持续更新）](https://hackerbs.com/apt%E7%AD%89%E5%90%84%E7%A7%8D%E6%BA%90%E5%88%97%E8%A1%A8%EF%BC%88%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0%EF%BC%89.html)

```shell
deb https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
deb-src https://mirrors.ustc.edu.cn/debian-security/ bookworm-security main contrib non-free non-free-firmware
```

同样的，还是编辑`/etc/apt/sources.list`

![20240603164707](https://img.hackerbs.com//20240603164707.png)

编辑后还是使用`apt update`更新源即可