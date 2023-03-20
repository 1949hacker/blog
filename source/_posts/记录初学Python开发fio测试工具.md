---
title: 记录初学Python开发fio测试工具
categories: [软件开发]
date: 2023-03-20 18:19:44
keywords: python,fio
tags:
    - python
---

```python
import subprocess,re


# TODO 将下面代码段整理为方法传参调用

print("IOPS标准同步4K设备区块路径挂载写、读、73读写测试开始！\n正在运行中...")

# 初始化用于存储运行结果的列表
bw = [0,0,0]
iops = [0,0,0]

# fio重复运行4次
for i in range(4):
    # 将fio运行结果标准输出到管道
    fio1 = subprocess.Popen(["fio", "-name=YEOS", "-size=100G", "-runtime=60s", "-bs=4k", "-direct=1", "-rw=randwrite", "-ioengine=libaio", "-numjobs=12", "-group_reporting", "-iodepth=64", "-filename=/data/"+str(i)], stdout=subprocess.PIPE)
    # 获取fio1管道内容进行格式化后再进行标准输出到fio2管道
    fio2 = subprocess.Popen(["grep","samples"], stdin=fio1.stdout, stdout=subprocess.PIPE)
    # 使用communicate获取子进程的标准输出并格式化为utf-8编码
    fio = fio2.communicate()[0].decode("utf-8")

    # 初始化列表
    bw_num = []
    iops_num = []

    # 匹配数字和小数点，并将其元素更新
    for line in fio.split("\n"):
        if "bw" in line:
            bw_num.extend(re.findall(r"\d+\.\d+|\d+", line))
        elif "iops" in line:
            iops_num.extend(re.findall(r"\d+\.\d+|\d+", line))

    # 将str类型转换为float后再转换为int
    bw_num = [int(float(e)) for e in bw_num]
    iops_num = [int(float(e)) for e in iops_num]

    # 将每次运行结果保存到列表
    bw[0] += bw_num[0]
    bw[1] += bw_num[1]
    bw[2] += bw_num[3]
    iops[0] += iops_num[0]
    iops[1] += iops_num[1]
    iops[2] += iops_num[2]
    print(f"第{i+1}次运行结果如下：\n带宽最小值：{bw_num[0]}，最大值{bw_num[1]}，均值{bw_num[3]}\nIOPS最小值：{iops_num[0]}，最大值{iops_num[1]}，均值{iops_num[2]}")

print(f"共4次运行，均值如下：\n带宽最小值：{bw[0]/4}，最大值{bw[1]/4}，均值{bw[2]/4}\nIOPS最小值：{iops[0]/4}，最大值{iops[1]/4}，均值{iops[2]/4}")

# TODO 将最终结果直接输出到Excel表格
```