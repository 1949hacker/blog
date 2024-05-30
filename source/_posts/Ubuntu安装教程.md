---
title: Ubuntu安装教程
comments: true
categories: [知识库]
date: 2024-05-30 11:51:57
keywords:
    - Ubuntu
    - Linux安装
tags:
    - Ubuntu
    - Ubuntu安装
    - Linux安装
    - Ubuntu服务器
    - Linux
---

## ubuntu-20.04.6-live-server-amd64安装教程

### 进入安装程序

**灰色/绿色高亮就是选定状态，Enter就是确认**

不动键盘的话，会自动选择第一个，手动回车进入第一个也可`Install Ubuntu Server`

![20240530120224](https://img.hackerbs.com//20240530120224.png)

<!-- more -->

### 选择安装语言

通常来讲，选择`English`，因为命令行只支持英文，选择其他语言会导致命令行输出的中文无法显示，呈现方块状。

![20240530120346](https://img.hackerbs.com//20240530120346.png)

### 跳过更新直接安装

因为安装的是20.04.6，所以会在此处提示有更新的版本，选择默认的`Continue without updating`即可

![20240530120638](https://img.hackerbs.com//20240530120638.png)

### 选择键盘

直接使用默认的`English`，Enter下一步即可

![20240530121043](https://img.hackerbs.com//20240530121043.png)

### 配置网络

如果是DHCP自动获取则直接`Done`下一步即可，否则参考配置静态IP

![20240530121137](https://img.hackerbs.com//20240530121137.png)

#### 配置静态IP

使用上箭头展开此处

![20240530121210](https://img.hackerbs.com//20240530121210.png)

选择`Edit IPv4`并使用`Manual`手动配置网络

Ubuntu的配置与其他Linux略有不同，如表所示

|配置项|内容|说明|
|---|---|---|
|Subnet|IP段/前缀|这一项需要配置IP段如192.168.2.0段，并指定前缀，前缀可以用子网掩码转前缀工具计算，或者手算|
|Address|IP地址||
|Gateway|网关地址||
|Name Servers|DNS地址|用,做分隔符，一般填119.29.29.29,8.8.8.8就足够了|

![20240530121329](https://img.hackerbs.com//20240530121329.png)

### 配置代理（几乎不需要）

配置网络后下一步会弹出配置代理，留空直接下一步即可

![20240530123748](https://img.hackerbs.com//20240530123748.png)

### 选择镜像站

Ubuntu会默认选择国内镜像站，所以这一步也是直接回车即可

![20240530123838](https://img.hackerbs.com//20240530123838.png)

### 配置磁盘

|名称|翻译|作用解析|
|---|---|---|
|Use an entire disk|使用整个磁盘||
|Set up this disk as an LVM group|将此磁盘设置为LVM组|使用lvm逻辑卷可以在后期非常方便的扩容管理，**安装系统必须使用lvm**|
|Encrypt the LVM group with LUKS|使用LUKS加密lvm组（磁盘加密，不需要）|

如图所示这种，选择了磁盘后确保启用了lvm直接下一步即可

![20240530123900](https://img.hackerbs.com//20240530123900.png)

这一步会显示磁盘的信息、分区等，直接回车下一步即可

![20240530124408](https://img.hackerbs.com//20240530124408.png)

提示是否确认继续，直接确认继续即可，此处英文的意思就是继续安装将格式化磁盘，丢失磁盘上的数据

![20240530124422](https://img.hackerbs.com//20240530124422.png)

### 配置用户

Ubuntu默认不会启用root用户，所以需要在此设置用户名密码

|配置项|翻译|说明|
|---|---|---|
|Your name|昵称|就是个显示名称，随便填|
|Your server's name|主机名|就是hostname，主机名，计算机名|
|Pick a username|选择一个用户名|就是登陆用的，账号|
|Choose a password|选择一个密码|设密码|
|Confirm your password|确认你的密码|二次输入再确认一遍|

![20240530124629](https://img.hackerbs.com//20240530124629.png)

### 安装SSH服务

勾选`Install OpenSSH server`，否则无法进行ssh连接，忽略`Import SSH identity`，不需要导入ssh key

![20240530124958](https://img.hackerbs.com//20240530124958.png)

### 跳过额外软件包安装

没必要在这里安装额外软件包，除非用户指定，但更建议安装完毕后手动安装，因为官方的包通常会落后一些而且并不如手动安装的，比如docker等

直接跳过，执行下一步

![20240530125216](https://img.hackerbs.com//20240530125216.png)

### 等待系统安装完毕

显示`Installing system`

![20240530125257](https://img.hackerbs.com//20240530125257.png)

当上面显示`Install complete`下面出现`Cancel update and reboot`时，直接选择下面的，意思是取消更新并重启

![20240530125349](https://img.hackerbs.com//20240530125349.png)

等待重启后进入系统

![20240530125451](https://img.hackerbs.com//20240530125451.png)

#### 移除安装介质失败

大概率会出现如图所示的错误，无法移除安装介质，其实并不影响，直接拔了U盘重启即可

![20240530125930](https://img.hackerbs.com//20240530125930.png)

### 安装完毕，成功进入系统

![20240530130045](https://img.hackerbs.com//20240530130045.png)

#### 删除安装过程中不得已创建的用户，并配置root登陆ssh

**除非客户需要，否则不需要执行该步骤**

登陆后先修改root用户密码

第一遍输安装时创建的用户的密码，是用于授权sudo命令

后面两步才是设置root密码

![20240530130218](https://img.hackerbs.com//20240530130218.png)

退出使用root用户登陆

然后使用`userdel -r 用户名`删除用户及其主目录

![20240530130436](https://img.hackerbs.com//20240530130436.png)

编辑`/etc/ssh/sshd_config`文件，使root用户可以通过密码登陆ssh

![20240530130601](https://img.hackerbs.com//20240530130601.png)

在原本的`#PermitRootLogin prohibit-password`下添加一行`PermitRootLogin yes`，作用是允许root用户使用密码登陆ssh，原本的`prohibit-password`作用是禁止密码登陆，会一直提示密码错误

如果是用vim编辑的配置文件则需要使用vim的语法`:wq`以便保存退出，在默认的命令模式下按`i`是编辑模式，按`ESC`是返回命令模式，vim语法自行参考网上教程

![20240530130722](https://img.hackerbs.com//20240530130722.png)

编辑完成保存退出后使用`systemctl restart ssh`重启ssh服务，使修改生效

![20240530131100](https://img.hackerbs.com//20240530131100.png)

使用`ssh root@ip`测试连接，成功，至此Ubuntu安装教程完毕！

![20240530131234](https://img.hackerbs.com//20240530131234.png)