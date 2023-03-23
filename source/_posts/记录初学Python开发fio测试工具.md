---
title: 记录初学Python开发fio测试工具
categories: [软件开发]
date: 2023-03-20 18:19:44
keywords: python,fio
tags:
    - python
---

# 使用Python自动测试IOPS性能并格式化输出

>代码过多,请点击**阅读全文**查看

<!-- more -->

```python
#!/usr/bin/python3

"""
TODO:
    此脚本已测试兼容环境为Debian 11.6
    注意自行根据盘位修改下列numjobs参数
"""

import subprocess, re


# 随机写
def randwrite():
    # 初始化用于存储运行结果的列表
    bw = [0, 0, 0]
    iops = [0, 0, 0]

    # fio重复运行4次
    print("随机写进行中...")
    for i in range(4):
        cmd = [
            "fio",
            "-name=YEOS",
            "-size=32G",
            "-runtime=120s",
            "-bs=4k",
            "-direct=1",
            "-rw=randwrite",
            "-ioengine=libaio",
            "-numjobs=12",
            "-group_reporting",
            "-iodepth=64",
            f"-filename=/dev/{disk}",
        ]

        # 将fio运行结果标准输出到管道
        fio1 = subprocess.Popen(cmd, stdout=subprocess.PIPE, shell=False)
        fio2 = subprocess.Popen(
            ["grep", "samples"], stdin=fio1.stdout, stdout=subprocess.PIPE, shell=False
        )

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
        if i == 0:
            continue
        else:
            # 将每次运行结果保存到列表
            bw[0] += bw_num[0]
            bw[1] += bw_num[1]
            bw[2] += bw_num[3]
            iops[0] += iops_num[0]
            iops[1] += iops_num[1]
            iops[2] += iops_num[2]

    bwMin = int(bw[0] / 3)
    bwMax = int(bw[1] / 3)
    bwAvg = int(bw[2] / 3)
    iopsMin = int(iops[0] / 3)
    iopsMax = int(iops[1] / 3)
    iopsAvg = int(iops[2] / 3)

    print(
        f"随机写均值如下:\n带宽最小值:{bwMin},最大值{bwMax},均值{bwAvg}\nIOPS最小值:{iopsMin},"
        f"最大值{iopsMax},均值{iopsAvg}"
    )


# 随机读
def randread():
    # 初始化用于存储运行结果的列表
    bw = [0, 0, 0]
    iops = [0, 0, 0]
    # fio重复运行4次
    print("随机读进行中...")

    for i in range(4):
        cmd = [
            "fio",
            "-name=YEOS",
            "-size=32G",
            "-runtime=120s",
            "-bs=4k",
            "-direct=1",
            "-rw=randwrite",
            "-ioengine=libaio",
            "-numjobs=12",
            "-group_reporting",
            "-iodepth=64",
            f"-filename=/dev/{disk}",
        ]

        # 将fio运行结果标准输出到管道
        fio1 = subprocess.Popen(cmd, stdout=subprocess.PIPE, shell=False)
        fio2 = subprocess.Popen(
            ["grep", "samples"], stdin=fio1.stdout, stdout=subprocess.PIPE, shell=False
        )

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
        if i == 0:
            continue
        else:
            # 将每次运行结果保存到列表
            bw[0] += bw_num[0]
            bw[1] += bw_num[1]
            bw[2] += bw_num[3]
            iops[0] += iops_num[0]
            iops[1] += iops_num[1]
            iops[2] += iops_num[2]

    bwMin = int(bw[0] / 3)
    bwMax = int(bw[1] / 3)
    bwAvg = int(bw[2] / 3)
    iopsMin = int(iops[0] / 3)
    iopsMax = int(iops[1] / 3)
    iopsAvg = int(iops[2] / 3)

    print(
        f"随机读均值如下:\n带宽最小值:{bwMin},最大值{bwMax},均值{bwAvg}\nIOPS最小值:{iopsMin},"
        f"最大值{iopsMax},均值{iopsAvg}"
    )


# 随机读写
def randrw():
    # 初始化用于存储运行结果的列表
    bw = [0, 0, 0, 0, 0, 0]
    iops = [0, 0, 0, 0, 0, 0]
    # fio重复运行4次
    print("随机读写进行中...")
    for i in range(4):
        cmd = [
            "fio",
            "-name=YEOS",
            "-size=32G",
            "-runtime=120s",
            "-bs=4k",
            "-direct=1",
            "-rw=randrw",
            "-ioengine=libaio",
            "-numjobs=12",
            "-group_reporting",
            "-iodepth=64",
            f"-filename=/dev/{disk}",
            "-rwmixwrite=30",
        ]

        # 将fio运行结果标准输出到管道
        fio1 = subprocess.Popen(cmd, stdout=subprocess.PIPE, shell=False)

        fio2 = subprocess.Popen(
            ["grep", "samples"], stdin=fio1.stdout, stdout=subprocess.PIPE, shell=False
        )

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

        if i == 0:
            continue
        else:
            # 读带宽
            bw[0] += bw_num[0]
            bw[1] += bw_num[1]
            bw[2] += bw_num[3]
            # 写带宽
            bw[3] += bw_num[6]
            bw[4] += bw_num[7]
            bw[5] += bw_num[9]
            # 读IOPS
            iops[0] += iops_num[0]
            iops[1] += iops_num[1]
            iops[2] += iops_num[2]
            # 写IOPS
            iops[3] += iops_num[5]
            iops[4] += iops_num[6]
            iops[5] += iops_num[7]

    RbwMin = int(bw[0] / 3)
    RbwMax = int(bw[1] / 3)
    RbwAvg = int(bw[2] / 3)

    WbwMin = int(bw[3] / 3)
    WbwMax = int(bw[4] / 3)
    WbwAvg = int(bw[5] / 3)

    RiopsMin = int(iops[3] / 3)
    RiopsMax = int(iops[4] / 3)
    RiopsAvg = int(iops[5] / 3)

    WiopsMin = int(iops[0] / 3)
    WiopsMax = int(iops[1] / 3)
    WiopsAvg = int(iops[2] / 3)
    print(
        f"随机读写均值如下:\n"
        f"读:\n带宽最小值:{RbwMin},最大值{RbwMax},均值{RbwAvg}\nIOPS最小值:{RiopsMin},"
        f"最大值{RiopsMax},均值{RiopsAvg}"
        "\n"
        f"写:\n带宽最小值:{WbwMin},最大值{WbwMax},均值{WbwAvg}\nIOPS最小值:{WiopsMin},"
        f"最大值{WiopsMax},均值{WiopsAvg}"
    )


def rm_file():
    print("请等待程序清除测试残留文件...")
    rm = subprocess.Popen(["rm", "-rf", "/test/*"], shell=False)
    rm.wait()
    print("清除完毕,程序结束!")


if __name__ == "__main__":
    print("欢迎使用群晖测试工具\n本工具测试内容:\n设备挂载模式下IOPS性能测试")
    disk = input("输入测试/dev/下哪个设备,例sdc,请输入:")
    randwrite()
    randread()
    randrw()
```