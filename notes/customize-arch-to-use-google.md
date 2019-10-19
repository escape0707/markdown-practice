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

为了启动`Firefox`、`Chromium`等图形化浏览器，建议安装好[显卡驱动](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)和一套[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。

### 显卡驱动

根据您的设备选择需要安装的显卡驱动，例如:

- 开源NVIDIA显卡驱动[`Nouveau`](https://wiki.archlinux.org/index.php/Nouveau)：

  ```bash
  sudo pacman -S xf86-video-nouveau
  ```

- [Intel显卡驱动](https://wiki.archlinux.org/index.php/Intel_graphics)（也有一说不安装而使用缺省驱动更快）：

  ```bash
  sudo pacman -S xf86-video-intel
  ```

> 如果是在虚拟机中，请安装`xf86-video-fbdev`驱动。并且Hyper-V不支持使用wayland作为显示后端，请直接不要安装plasma-wayland-session，也不要在sddm中选择（不装就自然没的选）。

### 桌面环境

您可以自己选择喜爱的[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。目前俺的综合建议是先当作依赖包安装好所有桌面环境都共用的[Xorg](https://wiki.archlinux.org/index.php/Xorg)显示服务器，和[xinit](https://wiki.archlinux.org/index.php/Xinit)启动工具：

```bash
sudo pacman -S xorg-server xorg-xinit --asdeps
```

之后再通过网络搜索几个能看对了眼的桌面环境，分别安装之后使用对应的方法启动并进行测试，以[Xfce4](https://wiki.archlinux.org/index.php/Xfce)为例：

```bash
sudo pacman -S xfce4 xfce4-goodies
startxfce4
```

用这种方法试几个之后自然就能找到适合自己习惯和电脑配置的桌面环境了。

另外俺建议在测试并确定好了想要使用的桌面环境，准备正式安装前删除并重建自己的账户和专用文件夹，以清除测试时各种软件遗留的配置和文件：

```bash
userdel -r your-username
useradd -m your-username
passwd your-username
```

俺已经体验过Ubuntu、GNOME~~，故此次尝试比较接近Windows桌面体验的~~以及本文第一版中提到的[`KDE`](https://wiki.archlinux.org/index.php/KDE)。经过两个月的体验，俺觉出KDE确实足够酷炫、定制丰富、对Windows难民友好，但对于俺的笔记本来说淡入淡出等特效还是复杂了一些，启动耗时相对长、资源占用也相对多。目前俺换到了Xfce这一更简化的GNOME系桌面环境。并且放弃了[显示管理器](https://wiki.archlinux.org/index.php/Display_manager)。

以下列举俺最近装Xfce和KDE的过程，如果想要安装KDE，[请见后](#kde)。

#### Xfce

需要安装的包：

- 支持回收站机制：`gvfs`
- 支持`xfce4-windowck-plugin`插件：`libwnck`
- 托盘区网络管理：`network-manager-applet`
- Noto汉字系字体：`noto-fonts-cjk`
- 声音服务器：`pulseaudio`
- Xfce基础包：`xfce4`
- Xfce额外组件包：`xfce4-goodies`

```bash
sudo pacman -S gvfs libwnck network-manager-applet noto-fonts-cjk pulseaudio xfce4 xfce4-goodies
```

#### KDE

安装`Noto`字体、KDE、[`wayland`](https://wiki.archlinux.org/index.php/KDE#KDE_applications)后端支持（可选）、以及文件管理、终端、记事本等应用，并且启用[SDDM](https://wiki.archlinux.org/index.php/SDDM)显示管理器：

```bash
sudo pacman -S noto-fonts-cjk plasma-meta plasma-wayland-session dolphin-plugins kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite
sudo systemctl enable sddm
```

全部安装后重启系统、输入密码便可以进入KDE桌面。