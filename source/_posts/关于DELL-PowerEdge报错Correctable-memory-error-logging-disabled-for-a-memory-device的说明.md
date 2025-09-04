---
title: >-
  关于DELL PowerEdge报错Correctable memory error logging disabled for a memory
  device的说明
comments: true
categories: [知识库]
date: 2025-09-04 16:16:54
keywords:
  - DELL
	- ECC
  - IT知识
  - 百科
	- 内存
	- iDRAC
tags:
  - DELL
	- ECC
  - IT知识
  - 百科
	- 内存
	- iDRAC
---

## 关于DELL PowerEdge R740XD iDRAC报内存设备的可纠正内存错误日志记录已禁用问题的说明及解答

> 巡检发现：iDRAC报错Correctable memory error logging disabled for a memory

致电800-858-0613核实BIOS固件版本低于2.10，固件过旧，属于遗留问题

关于该报错的解释为：BIOS关闭了对于ECC纠错记录的保存，仅仅是没有记录进行了ECC纠错，实际上并不影响使用，可能存在的风险为：无法知晓ECC进行了多少次纠错。

**2025-9-4已致电DELL800-858-0613核实，该信息可信**