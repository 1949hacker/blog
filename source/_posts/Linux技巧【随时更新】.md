---
title: Linux技巧【持续更新】
categories: [知识库]
tags:
  - Linux
comments: true
date: 2022-11-17 16:15:48
keywords:
  - Linux
sticky:
---

>若您有任何技术问题，可以通过本站展示的联系方式咨询我

<!--more-->

## Linux查看硬盘SN

```shell
# 将#替换为你对应的硬盘号 如sda
lsblk --nodeps -no serial /dev/sd*
```

## Linux解除文件、目录占用

```shell
# 使用该命令查看占用文件、文件夹的程序id
fuser -cu /你要查询的文件或目录
# 示例
fuser -cu docker-application/
# 我的输出结果
/mnt/TEST/docker-application: 16676m(root) 16702m(root)
# 其中16676m(root)和16702m(root)便是表明该目录由id为16676和16702的进程占用
# 使用kill id杀死进程后便可删除或使用mount -f强制卸载
```

## Linux使用iostat查看磁盘io信息

```shell
# 使用iostat可以查看简要的磁盘io信息
iostat

# 在此列出部分iostat参数:
# -c 查看CPU信息
# -d 仅显示磁盘信息,默认是显示CPU和磁盘信息
# -x 显示详细信息
# -k或-m分别是以KB/MB为显示单位
# -p显示磁盘及其分区的信息

# 演示实例
iostat -x -m 1
# 输出结果为每1秒刷新一次以MB为单位的详细信息,一直刷新
avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           1.12    1.12   20.66    5.81    0.00   71.29

Device            r/s     rMB/s   rrqm/s  %rrqm r_await rareq-sz     w/s     wMB/s   wrqm/s  %wrqm w_await wareq-sz     d/s     dMB/s   drqm/s  %drqm d_await dareq-sz     f/s f_await  aqu-sz  %util
dm-0             0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
dm-1             0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
dm-2             0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
dm-3             0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
dm-4             0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
md123            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
md124            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
md125            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
md126            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
md127            0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
nvme0n1          7.00      0.02     0.00   0.00    6.14     3.43  120.00      1.32     0.00   0.00    1.31    11.30    0.00      0.00     0.00   0.00    0.00     0.00    2.00    2.50    0.21   9.20
sda              2.00      0.00     0.00   0.00   38.00     0.00  920.00     32.91     2.00   0.22    0.52    36.63    0.00      0.00     0.00   0.00    0.00     0.00    2.00   38.50    0.63  63.60
sdb              3.00      0.00     0.00   0.00   35.33     1.33  964.00     34.25     3.00   0.31    0.37    36.38    0.00      0.00     0.00   0.00    0.00     0.00    2.00   38.00    0.54  61.20
sdc              2.00      0.00     0.00   0.00   42.00     0.00  883.00     34.25     3.00   0.34    0.82    39.72    0.00      0.00     0.00   0.00    0.00     0.00    2.00   42.00    0.89  71.20
sdd              2.00      0.00     0.00   0.00   39.50     0.00  956.00     34.21     2.00   0.21    0.36    36.65    0.00      0.00     0.00   0.00    0.00     0.00    2.00   39.00    0.50  59.20
sde              3.00      0.00     0.00   0.00   36.00     1.33  780.00     34.26     1.00   0.13    0.43    44.98    0.00      0.00     0.00   0.00    0.00     0.00    2.00   44.50    0.53  61.20
sdf              2.00      0.00     0.00   0.00   29.50     0.00  772.00     34.21     2.00   0.26    0.44    45.37    0.00      0.00     0.00   0.00    0.00     0.00    2.00   30.00    0.46  58.80
sdg              3.00      0.00     0.00   0.00   22.00     1.33  773.00     34.21     2.00   0.26    0.43    45.31    0.00      0.00     0.00   0.00    0.00     0.00    2.00   33.00    0.46  58.00
sdh              2.00      0.00     0.00   0.00   49.50     0.00  738.00     33.76     3.00   0.40    0.66    46.85    0.00      0.00     0.00   0.00    0.00     0.00    2.00   49.50    0.68  64.80
sdi              3.00      0.00     0.00   0.00   31.33     1.33  680.00     34.21     2.00   0.29    0.86    51.52    0.00      0.00     0.00   0.00    0.00     0.00    2.00   47.00    0.77  66.80
sdj              2.00      0.00     0.00   0.00   44.50     0.00  801.00     34.25     1.00   0.12    0.43    43.79    0.00      0.00     0.00   0.00    0.00     0.00    2.00   44.00    0.52  61.20
sdk              2.00      0.00     0.00   0.00   49.00     0.00  675.00     33.88     3.00   0.44    1.00    51.39    0.00      0.00     0.00   0.00    0.00     0.00    2.00   49.00    0.87  71.20
sdl              2.00      0.00     0.00   0.00   38.50     0.00  697.00     34.62     1.00   0.14    1.20    50.87    0.00      0.00     0.00   0.00    0.00     0.00    2.00   38.50    0.99  76.40
sdm              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00
sr0              0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00      0.00     0.00   0.00    0.00     0.00    0.00    0.00    0.00   0.00

# 其中1代表刷新频率,1秒,3代表次数,刷新3次,如果不填写3,则为一直刷新
iostat -x -m 1 3
```

