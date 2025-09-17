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
```

```shell
# 批量巡检脚本 - 适用于HDD

# 遍历可能的RAID设备（sda, sdb, sdc, sdd）
for raid in {a..d}; do
    # 检查设备是否存在
    if [ -b "/dev/sd$raid" ]; then
        # 遍历每个RAID设备上的物理磁盘（0到23号）
        for disk in {0..23}; do
            # 检查磁盘是否可访问（尝试获取基本信息，输出重定向到/dev/null）
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                # 获取完整的SMART数据
                output=$(smartctl -a -d megaraid,$disk /dev/sd$raid)

                # 从输出中提取序列号
                serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}')

                # 提取读取无法纠正错误次数（最后一列）
                read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}')

                # 提取验证无法纠正错误次数（最后一列）
                verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}')

                # 提取非介质错误次数（最后一列）
                non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}')

                # 检查是否有任何错误（任一错误计数大于0）
                if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then
                    # 将有错误的磁盘完整SMART数据追加到日志文件
                    echo "$output" >> disk_error.log

                    # 输出格式化错误信息
                    echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"
                    echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"
                    echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"
                    echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m"
                    echo -e "\n"
                fi
            fi
        done
    fi
done

# 整合为一行，一条龙服务
for raid in {a..d}; do if [ -b "/dev/sd$raid" ]; then for disk in {0..23}; do if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then output=$(smartctl -a -d megaraid,$disk /dev/sd$raid); serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}'); read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}'); verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}'); non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}'); if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then echo "$output" >> disk_error.log; echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"; echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"; echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"; echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m"; echo -e "\n"; fi; fi; done; fi; done
```

```shell
# 批量巡检脚本 - 适用于SSD

# 遍历可能的RAID设备（sda, sdb, sdc, sdd）
for raid in {a..d}; do
    # 检查设备是否存在
    if [ -b "/dev/sd$raid" ]; then
        # 遍历每个RAID设备上的物理磁盘（0到23号）
        for disk in {0..23}; do
            # 检查磁盘是否可访问（尝试获取基本信息，输出重定向到/dev/null）
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                # 获取完整的SMART数据
                output=$(smartctl -a -d megaraid,$disk /dev/sd$raid)
                
                # 从输出中提取序列号
                serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}')
                
                # 检测磁盘类型（SSD或HDD）
                if echo "$output" | grep -q "SSD"; then
                    # SSD磁盘：使用Media_and_Data_Integrity_Errors等属性
                    media_errors=$(echo "$output" | grep "Media_and_Data_Integrity_Errors" | awk '{print $10}')
                    media_errors=${media_errors:-0}
                    
                    # 对于SSD，我们主要关注媒体错误和ECC错误
                    uncorrected_errors=$(echo "$output" | grep "Error_Information_Log_Entries" | awk '{print $10}')
                    uncorrected_errors=${uncorrected_errors:-0}
                    
                    # 检查SSD错误
                    if [ "$media_errors" -gt 0 ] || [ "$uncorrected_errors" -gt 0 ]; then
                        echo "$output" >> disk_error.log
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"
                        echo -e "\033[36;1m媒体和数据完整性错误：\033[31;1m$media_errors\033[36;1m"
                        echo -e "\033[36;1m错误信息日志条目：\033[31;1m$uncorrected_errors\033[36;1m"
                        echo -e "\n"
                    fi
                else
                    # HDD磁盘：使用原有的错误计数器
                    read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}')
                    read_uncorrected=${read_uncorrected:-0}
                    
                    verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}')
                    verify_uncorrected=${verify_uncorrected:-0}
                    
                    non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}')
                    non_medium=${non_medium:-0}
                    
                    # 检查HDD错误
                    if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then
                        echo "$output" >> disk_error.log
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"
                        echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"
                        echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"
                        echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m"
                        echo -e "\n"
                    fi
                fi
            fi
        done
    fi
done

# 一条龙服务
for raid in {a..d}; do if [ -b "/dev/sd$raid" ]; then for disk in {0..23}; do if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then output=$(smartctl -a -d megaraid,$disk /dev/sd$raid); serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}'); if echo "$output" | grep -q "SSD"; then media_errors=$(echo "$output" | grep "Media_and_Data_Integrity_Errors" | awk '{print $10}'); media_errors=${media_errors:-0}; uncorrected_errors=$(echo "$output" | grep "Error_Information_Log_Entries" | awk '{print $10}'); uncorrected_errors=${uncorrected_errors:-0}; if [ "$media_errors" -gt 0 ] || [ "$uncorrected_errors" -gt 0 ]; then echo "$output" >> disk_error.log; echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"; echo -e "\033[36;1m媒体和数据完整性错误：\033[31;1m$media_errors\033[36;1m"; echo -e "\033[36;1m错误信息日志条目：\033[31;1m$uncorrected_errors\033[36;1m"; echo -e "\n"; fi; else read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}'); read_uncorrected=${read_uncorrected:-0}; verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}'); verify_uncorrected=${verify_uncorrected:-0}; non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}'); non_medium=${non_medium:-0}; if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then echo "$output" >> disk_error.log; echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"; echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"; echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"; echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m"; echo -e "\n"; fi; fi; fi; done; fi; done

# 测试版

#!/bin/bash

# 日志文件以日期时间命名
log_file="disk_error_$(date +%Y%m%d_%H%M%S).log"

