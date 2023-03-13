---
tags: [Blog/System Configurations]
title: 在虚拟机 / 物理机中安装 Arch Linux
---

## 目录 <!-- omit in toc -->

-   [前言](#前言)
-   [预备理论](#预备理论)
-   [准备安装](#准备安装)
-   [安装 Arch Linux](#安装-arch-linux)
-   [配置系统](#配置系统)
-   [重启](#重启)
-   [安装完成后的工作](#安装完成后的工作)

## 前言

本文将承接前文[安装 Arch Linux 前的准备工作](prepare-to-install-arch.md)和[为 Arch Linux 创建 Hyper-V 虚拟机](create-vm-for-arch.md)，按步骤介绍在虚拟机以及物理机中安装 Arch Linux 的步骤。个别步骤虚拟机同物理机的操作不一致，后文中会注明，请注意区分。

以下参考官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)。

## 预备理论

### 新：安利专用、自动安装器——archinstall

> 此小节编写于 2021 年 4 月，此工具的使用流程目前已经发生了变化，

从 2021 年~~ 愚人节~~ 4 月 1 日起发布的 ArchISO，官方重新加入了弃更已久的安装脚本，堪称爷青回 / 文艺复兴。用这个脚本，你可以轻松的在同学的电脑的虚拟机上 15 分钟安装 Arch 并进入桌面环境。不过截止到此时脚本已经发生了多次更新，笔者没有再使用，建议各位在了解这个工具存在的情况下自行选择手动还是脚本安装。想要使用脚本安装，必须先理解如何手动安装。否则出现任何问题，都会摸不着北。故下文先正常走一遍手动安装的流程。

### 与 Windows 共存双启动

如果担心自己无法完全脱离 Windows，在某些时间点可能还要用到 Windows 专有的工具或者游戏，那么不妨直接做成[双启动](https://wiki.archlinux.org/title/Dual_boot_with_Windows)，不受虚拟 Windows 的局限性。

接下来说的如果一开始看不明白没关系，可以先跳过这一小节，在虚拟机中按照后文安装一遍，了解了概念以后回来再看这里。之所以在开头提是希望避免读者实际安装的时候才发现需要双启动却没有做准备工作而浪费时间。

如果要安排上双启动，最理想（简单）的顺序是：

1. 在目标安装盘上创建并格式化一个至少 500 MB 的 [ESP](https://wiki.archlinux.org/title/EFI_system_partition)
2. 安装 Windows 的时候创建一个大小符合 Windows 需要的分区并安装在它里面。此时 Windows 会自动利用磁盘中已有的 ESP。
3. 在剩余的空间和已有的 ESP 上安装 Linux。
4. 重启，享用双启动的系统。

如果先安装了 Windows 并不方便格式化重装，ESP 的大小会在 100 MB 左右，虽然我目前双启动只用了 75.3 MB，但如果换用其他内核也[有可能不够存放内核和初始内存文件系统](https://wiki.archlinux.org/title/Dual_boot_with_Windows#The_EFI_system_partition_created_by_Windows_Setup_is_too_small)。此时，可以[使用 systemd-boot 的 XBOOTLDR 启动模式](https://wiki.archlinux.org/title/Systemd-boot#Installation_using_XBOOTLDR)。

### 打开 UEFI & 关闭 Secure Boot

启动机器前，请在机器固件设置（虚拟机配置或物理机主板配置）中打开 UEFI 模式。并且，俺建议和虚拟机安装过程一样关闭 [Secure Boot](https://www.rodsbooks.com/efi-bootloaders/secureboot.html)。支持这一建议的[原因](https://www.reddit.com/r/archlinux/comments/8nbau0/secure_boot_yay_or_nay/)为——Secure Boot 保护我们不受 [Evil Maid Attack](https://en.wikipedia.org/wiki/Evil_maid_attack) 之类物理攻击，但俺自用笔记本的环境几乎不存在收到这类攻击的可能，并且俺也不涉及绝密级别的工作，也没有做全盘加密，这种要看半个小时才弄得好的对我没有什么价值的安全措施暂时就不考虑了。如果有需要可以参看官方[Secure Boot 指南](https://wiki.archlinux.org/index.php/Secure_Boot)。

### 在物理机上查看文档

之前提到过尝鲜建议先用虚拟机安装，可以在原系统下方便查阅资料。

不过即便是在物理机上启动了 Arch 安装介质，也有办法查阅资料。在[连接到互联网](#连接到互联网)之后，可以执行`Installation_guide`来快速使用文本型网页浏览器`lynx`查看官方指南。

如果希望一边看网页 / 文档，一边进行安装，可以用`ctrl+alt+F1-6`或`alt+ 左 / 右键`切换终端。

## 准备安装

### 选择键盘布局

俺使用默认的 US 布局，故不做更改。

### 连接到互联网

虚拟机之前已经配置了虚拟网络适配器，故跳过。

物理机安装时如使用有线连接，无需额外操作；而如需使用 Wi-Fi 连接，则首先[检查无线功能是否被关掉](https://wiki.archlinux.org/title/Network_configuration/Wireless#Rfkill_caveat)：

```bash
rfkill list
```

如果是硬件关掉了，用机器上的开关开开；如果是被内核软件关掉了，可以执行命令打开：

```bash
rfkill unblock wifi
```

之后[使用 iwd 连接无线网络](https://wiki.archlinux.org/title/Iwd#iwctl)，按 `tab` 键自动补全（一般情况下只需要第一行和最后一行）：

```bash
iwctl
device list
station your-wireless-device scan
station your-wireless-device get-networks
station your-wireless-device connect your-wifi-SSID
```

一般来说，`archboot` 安装环境中已经包含了各种网卡的驱动，其中就有俺所需的高通专有驱动 `broadcom-wl`。但如果工具仍然报错，请根据 [无线网络配置](https://wiki.archlinux.org/index.php/Wireless_network_configuration) 检查常见问题。

如此便可连接上 Wi-Fi 网络。

### SSH 连接及远程安装

目标机器连接到网络后，如果可以使用另一台电脑连接到此台电脑的网络，则可以配置并使用 SSH 远程连接并进行安装操作。只需要给`root`用户设置密码：

```shell
passwd
```

之后在客户端机器上执行 `ssh root@目标机器的地址` 即可。这种方法可以充分利用客户端机器上更好的性能或者熟悉的操作系统、操作方式。还可以轻松将配置文件用 sftp 的方式拷贝到目标机器上，快速完成配置。

### 验证启动模式

运行如下命令，文件夹存在并有内容即正常。如不存在，则系统是在 BIOS 环境中启动的，请再次检查创建虚拟机时是否选择的 Generation 2。如是在物理机上安装，检查主板设置是否开启了 UEFI 模式。如果列出文件夹不存在，则启动的是 BIOS 模式。

```bash
ls /sys/firmware/efi/efivars
```

### 更新系统时钟

```bash
timedatectl set-ntp true
```

检查时钟状态：

```bash
timedatectl status
```

### 磁盘分区

列出所有硬盘：

```bash
fdisk -l
```

寻找类似`/dev/sda`或者`/dev/nvme0n1`的条目，并根据磁盘的大小等信息，确定您想要安装的磁盘。

Linux 系统需要至少一个 root 目录（`/`）分区。如果是 UEFI 启动，还需要一个 [EFI 系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition)；如果是 BIOS 启动，并且想俺一样想用 GPT 分区表和 GRUB 则需要一个 [BIOS 启动分区](<https://wiki.archlinux.org/title/GRUB#GUID_Partition_Table_(GPT)_specific_instructions>)。[Swap](https://wiki.archlinux.org/index.php/swap) 可以通过分区实现也可以通过动态 Swap 文件实现，不过我现在有 16 GB 内存所以不上 Swap 裸奔。注意，不用 Swap 不能休眠计算机，据说有一些日志性质的缓存也在 Swap 中，不过没有照样用的可以。

在 UEFI 启动下安装，俺参考了[官方分区模板](https://wiki.archlinux.org/title/Partitioning#Example_layouts)，选择将磁盘分为 512MB 的 EFI 系统分区和利用全部剩余空间的 Linux x86-64 根分区。~~和一个 8GB 的 Swap 分区。Swap 分区的大小可以参考 It's FOSS 上的 [一篇文章](https://itsfoss.com/swap-size/)。一般来说设为物理内存的两倍肯定够用了。~~

在只能 BIOS 启动的一台老机器上，俺同样按照[模板](https://wiki.archlinux.org/title/Partitioning#BIOS/GPT_layout_example)创建了一个 1MB 的 BIOS 启动分区、内存的两倍大小的一个 8GB 的 Swap 分区和利用全部剩余空间的 Linux x86-64 根分区。

分区时为了方便可以用 [`fdisk`](https://wiki.archlinux.org/index.php/Fdisk) 工具的图形版 `cfdisk`：

```bash
cfdisk
```

以下以常见的 UEFI / GPT 为例：

1. Select label type 选择`gpt`（如果未提示，检查屏幕上方第三行是否为`Lable: gpt`。如果不是则用 `fdisk` 新建分区表。）
2. New 一个`512M`的分区，Type 更改为`EFI System partition`
3. New 一个大小为剩余空间~~ -`8GB`（即`cfdisk`自动填写的大小减去`8GB`）~~的分区，并确保 Type 为`Linux Root partition (x86-64)`
4. ~~New 一个`8GB`的分区，Type 更改为`Linux swap`~~
5. 选择 Write 将分区更改写到磁盘
6. Quit

### 格式化分区

刚创建好的分区是没有文件系统的，我们要自行创建文件系统。

> 可以用`fdisk -l`来查看分区大小，用`lsblk -f`来查看分区格式

将 EFI 系统分区格式化为 FAT32 格式，将 Linux 文件系统分区格式化为 ext4 格式：

> 如果您的分区名和下述不一致，请自行替换

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

如果之前创建了 Swap 分区，应将其启用：

```bash
mkswap /dev/sda3
swapon /dev/sda3
```

### 挂载分区

将 Linux 文件系统分区挂载为`/mnt`，将 EFI 系统分区挂载为`/mnt/boot`。

> 系统启动时，会在 EFI 系统分区寻找一个 [Boot Loader（Manager）](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)或者使用 [EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB) 方式直接启动 Linux 系统内核。为了方便日后安装双系统或者调整启动参数建议安装`systemd-boot`或者`GRUB`作为启动器。笔记本中俺是安装了[双系统](https://wiki.archlinux.org/title/Dual_boot_with_Windows)。
>
> ~~根据官方 Wiki 上关于 [EFI 系统分区挂载](https://wiki.archlinux.org/index.php/EFI_system_partition#Mount_the_partition) 的说明，当选择 EFISTUB 直接启动的方法时，需将 EFI 系统分区挂载到 `/boot` 中。对应于我们将 root 挂载到 `/mnt` 的情况，应将 EFI 系统分区挂载到 `/mnt/boot` 目录。~~（是否必须？存疑。）

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## 安装 Arch Linux

### 选择镜像服务器

Arch Linux 默认在线安装。此外，更新时也会通过镜像服务器下载软件包。镜像服务器的列表在`/etc/pacman.d/mirrorlist`中，在安装过程中会被自动拷贝到新系统。在连接到互联网后，`reflector`会自动从镜像服务器列表中选择前 20 个最近完成同步的服务器。下载软件包时会自动按照列表中的先后顺序选择服务器。但这些服务器中有的从国内连接会很慢。如果发生这种情况，可以手动改成使用国内大厂、大学的源。

我们也可以按照 Wiki 上关于 [镜像服务器排序](https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors) 的说明测试并修改列表：

```bash
reflector --country China --sort rate --save /etc/pacman.d/mirrorlist
```

还可以手动编辑，将自己喜好的镜像服务器放置在列表的前列：

-   如不想用 Vim / Nano 等命令行文本编辑器的话，可以直接用某个服务器地址覆写列表文件：

    ```bash
    echo "Server = https://mirrors.cloud.tencent.com/archlinux/\$repo/os/\$arch" > /etc/pacman.d/mirrorlist
    ```

-   如果希望再附加一两行服务器，用 `>>` 来在文件末尾附加新内容，例如：

    ```bash
    cat >> /etc/pacman.d/mirrorlist
    Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/\$repo/os/\$arch
    Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/\$repo/os/\$arch
    Server = http://mirrors.163.com/archlinux/\$repo/os/\$arch
    #ctrl+d
    ```

    > 最后一行表示用组合键`ctrl+d`输入文件终止符`EOF`来完成该文件的编写。详见 [Linux 系统输入输出重定向简介](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-i-o-redirection)。

最后检查文件内容：

```bash
cat /etc/pacman.d/mirrorlist
```

### 启动 pacman 并行下载

在 `/etc/pacman.conf` 中将 `ParallelDownloads` 设置前的注释删掉其用并设置为你认为合适的线程数,比如 16 个或更多，即可[启用多个软件包并行下载](https://wiki.archlinux.org/title/Pacman#Enabling_parallel_downloads)。

### 使用 powerpill 缓存软件包

如果更新了服务器列表后还不够快，可以下载[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)，并用其先缓存各软件包。当然如果身边有可用的 Arch 机器，也可以用[`pacserve`](https://wiki.archlinux.org/index.php/Pacserve)，这里只说下`powerpill`。

由于`powerpill`属于 AUR 而非官方源，ArchISO 下`makepkg`又会比较麻烦，所以建议添加 ArchlinuxCN 源**或**作者的源，并[将`SigLevel`改成`PackageRequired`](https://wiki.archlinux.org/title/Pacman/Package_signing#Setup)：

```bash
vim /etc/pacman.conf
# 将通用 SigLevel 改成 PackageRequired
SigLevel = PackageRequired
# ArchLinuxCN
[archlinuxcn]
Server = https://mirrors.cloud.tencent.com/archlinuxcn/$arch
# powerpill 作者源（个人建议有 CN 源就够了，别加这个了）
[xyne-x86_64]
SigLevel = Required
Server = https://xyne.archlinux.ca/bin/repo.php?file=
```

安装`powerpill`并将各软件包缓存到新机上，（如果在虚拟机上，不需要安装`linux-firmware`。在容器中，还能不装`linux`内核）：

```bash
# 如用 ArchLinuxCN，则需要安装 keyring
pacman -Sy archlinuxcn-keyring
pacman -S powerpill
mkdir -p /mnt/var/cache/pacman/pkg
powerpill -Swb /tmp --cachedir /mnt/var/cache/pacman/pkg base base-devel linux linux-firmware
```

建议此时就将所有之后想要安装的软件包一次性缓存下来，节省后续手动安装过程中等待下载的时间。下一步的`pacstrap`工具会使用这些缓存直接安装。如果之前选择的镜像服务器不好用，可以`ctrl+c`中止下载并重新选择服务器。下次`pacman`会从中止的软件包开始续传。

### 安装必要的软件包

下载并安装 Arch Linux：

```bash
pacstrap /mnt base linux linux-firmware
```

> 注 1：此时便可在命令后续写其他想要安装的软件包名，用空格隔开。这些软件包会被直接安装到新系统中。
>
> 注 2：从 2019 年 10 月 6 日起，base 软件包组被同名的新软件包`base`[替换](https://www.archlinux.org/news/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)，且不再包含`linux`、`linux-firmware`、`netctl`、`vi`等包。俺已经根据这一变化、重新安装了系统并更新了这篇文章。
>
> 这一改动的好处在于日后`base`中添加的新成员可以跟随`base`的更新而自动安装。排除一些非必须的软件包可以给诸如进行批量部署的运维人员自定义需要的部件等使用场景带来方便。大家都可以安装自己需要的内核（`linux`）、文本编辑器（`neovim`）、网络管理器（`networkmanager`）、固件（`broadcom-wl-dkms`）等等，而不必用`ignore`来排除、安装后删除、又或者不管不顾造成冗余。

## 配置系统

### Fstab

> [fstab](https://wiki.archlinux.org/index.php/Fstab) 文件用来规定分区、磁盘等如何被（自动）挂载。如果像我一样只在一个 GPT 盘上装系统，可以不用在此时就生成这个文件，而是用 systemd [自动挂载](https://wiki.archlinux.org/title/Systemd#GPT_partition_automountinghttps://wiki.archlinux.org/title/Fstab#GPT_partition_automounting)。否则请看下面的常规操作。

创建 fstab 文件，用`-U`或`-L`来指定用 [UUID](https://wiki.archlinux.org/index.php/UUID) 还是标签来标识（建议使用 UUID）：

```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

检查生成的文件：

```bash
cat /mnt/etc/fstab
```

### Chroot

变更可见根目录到新系统中，开始新系统的设置：

```bash
arch-chroot /mnt
```

### 下载并配置默认文本编辑器

有`nano`、`vi`、`vim`、`neovim`等可以使用，俺使用的是`neovim，并且将其通过 [`symlink`](https://en.wikipedia.org/wiki/Symbolic_link) 用其彻底替换`vi`（也可以安装 `aur/neovim-symlinks` 包帮你完成替换）：

```bash
pacman -S neovim
ln -s /usr/bin/{nvim,vi}
```

### 时区

设置时区（亚洲 / 上海）（也可以暂时不设置，启动到新系统后用桌面环境自带设置或者 `timedatectl` 更改）：

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

将硬件时钟设置为与系统时钟相同：

```bash
hwclock --systohc
```

### 本地化

程序和系统均需要用 [Locale](https://wiki.archlinux.org/index.php/Locale) 来确定地域、货币、时间日期格式等等。要使用某个 locale 设置，就要先生成它。有时候，在系统使用过程中即使自己只用到一套 locale，也要为了支持切换、或者支持此电脑上的其他用户对不同的 locale 的需要，将所有可能用到的 locale 都一并生成。

移除`/etc/locale.gen`中您需要的 locale 前的`#`号注释。可以用 Nano/Vi(m) 等工具修改，也可以用`cat`命令直接编写，例如：

```bash
cat > /etc/locale.gen
en_US.UTF-8 UTF-8
ja_JP.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_HK.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
# ctrl+d
```

> 注：建议选择 UTF-8 字符集的 locale。

之后生成 locale 讯息：

```bash
locale-gen
```

创建`locale.conf`文件，设置`LANG`变量（也可以不设置，进入新系统后用桌面环境自带设置或`localectl`设置）：

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

> 注：建议操作系统均使用英文 locale，这样终端内的输出、软件报错信息均为英语，便于 Google 搜索和问题分享等等。反之，设置成汉字语系的 locale 还可能导致 TTY 乱码。

之后设置新系统的键盘布局，由于俺用默认的 US 键盘布局，故跳过。

### 网络配置

创建 [hostname](https://wiki.archlinux.org/index.php/Hostname) 文件（也可以不设置，进入新系统后用桌面环境自带的设置或者用`hostnamectl`设置）：

> 用您喜欢的主机名替换`myhostname`。

```bash
echo myhostname > /etc/hostname
```

[（这一步可以跳过）](https://wiki.archlinux.org/title/Network_configuration#Local_hostname_resolution)创建对应的 [hosts](https://jlk.fjfi.cvut.cz/arch/manpages/man/hosts.5) 文件：

```bash
cat > /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 myhostname.localdomain myhostname
# ctrl+d
```

> 如果此系统有一个永久 IP 地址，应该用此地址替换`127.0.1.1`。

完成其他[网络配置](https://wiki.archlinux.org/index.php/Network_configuration)：

-   虚拟机：

    对于虚拟机，只需要启动 [`dhcpcd`](https://wiki.archlinux.org/index.php/Dhcpcd#Running) 服务即可：

    ```bash
    systemctl enable dhcpcd
    ```

-   物理机：

    如使用有线网络连接，同上述虚拟机配置一样只需启动`dhcpcd`服务即可。如需在新系统中继续使用 Wi-Fi，则建议安装一个 [网络管理器](https://wiki.archlinux.org/index.php/Network_configuration#Network_managers)。

    鉴于 GNOME、KDE 等[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)常使用`networkmanager`来管理网络连接，Xfce 也支持`network-manager-applet`作为前端，此包会在安装桌面环境时作为依赖自动安装。俺建议在`netctl`被移出`base`的当下，使用它来联网。

    如果想和俺一样使用 Xfce 作为桌面环境，则直接安装 `network-manager-applet`

    ```bash
    pacman -S network-manager-applet
    ```

    如果使用 GNOME、KDE 等默认使用它的桌面环境，则作为依赖包安装 `networkmanager`

    ```bash
    pacman -S networkmanager --asdeps
    ```

    安装完成后启用它，下次启动到系统后就会自动启动网络管理服务了。

    ```bash
    systemctl enable NetworkManager
    ```

    此外，关于无线网卡驱动，俺的笔电使用的无线网卡芯片为高通的 BCM43142，不被高通官方开源网卡驱动支持。而官方的专有驱动 [`wl`](https://wiki.archlinux.org/index.php/Broadcom_wireless#broadcom-wl) 则似乎由于知识产权问题，不能由其他组织分发，所以没能被集成在 Linux 操作系统内核自带固件全家桶`linux-firmware`中，必须由自行安装。对应的，俺不用安装`linux-firmware`似乎也没事：

    ```bash
    pacman -S broadcom-wl
    ```

    该驱动有一种可以支持 [DKMS](https://wiki.archlinux.org/index.php/DKMS) 技术、随 Linux 系统内核更新自动适配的版本，怕部分更新而出问题也可以安装 DKMS 版：

    ```bash
    pacman -S linux-headers broadcom-wl-dkms
    ```

### Initramfs

一般来说不需要创建新的 initramfs，除非想要使用 [LVM](https://wiki.archlinux.org/index.php/LVM#Configure_mkinitcpio)，[系统加密](https://wiki.archlinux.org/index.php/Dm-crypt)，或者 [RAID](https://wiki.archlinux.org/index.php/RAID#Configure_mkinitcpio) 等进阶技术。

如果您之前没有生成`fstab`，希望使用 systemd 来自动挂载 root 等分区，则需要用 systemd init hook。

编辑`/etc/mkinitcpio.conf`，在`HOOKS`中将`base`改为`systemd`，并去掉`udev`、`usr`、`resume`。完成后创建新的 initramfs：

```bash
mkinitcpio -P
```

### 设置 Root 密码或创建其他管理员账户

我们可以选择设置 [root 账户密码](https://wiki.archlinux.org/index.php/Password)或不设置 root 账户密码，直接创建我们需要的：

```bash
passwd
```

### Boot loader

这是重启到新系统前最后的步骤了，以下参考了 [Arch 启动过程 #Boot loader](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader) 来配置系统的 Boot loader。

如果是在 **物理机** 上安装，需要先安装 [Microcode](https://wiki.archlinux.org/index.php/Microcode)，以便根据处理器品牌，配置早期微码更新。Intel 处理器的操作如下，Amd 处理器需要将`intel`替换为`amd`：

```bash
pacman -S intel-ucode
```

> 虚拟机上 [不需要考虑 Microcode](https://wiki.archlinux.org/index.php/Microcode#EFISTUB)

如果希望双启动的，可以用[`systemd-boot`](https://wiki.archlinux.org/title/Systemd-boot)来控制启动过程；如果不做双启动，可以选择更精简的 EFISTUB 启动方法，并参考 [Using UEFI directly](https://wiki.archlinux.org/index.php/EFISTUB#Using_UEFI_directly) 来进行配置。下面先描述 EFISTUB，再描述 systemd-boot。

#### EFISTUB

编写 EFISTUB 启动项的脚本：

1. 首先用`exit`命令或者组合键`ctrl+d`退出 chroot 环境。
2. 创建一个脚本文件来存贮将要执行的一条长命令，方便我们检查和修改参数：

    ```bash
    vi efibootmgr.sh
    ```

    > 不用 Vi(m)也可以用 Nano 等编辑器或者`cat`、`echo`等命令。但之后输入[分区 UUID](https://wiki.archlinux.org/index.php/Persistent_block_device_naming#by-partuuid) 俺是用 Vi(m)的[`:read`](https://vim.fandom.com/wiki/Append_output_of_an_external_command#Using_:read) 命令完成的。
    >
    > #### 一点 Vi 的小说明
    >
    > 进入 Vi 后是普通模式。
    >
    > 普通模式下按`h` `j` `k` `l`分别向 ←↓↑→ 移动光标。按`i`进入插入模式开始键入文字。
    >
    > 按`Esc`从各种模式返回普通模式。
    >
    > 普通模式下键入`:`和命令并回车来执行 Vi 命令，例如`:q`在未修改文件的情况下退出、`:q!`放弃修改并退出、`:w`保存、`:wq`/`:x`保存并退出等。上述`:read`也是如此执行。
    >
    > 普通模式下键入`dd`来剪切当前行，`p`来粘贴，`J`（即`shift+j`）来将下一行接到当前行末尾。
    >
    > 更多 Vi(m) 技巧请您自行了解，不在此赘述。

3. 在其中写入如下内容：

    > 将下文`/dev/sda`和 `1` 替换为您的 EFI 系统分区（ESP）的磁盘和分区号。`--disk /dev/sda --part 1` 对应的是俺之前创建的 ESP——`/dev/sda1`。

    - 虚拟机：

        ```bash
        efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=/initramfs-linux.img' --verbose
        ```

    - 物理机：

        > 多一段启动 Microcode 的 `initrd=/intel-ucode.img`

        ```bash
        efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=/intel-ucode.img initrd=/initramfs-linux.img' --verbose
        ```

4. `root`参数如果之前做好了使用 systemd 自动挂载的准备可以忽略掉，否则，将`root=`之后的 PARTUUID 的参数替换成 Linux 文件系统分区（俺这里是`/dev/sda2`）的分区 UUID。手动输入比较麻烦，这里俺使用的 Vi(m) 的`:read`命令，让 Vi(m) 将`:read !`后面的文字当作 Shell 命令执行，并将结果另起一行写在文件中。

    ```vim
    :read !lsblk -dno PARTUUID /dev/sda2
    ```

    > 注意：默认情况下 Vi 会 [防止用户使用退格键删除`自动缩进`、`换行符`以及`进入插入模式时的位置`之前的字符](https://vi.stackexchange.com/a/2163/22060)。要么直接在普通模式下用`J`（大写 J）将下一行接到当前行末尾，要么临时`:set backspace=indent,eol,start`。

5. 检查脚本中内容：

    ```bash
    cat efibootmgr.sh
    fdisk -l
    lsblk -dno PARTUUID /dev/sda2
    ```

6. 确认无误后，可将该脚本在机器磁盘中备份一份（可选），赋予该脚本可执行权限，并运行：

    ```bash
    cp efibootmgr.sh /mnt/home
    chmod u+x efibootmgr.sh
    ./efibootmgr.sh
    ```

7. 再次检查启动项是否配置无误：

    ```bash
    efibootmgr --verbose
    ```

8. 设置启动顺序（可选）：

    > 运行完脚本后，efibootmgr 自动将`Arch Linux`项设置为第一启动项，故此步骤可以跳过。

    ```bash
    efibootmgr -o XXXX,XXXX --verbose
    ```

    其中 XXXX 即 efibootmgr 的输出中各启动项前的四位数。

    > 俺在笔电上安装时会将 U 盘设为第一项，系统设为第二项，主板固件设置设为第三项。关机之后移除 U 盘便可从硬盘系统启动。之后若无法启动或缺少驱动，下次启动前插入 U 盘便能进入安装环境进行急救。确认启动正常后，可用 `systemctl reboot --firmware-setup` 重启到主板设置或者直接用 efibootmgr 将系统启动项调回第一项。

#### systemd-boot

[systemd-boot 启动](https://wiki.archlinux.org/title/Systemd-boot]其实很简单，而且也支持启动时选择进入 Windows 或者主板设置。

首先[安装 systemd-boot](https://wiki.archlinux.org/title/Systemd-boot#Installation)：

```bash
bootctl install
```

之后进行配置。先创建启动项，可以拷贝参考样例：

```bash
cp /usr/share/systemd/bootctl/arch.conf /boot/loader/entries
```

再将拷贝来的文件中的`options`行删掉，在`initrd /initramfs-linux.img`行之前加上载入 ucode 的配置（如是 amdcpu 则将 intel 改为 amd）：

```
initrd  /intel-ucode.img
```

最后是这样的：

```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
```

保存以后可以继续编写 systemd-boot 本身的配置`/boot/loader/loader.conf`，不过其实没啥重要的，全删了用默认值也没问题。

## 重启

1. 用组合键`ctrl+d`或者命令退出 chroot 环境：

    ```bash
    exit
    ```

2. 卸载文件系统（可选）：

    ```bash
    umount -R /mnt
    ```

    如果文件系统正处于使用中，可以用 [fuser(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/fuser.1) 找到。

3. 关机：

    ```bash
    shutdown now
    ```

    所有仍然挂载的文件系统会被`systemd`自动卸载。

    移除安装介质：

    - 虚拟机

        在虚拟机设置中将 SCSI 控制器里的 DVD 驱动器中的安装镜像关掉。

        ![remove-install-disk](../attachments/remove-install-disk.png)

    - 物理机

        确认关机后移除 U 盘即可。

4. 再次启动并用设置好的密码登录 root 账户后，终于进入了 Arch Linux。

    ![welcome-to-arch-linux](../attachments/welcome-to-arch-linux.png)

## 安装完成后的工作

至此，我们便可以在 Hyper-V 虚拟机 / 物理机中运行最小化、可联网的 Arch Linux 系统。可以关机稍作休息，择时进行后续配置过程。

您可以进一步参考官方[推荐的安装完成后的操作](https://wiki.archlinux.org/index.php/General_recommendations)。包括但不限于：

-   创建用户账户、配置 `sudo`
-   安装蓝牙、显卡等的闭源驱动
-   配置无线网络连接
-   安装桌面环境

俺也会在之后 [Arch 新机用上 Google](customize-arch-to-use-google.md) 一文中继续记录俺进一步折腾系统到可以用搜索引擎查资料的程度的步骤，详情请移步后文。
