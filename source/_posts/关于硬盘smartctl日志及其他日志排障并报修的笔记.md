---
title: 关于服务器硬盘故障但带外没有错误日志的排障与报修笔记
comments: true
categories: [知识库]
date: 2025-09-18 15:49:46
keywords:
  - 服务器
  - 运维
  - 排障
  - 带外
  - 浪潮
  - 联想
  - DELL
  - 华三
  - 超聚变
  - 硬盘
  - 故障
  - 日志
tags:
  - 服务器
  - 运维
  - 排障
  - 带外
  - 浪潮
  - 联想
  - DELL
  - 华三
  - 超聚变
  - 硬盘
  - 故障
  - 日志
---

## 情况说明

收到系统发出IO占用率和IO延迟的告警，登录带外排查无任何错误日志，随后进入操作系统使用脚本批量排查smartctl日志，发现存在错误计数，因smartctl并非厂家带外的告警日志，所以特此向Inspur、H3C、Lenovo、DELL发出了确认函，其中提到了一些日志参数的告警，目前已收到H3C、Inspur的书面回复，厂商对日志中以下内容的告警表示认可并作为报修依据

| 硬盘类型 | 参数 | 翻译 | 说明 | 来源 |
| --- | --- | --- | --- | --- |
| SSD  | ID 05 Reallocated Sector Count | 重分配扇区计数     | 因坏块被重新分配的扇区数量，值越高健康状况越差        | H3C书面邮件回复 |
| SSD  | ID 197 Current Pending Sector Count | 当前待处理扇区计数   | 有潜在读写错误、待重新映射的扇区数量             | H3C书面邮件回复 |
| HDD  | Total uncorrected errors | 总无法纠正错误     |  | H3C书面邮件回复 |
| HDD | Verify total uncorrected errors | 校验无法纠正错误 | 硬盘控制器自检时无法通过ECC纠正的错误总数，高值表示可靠性下降 | H3C书面邮件回复 |
| HDD | Read total uncorrected errors | 读无法纠正错误 | 读取/写入IO时无法通过ECC纠正的错误总数，高值表示可靠性下降 | H3C书面邮件回复 |
| HDD  | Elements in grown defect list | 已增长缺陷列表中的元素 | 硬盘运行中登记的坏块数量，用于追踪坏块增长          | [@Icenowy](https://github.com/icenowy)于清华TUNA协会技术群组内回复 |
| HDD  | Write total uncorrected errors | 写入无法纠正错误总数  | 实际日志中无此项，参考上方总无法纠正错误即可 | Inspur书面邮件回复 |
| SSD | Reported Uncorrectable Errors | 已报告的不可纠正错误 | 硬盘向主机报告的读/写过程中发生的不可恢复错误次数(>10更换) | Inspur书面邮件回复 |
| SSD | Current Pending Sector Count | 当前待处理扇区计数 | 检测到潜在读写错误、等待重新分配的扇区数量(>100更换) | Inspur书面邮件回复 |
| SSD | Offline Uncorrectable | 离线不可纠正错误 | 硬盘在离线自检/后台扫描时检测到的不可恢复错误次数(>0更换) | Inspur书面邮件回复 |

**以下是辅助日志，作为协助排障参考，不作为直接依据**

| 硬盘类型 | 参数 | 翻译 | 说明 | 来源 |
| --- | --- | --- | --- | --- |
| SSD | Reallocated Sector Count | 重分配扇区计数 | 记录因物理损坏被替换到备用扇区的次数，数值增加说明介质退化(>500为不可靠) | Inspur书面邮件回复 |
| SSD | CRC Error Count | CRC 错误计数 | 记录主机与硬盘之间传输数据时发生的 CRC 校验错误次数，常见原因包括数据线接触不良、电磁干扰或接口问题，单盘较多则可能为该盘本体故障，多个硬盘则进一步筛查是否位于同一个硬盘背板或同一个SAS端口 | Inspur书面邮件回复 |
| HDD | Non-medium error count | 非介质故障 | 与上方SSD的是一样的意思 | Inspur书面邮件回复 |

---

以下是回复的截图

<!-- more -->

Inspur 浪潮

![20250918162644](https://img.hackerbs.com//20250918162644.png)

H3C 华三

![20250918162724](https://img.hackerbs.com//20250918162724.png)

清华大学TUNA协会技术群组

非书面回复，感谢[@Icenowy](https://github.com/icenowy)于*清华TUNA协会技术群组*内回复提出参考`Elements in grown defect list`参数的值，该值是HDD独有的指标，记录的是在使用过长中新发现的物理坏块数，数值持续增长则意味着该硬盘可靠性正在下降

## 排查过程

### 对于JBOD模式的硬盘

```shell
# 使用smartctl直接查询即可
smartctl -a /dev/sdX

# 可以增加grep快速筛选想要查询的参数，在替换双引号中的即可
smartctl -a /dev/sdX | grep -E "Elements in grown defect list"

# 配合for迅速遍历磁盘，{a..z}则是遍历sda到sdz的所有盘
# 如果盘很多那可以直接写为{a..zz}，则是遍历sda到sdzz共702个盘
# [ -b "/dev/sd$i" ] test命令的简写，判断该设备是不是一个硬盘
# echo "sd$i"是在输出日志前输出盘号，避免不知道是哪个盘的报错
for i in {a..z}; do [ -b "/dev/sd$i" ] && echo "sd$i:" && sudo smartctl -a "/dev/sd$i" | grep "Elements in grown defect list"; done
```

### 对于LSI MegaRAID控制器，RAID模式的硬盘

```shell
# 由于硬盘由阵列卡接管，所以需要调用MegaRAID驱动程序才可访问
# /dev/sdX输入你挂载到系统的阵列，例如通常/dev/sda是RAID1系统盘
# 其中megaraid,后面的数字对应该硬盘是该阵列中的第几个盘，从编号0开始是第一个
smartctl -a -d megaraid,0 /dev/sdX

# 同样的grep筛选
smartctl -a -d megaraid,0 /dev/sdX | grep -E "Non-medium error count"

# 遍历快速排查所有盘
for raid in {a..z};do [ -b "/dev/sd$raid" ] && for disk in {0..20};do echo -e "\033[36;1m阵列 sd$raid 中硬盘 $disk 日志: " && smartctl -a -d megaraid,$disk /dev/sd$raid | grep -E "Non-medium error count";done;done

# 上方命令为单行，方便复制粘贴执行，以下是拆解后可读的版本
for raid in {a..z}; do # 遍历sda到sdz的设备
    [ -b "/dev/sd$raid" ] && # 判断是不是硬盘，不是就跳过
    for disk in {0..20}; do # 预设是0到20共21个硬盘，可以自行设置
        echo -e "\033[36;1m阵列 sd$raid 中硬盘 $disk 日志: " # 打印个开头，不然不知道是哪个设备的日志
        smartctl -a -d megaraid,$disk /dev/sd$raid | grep -E "Non-medium error count" # 根据你的需要调整grep筛选的内容
    done
done
```
## 进阶脚本

**该脚本会在当前目录输出`smartctl`的日志源码到`disk_error_$(date +%Y%m%d_%H%M%S).log`文件**

推荐使用此单行模式，复制粘贴直接用，不需要创建shell文件

```shell
log_file="disk_error_$(date +%Y%m%d_%H%M%S).log"; for raid in {a..d}; do [ -b "/dev/sd$raid" ] && for disk in {0..23}; do smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1 && output=$(smartctl -a -d megaraid,$disk /dev/sd$raid) && serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}') && if echo "$output" | grep -q "SSD"; then reallocated_sectors=$(echo "$output" | grep "Reallocated_Sector_Ct" | awk '{print $10}'); reallocated_sectors=${reallocated_sectors:-0}; current_pending=$(echo "$output" | grep "Current_Pending_Sector" | awk '{print $10}'); current_pending=${current_pending:-0}; reported_uncorrect=$(echo "$output" | grep "Reported_Uncorrect" | awk '{print $10}'); reported_uncorrect=${reported_uncorrect:-0}; offline_uncorrect=$(echo "$output" | grep "Offline_Uncorrectable" | awk '{print $10}'); offline_uncorrect=${offline_uncorrect:-0}; crc_errors=$(echo "$output" | grep "CRC_Error_Count" | awk '{print $10}'); crc_errors=${crc_errors:-0}; [ "$reallocated_sectors" -gt 0 ] || [ "$current_pending" -gt 0 ] || [ "$reported_uncorrect" -gt 0 ] || [ "$offline_uncorrect" -gt 0 ] || [ "$crc_errors" -gt 0 ] && echo "$output" >> "$log_file" && echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD 磁盘 \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) 检测到异常：\033[0m\n\033[36;1m05 重分配扇区计数(Reallocated Sector Count)：\033[31;1m$reallocated_sectors\033[0m\n\033[36;1m197 当前待处理扇区(Current Pending Sector Count)：\033[31;1m$current_pending\033[0m\n\033[36;1m已报告的不可纠正错误(Reported Uncorrectable Errors)：\033[31;1m$reported_uncorrect\033[0m\n\033[36;1m离线不可纠正错误(Offline Uncorrectable)：\033[31;1m$offline_uncorrect\033[0m\n\033[36;1m接口CRC错误计数(CRC Error Count)：\033[31;1m$crc_errors\033[0m\n"; else read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}'); read_uncorrected=${read_uncorrected:-0}; verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}'); verify_uncorrected=${verify_uncorrected:-0}; total_uncorrected=$(echo "$output" | grep "Total uncorrected errors" | awk '{print $NF}'); total_uncorrected=${total_uncorrected:-0}; grown_defect_list=$(echo "$output" | grep "Elements in grown defect list" | awk '{print $NF}'); grown_defect_list=${grown_defect_list:-0}; non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}'); non_medium=${non_medium:-0}; [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || [ "$total_uncorrected" -gt 0 ] || [ "$grown_defect_list" -gt 0 ] || [ "$non_medium" -gt 0 ] && echo "$output" >> "$log_file" && echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD 磁盘 \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) 检测到异常：\033[0m\n\033[36;1m读无法纠正错误(Read total uncorrected errors)：\033[31;1m$read_uncorrected\033[0m\n\033[36;1m校验无法纠正错误(Verify total uncorrected errors)：\033[31;1m$verify_uncorrected\033[0m\n\033[36;1m总无法纠正错误(Total uncorrected errors)：\033[31;1m$total_uncorrected\033[0m\n\033[36;1m已增长缺陷列表元素(Elements in grown defect list)：\033[31;1m$grown_defect_list\033[0m\n\033[36;1m非介质错误(Non-medium error count)：\033[31;1m$non_medium\033[0m\n"; fi; done; done
```

```shell
#!/bin/bash

# 日志文件以日期时间命名
log_file="disk_error_$(date +%Y%m%d_%H%M%S).log"

# 遍历可能的RAID设备（sda, sdb, sdc, sdd）
for raid in {a..d}; do
    if [ -b "/dev/sd$raid" ]; then
        for disk in {0..23}; do
            if smartctl -d megaraid,$disk /dev/sd$raid -i >/dev/null 2>&1; then
                output=$(smartctl -a -d megaraid,$disk /dev/sd$raid)

                serial=$(echo "$output" | grep "Serial number:" | awk '{print $3}')

                if echo "$output" | grep -q "SSD"; then
                    # ================= SSD =================
                    reallocated_sectors=$(echo "$output" | grep "Reallocated_Sector_Ct" | awk '{print $10}')
                    reallocated_sectors=${reallocated_sectors:-0}

                    current_pending=$(echo "$output" | grep "Current_Pending_Sector" | awk '{print $10}')
                    current_pending=${current_pending:-0}

                    reported_uncorrect=$(echo "$output" | grep "Reported_Uncorrect" | awk '{print $10}')
                    reported_uncorrect=${reported_uncorrect:-0}

                    offline_uncorrect=$(echo "$output" | grep "Offline_Uncorrectable" | awk '{print $10}')
                    offline_uncorrect=${offline_uncorrect:-0}

                    crc_errors=$(echo "$output" | grep "CRC_Error_Count" | awk '{print $10}')
                    crc_errors=${crc_errors:-0}

                    if [ "$reallocated_sectors" -gt 0 ] || [ "$current_pending" -gt 0 ] || \
                       [ "$reported_uncorrect" -gt 0 ] || [ "$offline_uncorrect" -gt 0 ] || [ "$crc_errors" -gt 0 ]; then
                        echo "$output" >> "$log_file"
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 SSD 磁盘 \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) 检测到异常：\033[0m\n"
                        echo -e "\033[36;1m05 重分配扇区计数(Reallocated Sector Count)：\033[31;1m$reallocated_sectors\033[0m"
                        echo -e "\033[36;1m197 当前待处理扇区(Current Pending Sector Count)：\033[31;1m$current_pending\033[0m"
                        echo -e "\033[36;1m已报告的不可纠正错误(Reported Uncorrectable Errors)：\033[31;1m$reported_uncorrect\033[0m"
                        echo -e "\033[36;1m离线不可纠正错误(Offline Uncorrectable)：\033[31;1m$offline_uncorrect\033[0m"
                        echo -e "\033[36;1m接口CRC错误计数(CRC Error Count)：\033[31;1m$crc_errors\033[0m\n"
                    fi
                else
                    # ================= HDD =================
                    read_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^read:" | awk '{print $NF}')
                    read_uncorrected=${read_uncorrected:-0}

                    verify_uncorrected=$(echo "$output" | grep -A 10 "Error counter log:" | grep "^verify:" | awk '{print $NF}')
                    verify_uncorrected=${verify_uncorrected:-0}

                    total_uncorrected=$(echo "$output" | grep "Total uncorrected errors" | awk '{print $NF}')
                    total_uncorrected=${total_uncorrected:-0}

                    grown_defect_list=$(echo "$output" | grep "Elements in grown defect list" | awk '{print $NF}')
                    grown_defect_list=${grown_defect_list:-0}

                    non_medium=$(echo "$output" | grep "Non-medium error count:" | awk '{print $NF}')
                    non_medium=${non_medium:-0}

                    if [ "$read_uncorrected" -gt 0 ] || [ "$verify_uncorrected" -gt 0 ] || \
                       [ "$total_uncorrected" -gt 0 ] || [ "$grown_defect_list" -gt 0 ] || [ "$non_medium" -gt 0 ]; then
                        echo "$output" >> "$log_file"
                        echo -e "\033[36;1mRAID \033[32;1msd$raid \033[36;1m中的 HDD 磁盘 \033[32;1m#$disk \033[36;1m(SN:\033[33;1m $serial) 检测到异常：\033[0m\n"
                        echo -e "\033[36;1m读无法纠正错误(Read total uncorrected errors)：\033[31;1m$read_uncorrected\033[0m"
                        echo -e "\033[36;1m校验无法纠正错误(Verify total uncorrected errors)：\033[31;1m$verify_uncorrected\033[0m"
                        echo -e "\033[36;1m总无法纠正错误(Total uncorrected errors)：\033[31;1m$total_uncorrected\033[0m"
                        echo -e "\033[36;1m已增长缺陷列表元素(Elements in grown defect list)：\033[31;1m$grown_defect_list\033[0m"
                        echo -e "\033[36;1m非介质错误(Non-medium error count)：\033[31;1m$non_medium\033[0m\n"
                    fi
                fi
            fi
        done
    fi
done
```

输出效果如下

![20250918173418](https://img.hackerbs.com//20250918173418.png)
