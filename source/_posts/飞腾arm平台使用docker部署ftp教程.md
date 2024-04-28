---
title: 飞腾arm平台使用docker部署ftp教程
categories: [知识库]
comments: true
date: 2023-09-12 20:50:38
keywords: vsftpd
tags:
    -   Docker
---

**因飞腾平台为armv8，暂未发现简单易用的vsftpd Docker镜像，特此提供打包完毕的Docker镜像及教程以供各位使用**

<!-- more -->

**该docker镜像为armv8版本，已测试适用飞腾2000平台**

第一步，拉取docker vsftpd映像

`docker pull 1949hacker/vsftpd`

如果你的环境为离线环境，则采用导入vsftpd映像方案

[点此下载vsftpd映像](http://117.176.215.230:20080/s/7PHKDK6qdb4Q8Qn/download?path=%2Fdocker%20images&files=1949hacker-vsftpd-arm.tar&downloadStartSecret=30n7yazxext)

其他资料可访问：[http://117.176.215.230:20080/s/7PHKDK6qdb4Q8Qn](http://117.176.215.230:20080/s/7PHKDK6qdb4Q8Qn)

将下载的`1949hacker-vsftpd-arm.tar`导入到你的系统中，使用`docker load -i 1949hacker-vsftpd-arm.tar`将该映像导入

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