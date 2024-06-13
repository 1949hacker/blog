---
title: 记录用docker部署nextcloud及配套mariadb、redis和持久化存储
comments: true
categories: [知识库]
date: 2024-06-13 16:45:32
keywords:
    - nextcloud
    - docker
    - docker compose
    - mariadb
    - redis
tags:
    - nextcloud
    - docker
    - docker compose
    - mariadb
    - redis
---

# 使用docker compose便捷的部署nextcloud及其配套的mariadb和redis并实现数据持久存储在本地的办法

**安装docker的教程在[apt等各种源列表（持续更新）](https://hackerbs.com/apt%E7%AD%89%E5%90%84%E7%A7%8D%E6%BA%90%E5%88%97%E8%A1%A8%EF%BC%88%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0%EF%BC%89.html)，docker hub被禁，处理的办法在[国内docker hub无法使用的解决办法](https://hackerbs.com/%E5%9B%BD%E5%86%85docker-hub%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8%E7%9A%84%E8%A7%A3%E5%86%B3%E5%8A%9E%E6%B3%95.html)**

新的docker已经自带`docker compose`命令了，所以不需要再安装`docker-compose`，且需注意是`docker空格compose`而不是以前的`-`

docker compose基础命令如下：

```shell
# 指定配置文件并后台启动
docker compose -f 指定配置文件.yaml up -d

# 停止容器并删除容器
docker compose -f 指定配置文件.yaml down
```

nextcloud.yaml配置文件如下

```yaml
services:
  db:
    image: mariadb:latest
    container_name: nextcloud-db
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: 设置你的数据库root密码
      MYSQL_DATABASE: 设置你的数据库名
      MYSQL_USER: 设置你的数据库用户名
      MYSQL_PASSWORD: 设置你的数据库用户密码
    volumes:
      - /mnt/nextcloud/db:/var/lib/mysql
      - 你的物理机路径:/var/lib/mysql # 这条是示例，该配置的作用是让数据库的所有文件持久存储到本地的该目录

  redis:
    image: redis:alpine
    container_name: nextcloud-redis
    restart: always

  app:
    image: nextcloud:latest
    container_name: nextcloud-app
    restart: always
    ports:
      - 80:80
    environment:
      MYSQL_HOST: db
      MYSQL_DATABASE: 你的数据库名
      MYSQL_USER: 你的数据库用户名
      MYSQL_PASSWORD: 你的数据库用户密码
      REDIS_HOST: redis
    volumes:
      - /mnt/nextcloud/data:/var/www/html
      - 你的服务器路径:/var/www/html # 这条是示例，该配置的作用是让nextcloud的所有文件持久存储到本地的该目录
    depends_on:
      - db
      - redis

volumes:
  db_data:
  nextcloud_data:
```
