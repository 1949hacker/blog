---
title: 关于Proxmox-VE系统创建UEFI虚拟机需要单独添加EFI磁盘的问题
categories: [知识库,技术笔记,软件]
tags:
  - Proxmox-VE
  - 虚拟机
date: 2023-01-03 12:00:25
---

>在传统物理机上，如需使用UEFI引导，仅仅需要在硬盘上创建EFI分区即可，无论是Windows还是Linux均是如此，但在使用Proxmox-VE创建虚拟机时，却需要额外添加EFI磁盘，对此提出疑问并进行探讨。

<!-- more -->

直接在网络上搜寻并无相关信息，查询[官方wiki](https://pve.proxmox.com/wiki/Qemu/KVM_Virtual_Machines)得到如下解释：

BIOS 和 UEFI

为了正确模拟计算机，QEMU 需要使用固件。在通常称为 BIOS 或 (U)EFI 的普通 PC 上，它作为启动 VM 时的第一步执行。它负责进行基本的硬件初始化，并为操作系统提供固件和硬件接口。默认情况下，QEMU 为此使用SeaBIOS，这是一个开源的 x86 BIOS 实现。SeaBIOS 是大多数标准设置的不错选择。

某些操作系统（例如 Windows 11）可能需要使用 UEFI 兼容实现。在这种情况下，您必须使用OVMF，它是一种开源 UEFI 实现。

在其他情况下，SeaBIOS 可能不是理想的启动固件，例如，如果您想进行 VGA 直通。

如果要使用 OVMF，需要考虑以下几点：

为了保存引导顺序之类的东西，需要有一个 EFI 磁盘。该磁盘将包含在备份和快照中，并且只能有一个。

您可以使用以下命令创建这样的磁盘：

```shell
qm set <vmid> -efidisk0 <storage>:1,format=<format>,efitype=4m,pre-enrolled-keys=1
```

其中**storage**是您希望拥有磁盘的存储，而**format**是存储支持的格式。或者，您可以通过VM 硬件部分中的添加 EFI 磁盘的 Web 界面创建这样的磁盘。

efitype选项指定应使用哪个版本的 OVMF 固件。对于新的 VM，这应该始终是4m，因为它支持安全启动并且分配了更多空间来支持未来的开发（这是 GUI 中的默认值）。

pre-enroll-keys指定 efidisk 是否应预加载特定于发行版和 Microsoft 标准安全启动密钥。它还默认启用安全启动（尽管它仍然可以在 VM 的 OVMF 菜单中禁用）。

笔记	如果要在现有 VM（仍使用2m efidisk）中开始使用安全启动，则需要重新创建 efidisk。为此，删除旧的 ( `qm set <vmid> -delete efidisk0` ) 并添加一个新的，如上所述。这将重置您在 OVMF 菜单中所做的任何自定义配置！
当使用带有虚拟显示器的 OVMF（没有 VGA 直通）时，您需要在 OVMF 菜单中设置客户端分辨率（您可以在启动期间按 ESC 按钮到达），或者您必须选择 SPICE 作为显示类型.

---

为了保存引导顺序？或许是类似于物理机上，启动菜单中除了UEFI的硬盘启动项外，还有一个额外的操作系统名称的启动项？如`Windows Boot Manager`或是`Debian`

EFI磁盘 = 操作系统引导项？
