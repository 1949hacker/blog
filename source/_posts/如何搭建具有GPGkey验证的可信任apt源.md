---
title: 如何搭建具有GPGkey验证的可信任apt源
comments: true
categories: [知识库]
date: 2024-06-21 17:13:04
keywords:
    - apt
    - GPG key
    - 可信任apt源
    - apt源
tags:
    - apt
    - GPG key
    - 可信任apt源
    - apt源
---

# 本文将介绍如何搭建一个像docker-ce一样具有GPG key验证的源

普通的用`deb [trusted=yes] http://url/ bookworm main`添加的源在内网环境用还行，但若是在公网，就会存在没有验证，被篡改的风险，且具有GPG key验证的源也更专业

搭建自己的apt源后便可将自己的deb包上传到服务器，然后添加自己的apt源并导入GPG密钥

<!-- more -->

首先，使用docker创建一个nginx，以下是我的docker-compose.yaml文件

```yaml
services:
    apt:
        image: nginx:latest
        container_name: apt
        restart: always
        ports:
        - 443:443
        volumes:
        - /mypath/html:/usr/share/nginx/html
        - /mypath/conf:/etc/nginx/conf.d
```

然后使用`docker compose up -d`启动这个nginx容器

进入到我的目录的conf中，新建一个apt.conf配置文件

```
server {
    listen 443;
    server_name mydomain;

    root /usr/share/nginx/html;
    autoindex on;

    location / {
        try_files $uri $uri/ =404;
    }
}
```

然后使用`docker exec apt nginx -t`测试配置文件

继续使用`docker exec apt nginx -s reload`重载配置文件

此时nginx已经上线

