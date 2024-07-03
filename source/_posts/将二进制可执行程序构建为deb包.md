---
title: 将二进制可执行程序构建为deb包
categories: [知识库]
comments: true
date: 2023-10-17 15:42:12
keywords: 构建deb包
tags:
    - Debian
    - Linux
    - deb
---

构建deb包所需依赖：
`apt install dh-make`

本文使用本站aptdownloader包做为演示

<!-- more -->

第一步，创建项目目录，项目目录名称格式应为`包名-版本号`如`aptdownloader-1.0.0`

```shell
# 我在我的/root/目录下创建了一个deb文件夹，也就是https://github.com/1949hacker/deb的仓库
# 使用tree列出目录结构并指定只显示3层
tree -L 3
# 目录结构如下
deb
├── LICENSE # 这个就不用解释了，git仓库的许可
├── package # 这个是我用来存构建deb的文件的文件夹
│   └── aptdownloader # aptdownloader包
│       ├── aptdownloader-1.0.0 # 按照包名-版本规则创建的文件夹
│       ├── aptdownloader_1.0.0-1_amd64.buildinfo # dpkg-buildpackage -us -uc -nc生成的文件
│       ├── aptdownloader_1.0.0-1_amd64.changes # dpkg-buildpackage -us -uc -nc生成的文件
│       ├── aptdownloader_1.0.0-1_amd64.deb # dpkg-buildpackage -us -uc -nc生成的deb包
│       └── aptdownloader_1.0.0.orig.tar.xz # dpkg-buildpackage -us -uc -nc生成的文件
├── python-source # 这个是我用来存放python源码的目录
│   ├── 64pxlogo.ico # logo的ico
│   ├── aptdownloader # aptdownloader的源码文件夹
│   │   └── aptdownloader.py # aptdownloader的源码
│   ├── dist # pyinstaller生成的可运行的二进制文件
│   │   └── aptdownloader # 二进制文件
│   └── pip # python venv的文件夹
│       ├── bin
│       ├── include
│       ├── lib
│       ├── lib64 -> lib
│       └── pyvenv.cfg
├── README_en.md # 自述文件英文版
└── README.md # 自述文件中文版

# 首先，进入python-source目录，使用以下命令激活python虚拟环境
# source后面的路径以你的python venv地址为准，your_path/bin/active
source pip/bin/activate
# 使用pyinstaller命令将aptdownloader.py转换为二进制可执行程序
# -i是指定图标，我linux都是用命令行，没试过图形化里会不会显示图标
# -F是生成单文件，后面紧跟着的就是源代码
# --distpath bin表示将生成的可执行程序输出到当前目录的bin文件夹中
# --workpath temp_build和--specpath temp_spec都是指定一个临时构建目录temp_build和一个临时spec配置文件目录temp_spec
# 使用&&链接符确保执行成功后分别rm删除两个临时目录
pyinstaller -F aptdownloader/aptdownloader.py -i 64pxlogo.ico --distpath bin --workpath temp_build --specpath temp_spec && rm temp_build -rf && rm temp_spec -rf

# 最终会将生成的程序输出到当前目录的bin文件夹
deb/python-source
├── 64pxlogo.ico # 本就存在的图标文件
├── aptdownloader # 本就存在的源码文件夹
│   └── aptdownloader.py # python源码
├── bin # 存放可执行程序的目录
│   └── aptdownloader # 生成的可执行程序
└── pip # 我创建的python venv环境目录

# 返回最外层目录，我的是deb目录且位于用户主目录，然后进入package目录
cd ~/deb
cd package
# 新建aptdownloader目录并进入
mkdir aptdownloader && cd aptdownloader

# 按照包名+版本的规则创建目录
mkdir aptdownloader-1.0.0
# 进入并使用dh_make创建项目文件
cd aptdownloader-1.0.0
dh_make --createorig
# 输入该命令后终端显示内容如下
(pip) root@vladimir:~/deb/package/aptdownloader/aptdownloader-1.0.0# dh_make --createorig
/usr/bin/dh_make:10: DeprecationWarning: 'nis' is deprecated and slated for removal in Python 3.13
  import nis
Type of package: (single, indep, library, python)
[s/i/l/p]? # 我要生成的是单包，所以这一步我按的是s键，按键就行，不需要Enter
Maintainer Name     : root
Email-Address       : root@localhost.localdomain
Date                : Fri, 21 Jun 2024 16:47:31 +0800
Package Name        : aptdownloader
Version             : 1.0.0
License             : blank
Package Type        : single
Are the details correct? [Y/n/q] # 这一步按Y确认
Currently there is not top level Makefile. This may require additional tuning
Done. Please edit the files in the debian/ subdirectory now.

# 现在当前目录便生成了debian文件夹
# 编辑debian/control文件，以你的为准，我的源内容如下，参照我给的注释修改即可
Source: aptdownloader # 来源，这个是默认生成的，不管
Section: unknown # 默认是unknown，我改成了utils（工具）
Priority: optional # 默认不用改
Maintainer: root <root@localhost.localdomain> # 这一步改成你的信息，参考下方我的control文件内容
Rules-Requires-Root: no # 默认不用改
Build-Depends: # 默认不用改
 debhelper-compat (= 13), # 默认不用改
Standards-Version: 4.6.2 # 默认不用改
Homepage: <insert the upstream URL, if relevant> # 改成你的官网，没有就不管
#Vcs-Browser: https://salsa.debian.org/debian/aptdownloader # 默认不用改
#Vcs-Git: https://salsa.debian.org/debian/aptdownloader.git # 默认不用改

Package: aptdownloader # 根据你的文件夹生成，默认不用改
Architecture: any # 默认不用改
Depends: # 默认不用改
 ${shlibs:Depends}, # 默认不用改
 ${misc:Depends}, # 默认不用改，需要前置依赖就换行写你的依赖，参考我的control文件
Description: <insert up to 60 chars description> # 60字简短描述
 <Insert long description, indented with spaces.> # 换行后再写详细描述

 # 以下是我的deb/control文件内容
Source: aptdownloader
Section: utils
Priority: optional
Maintainer: Vladimir Yang <0@hackerbs.com>
Rules-Requires-Root: no
Build-Depends:
 debhelper-compat (= 13),
Standards-Version: 4.6.2
Homepage: https://hackerbs.com
#Vcs-Browser: https://salsa.debian.org/debian/aptdownloader
#Vcs-Git: https://salsa.debian.org/debian/aptdownloader.git

Package: aptdownloader
Architecture: any
Depends:
 ${shlibs:Depends},
 ${misc:Depends},
 apt-rdepends (>= 1.0) # 我的前置需要该包，所以要这样写
Description: A tool for downloading apt packages and their dependencies.
 Use 'aptdownloader <package name>' to download the software package and its dependencies to the current directory.
 example:
    aptdownloader docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 编辑deb/install文件（需新建），告诉dpkg我的aptdownloader可执行程序应该放到/usr/bin下
# 格式为：包名 安装目录
# 我的install内容：
aptdownloader usr/bin

# 回到aptdownloader-1.0.0目录
cd /root/deb/package/aptdownloader/aptdownloader-1.
0.0/
# 将我的可执行程序aptdownloader放到该目录
cp ../../../python-source/bin/aptdownloader ./
# 现在便可执行最后一步，运行dpkg-buildpackage
dpkg-buildpackage -us -uc -nc
# 然后退回到上层aptdownloader目录，即可看到
/root/deb/package/aptdownloader
├── aptdownloader-1.0.0 # 项目版本目录
├── aptdownloader_1.0.0-1_amd64.buildinfo # dpkg-buildpackage生成的构建信息
├── aptdownloader_1.0.0-1_amd64.changes # dpkg-buildpackage生成的变更信息
├── aptdownloader_1.0.0-1_amd64.deb # dpkg-buildpackage构建好的deb包
└── aptdownloader_1.0.0.orig.tar.xz # dpkg-buildpackage生成的其他文件的归档
```

至此，构建deb包的教程结束，请移步[如何搭建具有GPGkey验证的可信任apt源]