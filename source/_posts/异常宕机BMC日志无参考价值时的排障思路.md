---
title: 异常宕机BMC日志无参考价值时的排障思路
comments: true
categories: [知识库]
date: 2025-09-10 18:35:21
keywords:
  - 带外
  - 宕机
  - 排障
  - Linux
  - BMC
tags:
  - 带外
  - 宕机
  - 排障
  - Linux
  - BMC
---

## 巡检发现异常宕机，带外日志仅有一条热复位的记录

BMC日志如下：

`SYS_Restart    系统启动/重新启动   开始热复位 - 触发`

仅有一条热复位记录，无物理按键记录，无操作系统发起重启记录

<!-- more -->

|排障思路|命令|
|---|---|
排查启动日志|`journalctl -b | grep -i "error\|fail\|panic\|oops" | head -20`
排查内核缓冲区|`dmesg --level=err,warn`
排查是否有内核崩溃转储文件|`ls -la /var/crash/`
排查是否有硬件错误|`journalctl -k | grep -i "hardware error"`和`dmesg | grep -i "MCE"`
排查watchdog是否触发|`journalctl -k | grep -i "watchdog"`和`dmesg | grep -i "watchdog"`
排查过热日志|`journalctl -k | grep -i "thermal\|overheat"`和`dmesg | grep -i "thermal"`
检查内存不足事件|`journalctl -k | grep -i "out of memory"`和`dmesg | grep -i "oom"`
检查 CPU 或 I/O 阻塞|`sar -q -f /var/log/sa/sa10`
检查内存错误计数|`dmidecode -t memory | grep -i "error"`
检查电源日志|`journalctl -k | grep -i "power"`