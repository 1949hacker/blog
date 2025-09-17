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

# 批量巡检脚本
for raid in {a..d}; do
    if [ -b "/dev/sd$raid" ]; then
        for disk in {0..23}; do
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                # 获取并分析smart数据
                output=$(smartctl -a -d megaraid,$disk /dev/sd$raid)
                echo "$output" >> disk_error.log

                # 检查是否有无法纠正的错误或非介质错误
                uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep -E "read:|write:|verify:" | awk '$NF > 0')
                non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '$NF > 0')

                # 如果有错误则输出简洁信息
                if [ -n "$uncorrected" ] || [ -n "$non_medium" ]; then
                    echo "RAID sd$raid 中的 Disk #$disk 存在以下错误："
                    [ -n "$uncorrected" ] && echo "$uncorrected"
                    [ -n "$non_medium" ] && echo "$non_medium"
                    echo -e "\n"
                fi
            fi
        done
    fi
done

# 整合为一行，一条龙服务
for raid in {a..d}; do if [ -b "/dev/sd$raid" ]; then for disk in {0..23}; do if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then output=$(smartctl -a -d megaraid,$disk /dev/sd$raid); echo "$output" >> disk_error.log; uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep -E "read:|write:|verify:" | awk '$NF > 0'); non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '$NF > 0'); if [ -n "$uncorrected" ] || [ -n "$non_medium" ]; then echo "RAID sd$raid 中的 Disk #$disk 存在以下错误："; [ -n "$uncorrected" ] && echo "$uncorrected"; [ -n "$non_medium" ] && echo "$non_medium"; echo -e "\n"; fi; fi; done; fi; done
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
