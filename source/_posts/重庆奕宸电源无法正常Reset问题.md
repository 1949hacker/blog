---
title: 重庆奕宸电源无法正常Reset问题
categories: [知识库]
tags:
  - 电源
comments: true
date: 2022-11-21 02:39:29
keywords: 电源
---

## 重庆奕宸电源无法正常Reset问题

## 问题详情

时间：2022/10/17
故障：在TyanS7106主板上完成一次正常启动后按PowerSW5秒强制断电，再次启动时在进入BIOS之前，自检过程中按ResetSW后异常断电无法启动

<!--more-->

## 提出疑问

1. 主板PowerSW和ResetSW给到电源的控制信号是否不同，是否可能存在Tyan给到电源的信号错误
2. IPMI日志是否有报告与电源相关的日志
3. 电源的控制电路是否存在问题

## 寻找答案

1. 向电源厂商了解到：主板Power/Reset SW给到电源的控制信号是由电源控制芯片输出高低电平，从而触发电源的响应，实现对电源的控制
2. IPMI日志报告PCH_P1V05/PVNN/P1V8，VCC3_AU 临界值过低，厂商回复该日志的意思是电压过低，属于正常现象，但经对比测试，其他电源无此现象。
3. 主板控制电源的信号通用、电源响应主板控制信号的方式通用、排除主板和电源程序上的故障，和厂商进一步探讨后，得知电源收到PS-ON信号后，开机通常延迟300ms后输出PWR-OK信号，关机则是100ms，但关机的100ms延迟参数来源于网上非权威渠道，同时提出疑问，主板Reset的逻辑是怎样的？
   > 发出关机指令，等待电源回复PWR-OK后再次给出PS-ON完成开机？
   > 还是发出PS-ON后，不等待电源返回PWR-OK，直接延迟一会儿后再次发出PS-ON进行开机？
4. 等待Tyan回复的同时，和电源厂家沟通安排测试，将关机100ms的延迟在多个主板上进行复现，观察是否存在错误：
   >逐步递减100ms、80ms、60ms.........
   >直接设置为最低值，无限接近0ms进行观测
5. 根据测试结果，将电源关机时序调整为最优默认值

## 总结
在发现问题时，善用对比法、排除法进行思路梳理