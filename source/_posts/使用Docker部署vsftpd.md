---
title: 使用Docker部署vsftpd
categories: [知识库]
date: 2023-09-07 17:12:38
keywords: vsftpd
tags:
    -   Docker
---

**为避免不同 系统环境导致vsftpd的各种问题，在此强烈建议使用docker进行vsftpd部署！**

<!-- more -->

第一步，拉取docker vsftpd映像

`docker pull oscarenzo/vsftpd`

如果你的环境为离线环境，则采用导入vsftpd映像方案

[点此下载vsftpd映像](http://pan.1949hacker.cn:20080/s/7PHKDK6qdb4Q8Qn/download?path=%2Fdocker%20images&files=vsftpd.tar&downloadStartSecret=hzc48g1udq6)

其他资料可访问：[http://pan.1949hacker.cn:20080/s/7PHKDK6qdb4Q8Qn](http://pan.1949hacker.cn:20080/s/7PHKDK6qdb4Q8Qn)

将下载的`vsftpd.tar`导入到你的系统中，使用`docker load -i vsftpd.tar`将该映像导入

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
docker run -d --name vsftpd-server -p 20:20 -p 21:21 -p 4559-4564:4559-4564 -e FTP_USER=ftptest -e FTP_PASSWORD=123456 -e PASV_ADDRESS=192.168.0.115 -e FTP_REPOSITORY=/opt/ftp -v /opt/ftp:/opt/ftp --restart=always d8e46f0dec88

# 注：离线环境导入images需要将命令中的fauria/vsftpd替换为images ID 9bfb39139661，如上方示例
```

**docker开机自动运行命令：`systemctl enable docker`**

![20230907174353](https://img.1949hacker.cn//20230907174353.png)