---
title: 将二进制可执行程序构建为deb包
categories: [知识库]
date: 2023-10-17 15:42:12
keywords: 构建deb包
tags:
    - Debian
    - Linux
    - deb
---

构建deb包所需依赖：
`apt install dh-make`

<!-- more -->

第一步，创建项目目录，项目目录名称格式应为`包名-版本号`如`aptdownloader-1.0.0`

进入项目目录`cd aptdownloader-1.0.0`，使用`dh_make --createorig`命令创建项目文件，并根据提示选择包类型`single, indep, library, python`，示例中创建的是`single`类型，所以按`s`，紧接着确认项目信息，确认按y，退出按q，项目信息如下：

|名称|翻译|类型|
|---|---|
|Maintainer Name|维护人员名称|根据系统`DEBFULLNAME`自动获取|
|Email-Address|电子邮件地址（邮箱）|根据系统`DEBEMAIL`自动获取|
|Date|日期|自动生成|
|Package Name|包名称|根据当前项目目录名称生成`aptdownloader`|
|Version|版本|根据当前项目目录名称生成`-1.0.0`|
|License|许可证|自动生成|
|Package Type|包类型|根据s/i/l/p选择生成|

维护人员名称修改方式：`export DEBFULLNAME="Your Name"`
电子邮件地址修改方式：`export DEBEMAIL="your.email@example.com"`

修改后再运行`dh_make --createorig`便可看到修改后的信息。

**注：项目目录名称必须严格按照`包名-版本号`命名，否则dh_make会报错**

![20231017163920](https://img.1949hacker.cn//20231017163920.png)

上述命令执行完毕后会生成`debian`目录，其中需要修改的是`control`和`rules`，以及创建`install`文件

**本文因只需要将二进制可执行程序打包为deb，所以采用默认`rules`即可，故不对rules进行讲解**

`control`文件内容如下：

```shell
Source: test
Section: unknown
Priority: optional
Maintainer: root <root@localhost.localdomain>
Rules-Requires-Root: no
Build-Depends:
 debhelper-compat (= 13),
Standards-Version: 4.6.2
Homepage: <insert the upstream URL, if relevant>
#Vcs-Browser: https://salsa.debian.org/debian/test
#Vcs-Git: https://salsa.debian.org/debian/test.git

Package: test
Architecture: any
Depends:
 ${shlibs:Depends},
 ${misc:Depends},
Description: <insert up to 60 chars description>
 <Insert long description, indented with spaces.>
 ```

这里直接给出我的修改结果：

```shell
Source: aptdownloader
Section: utils
Priority: optional
Maintainer: Vladimir Yang <scepter@1949hacker.cn>
Build-Depends: debhelper-compat (= 13)
Standards-Version: 4.5.1
Homepage: https://1949hacker.cn

Rules-Requires-Root: no

Package: aptdownloader
Architecture: any
Depends: ${shlibs:Depends}, ${misc:Depends}, apt-rdepends (>= 1.0)
Description: A tool for downloading apt packages and their dependencies.
 Use 'aptdownloader <package name>' to download the software package and its dependencies to the current directory.
 example:
    aptdownloader docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

其中手动修改的内容有：

|名称|修改为|
|---|---|
|Maintainer|姓名 电子邮箱|
|Homepage|建议修改为该项目的介绍网页|
|Depends|该项较为关键，除初始的`${shlibs:Depends}, ${misc:Depends}`外，如果你的项目依赖额外的deb包，则需要在此注明，以逗号分隔|
|Description|冒号后面写不超过60个字符的简介，然后换行缩进1个空格再写完整描述|

`install`文件决定了你的程序放到哪个目录，本文的示例将`aptdownloader`文件放到`/usr/bin`目录，则`install`文件内容如下：

```shell
aptdownloader usr/bin
```

将你的程序放到项目的根目录，例如我的项目名为`aptdownloader-1.0.0`则我的二进制可执行程序`aptdownloader`路径为`aptdownloader-1.0.0/aptdownloader`，随后在项目根目录运行命令`dpkg-buildpackage -us -uc -nc`，随后便会在项目目录的上级目录中生成`.buildinfo`、`.changes`、`.deb`三个文件，如图所示

![20231017171756](https://img.1949hacker.cn//20231017171756.png)