### 参数详解

|参数|释义|
|---|---|
|-c|显示CPU利用率报告|
|--compact|不要将“设备利用率报告”拆分为子报告，以便所有指标都显示在一行中|
|-d|显示设备利用率报告|
|--dec={ 0 \| 1 \| 2 }|指定要使用的小数位数（0到2，默认值为2）|
|-f|官方对该参数的解释略微复杂,烦请查阅[官方文档](https://manpages.debian.org/testing/sysstat/iostat.1.en.html#f)|
|-g|将所有设备汇总显示,-g参数后面填group_name|
|-H|此选项必须与选项-g一起使用，并指示只显示组的全局统计信息，而不显示组中单个设备的统计信息|
|-h|该选项将输出结果分类显示,便于阅读|
|--human|以人类可读格式打印大小（例如1.0k、1.2M等）。使用此选项显示的单位将取代与度量相关的任何其他默认单位（例如千字节、扇区…）|
|-j|在参数后跟(ID/LABEL/PATH/UUID)等,以替换原/dev/sd*的显示,[官方文档](https://manpages.debian.org/testing/sysstat/iostat.1.en.html#j)|
|-k|以KB为单位进行显示|
|-m|以MB为单位进行显示|
|-N|显示任何设备映射器设备的注册设备映射器名称。用于查看LVM2统计信息|
|-o JSON|以json格式输出结果|
|-p|显示磁盘及其分区信息,可以在后跟指定磁盘以显示单个磁盘的分区信息|
|--pretty|将报告以更易于阅读的方式显示,同--human|
|-s|显示报告的短（窄）版本，该版本应适合80个字符宽的屏幕|
|-t|打印显示的每个报告的时间。时间戳格式可能取决于S_TIME_format环境变量的值|
|-V|打印iostat版本号|
|-x|显示扩展统计信息|
|-y|如果在给定的时间间隔内显示多个记录，则省略系统启动后的第一个统计报告|
|-z|告诉iostat忽略在采样期间没有活动的任何设备的输出|

### 输出结果参数解读

|参数|释义|
|---|---|
|%user|显示在用户级别（应用程序）执行时发生的 CPU 使用率百分比|
|%nice|显示在具有良好优先级的用户级别执行时发生的 CPU 使用率百分比|
|%system|显示在系统级别（内核）执行时发生的 CPU 使用率百分比|
|%iowait|显示系统有未完成的磁盘 I/O 请求期间 CPU 或 CPU 空闲的时间百分比|
|%steal|显示虚拟机管理程序为另一个虚拟处理器提供服务时虚拟 CPU 或 CPU 非自愿等待所花费的时间百分比|
|%idle|显示 CPU 或 CPU 空闲且系统没有未完成的磁盘 I/O 请求的时间百分比|
|r/s|设备每秒完成的读取请求数（合并后）|
|rMB/s|每秒读字节数,单位随参数-k/-m分别对应KB和MB|
|rrqm/s|每秒合并到设备队列的读取请求数|
|%rrqm|在发送到设备之前合并在一起的读取请求的百分比|
|r_await|平均每次设备读I/O操作的等待时间 (毫秒)|
|rareq-sz|向设备发出的读取请求的平均大小（以KB为单位）|
|w/s|设备每秒完成的写入请求数（合并后）|
|wMB/s|每秒写字节数,单位随参数-k/-m分别对应KB和MB|
|wrqm/s|每秒合并到设备队列的写入请求数|
|%wrqm|在发送到设备之前合并在一起的写入请求的百分比|
|w_await|平均每次设备写I/O操作的等待时间 (毫秒)|
|wareq-sz|向设备发出的写入请求的平均大小（以KB为单位）|
|d/s|设备每秒完成的丢弃请求数（合并后）|
|dMB/s|每秒为设备丢弃的扇区数,单位随参数-k/-m分别对应KB和MB|
|drqm/s|每秒排队到设备的合并丢弃请求数|
|%drqm|在发送到设备之前合并在一起的丢弃请求的百分比|
|d_await|向要服务的设备发出的丢弃请求的平均时间（以毫秒为单位）。这包括队列中的请求所花费的时间以及为它们提供服务所花费的|
|dareq-sz|向设备发出的丢弃请求的平均大小（以KB为单位）|
|f/s|设备每秒完成的刷新请求数（合并后）。这对磁盘执行的刷新请求进行计数。不跟踪分区的刷新请求。在合并之前，刷新操作算作写入操作|
|f_await|向要服务的设备发出刷新请求的平均时间（以毫秒为单位）。块层合并刷新请求并且一次最多执行一个。因此刷新操作可能是两倍长：等待当前刷新请求，然后执行它，然后等待下一个|
|aqu-sz|向设备发出的请求的平均队列长度,在以前的版本中,此字段称为 avgqu-sz|
|%util|向设备发出 I/O 请求所用时间的百分比（设备的带宽利用率）。对于串行服务请求的设备，当此值接近 100% 时，就会发生设备饱和。但对于并行处理请求的设备，例如 RAID 阵列和现代 SSD，这个数字并不反映它们的性能限制|

> %iowait的值过高,表示硬盘存在I/O瓶颈
> %idle值高,表示CPU较空闲
> %idle值高但系统响应慢时,有可能是CPU等待分配内存,此时应加大内存容量
> %idle值如果持续低于10,那么系统的CPU处理能力相对较低,表明系统中最需要解决的资源是CPU
> %util 接近 100%，说明产生的I/O请求太多，I/O系统已经满负荷，该磁盘可能存在瓶颈
>
> 该文章内容参考[iostat官方文档https://manpages.debian.org/testing/sysstat/iostat.1.en.html](https://manpages.debian.org/testing/sysstat/iostat.1.en.html)

## fio测试工具详细教程

> 待完善,先临时记录一条

```shell
fio -name=stress_disk -filename=/mnt/DATA/test/a -size=30T -runtime=240h -bs=1m -direct=1 -rw=randrw -ioengine=libaio -numjobs=12 -group_reporting -iodepth=8 -rwmixwrite=50 -time_based -ramp_time=60s

# 其中name是本次测试任务的名称
# filename则为指定测试目录和文件名
# size则为指定测试文件大小,此处为30T,无论运行多久,该测试文件a都不会超过30T
# runtime为指定运行时间,240h则为240小时,单位可以为秒s分m时h
# bs为块大小,顺序读写通常为1m,4k则填写4k即可
# direct有0和1选项,0为使用文件系统缓存,1则为直接操作磁盘
# rw可选的参数有read/write/rw/randread/randwrite/randrw,分别为读/写/读写/随机读/随机写/随机读写
# ioengine IO测试引擎,在linux下选择libaio即可
# numjobs线程数,一般设置为硬盘数量,根据实际需要自行调整
# group_reporting合并报告,将性能测试结果合并汇报,显示所有硬盘性能测试的总结果
# iodepth IO队列深度,指定IO队列深度,提高每次请求的IO数量
# rwmixwrite/rwmixread 这两个参数都可以指定,按百分比指定,如rwmixwrite=30则表示写占比30%,只需使用其中一个即可,无需同时指定两个参数
# time_based 设置即便fio写完了指定的size依旧不结束，直至runtime的时间
# ramp_time=60s 设置fio执行任务时的预热时间，可以使性能测试结果更加精确可靠
```

## Linux NFS挂载及使用systemctl自动挂载

```shell
# NFS挂载基础命令及其作用
mount -t nfs -o vers=3,rw,hard,sync 10.0.0.100:/mnt/TEST/nfs /mnt/100nfs
# mount linux挂载命令
# -t nfs 指定挂载协议为nfs -o也就是options，选项的意思
# vers=3 指定NFS版本，NFSv3
# rw 读写
# hard硬链接
# 补充：soft为软连接，二者的特点是硬链接保障数据安全但需要持续等待服务器响应，软连接的特点是快速响应但如果服务器断开则可能导致数据损坏或丢失
# 简单来说，硬链接适合网络环境较差或数据安全要求严格的场景，软连接适合网络稳定需要快速响应的场景
# 【技术是严谨的，如果我的解释有误烦请立刻指正，十分感谢！】
# sync，同步写入，写入到磁盘，IO响应较慢但数据安全，async，异步写入，IO响应快，数据安全性低，类似RAID write back
# 参数指定完成后便是服务器路径和本地路径，格式为 服务器地址:路径 本地路径，如 example:/example /example

# 使用systemctl管理nfs挂载
# 创建nfs.service文件【我的环境为Debian 11.6，该文件放在/usr/lib/systemd/system/下】
# 2024/1/13更新，因实际使用过程中遇到过网络并未第一时间连通导致的挂载失败，所以在mount操作前增加ping操作
# 原命令为：ExecStart=/usr/bin/mount -t nfs -o vers=3,rw,hard,sync 10.0.0.100:/mnt/data /data
# 下列代码中10.0.0.100为我的nfs服务器地址，需要替换为你自己的，包括服务器的路径:/mnt/data/storage和挂载路径/mnt/nfs都需要替换为你自己的
# 如有问题可以通过文末的联系方式咨询我
[Unit]
Description=auto mount nfs
After=network.target

[Service]
Type=simple
ExecStart=/bin/bash -c 'until ping -c1 10.0.0.100 &>/dev/null; do sleep 5; done && /usr/bin/mount -t nfs -o vers=4,rw,hard,sync 10.0.0.100:/mnt/data/storage /mnt/nfs'
Restart=no

[Install]
WantedBy=multi-user.target

# 挂载nfs
systemctl start nfs.service
# 开机自动挂载
systemctl enable nfs.service
```

## mdadm管理软RAID阵列

mdadm全称Multiple Disk and Device Administration（多磁盘和设备管理），是一个Linux软件RAID管理工具，可以使用它来创建各种级别的RAID阵列，包括RAID 0、RAID 1、RAID 5、RAID 6等。

### 使用mdadm创建RAID的一般步骤：

1. 安装mdadm软件

在大多数Linux发行版中，mdadm已经预装了，如果没有预装，则可以使用`sudo apt install mdadm`命令进行安装

1. 准备好磁盘分区，推荐使用`parted`进行磁盘分区

2. 创建RAID设备

使用mdadm命令创建RAID设备。例如，以下命令将使用指定的磁盘分区创建一个RAID5设备

```shell
sudo mdadm --create /dev/md0 --level=5 --raid-devices=3 /dev/sda1 /dev/sdb1 /dev/sdc1
```

其中，/dev/md0是要创建的RAID设备的名称，自定义为md开头的设备即可，如`md233`等，--level=5指定RAID级别为5（RAID 5），--raid-devices=3指定有3个磁盘参与RAID，/dev/sda1、/dev/sdb1和/dev/sdc1是要组成RAID设备的分区。

4. 格式化RAID设备

创建RAID设备后，需要对其进行格式化，以便可以在其中存储数据。可以使用mkfs命令格式化RAID设备。例如，以下命令将对RAID设备进行ext4格式化：

```shell
sudo mkfs.ext4 /dev/md0
```

1. 挂载RAID设备

格式化RAID设备后，需要将其挂载到文件系统中，以便可以访问其中的数据。可以使用mount命令将RAID设备挂载到指定的挂载点。例如，以下命令将RAID设备挂载到/mnt/raid5目录：

```shell
sudo mount /dev/md0 /mnt/raid5
```

### 使用mdadm删除RAID的一般步骤：

1. 卸载RAID设备

在删除RAID设备之前，需要先将其从文件系统中卸载。可以使用以下命令卸载挂载在/mnt/raid0目录的RAID设备：

```shell
sudo umount /mnt/raid5
```

2. 停用RAID设备

在删除RAID设备之前，需要停用该设备。可以使用以下命令停用RAID设备：

```shell
sudo mdadm --stop /dev/md0
```

其中，/dev/md0是要停用的RAID设备的名称。

3. 删除RAID设备

在停用RAID设备之后，可以使用以下命令删除RAID设备：

```shell
sudo mdadm --remove /dev/md0
```

其中，/dev/md0是要删除的RAID设备的名称。

4. 清除RAID设备元数据

在删除RAID设备之后，需要清除RAID设备的元数据，以确保在以后不会误认为该设备是RAID设备。可以使用以下命令清除RAID设备的元数据：

```shell
sudo mdadm --zero-superblock /dev/sda1 /dev/sdb1 /dev/sdc1
```

其中，/dev/sda1、/dev/sdb1和/dev/sdc1是原始磁盘分区，是用来创建RAID设备的。使用该命令清除元数据可以确保将来使用这些分区时不会出现问题。

以上就是删除mdadm创建的RAID设备的步骤。请注意，在删除RAID设备之前，务必备份其中的数据，以免误删除数据。

## fstab自动挂载失败导致无法开机的解决办法

首先，fstab的挂载强烈建议使用UUID，可以通过`blkid`命令查询UUID，UUID是不会变的，但是你的sda可能变成sdb，尤其是在iscsi挂载时

通常`fstab`的挂载命令中有一条`default`，改为`nofail`即可。
```shell
# 原挂载方式
/dev/sdc1 /mnt/data xfs default 0 0
# 修改为nofail，该命令的作用为在挂载时如果失败则不会阻止系统启动。使用UUID挂载
UUID="4048c1b1-1bde-49ee-8d40-8d437ce32783" /mnt/data xfs nofail 0 0
```

**注意！修改fstab后强烈建议使用`mount -a`来测试挂载是否正常**

**使用`blkid`查询UUID时，如果你的设备过多，可以指定某个设备或其分区查询，这样还可避免粗心错误挂载了设备本身，例如`blkid /dev/sdc1`这样查询的便是sdc设备的sdc1分区**

## Linux使用stress-ng让CPU始终维持在百分比占用的办法

首先，你需要安装stress-ng，同时，也推荐你使用dstat观察CPU占用率
这两个软件通过`apt install stress-ng dstat`即可安装

```shell
stress-ng --cpu <线程数> --cpu-method matrixprod --cpu-load <百分比数字> --matrix-size 400 --timeout 0 --metrics-brief
# 参数解析
# --cpu <线程数>是使用多少个线程
# --cpu-load <百分比数字>是占用率维持在多少
# 上面这个两个参数需要配合使用，假设你的CPU是4核8线程，想要达到整个CPU50%占用率
# 则需要设置为--cpu 8 --cpu-load 50或者是--cpu 4 --cpu-load 100，总占用率=线程数*百分比
# --cpu-method matrixprod 这个选项指定了要使用的 CPU 负载类型，这里设置为 matrixprod，表示使用矩阵乘法来进行负载测试
# --matrix-size 400 这个选项指定了矩阵乘法中的矩阵大小，这里设置为 400x400，表示要进行 400x400 的矩阵乘法计算
# --timeout 0表示一直运行，也可设置为秒s分m时h的自定义时间如120s，也可写为2m
# --metrics-brief 这个选项指定了在结束测试后输出的统计信息，这里设置为 brief，表示只输出简要的统计信息 -- 如果只是单纯为了占用CPU，可以不用这个

# 示例，56核112线程的CPU使其占用维持在64%
# 可以采用两个方案，控制满载的线程，或者直接设定总负载率
# 因为我需要其余线程用于测试，所以下面采用的是控制满载线程的方式，使剩余线程空闲
stress-ng --cpu 72 --cpu-method matrixprod --cpu-load 100 --matrix-size 400 --timeout 0 --metrics-brief

# 补充，因为stress-ng需要使用内存，考虑到你可能需要限制占用的内存量，可以使用以下方式实现
# 指定--vm-bytes参数，该参数的作用是每个线程能使用的内存大小
# --vm-bytes参数允许的范围是4KB-256TB，默认似乎是20MB
# 我修改后的测试代码如下：
stress-ng --cpu 72 --cpu-method matrixprod --cpu-load 100 --vm-bytes 4K --matrix-size 400 --timeout 0 --metrics-brief
```

## Linux使用vsftp部署 FTP服务器的办法

---
本站新发布了使用docker部署 vsftpd的方法，推荐使用docker部署，请查看文章《使用Docker部署vsftpd》
---

第一步，根据你的发行版使用`apt`或`yum`及其他命令，安装`vsftp`软件包

![20230906162643](https://img.hackerbs.com//20230906162643.png)

第二步，编辑`vsftpd.conf`文件

![20230906162829](https://img.hackerbs.com//20230906162829.png)

确认以下内容：

|参数|用途|
|---|---|
|anonymous_enable=NO|禁止匿名访问|
|local_enable=YES|允许本地用户登录|
|write_enable=YES|允许用户上传文件|
|chroot_local_user=YES|限制用户的根目录为其主目录|

第三步，添加以下行以指定允许登录的用户列表，这里我们创建一个名为`test`的用户

```shell
userlist_enable=YES
userlist_file=/etc/vsftpd.userlist
userlist_deny=NO
local_root=指定目录
```

编辑完成后保存退出

![20230906171602](https://img.hackerbs.com//20230906171602.png)

第四步，创建指定用户，此处以`abc`用户作为演示

使用`useradd abc`创建`abc`用户

**各发行版创建过程并不一致，请以你的发行版为准！**

![20230906171833](https://img.hackerbs.com//20230906171833.png)

使用`mkdir`命令，创建一个用于ftp的目录

修改该目录所有者及所有组权限为`abc`，并赋予755权限

```shell
# 因vsftp安全策略原因，用户主目录，也就是/opt/ftp，将无法具备写权限，所以需要创建子目录用于ftp文件传输
mkdir -p /opt/ftp/data
#设置abc用户主目录为/opt/ftp/abc
usermod -d /opt/ftp abc
#移除abc用户主目录/opt/ftp的写权限，否则无法登录
chmod a-w /opt/ftp
#修改目录所有者及所有组
chown -R abc:abc /opt/ftp/data
#修改目录权限为755
chmod -R 755 /opt/ftp/data

#此处/opt/ftp则为上文中local_root=指定目录：local_root=/opt/ftp
```

根据上文的`userlist_file=/etc/vsftpd.userlist`，在`/etc`目录下创建`vsftpd.userlist`文件并编辑内容

![20230906164114](https://img.hackerbs.com//20230906164114.png)

输入用户名`abc`退出保存即可，如需添加多个用户，每行一个即可

![20230906171208](https://img.hackerbs.com//20230906171208.png)

第五步，重启`vsftp`服务

`systemctl restart vsftpd`

![20230906164916](https://img.hackerbs.com//20230906164916.png)

如果需要让该ftp用户`abc`无法登录到系统，仅访问ftp，则按照以下步骤设置

首先，找到系统中的`/etc/base`或`/etc/shells`，通常系统中会同时存在这两个文件，需要分别查看这两个文件，选择有内容的那一个，如图，`/etc/bash`为空，则该文件无效

![20230906183203](https://img.hackerbs.com//20230906183203.png)

编辑有文件的`/etc/shells`，在文件末尾添加`/bin/false`后保存退出

![20230906183254](https://img.hackerbs.com//20230906183254.png)

随后使用`usermod -s /bin/false abc`即可使`abc`用户无法登录到系统，但可正常登录ftp，若ftp依旧无法登录，使用上文中的方式重启`vsftp`服务即可。