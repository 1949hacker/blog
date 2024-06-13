---
title: 记录一下我的Debian配置v2ray config.json
comments: true
categories: [知识库]
date: 2024-06-13 16:41:09
keywords:
    - debian
    - v2ray
    - v2ray客户端
    - config.json
tags:
    - debian
    - v2ray
    - v2ray客户端
    - config.json
---

# 配置内容如下

`vim /etc/v2ray/config.json`

```json
{
  "log": {
    "access": "",
    "error": "",
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "tag": "socks",
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ],
        "routeOnly": false
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "allowTransparent": false
      }
    }
  ],
  "outbounds": [
    {
      "tag": "proxy",
      "protocol": "shadowsocks",
      "settings": {
        "servers": [
          {
            "address": "你的服务器地址",
            "method": "加密方式",
            "ota": false,
            "password": "你的密码",
            "port": 服务器端口,
            "level": 1
          }
        ]
      },
      "streamSettings": {
        "network": "tcp"
      },
      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },
    {
      "tag": "direct",
      "protocol": "freedom",
      "settings": {}
    },
    {
      "tag": "block",
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      }
    }
  ]
}
```
