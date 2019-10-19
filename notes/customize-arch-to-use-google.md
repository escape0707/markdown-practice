---
tags: [Blog/System Configurations]
title: Arch新机用上Google
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [创建账户与配置sudo](#创建账户与配置sudo)

## 前言

在前文[在虚拟机/物理机中安装Arch Linux](install-arch-on-laptop-and-vm.md)中，俺已经记录了安装最小化、可联网的Arch Linux的过程。本文将继续描述俺在前述系统中折腾到可以在桌面环境下使用搜索引擎的程度所需的步骤。

## 创建账户与配置sudo

在完成了最小化Arch Linux安装之后，为了安全使用系统，还需要创建属于自己的个人账户，安装并配置好sudo。

### 用户和用户组

新安装的系统只有`root`一个超级用户。一直使用`root`账户并不安全，可能会意外地执行或修改了文件，以及给予第三方程序过高权限带来风险。故平时应当使用普通权限的账户。（KDE桌面环境也需要您创建非`root`账户才能登录。）

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

由于俺只是单人、单账户并作为客户端使用机器，仅用`sudo`防手滑和防其他进程滥用权限。故在此不做[进阶的设置](https://wiki.archlinux.org/index.php/Sudo#Example_entries)，仅将之前新建的用户添加到`sudoers`中去：

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