# 遍历可能的RAID设备（sda, sdb, sdc, sdd）
for raid in {a..d}; do
    # 检查设备是否存在
    if [ -b "/dev/sd$raid" ]; then
        # 遍历每个RAID设备上的物理磁盘（0到23号）
        for disk in {0..23}; do
            # 检查磁盘是否可访问（尝试获取基本信息，输出重定向到/dev/null）
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                # 获取完整的SMART数据
                output=$(smartctl -a -d megaraid,$disk /dev/sd$raid)

                # 从输出中提取序列号
                serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}')

                # 检测磁盘类型（SSD或HDD）
                if echo "$output" | grep -q "SSD"; then
                    # SSD磁盘：检查关键错误属性
                    reallocated_sectors=$(echo "$output" | grep "Reallocated_Sector_Ct" | awk '{print $10}')
                    reallocated_sectors=${reallocated_sectors:-0}

                    reported_uncorrect=$(echo "$output" | grep "Reported_Uncorrect" | awk '{print $10}')
                    reported_uncorrect=${reported_uncorrect:-0}

                    current_pending=$(echo "$output" | grep "Current_Pending_Sector" | awk '{print $10}')
                    current_pending=${current_pending:-0}

                    offline_uncorrect=$(echo "$output" | grep "Offline_Uncorrectable" | awk '{print $10}')
                    offline_uncorrect=${offline_uncorrect:-0}

                    crc_errors=$(echo "$output" | grep "CRC_Error_Count" | awk '{print $10}')
                    crc_errors=${crc_errors:-0}

                    # 检查SSD错误
                    if [ "$reallocated_sectors" -gt 0 ] || [ "$reported_uncorrect" -gt 0 ] || \
                       [ "$current_pending" -gt 0 ] || [ "$offline_uncorrect" -gt 0 ] || [ "$crc_errors" -gt 0 ]; then
                        echo "$output" >> "$log_file"
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"
                        echo -e "\033[36;1m重分配扇区计数：\033[31;1m$reallocated_sectors\033[36;1m"
                        echo -e "\033[36;1m报告无法纠正错误：\033[31;1m$reported_uncorrect\033[36;1m"
                        echo -e "\033[36;1m当前待处理扇区：\033[31;1m$current_pending\033[36;1m"
                        echo -e "\033[36;1m离线不可校正错误：\033[31;1m$offline_uncorrect\033[36;1m"
                        echo -e "\033[36;1m接口CRC错误次数：\033[31;1m$crc_errors\033[36;1m"
                        echo -e "\n"
                    fi
                else
                    # HDD磁盘：使用原有的错误计数器
                    read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}')
                    read_uncorrected=${read_uncorrected:-0}

                    verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}')
                    verify_uncorrected=${verify_uncorrected:-0}

                    non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}')
                    non_medium=${non_medium:-0}

                    # 检查HDD错误
                    if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then
                        echo "$output" >> "$log_file"
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"
                        echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"
                        echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"
                        echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m"
                        echo -e "\n"
                    fi
                fi
            fi
        done
    fi
done

# 一条龙服务
log_file="disk_error_$(date +%Y%m%d_%H%M%S).log"; for raid in {a..d}; do if [ -b "/dev/sd$raid" ]; then for disk in {0..23}; do if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then output=$(smartctl -a -d megaraid,$disk /dev/sd$raid); serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}'); if echo "$output" | grep -q "SSD"; then reallocated_sectors=$(echo "$output" | grep "Reallocated_Sector_Ct" | awk '{print $10}'); reallocated_sectors=${reallocated_sectors:-0}; reported_uncorrect=$(echo "$output" | grep "Reported_Uncorrect" | awk '{print $10}'); reported_uncorrect=${reported_uncorrect:-0}; current_pending=$(echo "$output" | grep "Current_Pending_Sector" | awk '{print $10}'); current_pending=${current_pending:-0}; offline_uncorrect=$(echo "$output" | grep "Offline_Uncorrectable" | awk '{print $10}'); offline_uncorrect=${offline_uncorrect:-0}; crc_errors=$(echo "$output" | grep "CRC_Error_Count" | awk '{print $10}'); crc_errors=${crc_errors:-0}; if [ "$reallocated_sectors" -gt 0 ] || [ "$reported_uncorrect" -gt 0 ] || [ "$current_pending" -gt 0 ] || [ "$offline_uncorrect" -gt 0 ] || [ "$crc_errors" -gt 0 ]; then echo "$output" >> "$log_file"; echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"; echo -e "\033[36;1m重分配扇区计数：\033[31;1m$reallocated_sectors\033[36;1m"; echo -e "\033[36;1m报告无法纠正错误：\033[31;1m$reported_uncorrect\033[36;1m"; echo -e "\033[36;1m当前待处理扇区：\033[31;1m$current_pending\033[36;1m"; echo -e "\033[36;1m离线不可校正错误：\033[31;1m$offline_uncorrect\033[36;1m"; echo -e "\033[36;1m接口CRC错误次数：\033[31;1m$crc_errors\033[36;1m\n"; fi; else read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}'); read_uncorrected=${read_uncorrected:-0}; verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}'); verify_uncorrected=${verify_uncorrected:-0}; non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}'); non_medium=${non_medium:-0}; if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$non_medium" -gt 0 ]; then echo "$output" >> "$log_file"; echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD Disk \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) \033[36;1m存在以下错误：\n"; echo -e "\033[36;1m读取错误无法纠正次数：\033[31;1m$read_uncorrected\033[36;1m"; echo -e "\033[36;1m磁盘自检无法纠错次数：\033[31;1m$verify_uncorrected\033[36;1m"; echo -e "\033[36;1m非介质错误次数：\033[31;1m$non_medium\033[36;1m\n"; fi; fi; fi; done; fi; done
```

输出效果如下



#### 原始错误日志解读

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
