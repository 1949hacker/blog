---
title: YGENAS进阶手册
comments: true
categories: [知识库]
date: 2024-07-16 10:27:36
keywords:
    - YGENAS
    - 昱格NAS
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
tags:
    - YGENAS
    - 昱格NAS
    - NAS
    - DDNS
    - 存储
    - zfs
    - RAID
    - 磁盘阵列
---

# 本文为昱格NAS（YGENAS）产品进阶版使用手册

本文主要讲解昱格NAS的高级配置项，如公网端口映射、DDNS配置（配套的阿里云域名购买和子账户Access Key配置）、视频分享、filestaion配置、昱格云盘、移动设备图片智能归档等

如您需要技术支持，可联系对应的销售，或拨打400-028-0061并告知来自本站

<!-- more -->

## YGENAS配置公网访问

若您需要在公网（指办公室之外的网络，比如出差时访问NAS）环境访问NAS，首先需要您的宽带具有公网IP，可以通过光猫查看，在光猫WAN中会显示IP地址，在[https://ipinfo.io](https://ipinfo.io)中输入IP地址即可看到该地址的信息，若为非公网地址则会提示`Error: The IP address is a bogon IP. No data for this IP address.`，此时则需要您联系运营商，告知需要转公网模式，建议转为`公网桥接`模式，也叫`公网拨号`模式，该操作实测在成都电信无需任何额外费用，拨打10000号客服直接操作后重启光猫即可，随后在路由器进行拨号上网，切换公网拨号并在路由器配置拨号连接需要熟悉网络配置的人员操作，断网状态下无法提供远程技术支持，所以建议联系宽带师傅处理。
