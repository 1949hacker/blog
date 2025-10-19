---
title: 手机救砖强刷 | 自制EDL工程线进入EDL9008模式并将澎湃降级为MIUI然后解BL
comments: true
categories: [知识库]
date: 2025-10-19 12:56:52
keywords:
  - EDL
  - EDL工程线
  - EDL9008
  - 救砖
  - 刷机
  - 澎湃OS
  - MIUI
  - 解BL
  - 硬核
tags:
  - EDL
  - EDL工程线
  - EDL9008
  - 救砖
  - 刷机
  - 澎湃OS
  - MIUI
  - 解BL
  - 硬核
---

本文将使用我唯一的红米K40PRO+来进行一次制作EDL工程线并强制刷写MIUI rom并解锁BL

EDL 9008(Emergency Download Mode)是高通CPU的一种特殊模式，可以绕过任何限制，直接与CPU通信刷写ROM

该方式适用于各种救砖，并且我已成功救砖过，可靠！

你需要准备：

你的手机（你最好有个备用机，别像我一样这么莽）

两根USB数据线（数据线是插到电脑上能识别并和手机传输数据的，不是纯充电线，线芯至少有4根）

你手机版本对应的ROM，例如我的是MIUI 14的中国ROM，没有解BL就直接刷跨区ROM会变砖，我就是这么变砖的

一台电脑，最好是Win11，免得win7一堆麻烦问题

[Mi Flash Pro](https://miflashpro.com/miflash-pro-v7-3-706-21)，小米官方的不行，缺驱动，其实就是小米不想让你刷

> 著名的[Xiaomi-HyperOS-BootLoader-Bypass](https://github.com/MlgmXyysd/Xiaomi-HyperOS-BootLoader-Bypass/blob/master/docs/README-zh.md)已经彻底凉了，实测最终会卡在还需等待144小时，然后一动不动

## 自制EDL工程线

一根USB数据线，剥开，找到绿色(D+, 数据正极)和黑色(GND, 接地线，也叫负极)，将这两根线短接

> 虽然没有直接证据证明，但我猜测这和短接主板上的EDL9008触点应该是一个原理

![20251019130733](https://img.hackerbs.com//20251019130733.png)

## 失败版 - 进入EDL9008模式

使用该线进入EDL9008模式

注意，电脑识别到EDL9008后断开该线，使用正常数据线连接

开始操作：

打开电脑设备管理器 -> 手机关机 -> 同时按住音量+和-键，不要松开，然后插入工程线 -> 电脑设备管理器出现EDL 9008后断开工程线

然后你就进入EDL 9008了，接下来开始降级刷MIUI14

不出意外的，失败了，小米6之后的机型，或者说，新的高通处理器已经屏蔽了这种简单的短接D+的方式

最终还是拆机短接主板上的触点

![20251019133611](https://img.hackerbs.com//20251019133611.png)

