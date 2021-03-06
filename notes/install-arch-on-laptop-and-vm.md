---
tags: [Blog/System Configurations]
title: 在虚拟机/物理机中安装Arch Linux
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [预备理论](#预备理论)
- [准备安装](#准备安装)
- [安装Arch Linux](#安装arch-linux)
- [配置系统](#配置系统)
- [重启](#重启)
- [安装完成后的工作](#安装完成后的工作)

## 前言

本文将承接前文[安装Arch Linux前的准备工作](prepare-to-install-arch.md)和[为Arch Linux创建Hyper-V虚拟机](create-vm-for-arch.md)，按步骤介绍在虚拟机以及物理机中安装Arch Linux的步骤。个别步骤虚拟机同物理机的操作不一致，后文中会注明，请注意区分。

以下参考官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)。

## 预备理论

### 打开UEFI & 关闭Secure Boot

启动机器前，请在机器固件设置（虚拟机配置或物理机主板配置）中打开UEFI模式。并且，俺建议和虚拟机安装过程一样关闭[Secure Boot](https://www.rodsbooks.com/efi-bootloaders/secureboot.html)。支持这一建议的[原因](https://www.reddit.com/r/archlinux/comments/8nbau0/secure_boot_yay_or_nay/)为——Secure Boot保护我们不受[Evil Maid Attack](https://en.wikipedia.org/wiki/Evil_maid_attack)之类物理攻击，但俺自用笔记本的环境几乎不存在收到这类攻击的可能，并且俺也不涉及绝密级别的工作。鉴于俺已经不熟练（xiǎng tōu lǎn）到BootLoader都不会装、全盘加密也不会做、sudoers还不会配的份儿上，这种Windows平台下主板里按个回车就能够轻松享受，Linux却要看半个小时才弄得好的只有理论价值的安全措施暂时就不考虑了。如果有需要可以参看官方[Secure Boot指南](https://wiki.archlinux.org/index.php/Secure_Boot)。

### 在物理机上查看文档

之前提到过尝鲜建议先用虚拟机安装，可以在原系统下方便查阅资料。

如果在物理机上启动了Arch安装介质后，尽管相对抽象，但也有办法查阅资料。在[连接到互联网](#连接到互联网)之后，可以使用`elinks`文本型网页浏览器查看官方指南：

```bash
elinks https://wiki.archlinux.org/index.php/Installation_guide
```

> 如果觉得白色背景亮瞎了眼，可以按`esc`唤出菜单 -> `Setup` -> `Options Manager` -> 按空格键展开`Default color settings` -> `Use document-specified colors` -> `Edit` -> `Value` 改为 `1` （或 `0`）

也可以查看ArchISO压制时附带的纯文本安装指南：

```bash
ls
less install.txt
```

如果希望一边看网页/文档，一边进行安装，可以用`ctrl+alt+F1`(`F1`到`F7`均可)和`alt+左/右键`切换终端。

## 准备安装

### 选择键盘布局

俺使用默认的US布局，故不做更改。

### 验证启动模式

运行如下命令，文件夹存在并有内容即正常。如不存在，则系统是在BIOS环境中启动的，请再次检查创建虚拟机时是否选择的Generation 2。（如是在物理机上安装，检查主板设置是否开启了UEFI模式。）

```bash
ls /sys/firmware/efi/efivars
```

### 连接到互联网

虚拟机之前已经配置了虚拟网络适配器，故跳过。

物理机安装时如使用有线连接，无需额外操作；而如需使用Wi-Fi连接，可以使用图形化工具：

```bash
wifi-menu -o
```

一般来说，`archboot`安装环境中已经包含了各种网卡的驱动，其中就有俺所需的高通专有驱动`broadcom-wl`。但如果工具仍然报错，请根据[无线网络配置](https://wiki.archlinux.org/index.php/Wireless_network_configuration)检查常见问题。

根据向导选择网络，输入密码之后，工具便会在`/etc/netctl`创建一个连接配置文件，接下来启动它：

> 用您指定的配置文件名称替换下文的`profile`，可在输入`start`和空格后，按`tab`键自动补全。

```bash
netctl start profile
```

如此便可连接上Wi-Fi网络。

### 更新系统时钟

```bash
timedatectl set-ntp true
```

检查时钟状态：

```bash
timedatectl status
```

![ArchISO运行时截图](../attachments/check-timedate-status.png)

### 磁盘分区

列出所有硬盘：

```bash
fdisk -l
```

寻找类似`/dev/sda`或者`/dev/nvme0n1`的条目，并根据磁盘的大小等信息，确定您想要安装的磁盘。

Linux系统需要至少一个root目录（`/`）分区，如果启用了UEFI，还需要一个[EFI系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition)。[Swap](https://wiki.archlinux.org/index.php/swap)可以通过分区实现也可以通过动态Swap文件实现。

俺参考了官方分区模板，选择将磁盘分为512MB的EFI系统分区、利用全部剩余空间的Linux文件系统分区和一个8GB的Swap分区。Swap分区的大小可以参考It's FOSS上的[一篇文章](https://itsfoss.com/swap-size/)。一般来说设为物理内存的两倍肯定够用了。

分区时为了方便可以用[`fdisk`](https://wiki.archlinux.org/index.php/Fdisk)工具的图形版`cfdisk`：

```bash
cfdisk
```

1. Select label type选择`gpt`（如果未提示，检查屏幕上方第三行是否为`Lable: gpt`）
2. New一个`512M`的分区，Type更改为`EFI System`
3. New一个大小为剩余空间-`8GB`（即`cfdisk`自动填写的大小减去`8GB`）的分区，并确保Type为`Linux filesystem`
4. New一个`8GB`的分区，Type更改为`Linux swap`
5. 选择Write将分区更改写到磁盘
6. Quit

### 格式化分区

刚创建好的分区是没有文件系统的，我们要自行创建文件系统。

> 可以用`fdisk -l`来查看分区大小，用`lsblk -f`来查看分区格式

将EFI系统分区格式化为FAT32格式，将Linux文件系统分区格式化为ext4格式：

> 如果您的分区名和下述不一致，请自行替换

```bash
mkfs.fat -F32 /dev/sda1
mkfs.ext4 /dev/sda2
```

![格式化分区](../attachments/format-partitions.png)

如果之前创建了Swap分区，应将其启用：

```bash
mkswap /dev/sda3
swapon /dev/sda3
```

### 挂载分区

将Linux文件系统分区挂载为`/mnt`，将EFI系统分区挂载为`/mnt/boot`。

> 系统启动时，会在EFI系统分区寻找一个[Boot Loader](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)、[Boot Manager](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)或者使用[EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB)方式直接启动Linux系统内核。由于俺在虚拟机及笔电中只安装一个系统，不涉及双系统及其他复杂情况，所以选择EFISTUB方式。
>
> 根据官方Wiki上关于[EFI系统分区挂载](https://wiki.archlinux.org/index.php/EFI_system_partition#Mount_the_partition)的说明，当选择EFISTUB直接启动的方法时，需将EFI系统分区挂载到`/boot`中。对应于我们将root挂载到`/mnt`的情况，应将EFI系统分区挂载到`/mnt/boot`目录。

```bash
mount /dev/sda2 /mnt
mkdir /mnt/boot
mount /dev/sda1 /mnt/boot
```

## 安装Arch Linux

### 选择镜像服务器

Arch Linux默认在线安装。此外，更新时也会通过镜像服务器下载软件包。镜像服务器的列表在`/etc/pacman.d/mirrorlist`中，在安装过程中会被自动拷贝到新系统。为了方便，此时就应选择一些连通性良好、速度够快的服务器，可以节省很多时间。

下载软件包时会自动按照列表中的先后顺序选择服务器。我们可以按照Wiki上关于[镜像服务器排序](https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors)的说明测试并修改列表：

```bash
pacman -Sy reflector
reflector --country China --sort rate --save /etc/pacman.d/mirrorlist
```

> 如果出现：`ImportError: No module named Reflector`的错误。可能是因为ArchISO的`python`版本过旧。用`pacman -S python --needed`更新一下并再次运行`reflector`也许可以解决问题。

也可以手动编辑，将自己喜好的镜像服务器放置在列表的前列：

- 如不想用Vim/Nano等命令行文本编辑器的话，可以直接用某个服务器地址覆写列表文件：

  ```bash
  echo "Server = https://mirrors.cloud.tencent.com/archlinux/\$repo/os/\$arch" > /etc/pacman.d/mirrorlist
  ```

- 如果希望再附加一两行服务器，用`>>`来在文件末尾附加新内容，例如：

  ```bash
  cat >> /etc/pacman.d/mirrorlist
  Server = https://mirrors.sjtug.sjtu.edu.cn/archlinux/\$repo/os/\$arch
  Server = http://mirrors.163.com/archlinux/\$repo/os/\$arch
  #ctrl+d
  ```

  > 最后一行表示用组合键`ctrl+d`输入文件终止符`EOF`来完成该文件的编写。详见[Linux系统输入输出重定向简介](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-i-o-redirection)。

最后检查文件内容：

```bash
cat /etc/pacman.d/mirrorlist
```

### 使用powerpill缓存软件包

如果更新了服务器列表后还不够快，可以下载[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)，并用其先缓存各软件包。当然如果身边有可用的Arch机器，也可以用[`pacserve`](https://wiki.archlinux.org/index.php/Pacserve)，这里只说下`powerpill`。

由于`powerpill`属于AUR而非官方源，ArchISO下`makepkg`又会非常麻烦，所以建议添加ArchlinuxCN源**或**作者的源，并[将`SigLevel`改成`PackageRequired`](https://wiki.archlinux.org/index.php/Powerpill#Troubleshooting)：

```bash
vim /etc/pacman.conf
# 将通用SigLevel改成PackageRequired
SigLevel = PackageRequired
# ArchLinuxCN
[archlinuxcn]
Server = https://mirrors.cloud.tencent.com/archlinuxcn/$arch
# powerpill作者源（个人建议有CN源就够了，别加这个了）
[xyne-x86_64]
SigLevel = Required
Server = https://xyne.archlinux.ca/bin/repo.php?file=
```

安装`powerpill`并将各软件包缓存到新机上：

```bash
# 如用ArchLinuxCN，则需要安装keyring
pacman -Sy archlinuxcn-keyring
pacman -S powerpill
mkdir -p /mnt/var/cache/pacman/pkg
powerpill -Sw --dbpath /tmp --cachedir /mnt/var/cache/pacman/pkg base base-devel linux linux-firmware
```

建议此时就将所有之后想要安装的软件包一次性缓存下来，节省后续手动安装过程中等待下载的时间。下一步的`pacstrap`工具会使用这些缓存直接安装。

### 安装必要的软件包

下载并安装Arch Linux：

```bash
pacstrap /mnt base linux linux-firmware
```

如果之前选择的镜像服务器不好用，可以`ctrl+c`中止下载并重新选择服务器。下次`pacman`会从中止的软件包开始续传。

> 注1：此时便可在命令后续写其他想要安装的软件包名，用空格隔开。这些软件包会被直接安装到新系统中。但是部分软件包的安装可能依赖于执行附加脚本，用`pacstrap`安装可能会遇到路径、变量等问题。俺的建议是在初体验Arch安装、未经实践的情况下，只安装原`base`组和`base-devel`这类大家都这样装过的软件包。其余的在chroot和重启到新系统之后再行安装。
>
> 注2：从2019年10月6日起，base软件包组被同名的新软件包`base`[替换](https://www.archlinux.org/news/base-group-replaced-by-mandatory-base-package-manual-intervention-required/)，且不再包含`linux`、`linux-firmware`、`netctl`、`vi`等包。俺已经根据这一变化、重新安装了系统并更新了这篇文章。
>
> 这一改动的好处在于日后`base`中添加的新成员可以跟随`base`的更新而自动安装。排除一些非必须的软件包可以给诸如进行批量部署的运维人员自定义需要的部件等使用场景带来方便。大家都可以安装自己需要的内核（`linux`）、文本编辑器（`neovim`）、网络管理器（`networkmanager`）、固件（`broadcom-wl-dkms`）等等，而不必用`ignore`来排除、安装后删除、又或者不管不顾造成冗余。

## 配置系统

### Fstab

创建[fstab](https://wiki.archlinux.org/index.php/Fstab)文件，用`-U`或`-L`来指定用[UUID](https://wiki.archlinux.org/index.php/UUID)还是标签来标识（建议使用UUID）：

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

有`nano`、`vi`、`vim`、`neovim`等可以使用，俺使用的是`neovim`，并且将其通过[`symlink`](https://en.wikipedia.org/wiki/Symbolic_link)用其彻底替换`vi`：

```bash
pacman -S neovim
ln -s /usr/bin/{nvim,vi}
```

### 时区

设置时区（亚洲/上海）：

```bash
ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```

将硬件时钟设置为与系统时钟相同：

```bash
hwclock --systohc
```

### 本地化

程序和系统均需要用[Locale](https://wiki.archlinux.org/index.php/Locale)来确定地域、货币、时间日期格式等等。要使用某个locale设置，就要先生成它。有时候，在系统使用过程中即使自己只用到一套locale，也要为了支持切换、或者支持此电脑上的其他用户对不同的locale的需要，将所有可能用到的locale都一并生成。

移除`/etc/locale.gen`中您需要的locale前的`#`号注释。可以用Nano/Vi(m)等工具修改，也可以用`cat`命令直接编写，例如：

```bash
cat > /etc/locale.gen
en_US.UTF-8 UTF-8
ja_JP.UTF-8 UTF-8
zh_CN.UTF-8 UTF-8
zh_HK.UTF-8 UTF-8
zh_TW.UTF-8 UTF-8
# ctrl+d
```

> 注：建议选择UTF-8字符集的locale。

之后生成locale讯息：

```bash
locale-gen
```

创建`locale.conf`文件，设置`LANG`变量：

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
```

> 注：建议操作系统均使用英文locale，这样终端内的输出、软件报错信息均为英语，便于Google搜索和问题分享等等。反之，设置成汉字语系的locale还可能导致TTY乱码。

之后设置新系统的键盘布局，由于俺用默认的US键盘布局，故跳过。

### 网络配置

创建[hostname](https://wiki.archlinux.org/index.php/Hostname)文件：

> 用您喜欢的主机名替换`myhostname`。

```bash
echo myhostname > /etc/hostname
```

创建对应的[hosts](https://jlk.fjfi.cvut.cz/arch/manpages/man/hosts.5)文件：

```bash
cat > /etc/hosts
127.0.0.1 localhost
::1 localhost
127.0.1.1 myhostname.localdomain myhostname
# ctrl+d
```

> 如果此系统有一个永久IP地址，应该用此地址替换`127.0.1.1`。

完成其他[网络配置](https://wiki.archlinux.org/index.php/Network_configuration)：

- 虚拟机：

  对于虚拟机，只需要启动[`dhcpcd`](https://wiki.archlinux.org/index.php/Dhcpcd#Running)服务即可：

  ```bash
  systemctl enable dhcpcd
  ```

- 物理机：

  如使用有线网络连接，同上述虚拟机配置一样只需启动`dhcpcd`服务即可。如需在新系统中继续使用Wi-Fi，则建议安装一个[网络管理器](https://wiki.archlinux.org/index.php/Network_configuration#Network_managers)。

  鉴于GNOME、KDE等[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)常使用`networkmanager`来管理网络连接，Xfce也支持`network-manager-applet`作为前端，并且此包会在安装KDE桌面环境时作为依赖自动安装。俺建议在`netctl`被移出`base`的当下，使用来联网。

  如果想和俺一样使用Xfce作为桌面环境，则直接安装`network-manager-applet`

  ```bash
  pacman -S network-manager-applet
  ```

  如果使用GNOME、KDE等默认使用它的桌面环境，则作为依赖包安装`networkmanager`

  ```bash
  pacman -S networkmanager --asdeps
  ```

  安装完成后启用它，下次启动到系统后就会自动启动网络管理服务了。

  ```bash
  systemctl enable NetworkManager
  ```

  此外，关于无线网卡驱动，俺的笔电使用的无线网卡芯片为高通的BCM43142，不被高通官方开源网卡驱动支持。而官方的专有驱动[`wl`](https://wiki.archlinux.org/index.php/Broadcom_wireless#broadcom-wl)则似乎由于知识产权问题，不能由其他组织分发，所以没能被集成在Linux操作系统内核自带固件全家桶`linux-firmware`中，必须由自行安装。对应的，俺不用安装`linux-firmware`似乎也没事：

  ```bash
  pacman -S broadcom-wl
  ```

  该驱动有一种可以支持[DKMS](https://wiki.archlinux.org/index.php/DKMS)技术、随Linux系统内核更新自动适配的版本，怕出问题也可以安装DKMS版：

  ```bash
  pacman -S linux-headers broadcom-wl-dkms
  ```

### Initramfs

一般来说不需要创建新的initramfs，除非想要使用[LVM](https://wiki.archlinux.org/index.php/LVM#Configure_mkinitcpio)，[系统加密](https://wiki.archlinux.org/index.php/Dm-crypt)，或者[RAID](https://wiki.archlinux.org/index.php/RAID#Configure_mkinitcpio)等进阶技术。这里跳过。

### Root密码

设置[root账户密码](https://wiki.archlinux.org/index.php/Password)：

```bash
passwd
```

### Boot loader

这是重启到新系统前最后的步骤了，以下参考了[Arch启动过程#Boot loader](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)来配置系统的Boot loader。

鉴于选择了EFISTUB启动方法，故参考[Using UEFI directly](https://wiki.archlinux.org/index.php/EFISTUB#Using_UEFI_directly)来进行配置。

如果是在**物理机**上安装，配置EFISTUB启动时还多一首小插曲——[Microcode](https://wiki.archlinux.org/index.php/Microcode)，即根据处理器品牌，配置早期微码更新。Intel处理器的操作如下，Amd处理器需要将`intel`替换为`amd`：

```bash
pacman -S intel-ucode
```

> 虚拟机上[不需要考虑Microcode](https://wiki.archlinux.org/index.php/Microcode#EFISTUB)

之后编写EFISTUB启动项的脚本：

1. 首先用`exit`命令或者组合键`ctrl+d`退出chroot环境。
2. 创建一个脚本文件来存贮将要执行的一条长命令，方便我们检查和修改参数：

   ```bash
   vi efibootmgr.sh
   ```

   > 不用Vi(m)也可以用Nano等编辑器或者`cat`、`echo`等命令。但之后输入[分区UUID](https://wiki.archlinux.org/index.php/Persistent_block_device_naming#by-partuuid)俺是用Vi(m)的[`:read`](https://vim.fandom.com/wiki/Append_output_of_an_external_command#Using_:read)命令完成的。
   >
   > #### 一点Vi的小说明
   >
   > 进入Vi后是普通模式。
   >
   > 普通模式下按`h` `j` `k` `l`分别向←↓↑→移动光标。按`i`进入插入模式开始键入文字。
   >
   > 按`Esc`从各种模式返回普通模式。
   >
   > 普通模式下键入`:`和命令并回车来执行Vi命令，例如`:q`在未修改文件的情况下退出、`:q!`放弃修改并退出、`:w`保存、`:wq`/`:x`保存并退出等。上述`:read`也是如此执行。
   >
   > 普通模式下键入`dd`来剪切当前行，`p`来粘贴，`J`（即`shift+j`）来将下一行接到当前行末尾。
   >
   > 更多Vi(m)技巧请您自行了解，不在此赘述。

3. 在其中写入如下内容：

   > 将下文`/dev/sda`和`1`替换为您的EFI系统分区（ESP）的磁盘和分区号。`--disk /dev/sda --part 1`对应的是俺之前创建的ESP——`/dev/sda1`。

   - 虚拟机：

     ```bash
     efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=/initramfs-linux.img' --verbose
     ```

   - 物理机：

     > 多一段启动Microcode的`initrd=/intel-ucode.img`

     ```bash
     efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=/intel-ucode.img initrd=/initramfs-linux.img' --verbose
     ```

4. 将`root=`之后的PARTUUID的参数替换成Linux文件系统分区（即`/dev/sda2`）的分区UUID。手动输入比较麻烦，这里俺使用的Vi(m)的`:read`命令，让Vi(m)将`:read !`后面的文字当作Shell命令执行，并将结果另起一行写在文件中。

   ```vim
   :read !lsblk -dno PARTUUID /dev/sda2
   ```

   > 注意：默认情况下Vi会[防止](https://vi.stackexchange.com/a/2163/22060)用户使用退格键删除`自动缩进`、`换行符`以及`进入插入模式时的位置`之前的字符的。要么直接在普通模式下用`J`（大写J）将下一行接到当前行末尾，要么临时`:set backspace=indent,eol,start`。

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

   > 运行完脚本后，efibootmgr自动将`Arch Linux`项设置为第一启动项，故此步骤可以跳过。

   ```bash
   efibootmgr -o XXXX,XXXX --verbose
   ```

   其中XXXX即efibootmgr的输出中各启动项前的四位数。

   > 俺在笔电上安装时会将U盘设为第一项，系统设为第二项，主板固件设置设为第三项。关机之后移除U盘便可从硬盘系统启动。之后若无法启动或缺少驱动，下次启动前插入U盘便能进入安装环境进行急救。确认启动正常后，可用`systemctl reboot --firmware-setup`重启到主板设置将系统调回第一项。

## 重启

1. 用组合键`ctrl+d`或者命令退出chroot环境：

   ```bash
   exit
   ```

2. 卸载文件系统（可选）：

   ```bash
   umount -R /mnt
   ```

   如果文件系统正处于使用中，可以用[fuser(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/fuser.1)找到。

3. 关机：

   ```bash
   shutdown now
   ```

   所有仍然挂载的文件系统会被`systemd`自动卸载。

   移除安装介质：

   - 虚拟机

     在虚拟机设置中将SCSI控制器里的DVD驱动器中的安装镜像关掉。

     ![remove-install-disk](../attachments/remove-install-disk.png)

   - 物理机

     确认关机后移除U盘即可。

4. 再次启动并用设置好的密码登录root账户后，终于进入了Arch Linux。

   ![welcome-to-arch-linux](../attachments/welcome-to-arch-linux.png)

## 安装完成后的工作

至此，我们便可以在Hyper-V虚拟机/物理机中运行最小化、可联网的Arch Linux系统。可以关机稍作休息，择时进行后续配置过程。

您可以进一步参考官方[推荐的安装完成后的操作](https://wiki.archlinux.org/index.php/General_recommendations)。包括但不限于：

- 创建用户账户、配置`sudo`
- 安装蓝牙、显卡等的闭源驱动
- 配置无线网络连接
- 安装桌面环境

俺也会在之后[Arch新机用上Google](customize-arch-to-use-google.md)一文中继续记录俺进一步折腾系统到可以用搜索引擎查资料的程度的步骤，详情请移步后文。
