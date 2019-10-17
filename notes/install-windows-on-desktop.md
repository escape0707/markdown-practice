---
tags: [Blog/System Configurations]
title: 在台式机上安装Windows 10
---

- [前言](#前言)
- [下载最新Windows安装镜像](#下载最新windows安装镜像)
- [使用Rufus制作USB启动盘](#使用rufus制作usb启动盘)
- [UEFI下Secure Boot USB启动盘](#uefi下secure-boot-usb启动盘)
- [Fresh Start](#fresh-start)
- [修改计算机名](#修改计算机名)
- [移动各种User文件夹](#移动各种user文件夹)
- [安装Chocolatey软件管理器和其他软件](#安装chocolatey软件管理器和其他软件)
- [检查Windows更新](#检查windows更新)
- [启用Hyper-V并导入虚拟机](#启用hyper-v并导入虚拟机)
- [禁用多余启动项和功能](#禁用多余启动项和功能)

## 前言

俺自用的Windows 10重装备忘录。

## 下载最新Windows安装镜像

下载[Media Creation Tool](https://www.microsoft.com/software-download/windows10)，运行向导中选择下载ISO。

## 使用Rufus制作USB启动盘

### 下载Rufus

从[官网](https://rufus.ie/)手动下载，或用Chocolatey下载：

```powershell
choco install rufus
```

### 制作启动盘

- 选择要使用的USB盘
- 选择下载的Windows ISO镜像
- 分区类型选择`GPT`、目标系统类型选择`UEFI`
- 文件系统选择FAT32
- `START`开始制作

## UEFI下Secure Boot USB启动盘

俺使用的MSI B150 Krait Gaming，需要开启UEFI、Secure Boot，以及legacy USB support，之后UEFI模式的USB启动才会出现。

## Fresh Start

初次安装Windows 10后再次进行Fresh Start可以彻底排除一堆微软自带的垃圾软件/游戏。手动删除也行，可选步骤。

## 修改计算机名

```text
JayChu-Desktop
```

## 移动各种User文件夹

```text
Desktop
Documents
Downloads
Music
Pictures
Videos
```

## 安装Chocolatey软件管理器和其他软件

管理员模式下，在`cmd.exe`中运行命令[安装Chocolatey](https://chocolatey.org/install)：

```powershell
@"%SystemRoot%\System32\WindowsPowerShell\v1.0\powershell.exe" -NoProfile -InputFormat None -ExecutionPolicy Bypass -Command "iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))" && SET "PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin"
```

并安装其他软件：

```powershell
choco install -y 7zip aria2 authy-desktop autohotkey emule Firefox geforce-experience geforce-game-ready-driver git logitech-options logitechgaming steam teamviewer tim toggl vlc vscode wechat
```

以及自行安装一些软件：

```text
GoldenDict
Microsoft Edge Canary
Microsoft Office Professional Plus 2019
PanDownload
QQ音乐
ShadowFox
shadowsocks（之前choco安装的在运行时会受到权限限制，将可执行程序文件拷贝出来自己建立文件夹使用）
```

还要去Microsoft Store中安装一些软件：

```text
Bitwarden
iTunes
Trello
哔哩哔哩动画
哔哩哔哩动画UWP
```

当使用Vim扩展插件时，为了不用经常去够`Esc`和`Ctrl`，Windows下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)或者[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/Use%20CapsLock%20in%20Vim.ahk)，Linux下可以使用[`caps2esc`](https://aur.archlinux.org/packages/interception-caps2esc)。

最后配置GoldenDict词典，以及下载[GoldenDict划译脚本](https://github.com/escape0707/scripts)。

## 检查Windows更新

## 启用Hyper-V并导入虚拟机

## 禁用多余启动项和功能

### 启动项

```text
iTunes Helper
LogiOptions
Logitech Gaming Framework
Microsoft Edge Update
Microsoft Onedrive
Windows Security notification icon
```

### 组策略

`Win+r`：`gpedit.msc`

![组策略中禁用Cortana和OneDrive](../attachments/gpedit-disable-cortana-onedrive.png)

### 注册表

将下列内容放入`.reg`文件中并运行，移除This PC中的3D Objects、设置启动时打开Num Lock：

```text
Windows Registry Editor Version 5.00
[-HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]
[-HKEY_LOCAL_MACHINE\SOFTWARE\WOW6432Node\Microsoft\Windows\CurrentVersion\Explorer\MyComputer\NameSpace\{0DB7E03F-FC29-4DC6-9020-FF41B59E513A}]

[HKEY_USERS\.DEFAULT\Control Panel\Keyboard]
"InitialKeyboardIndicators"="2"
```

### 关闭系统托盘区图标

- 麦克风
- 地理位置
