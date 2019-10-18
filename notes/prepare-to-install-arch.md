---
tags: [Blog/System Configurations]
title: 安装Arch Linux前的准备工作
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [为何选择Linux](#为何选择linux)
- [为何选择Arch](#为何选择arch)
- [推荐在虚拟机上初体验](#推荐在虚拟机上初体验)
- [下载 & 校验Arch Linux ISO安装镜像](#下载--校验arch-linux-iso安装镜像)
- [创建USB启动盘](#创建usb启动盘)
- [准备完成](#准备完成)

## 前言

作为计划投放在个人博客上的第一篇文章，俺希望能在其中记录下之前在台式机和笔记本电脑上安装Arch Linux的过程。俺~~可怜又可爱~~的小笔记本电脑从高中二年级开始就在稳健地发挥着热量，直到大学毕业准备工作仍未下岗。其硬盘、CPU、内存难以满足当下一套Windows 10 + Firefox Quantum + Electron Apps(Visual Studio Code / Trello)的欲求。恰巧最近想要在Linux环境下学习一些别的框架，挑挑拣拣便看中了这个配置相对复杂但是内容精简的Arch Linux往笔电上试着装了装。

此文不妄为各位读者解惑，仅能够作为下次安装或给同学朋友的参考便甚好。如能对万维网另一端的您有所帮助，不胜荣幸。

> 跳过预备部分的海量废话、直入正题的传送门：从[下载 & 校验Arch Linux ISO安装镜像](#下载--校验arch-linux-iso安装镜像)开始

## 为何选择Linux

- 开源免费
- 部分Linux[发行版](https://en.wikipedia.org/wiki/Linux_distribution)的资源占用显著小于Windows 10，俺的小笔记本电脑可以接受。软硬件资源得以更高效地服务于实际应用
- 部分Linux发行版相较于Windows 10提供更少的功能的同时，提供了更高的定制性。在用户有耐心有能力理解如何精准定制、以及能接触高质足量的指引的前提下，仅对用户造成最小的干扰、引入最少的不确定因素，便能精准调校到趁手好用的程度
- 干净的软件包管理设计在有连通性良好的镜像服务器的前提下极大提升软件下载安装更新的舒适度，爱了爱了
- 大部分的病毒与攻击均针对Windows平台，具体细分到某个Linux发行版适用的病毒与攻击较少，也许能侥幸提高安全度

对于计算机相关行业的学生、从业人员：

- 可以提供非常标准统一可靠、安全、广为应用、免费的整套工具和环境
- 深入设计理念的软件包管理机制完爆Windows下[Chocolatey](https://chocolatey.org/)能够模拟出来的体验
- 通过深入了解Linux操作系统，拓宽知识面，了解操作系统原理，尤其是能为熟悉嵌入式环境和远程操作云服务器（常搭载*nix系统）提供经验
- 部分框架最初便在Linux下开发使用，之后才适配到其他平台。往往其在Linux下才能发挥最大作用，特别是一些远离民用的领域（如Docker等容器、深度学习框架等）

一些缺点：

- 确实需要花一些时间上手，就像你小时候刚摸到电脑一样，问题一开始可能会比较多，出错后可能比较迷茫
- 未必能有人像小时候爸爸手把手教你一样帮你解决问题（但Linux社区质量显著高于Windows社区！特别是没有[百不知道](https://zhidao.baidu.com/)的内容鱼龙混杂。）
- 很多公司特别是国内公司选择不开发其软件的Linux版，例如没有原生Adobe套件（这没救，有钱请上Mac吧屏幕还好）、QQ、微信。虽大多有解决方案，但毕竟多一层麻烦。（你可以用Telegram和Slack啊拜托~）
- 游戏少、治不了、没救了、买主机吧。（但有Dota2！有LoL外服！而且你为什么不买主机？？是我最后生还者不好玩了还是我光环致远星镇不住河山了？）
- Nvidia不认真支持其显卡在Linux下的使用体验，特别是多显卡交火、双显卡切换等在Linux下的发挥可能不如Windows（但对俺这种看网页、VSCode打打代码的菜鸡来说感觉不到，除非俺也有四路泰坦。）

总体而言，其实仅就资源占用这一点，在俺的小笔记本上，Linux就已经一刀处决Windows 10了。俺便在认为利大于弊的时候，趁着有闲，玩玩Linux。

## 为何选择Arch

[Arch Linux的设计理念](https://wiki.archlinux.org/index.php/Arch_Linux)

- 最小，适合俺的第一原因
- 前沿，滚动更新最新稳定版，防止了俺在Ubuntu上遇到过的编译器和依赖太老总瞎编，查一小时才查出来的问题
- 泛用，特别是AUR能让人用上更多软件
- 从众，以前看博客都在说Ubuntu运行某某命令，现在总看到Arch下怎样怎样做。少走弯路、少造轮子
- 官方Wiki和社区很强、大神云集
- pacman有点酷
- 稍微打开了Linux系统黑箱，看到更内部结构的冰山一角
- （年幼无知时折腾Ubuntu和显卡驱动PTSD了）

## 推荐在虚拟机上初体验

在尝试于一台物理机上安装Arch Linux之前，不妨先在虚拟机中体验一下安装Arch Linux的过程。况且，在虚拟机中安装，可以直接使用物理机的软硬件阅读安装指南、搜索资料，更为便利。另外，如遇困难，想中途停止安装，也可以轻松回滚。

本文将描述Arch Linux安装介质的准备工作，后文将分别记录俺在一台Windows 10台式机和一台已运行Arch Linux的笔电上，安装最小化、有网络连接的Arch Linux，配置可以使用搜索引擎的环境，以及安装自用软件的过程。

> 俺是在下述环境进行安装的：
>
> 操作系统：Windows 10 Pro 1903 18362.239
>
> 虚拟技术：Hyper-V
>
> 并不清楚Hyper-V与Oracle VM VirtualBox上仅仅是进行Arch Linux安装会有什么体验上的不同。之前已经试过VirtualBox，此次为了少安装一个第三方软件尝试一下微软的Hyper-V。

## 下载 & 校验Arch Linux ISO安装镜像

我们可以按照官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)来下载和校验安装镜像。

### 下载

在[官网下载](https://www.archlinux.org/download/)镜像时，可选择使用[网易](http://mirrors.163.com/archlinux/iso/2019.07.01/)等速度较快的镜像服务器。

### 校验

Windows下可在之后使用Rufus工具制作USB安装盘时，直接点击校验按钮校验SHA1。

或直接用`certutil`校验SHA1：

> 将`version`替换为您下载到的版本。

```powershell
certutil -hashfile archlinux-version-x86_64.iso.sig sha1
```

或校验PGP签名，在Windows下可以使用[Gpg4win](https://www.gpg4win.org/)：

1. 安装Gpg4win前确认UAC提示的安装包签名是否正确。
2. 将下载来的Arch Linux ISO安装镜像和PGP签名文件放在同一文件夹下，之后在此文件夹运行PowerShell或Command Prompt。
3. 运行命令：

   ```powershell
   gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
   ```

4. 看到`Good signature from`等即表明文件的**完整性**符合签名描述：

   ```bash
   gpg: assuming signed data in '.\archlinux-2019.07.01-x86_64.iso'
   gpg: Signature made 07/01/19 23:07:38 China Standard Time
   gpg:                using RSA key 4AA4767BBC9C4B1D18AE28B77F2D434B9741E8AC
   gpg: C:/Users/tothe/AppData/Roaming/gnupg/trustdb.gpg: trustdb created
   gpg: key 7F2D434B9741E8AC: public key "Pierre Schmitz <pierre@archlinux.de>" imported
   gpg: Total number processed: 1
   gpg:               imported: 1
   gpg: Good signature from "Pierre Schmitz <pierre@archlinux.de>" [unknown]
   gpg: WARNING: This key is not certified with a trusted signature!
   gpg:          There is no indication that the signature belongs to the owner.
   Primary key fingerprint: 4AA4 767B BC9C 4B1D 18AE  28B7 7F2D 434B 9741 E8AC
   ```

5. 其中的`WARNING`表示此密钥[没有被信任或是伪造的](https://gnupg.org/download/integrity_check.html#sec-1-1)。依据[安装指南#校验签名](https://wiki.archlinux.org/index.php/Installation_guide#Verify_signature)，我们需要检查签名者信息以及`fingerprint`来确认其是否可信。

6. 根据Arch Linux官网的[开发人员信息](https://www.archlinux.org/people/developers/#pierre)我们可以找到并确认这个公钥。

   > 当然我们也需要确认页面的可信任性等等，这里不再[详述](https://pierre-schmitz.com/trust-the-master-keys/)。

或者在Arch Linux下，用`pacman-key`校验签名：

```bash
pacman-key -v archlinux-version-x86_64.iso.sig
```

## 创建USB启动盘

> 虚拟机可直接从ISO镜像启动，无需制作成USB启动盘。请跳过这一步。

Windows下可以用[Rufus工具创建USB启动盘](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_Windows)：

- SELECT您的ArchISO
- Partition Scheme选择GPT
- File System选择FAT32
- START，选Write in DD Image mode

Linux下可以用[dd工具创建USB启动盘](https://wiki.archlinux.org/index.php/USB_flash_installation_media#In_GNU/Linux)：

- 用`lsblk`命令找到USB盘的名称，例如`sdb`
- 将如下命令中的`/dev/sdx`替换为您的USB盘的名称，例如`/dev/sdb`。注意不要后缀分区数，即不要用`/dev/sdb1`

  ```bash
  sudo dd bs=4M if=path/to/archlinux.iso of=/dev/sdx status=progress oflag=sync
  ```

之后请参照您的主板设置调整为USB启动。

## 准备完成

至此，准备工作已经完成。俺之后将会继续描述[在Hyper-V虚拟机上](create-vm-for-arch.md)和[在笔电上](install-arch-on-laptop-and-vm.md)安装Arch Linux的过程，详情请移步后文。
