---
title: DELL服务器硬盘IO告警排障思路
comments: true
categories: [知识库]
date: 2025-09-17 11:04:28
keywords:
  - DELL
  - IO告警
  - 硬盘
  - 排障
  - 阵列
  - RAID
tags:
  - DELL
  - IO告警
  - 硬盘
  - 排障
  - 阵列
  - RAID
---

## 系统报IO告警，在带外无异常的情况下，在操作系统中进行排障

### 故障现象

监测平台报障IO占用率和延迟过高

### 初步排障

登录带外观察是否有故障日志，无论是否有故障日志，均需要进一步进行二次核对

### JBOD直通无RAID排障

<!-- more -->

使用`smartctl`命令观察磁盘健康状态，网络上有大量教程，不再赘述

如果磁盘本身无故障，观察输出结果中的`199  UltraDMA CRC Error Count`这一行是否有记录，这是硬件通信错误的日志，如果有记录到纠正次数，那就说明发生了硬件连接的问题，如硬盘背板故障，连接线故障，接口松动等

批量排障命令参考

`for i in {a..z}; do [ -b "/dev/sd$i" ] && echo "sd$i:" && sudo smartctl -a "/dev/sd$i" | grep "199"; done`

### RAID阵列模式排障

做了阵列的磁盘无法直接使用`smartctl`进行排障，需要改为`smartctl -a -d megaraid,3 /dev/sdX`的方式排障，`megaraid,`后面跟你的硬盘的编号，`/dev/sdX`则是对应这个阵列在系统中的编号

范例：

```shell
# raid5阵列，挂载为/dev/sdb
# 阵列中disk2存在故障
smartctl -a -d megaraid,2 /dev/sdb

# 批量查询所有阵列所有盘
# 其中a..d是用于遍历/dev/sda-d，根据你的阵列情况调整
# 其中0..10是用于遍历一个阵列下有多少个硬盘，根据你的阵列情况调整
for raid in {a..d};do for disk in {0..10};do smartctl -a -d megaraid,$disk /dev/sd$raid | grep -i "CRC\|Error";done;done

for raid in {a..d}; do
    # 检查RAID虚拟设备（如/dev/sda）是否存在
    if [ -b "/dev/sd$raid" ]; then
        for disk in {0..23}; do # 扩大范围以覆盖更多磁盘
            # 尝试获取设备信息，如果命令成功退出（设备存在）则执行查询
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                echo "==== Checking /dev/sd$raid, Physical Disk #$disk ===="
                smartctl -a -d megaraid,$disk /dev/sd$raid | grep -A 20 -B 2 "Error counter log:"
                echo -e "\n"
            fi
        done
    else
        echo "RAID device /dev/sd$raid does not exist, skipping..."
    fi
done

# 整合为一行，方便直接粘贴运行
for raid in {a..d}; do if [ -b "/dev/sd$raid" ]; then for disk in {0..23}; do if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then echo "==== Checking /dev/sd$raid, Physical Disk #$disk ===="; smartctl -a -d megaraid,$disk /dev/sd$raid | grep -A 20 -B 2 "Error counter log:"; echo -e "\n"; fi; done; fi; done
```

#### 错误日志解读

范例：

![20250917112407](https://img.hackerbs.com//20250917112407.png)

|报错内容|翻译|说明|
|---|---|
|Error Corrected by ECC|ECC纠错|硬盘自纠错，某个扇区存在错误时的自动纠正|
|Total errors corrected|已纠错总次数|记录总共纠正多少次|
|Correction algorithm invocations|纠错算法调用次数|忽略|
|Gigabytes processed|纠错数据数量GB|记录总共纠正了多少GB数据|
|Total uncorrected errors|未能纠错总数|无法纠错的次数|
|Non-medium error count|非硬盘故障错误次数|非存储介质故障，例如线缆、背板、接口等硬件通信故障|
|fast rereads|快速重读|ECC快速无感纠错|
|delayed rereads|延迟重读|ECC延迟重读，导致IO延迟高|
|rewrites|重写|重读成功后重写数据|

范例中ECC纠错高达2555次，且存在4次无法纠错，且非介质故障高达13次，该硬盘急需维护。
