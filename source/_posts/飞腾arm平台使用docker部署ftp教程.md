---
title: 飞腾arm平台使用docker部署ftp教程
categories: [知识库]
comments: true
date: 2023-09-12 20:50:38
keywords: vsftpd
tags:
    - Docker
---

**因飞腾平台为armv8，暂未发现简单易用的vsftpd Docker镜像，特此提供打包完毕的Docker镜像及教程以供各位使用**

<!-- more -->

**该docker镜像为armv8版本，已测试适用飞腾2000平台**

第一步，拉取docker vsftpd映像

*因docker hub已被墙，拉去docker images请参考[国内docker hub无法使用的解决办法](https://hackerbs.com/%E5%9B%BD%E5%86%85docker-hub%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8%E7%9A%84%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95.html)，或者使用下面手动导入的方式*

`docker pull hackerbs/vsftpd`

如果你的环境为离线环境，则采用导入vsftpd映像方案

docker离线镜像位于[网盘](https://disk.ygeit.cn/s/RpAtcoJcbpYtaoa)docker_images文件夹中，名为`hackerbs-vsftpd-arm.tar`

将下载的`hackerbs-vsftpd-arm.tar`导入到你的系统中，使用`docker load -i hackerbs-vsftpd-arm.tar`将该映像导入

第二步，使用该映像启动容器

```shell
# 首先，创建一个路径用于存放ftp文件
mkdir -p 路径
# 示例
mkdir -p /opt/ftp

# 请用你的参数替代{参数}内容
docker run -d --name {容器名} -p 20:20 -p 21:21 -p 4559-4564:4559-4564 -e FTP_USER={ftp用户名} -e FTP_PASSWORD={ftp密码} -e PASV_ADDRESS={服务器地址}  -e FTP_REPOSITORY=/opt/ftp -v {主机FTP目录}:/opt/ftp --restart=always oscarenzo/vsftpd

# 若无特别需求，仅更改“容器名、ftp用户名、密码、服务器地址”即可
# 示例如下
docker run -d --name vsftpd-server -p 20:20 -p 21:21 -p 4559-4564:4559-4564 -e FTP_USER=ftptest -e FTP_PASSWORD=123456 -e PASV_ADDRESS=192.168.2.254 -e FTP_REPOSITORY=/opt/ftp -v /opt/ftp:/opt/ftp --restart=always f8044caf3727

# 注：离线环境导入images需要将命令中的fauria/vsftpd替换为images ID f8044caf3727，如上方示例

# 上述命令执行后可以使用docker ps查看容器是否运行成功
# 容器运行成功后修改本机FTP目录权限为777，示例

chmod -R 777 /opt/ftp

# docker开机自动运行命令
systemctl enable docker
```

完整过程如下图所示，若有任何问题可以通过文末联系方式咨询

![20230912213003](https://img.hackerbs.com//20230912213003.png)