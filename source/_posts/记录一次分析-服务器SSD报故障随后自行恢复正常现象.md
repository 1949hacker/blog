---
title: 记录一次分析[服务器SSD报故障随后自行恢复正常现象]
comments: true
categories: [知识库]
date: 2025-09-09 17:36:21
keywords:
  - Inspur
  - 浪潮
  - IT知识
  - 百科
  - 硬盘
  - 带外
  - BMC
tags:
  - Inspur
  - 浪潮
  - IT知识
  - 百科
  - 硬盘
  - 带外
  - BMC
---

日常巡检时发现带外出现INTEL SSD故障告警，随后又自行恢复，间隔约30分钟，无人工介入。

`smartctl -a /dev/sda`

|故障原因推测|排障思路|
|---|---|
|硬盘背板接触问题|检查smartctl中`ID 199 UltraDMA CRC Error Count`的值，是否存在CRC校验错误|
|NAND闪存寿命是否不足|检查smartctl中`ID 233 Media Wearout Indicator`的值|
|是否存在坏扇区|检查smartctl中`ID 5 Reallocated sector ct`的值|
|硬盘备用空间是否不足|检查smartctl中`ID 232 Available Reservd space`的值|

<!-- more -->

实例：

```shell
Media Wearout Indicator: 当前值0 临界值100 - 健康
Reallocated Sector Count: 当前值0 临界值75 - 健康
Available Reservd Spare: 当前值0 临界值100 - 健康
```