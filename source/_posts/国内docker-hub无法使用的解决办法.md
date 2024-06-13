---
title: 国内docker hub无法使用的解决办法
sticky: 100
comments: true
categories: [知识库]
date: 2024-06-12 18:16:26
keywords:
    - docker
    - docker hub
    - docker pull
    - docker 代理
tags:
    - docker
    - docker hub
    - docker pull
    - docker 代理
---

# 首先你得有代理，比如v2ray，然后系统里有socks5，这个就不能放在网站上说明了，懂得都懂，需要技术支持的页面底部，联系我

**在QQ`粒子｜服主社区 ①群`的用户，直接群里@hackerbs即可**

因为有些东西不能直接放到网站上讲，所以网站上的教程直接从假设你已经在你的Linux系统里有了v2ray，并在系统中有socks5，如果你不懂，那就按上面说的，联系我或者群里@我

如果你想google，想ChatGPT，但是不知道在哪儿买便宜好用的v2ray服务，一样，联系我

如果你真的很菜很菜，完全不懂Linux，但是又需要在Linux里使用v2ray，可以联系我，如果我空的时候，会友情免费帮你，或者根据你的经济情况有偿协助。

> 怕小白不懂，在此说的明白点：使用代理，也就是fan qiang，违法，不能明目张胆的说，懂嘛？

展开阅读

<!-- more -->

那么，直接从已经配置好socks5和安装好了docker开始

**如果你还没安装docker，那么搜本站的[apt等各种源列表（持续更新）](https://hackerbs.com/apt%E7%AD%89%E5%90%84%E7%A7%8D%E6%BA%90%E5%88%97%E8%A1%A8%EF%BC%88%E6%8C%81%E7%BB%AD%E6%9B%B4%E6%96%B0%EF%BC%89.html)**

使用`mkdir /etc/systemd/system/docker.service.d`创建目录，然后使用`vim /etc/systemd/system/docker.service.d/http-proxy.conf`编辑该文件，文件内容如下：

```json
[Service]
Environment="HTTP_PROXY=socks5://IP地址:端口/"
Environment="HTTPS_PROXY=socks5://127.0.0.1:10808/"
```

保存退出后使用`systemctl daemon-reload`重载systemd，然后使用`systemctl restart docker`重启docker，接下来你就可以愉快的使用docker pull了

如果你之前配置了国内docker hub，记得删掉，也就是`/etc/docker/daemon.json`中的内容，如下所示：

```json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```
