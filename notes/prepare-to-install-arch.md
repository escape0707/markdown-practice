---
tags: [Blog/System Configurations]
title: 安装 Arch Linux 前的准备工作
---

## 目录 <!-- omit in toc -->

- [对前言的更新](#对前言的更新)
- [前言](#前言)
- [为何选择 Linux](#为何选择-linux)
- [为何选择 Arch Linux](#为何选择-arch-linux)
- [推荐在虚拟机或者~~二奶机~~淘汰的老机器上初体验](#推荐在虚拟机或者二奶机淘汰的老机器上初体验)
- [下载 \& 校验 Arch Linux ISO 安装镜像](#下载--校验-arch-linux-iso-安装镜像)
- [制作 USB 启动盘](#制作-usb-启动盘)
- [准备完成](#准备完成)

## 对前言的更新

距离俺首次开始写这篇文章已经过去了几年，现在也有了别的设备来使用，对 Linux 的了解也稍微丰富了些。在这过程中俺经常向俺的朋友们推荐一些开源的、甚至是可以自建的软件产品来使用，只是希望大家能够在这个过程中些许强化自己的网络生活的安全和隐私。

最初安利的过程还是比较困难的，不过拜现在更为方便的 `archinstall` 安装脚本、远程桌面软件、和 Hyper-V 虚拟机 USB 直通所赐，俺可以更快捷、远程的指导朋友们在一张闲置的 USB 移动硬盘上安装正式的 ArchLinux 来原生使用。

这种入门方法的好处不少：首先，在 Windows 中直接安装、不需要额外制作 USB 启动盘，全程可以用朋友们熟悉的远程桌面工具帮助他们顺利完成安装。其次，在虚拟机中进行 USB 直通安装，朋友完全不用担心意外抹除电脑上的任何数据或者影响到 Windows 的启动项，就算安装过程的操作或者安装脚本有错误，也可以放心的抹除重来，顶多苦一苦那张移动硬盘。最后，因为 Linux 内核和 `linux-firmware` 两者基本上包括了所有我们需要的驱动，一次制作的系统盘可以随便接到 PC 上用。只要选用这张盘启动，在正常的机器都能用上自己的 Linux 系统。甚至在担心显示性能不够的情况下，也可以默认不启动到图形化界面（`systemctl set-default multi-user.target`），而是在命令行模式下手动启动图形界面（`systemctl isolate graphical.target`）。这种灵活性只有 Linux 才能帮助我们做到。

鉴于现在的 `archinstall` 脚本更加可靠、便于使用，并且现在有了如此方便的安装与体验方式，俺之后在自己博客维护的 Linux 安装文章的内容也会从原本的“忠实记录手动安装过程”转移到“给朋友们安利 Arch Linux 时的安装流程”和“自己实际安装并使用 Arch Linux 一定会采用的安装指南中没有要求的配置”两个方面。

其实，主要还是之前的安装过程过于接近官方安装指南的中文翻译，并不完全服务于俺“让一直觉得尝试 Linux 系统很麻烦的朋友也能轻松开始”的目标，而是着重于讲明白安装过程中接触的概念。现在看来，这部分还是大家齐心协力一起贡献翻译力量给中文版 Wiki 为好。

## 前言

作为计划投放在个人博客上的第一篇文章，俺希望能在其中记录下之前在台式机和笔记本电脑上安装 Arch Linux 的过程。俺~~可怜又可爱~~的小笔记本电脑从高中二年级开始就在稳健地发挥着热量，直到大学毕业准备工作仍未下岗。其硬盘、CPU、内存难以满足当下一套 Windows 10 + Firefox Quantum + Electron Apps (Visual Studio Code / Trello) 的欲求。恰巧最近想要在 Linux 环境下学习一些别的框架，挑挑拣拣便看中了这个配置相对复杂但是内容精简的 Arch Linux 往笔电上试着装了装。

此文不妄为各位读者解惑，仅能够作为下次安装或给同学朋友的参考便甚好。如能对万维网另一端的您有所帮助，不胜荣幸。

> 跳过预备部分的海量废话、直入正题的传送门：从[下载 & 校验 Arch Linux ISO 安装镜像](#下载--校验-arch-linux-iso-安装镜像) 开始

## 为何选择 Linux

- 开源免费，隐私性与安全性的天花板更高
- 部分 Linux [发行版](https://en.wikipedia.org/wiki/Linux_distribution)的资源占用显著小于 Windows 10，俺的小笔记本电脑可以接受。软硬件资源得以更高效地服务于实际应用
- 部分 Linux 发行版相较于 Windows 10 提供更少的功能的同时，提供了更高的定制性。在用户有耐心有能力理解如何精准定制、以及能接触高质足量的指引的前提下，仅对用户造成最小的干扰、引入最少的不确定因素，便能精准调校到趁手好用的程度
- 干净的软件包管理设计在有连通性良好的镜像服务器的前提下极大提升软件下载安装更新的舒适度，爱了爱了
- 大部分的病毒与攻击均针对 Windows 平台，具体细分到某个 Linux 发行版适用的病毒与攻击较少，也许能侥幸提高安全度

对于计算机相关行业的学生、从业人员：

- 可以提供非常标准统一可靠、安全、广为应用、免费的整套工具和环境
- 深入设计理念的软件包管理机制完爆 Windows 下 Chocolatey, Scoop, WinGet 能够模拟出来的体验
- 通过深入了解 Linux 操作系统，拓宽知识面，了解操作系统原理和网络配置，积累 Linux 系统工具使用经验
- 部分框架设计之初便服务于 Linux 下的开发使用，之后才适配到其他平台。往往其在 Linux 下才能发挥最大作用，特别是一些远离民用的领域（如 Docker 等容器、深度学习框架等）

一些缺点：

- 确实需要花一些时间上手，就像你小时候刚摸到电脑一样，问题一开始可能会比较多，出错后可能比较迷茫
- 未必能有人像小时候爸爸手把手教你一样帮你解决问题（但 Linux 社区质量显著高于 Windows 社区！特别是没有[百不知道](https://zhidao.baidu.com/)的内容鱼龙混杂。）
- 很多公司特别是国内公司选择不开发其软件的 Linux 版，例如没有原生 Adobe, Autodesk 套件、QQ、微信。虽大多有解决方案，但毕竟多一层麻烦。不过这些东西我都不用就是了，希望美术软件方面以后也再卷一卷，让美术设计师们也能有自由的操作系统选择
- ~~游戏少、治不了、没救了、我都懒得调 Wine 等适配层，直接双启动吧，或者买主机。~~
- ~~Nvidia 不认真支持其显卡在 Linux 下的使用体验，特别是多显卡交火、双显卡切换等在 Linux 下的发挥可能不如 Windows（但对俺这种看网页、VSCode 打打代码的菜鸡来说感觉不到，除非俺也有四路泰坦。）~~
- 随着各大厂推行云游戏和 Nvidia 深耕深度学习领域，Linux 上的显卡驱动和游戏兼容性已经在逐步好转了。Steam Deck 出来以后，可以遇见 Linux 上的游戏体验将会越来越好。Proton 了解一下。何况你可以双启动嘛，俺现在就是无脑双启动 Windows 10 LTSC。

总体而言，其实仅就资源占用这一点，在俺的小笔记本上，Linux 就已经一刀处决 Windows 10 了。俺便在认为利大于弊的时候，趁着有闲，玩玩 Linux。

## 为何选择 Arch Linux

[Arch Linux 的设计理念](https://wiki.archlinux.org/index.php/Arch_Linux)

- 最小，适合俺的第一原因
- 前沿，滚动更新最新稳定版，防止了俺在 Ubuntu 上遇到过的编译器和依赖太老总瞎编，查一小时才查出来的问题
- 泛用，特别是 AUR 能让人用上更多软件
- 从众，以前看博客都在说 Ubuntu 运行某某命令，现在总看到 Arch 下怎样怎样做。少走弯路、少造轮子
- 官方 Wiki 和社区很强、大神云集
- 包管理工具 pacman 用起来很舒服
- 稍微打开了 Linux 系统黑箱，看到更内部结构的冰山一角
- （年幼无知时折腾 Ubuntu 和显卡驱动 PTSD 了）
- （太菜太爱玩，暂时搞不了 Gentoo 或者 Linux From Scratch）

## 推荐在虚拟机或者~~二奶机~~淘汰的老机器上初体验

作为 Linux 的初次使用者，相信各位读者一定是选择使用图形化操作界面的。同样作为初次使用者，俺建议各位在比较像 Max OS X 的 Gnome 和 比较像 Windows 的 KDE Plasma 之中选择自己初次使用的桌面环境，让它们帮你解决包括网络配置、文件检索、窗口、软件自启动等等问题。而试用这两种界面的去别的最方便的方法就是在虚拟机中运行最新版的 OpenSUSE（另一个发行版，同样滚动更新，有图形化的安装 ISO）的 Gnome 和 KDE 的 LiveCD 启动盘实际感受一下，并选好心仪的桌面环境。

如果机器的性能实在受限严重，无法流畅渲染 Gnome、KDE等桌面环境，则推荐选择其他稍微轻量一些的支持 Wayland 的桌面环境。俺初次最初写这篇文章的时候用的老笔记本，只能支撑 Xfce。但现在 Xfce 仍然无法跟上对 Wayland 和 GTK 4 的支持，所以还是作罢。基本上老机器俺现在全部拿来纯 SSH 连接当作无头服务器、软路由使用了。

俺对 OpenSUSE 还不是很熟悉，在中文 Linux 社区中它的流行度也相对低，对于部分希望入门 Linux 日用的读者来说还是从 Arch 入门感觉会更容易得到技术支持。

之后俺推荐给各位朋友的安装使用过程也是在虚拟机中进行的，过程中可以直接使用物理机的软硬件阅读安装指南、搜索资料，甚至通过远程桌面接受帮助，更为便利。另外，也更加不容易破坏已有的软硬件配置。如遇困难，想中途停止安装，也可以轻松移除。

本文将描述 Arch Linux 安装介质的准备工作，后文将分别记录俺在一台 Windows 10 台式机和一台已运行 Arch Linux 的笔电上，安装最小化、可联网的 Arch Linux，配置可以使用搜索引擎的环境，以及安装自用软件的过程。

## 下载 & 校验 Arch Linux ISO 安装镜像

我们可以按照官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)来下载和校验安装镜像。

### 下载

在[官网下载](https://www.archlinux.org/download/)镜像时，可选择使用[腾讯](https://mirrors.tencent.com/archlinux/iso/latest)、[网易](https://mirrors.163.com/archlinux/iso/latest)等速度较快的镜像服务器。

### 校验

Windows 下可在之后使用 Rufus 工具制作 USB 安装盘时，直接点击校验按钮校验 SHA1。

或直接用 `Get-FileHash` 校验 SHA1：

> 将 `version` 替换为您下载到的版本。

```powershell
Get-FileHash archlinux-version-x86_64.iso -Algorithm SHA1
```

或校验 PGP 签名，在 Windows 下可以使用 [Gpg4win](https://www.gpg4win.org/)：

1. 安装 Gpg4win 前确认 UAC 提示的安装包签名是否正确。
2. 将下载来的 Arch Linux ISO 安装镜像和 PGP 签名文件放在同一文件夹下，之后在此文件夹运行 PowerShell 或 Command Prompt。
3. 运行命令：

   ```powershell
   gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig
   ```

4. 看到 `Good signature from` 等即表明文件的**完整性**符合签名描述：

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

5. 其中的 `WARNING` 表示此密钥[没有被信任或是伪造的](https://gnupg.org/download/integrity_check.html#sec-1-1)。依据[安装指南#校验签名](https://wiki.archlinux.org/index.php/Installation_guide#Verify_signature)，我们需要检查签名者信息以及 `fingerprint` 来确认其是否可信。

6. 根据 Arch Linux 官网的[开发人员信息](https://www.archlinux.org/people/developers/#pierre)我们可以找到并确认这个公钥。

   > 当然我们也需要确认页面的可信任性等等，这里不再[详述](https://pierre-schmitz.com/trust-the-master-keys/)。

或者在 Arch Linux 下，用 `pacman-key` 校验签名：

```bash
pacman-key -v archlinux-version-x86_64.iso.sig
```

## 制作 USB 启动盘

> 虚拟机可直接从 ISO 镜像启动，无需制作成 USB 启动盘。请跳过这一步。

请参考俺的另一短篇[使用 Ventoy 制作 USB 启动盘](use-ventoy-to-create-bootable-usb.md)

## 准备完成

至此，准备工作已经完成。俺之后将会继续描述[在 Hyper-V 虚拟机上](create-vm-for-arch.md)和[在笔电上](install-arch-on-laptop-and-vm.md)安装 Arch Linux 的过程，详情请移步后文。
