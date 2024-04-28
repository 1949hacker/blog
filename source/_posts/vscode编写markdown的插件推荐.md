---
title: vscode编写markdown的插件推荐
comments: true
date: 2022-10-26 11:09:08
keywords: vscode,markdown
categories: [知识库]
tags:
    - markdown
---

# 使用 VScode 编写 Markdown 的插件推荐及其技巧

<!--more-->

## Markdown 插件推荐

- Markdown Preview Enhanced
- Markdown PDF
- LaTeX Workshop
- Paste Image
- Markdown All in One

## Markdown 插件详细介绍

### Markdown Preview Enhanced

该插件可以对 Markdown 源码进行实时渲染预览，通过配置插件的

> Markdown-preview-enhanced: Automatically Show Preview Of Markdown Being Edited
> 
> - [x] Automatically show preview of markdown being edited.

可以实现点击 md 文件编辑的同时自动打开实时渲染界面

![2022-10-14-16-18-21](https://img.hackerbs.com//2022-10-14-16-18-21.png)

### Markdown PDF

该插件可以将 md 文件转换为**pdf、html、png、jpeg**四种格式，配合 vscode 快捷键`Ctrl+Shift+P`并输入`>export (pdf)`后回车即可将当前打开的 md 文件转换为 PDF 格式，替换`(pdf)`中的`pdf`为`html、png、jpeg`其中任意一种，即可导出对应的格式
使用`>export (all: pdf,html,png,jpeg)`可以一次性导出为 4 种格式

### LaTeX Workshop

该插件可以使 vscode 支持 latex 数学公式代码
示例如下：

```LaTex
$$
M =
\begin{pmatrix}
1 & 0 \\
0 & 1
\end{pmatrix}
\begin{pmatrix}
1 & 0 \\
0 & 1
\end{pmatrix}
$$
```

LaTeX 语法可以自行搜索，此处给出一条参考链接：[LaTeX 语法](http://www.uinio.com/Math/LaTex/)

### Paste Image

使用 Paste Image 可以快速将粘贴板中的图片粘贴到 Markdown 源码中，快捷键是`Ctrl+Shift+V`
示例：

![2022-10-14-16-37-29](https://img.hackerbs.com//2022-10-14-16-37-29.png)

Paste Image 默认会将图片复制到当前 md 文件所在目录，并根据**年月日时分秒**自动命名，可以在 vscode 设置中找到 paste image 的配置参数进行修改
图片自动复制的目录参数为：

![2022-10-14-16-42-28](https://img.hackerbs.com//2022-10-14-16-42-28.png)

其中`${currentFileDir}`为当前 md 文件所在目录，可以修改为项目根目录`${projectRoot}`，还可在二者后面跟上`\`，如`${currentFileDir}/md_img`，则会将图片复制到当前 md 文件所在目录下的**md_img**目录，若没有**md_img**目录则自动创建

**修改图片保存路径后需要修改`Paste Image: Prefix`，以便路径可以正常匹配**

```shell
Paste Image: Prefix
The string prepend to the resolved image path before paste.
```

如上文示例，将图片目录自定义为当前目录中`/md_img`文件夹，那么此处需修改为：

![20221027132513](https://img.hackerbs.com//20221027132513.png)

当使用`Ctrl+Shift+V`粘贴后，图片路径会自动设定为`md_img/图片名称`

图片自动命名的参数为：

![2022-10-14-16-45-50](https://img.hackerbs.com//2022-10-14-16-45-50.png)

其中`Y-MM-DD-HH-mm-ss`为**年-月-日-时-分-秒**，该参数采用的是**Moment.js**
[Moment.js 语法教程](https://momentjs.com/#/displaying/format)

### Markdown All in One

该插件可以使vscode自动补全markdown语法，如使用`- 无序列表`时，回车将在下一行自动补全`- `
示例如图：

![2022-10-14-16-55-11](https://img.hackerbs.com//2022-10-14-16-55-11.png)

基于vscode的特点，按下`Ctrl+Shift+P`调出控制台后，可以使用**maio**的缩写快速筛出本插件的命令，如图：

![2022-10-14-17-00-34](https://img.hackerbs.com//2022-10-14-17-00-34.png)

[Markdown All in One官网详细介绍](https://markdown-all-in-one.github.io/guide/#features)