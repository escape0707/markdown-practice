---
tags: [Blog/System Configurations]
title: Arch新机用上Google
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [创建账户与配置sudo](#创建账户与配置sudo)
- [显卡驱动与图形化界面](#显卡驱动与图形化界面)

## 前言

在前文[在虚拟机/物理机中安装Arch Linux](install-arch-on-laptop-and-vm.md)中，俺已经记录了安装最小化、可联网的Arch Linux的过程。本文将继续描述俺在前述系统中折腾到可以在桌面环境下使用搜索引擎的程度所需的步骤。

## 创建账户与配置sudo

在完成了最小化Arch Linux安装之后，为了安全使用系统，还需要创建属于自己的个人账户、安装并配置好sudo。

### 新建用户

新安装的系统只有`root`一个超级用户。一直使用`root`账户并不安全，可能会意外地执行或修改了文件，以及给予第三方程序过高权限带来风险。平时应当使用普通权限的账户。否则无法使用`makepkg`、`sddm`等工具。

[新建一个用户](https://wiki.archlinux.org/index.php/Users_and_groups#Example_adding_a_user)：

> 用您喜欢的用户名替换`your-username`。

```bash
useradd -m your-username
passwd your-username
```

### 提升权限

普通账户可以用[`su`](https://wiki.archlinux.org/index.php/Su)或[`sudo`](https://wiki.archlinux.org/index.php/Sudo)来提权（`root`账户也可以用`su`来[进入普通账户的环境](https://wiki.archlinux.org/index.php/Su#Tips_and_tricks)修改配置）。使用`su`在shell中提升到`root`账户的权限进而为所欲为，而使用`sudo`则可以作为`root`账户执行一条命令。

安装`sudo`：

```bash
pacman -S sudo
```

由于俺只是单人、单账户、客户端自闭性使用机器，仅用`sudo`防手滑和其他进程滥用权限。故在此不做[进阶的设置](https://wiki.archlinux.org/index.php/Sudo#Example_entries)，仅将之前新建的用户添加到`sudoers`中去：

```bash
visudo
```

通过启动的Vi编辑器，在`root`的权限之后添加自用账户的权限，如下：

```bash
##
## User privilege specification
##
root ALL=(ALL) ALL
your-username ALL=(ALL) ALL
```

注销并登录到新账户：

```bash
logout
```

## 显卡驱动与图形化界面

### 显卡驱动

根据您的设备选择需要安装的[显卡驱动](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)，例如:

- 开源NVIDIA显卡驱动[`Nouveau`](https://wiki.archlinux.org/index.php/Nouveau)：

  ```bash
  sudo pacman -S xf86-video-nouveau
  ```

- [Intel显卡驱动](https://wiki.archlinux.org/index.php/Intel_graphics)（也有一说不安装而使用缺省驱动更快）：

  ```bash
  sudo pacman -S xf86-video-intel
  ```

> 如果是在虚拟机中，请安装`xf86-video-fbdev`驱动。并且Hyper-V不支持使用wayland作为显示后端，请直接不要安装plasma-wayland-session，也不要在sddm中选择（不装就自然没的选）。

### 图形化界面

您可以自己选择喜爱的[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。俺已经体验过Ubuntu和GNOME，故此次尝试比较接近Windows桌面体验的[`KDE`](https://wiki.archlinux.org/index.php/KDE)桌面环境。

安装`Noto`字体、KDE、[`wayland`](https://wiki.archlinux.org/index.php/KDE#KDE_applications)后端支持、以及包括文件管理、终端、记事本等应用，并且启用[SDDM](https://wiki.archlinux.org/index.php/SDDM)显示管理器：

```bash
sudo pacman -S noto-fonts-cjk plasma-meta plasma-wayland-session dolphin-plugins kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite
sudo systemctl enable sddm
```

之前提到，KDE环境整合了`NetworkManager`提供图形化Wi-Fi连接配置界面。但是由于这类网络管理软件（包括前文用到的`netctl`）同时运行会导致错误，此软件不会自动启动，需要我们手动启动：

```bash
systemctl enable NetworkManager
```

全部安装后重启系统、输入密码便可以进入KDE桌面。
