---
tags: [Blog/System Configurations]
title: 为 Arch Linux 创建 Hyper-V 虚拟机
---

## 目录 <!-- omit in toc -->

- [前言](#前言)
- [启用 Hyper-V & 创建虚拟机](#启用-hyper-v--创建虚拟机)
- [配置虚拟机](#配置虚拟机)
- [连接并启动虚拟机](#连接并启动虚拟机)
- [正式安装](#正式安装)

## 前言

本文承接前文 [安装 Arch Linux 前的准备工作](prepare-to-install-arch.md)，介绍利用 Windows 10 整合的 Hyper-V 技术，创建一台虚拟机并启动到 Arch Linux 安装介质以便安装的步骤。

## 启用 Hyper-V & 创建虚拟机

可以参照官方[虚拟机安装指南](https://wiki.archlinux.org/index.php/Hyper-V)。这里为了方便读者，简单翻译一下。

### 启用 Hyper-V

[用管理员权限执行 PowerShell 命令](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v#enable-hyper-v-using-powershell)：

```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

或者，在开始菜单搜索并选择“Turn Windows features on or off”，找到“Hyper-V”并勾选，点击“OK”按钮。（勾选后为对勾抑或方块并不影响）

![启用 Hyper-V](../attachments/enable-hyper-v.png)

启用之后往往需要重启。

### 配置虚拟机网络（可选）

Arch Linux 的安装和使用均需要连接到网络，因此要给虚拟机分配一个 Virtual Switch（虚拟交换器）。这里有使用外部交换器和内部交换器两种选择。

因无特殊需求，用随 Windows 10 Fall Creators Update 更新而来的内建 NAT internal switch——“Default Switch”即可。

### 创建虚拟机

1. 在开始菜单搜索并运行“Hyper-V Manager”。选择“New”，“Virtual Machine”，进入新建虚拟机向导。

   ![新建虚拟机](../attachments/new-virtual-machine.png)

2. 在“Specify Generation”时，选择“Generation 2”来使用 UEFI 环境。

   ![选择 Generation 2](../attachments/specify-generation-2.png)

3. 在“Assign Memory”时，选择恰当大小的内存，俺使用了默认值。

4. 在“Configure Networking”时，“Connection”选择“Default Switch”

   ![图片](../attachments/choose-default-switch.png)

5. 在“Connect Virtual Hard Disk”时，选择“Create a virtual hard disk”

   ![创建 VHD](../attachments/create-a-virtual-hard-disk.png)

6. 在“Installation Options”时，选择“Install an operating system from a bootable CD/DVD-ROM”，并选择之前下载的 Arch Linux ISO 镜像

   ![选择安装镜像](../attachments/choose-installation-image-file.png)

## 配置虚拟机

选择我们刚刚创建的虚拟机，打开其设置

![打开虚拟机设置](../attachments/open-virtual-machine-settings.png)

- 关闭 Secure Boot：

  ![关闭 Secure Boot](../attachments/virtual-machine-disable-secure-boot.png)

## 连接并启动虚拟机

1. 连接并启动：双击创建的虚拟机，选择 Start。

2. 自此可以开始在 Arch Linux Live Environment 中进行安装步骤，建议先测试网络可用性：

   ```bash
   ping baidu.com
   ```

## 正式安装

[在虚拟机 / 物理机中安装 Arch Linux](install-arch-on-laptop-and-vm.md) 的步骤请移步后文。
