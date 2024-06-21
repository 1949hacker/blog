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

## 使用aptly管理apt仓库

```shell
# 创建一个aptly的工作地址
mkdir -p /mypath/apt
# 使用aptly创建仓库
# bookworm是debian 12的代号
# comment则是填仓库的描述
# hackerbs是我的仓库名，修改为你的即可
aptly repo create -distribution="bookworm" -comment="hackerbs.com's repository" hackerbs

# 运行该命令后会在用户目录下生成一个.aptly.conf配置文件，和.aptly的目录
# 建议编辑.aptly.conf配置文件，修改原
"rootDir": "/root/.aptly",
# 改为指定的，如我的/mypath/apt目录
"rootDir": "/mypath/apt",
# 重新运行
aptly repo create -distribution="bookworm" -comment="hackerbs.com's repository" hackerbs
# 现在指定的/mypath/apt下便会出现一个db文件夹
# 继续往仓库添加deb包
# add后面的hackerbs是上一步创建的仓库名，仓库名后面则是包的路径，跟部署无关，这里只是添加到仓库
aptly repo add hackerbs /deb/package/aptdownloader/aptdownloader_1.0.0-1_amd64.deb
# 添加完成后创建一个快照
# create后面的aptdownloader只是我的快照名称，repo后面接你的仓库名
aptly snapshot create aptdownloader from repo hackerbs
# 使用快照发布apt存储库
# snapshot后面跟快照名
aptly publish snapshot aptdownloader
```

至此，aptly发布存储库完成，/mypath/apt/public文件夹就是网站的文件夹，所以下文中nginx指向的便是该路径

以下是aptly命令的补充

```shell
aptly publish drop buster # 取消已发布的存储库，这里指定的名字不是存储库的名字而是distribution
aptly snapshot drop imlala_snapshot # 删除快照（如果当前快照正在对外服务，需要先取消快照对应的存储库发布后才能删除）
aptly snapshot show -with-packages imlala_snapshot # 显示某个快照内有多少个包
aptly snapshot list # 列出所有快照
aptly repo show -with-packages imlala # 显示某个存储库内有多少包
aptly repo list # 列出所有存储库
aptly repo remove imlala qbittorrent-nox_4.1.7.1_amd64 # 从存储库删除包
aptly serve # 启动自带的http服务器
```

## 使用docker创建一个nginx，以下是我的docker-compose.yaml文件

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
    listen [::]:443;
    server_name mydomain;

    root /mypath/public;
    autoindex on;
    index index.html
    allow all;
}
```

然后使用`docker exec apt nginx -t`测试配置文件

继续使用`docker exec apt nginx -s reload`重载配置文件

此时nginx已经上线

## 配置GPG key

首先使用`gpg --full-generate-key`命令并按图所示创建gpg密钥

![20240621183309](https://img.hackerbs.com//20240621183309.png)

![20240621183514](https://img.hackerbs.com//20240621183514.png)

然后建议导出密钥用于备份

**私钥绝对不要泄露**

```shell
# 导出私钥，邮箱替换成你的，private.gpg也可以按你的习惯命名
gpg --export-secret-keys --armor 0@hackerbs.com > private.gpg

# 导出公钥，邮箱替换成你的，public.gpg也可以按你的习惯命名
gpg --export --armor 0@hackerbs.com > public.gpg

# 查看所有gpg key信息
 gpg --list-keys
# 我的输出如下
/root/.gnupg/pubring.kbx
------------------------
pub   rsa4096 2024-06-21 [SC]
      0380F37B181615738D90FF46C1763CF4CC699C74
uid           [ultimate] hackerbs (hackerbs.com) <0@hackerbs.com>
sub   rsa4096 2024-06-21 [E]
# 其中0380F37B181615738D90FF46C1763CF4CC699C74便是我的公钥，使用下面的命令进行导出
gpg --export --armor 0380F37B181615738D90FF46C1763CF4CC699C74 > hackerbs.asc

# 导出的公钥放到网站根目录以供下载
mv hackerbs.asc /mypath/public
```

*因为apt-key被淘汰了，所以没有推荐用apt-key的方式，不过还是补充下*

```shell
# 将公钥上传到keyserver.ubuntu.com
gpg --keyserver keyserver.ubuntu.com --send-key 630D583FFE07EDA9B9531E029CA58840D67352EE
```

在[https://keyserver.ubuntu.com/](https://keyserver.ubuntu.com/)输入邮箱地址即可查询到公钥

![20240621190029](https://img.hackerbs.com//20240621190029.png)

## 客户端添加gpg密钥和源

```shell
# 以下是本站源的演示
# 其中hackerbs.asc名称可以更改，如myname.asc，后缀要保留
# 后面的连接就是上方为什么需要将asc放到网站根目录
# &&是表示执行成功后继续执行mv将asc密钥放到/etc/apt/trusted.gpg.d/目录下
# 后面的添加到sources.list就不用再讲解了
curl -o hackerbs.asc https://apt.ygeit.cn/hackerbs.asc && mv hackerbs.asc /etc/apt/trusted.gpg.d/
echo "deb https://apt.ygeit.cn bookworm main" >> /etc/apt/sources.list
apt update
```

## 补充

如果访问有问题，记得修改public的权限为755

```shell
chmod -R 755 /mypath/apt/public
```