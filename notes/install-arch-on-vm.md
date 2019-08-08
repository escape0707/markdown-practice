---
tags: [Blog/System Configurations]
title: 在Hyper-V虚拟机上安装Arch Linux
created: '2019-07-29T01:23:20.716Z'
modified: '2019-07-31T14:03:10.185Z'
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [下载 & 校验Arch Linux ISO安装镜像](#下载--校验arch-linux-iso安装镜像)
- [启用Hyper-V & 创建虚拟机](#启用hyper-v--创建虚拟机)
- [配置虚拟机](#配置虚拟机)
- [连接并启动虚拟机](#连接并启动虚拟机)
- [准备安装](#准备安装)
- [安装Arch Linux](#安装arch-linux)
- [配置系统](#配置系统)
- [重启](#重启)
- [安装完成后的工作](#安装完成后的工作)

## 前言

作为计划投放在个人博客上的第一篇文章，记录些前几天在台式机和笔记本电脑上安装Arch Linux的过程。我~~可怜又可爱~~的小笔记本电脑从高中二年级开始就一直稳健地发挥着热量，直到如今大学毕业。其硬盘、CPU、内存难以满足当下一套Windows 10 + Firefox Quantum + Electron App(Visual Studio Code / Trello)的欲求。恰巧最近想要在Linux环境下学习一些别的框架，挑挑拣拣便看中了这个配置相对复杂但是内容精简的Arch Linux + KDE往笔电上试着装了装。

此文不妄为各位读者解惑，仅能够作为下次安装或给同学朋友的参考便好。如对网络另一头的他人能有所帮助，不胜荣幸。

在尝试于一台物理机上安装Arch Linux之前，不妨先在虚拟机中体验一下安装Arch Linux的过程。况且，在虚拟机中安装，可以直接使用物理机的软硬件阅读安装指南、搜索资料，更为便利。另外，如遇困难，想中途停止安装，也可以轻松回滚。

本文描述笔者在一台Windows 10机器上，从开启Hyper-V虚拟功能，到能够在虚拟机中运行有网络连接的Arch Linux的过程。

> 笔者是在下述环境安装的：
>
> 操作系统：Windows 10 Pro 1903 18362.239
> 虚拟技术：Hyper-V
>
> 并不清楚Hyper-V与Oracle VM VirtualBox上仅仅是进行Arch Linux安装，会有什么体验上的不同。之前已经试过VirtualBox，此次为了少安装一个第三方软件尝试一下微软的Hyper-V。

## 下载 & 校验Arch Linux ISO安装镜像

我们可以按照官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)来下载和校验安装镜像。

### 下载

在[官网下载](https://www.archlinux.org/download/)镜像时，可选择使用[网易](http://mirrors.163.com/archlinux/iso/2019.07.01/)等速度较快的镜像服务器。

### 校验

Windows下可直接用`certutil`校验SHA1：

> 将`version`替换为您下载到的版本。

```powershell
certutil -hashfile archlinux-version-x86_64.iso.sig sha1
```

或在使用Rufus工具制作USB安装盘时，直接点击校验按钮校验SHA1。

或校验PGP签名：

1. Windows下可以使用[Gpg4win](https://www.gpg4win.org/)，安装前确认UAC提示的安装包签名是否正确。
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

## 启用Hyper-V & 创建虚拟机

可以参照官方[虚拟机安装指南](https://wiki.archlinux.org/index.php/Hyper-V)。这里为了方便读者，简单翻译一下。

### 启用Hyper-V

在开始菜单搜索并选择“Turn Windows features on or off”，找到“Hyper-V”并勾选，点击“OK”按钮。（勾选后为对勾抑或方块并不影响）

![启用Hyper-V](../attachments/enable-hyper-v.png)

### 配置虚拟机网络（可选）

Arch Linux的安装和使用均需要连接到网络，因此要给虚拟机分配一个Virtual Switch（虚拟交换器）。这里有使用外部交换器和内部交换器两种选择。

因无特殊需求，用随Windows 10 Fall Creators Update更新而来的内建NAT internal switch——“Default Switch”即可。

### 创建虚拟机

1. 在开始菜单搜索并运行“Hyper-V Manager”。选择“New”，“Virtual Machine”，进入新建虚拟机向导。

   ![新建虚拟机](../attachments/new-virtual-machine.png)

2. 在“Specify Generation”时，选择“Generation 2”来使用UEFI环境。

   ![选择Generation 2](../attachments/specify-generation-2.png)

3. 在“Assign Memory”时，选择恰当大小的内存，笔者使用了默认值。

4. 在“Configure Networking”时，“Connection”选择“Default Switch”

   ![图片](../attachments/choose-default-switch.png)

5. 在“Connect Virtual Hard Disk”时，选择“Create a virtual hard disk”

   ![创建VHD](../attachments/create-a-virtual-hard-disk.png)

6. 在“Installation Options”时，选择“Install an operating system from a bootable CD/DVD-ROM”，并选择之前下载的Arch Linux ISO镜像

   ![选择安装镜像](../attachments/choose-installation-image-file.png)

## 配置虚拟机

选择我们刚刚创建的虚拟机，打开其设置

![打开虚拟机设置](../attachments/open-virtual-machine-settings.png)

- 关闭Secure Boot：

  ![关闭Secure Boot](../attachments/virtual-machine-disable-secure-boot.png)

- 分配更多处理器核心（可选）：

  ![分配更多处理器核心](../attachments/virtual-machine-change-virtual-processors-number.png)

## 连接并启动虚拟机

1. 连接并启动：双击创建的虚拟机，选择Start。

2. 自此可以开始在Arch Linux Live Environment中进行安装步骤，先测试网络可用性：

   ```bash
   ping baidu.com
   ```

## 准备安装

   以下参考官方[安装指南](https://wiki.archlinux.org/index.php/Installation_guide#Pre-installation)。

### 选择键盘布局

笔者使用默认的US布局，故不做更改。

### 验证启动模式

运行如下命令，文件夹存在并有内容即正常。如不存在，则系统是在BIOS环境中启动的，请再次检查创建虚拟机时是否选择的Generation 2。（如是在物理机上安装，检查主板设置是否开启了UEFI模式。）

```bash
ls /sys/firmware/efi/efivars
```

### 连接到互联网

之前已经配置了虚拟网络适配器，故跳过。

*物理机安装时如需使用Wi-Fi连接，请参见笔者的后续文章：[在笔记本电脑上安装Arch Linux的额外事项#连接到互联网](install-arch-on-laptop.md#连接到互联网)。*

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

Linux系统需要至少一个root目录（`/`）分区，如果启用了UEFI，还需要一个[EFI系统分区](https://wiki.archlinux.org/index.php/EFI_system_partition)。Swap可以不通过分区实现而是通过动态Swap文件实现。

笔者参考了官方分区模板，选择将磁盘分为512MB的EFI系统分区和利用全部剩余空间的Linux文件系统分区。

分区时为了方便可以用[`fdisk`](https://wiki.archlinux.org/index.php/Fdisk)工具的图形版`cfdisk`

1. `Select label type`选择`gpt`（如果未提示，检查屏幕上方第三行是否为`Lable: gpt`）
2. `New`一个`512M`的分区，`Type`更改为`EFI System`
3. `New`一个大小为剩余空间（即`cfdisk`自动填写的大小）的分区，并确保`Type`为`Linux filesystem`
4. 选择`Write`将分区更改写到磁盘
5. `Quit`

![cfdisk分区结果](../attachments/cfdisk-result.png)

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

### 挂载分区

将Linux文件系统分区挂载为`/mnt`，将EFI系统分区挂载为`/mnt/boot`。

> 系统启动时，会在EFI系统分区寻找一个[Boot Loader](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)、[Boot Manager](https://wiki.archlinux.org/index.php/Arch_boot_process#Boot_loader)或者使用[EFISTUB](https://wiki.archlinux.org/index.php/EFISTUB)方式直接启动Linux系统内核。由于笔者在虚拟机及笔电中只安装一个系统，不涉及双系统及其他复杂情况，所以选择EFISTUB方式。
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

下载软件包时会自动按照列表中的先后顺序选择服务器。我们可以按照Wiki上关于[镜像服务器排序](https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors)的说明测试并修改列表。也可以手动编辑，将自己喜好的镜像服务器放置在列表的前列。

- 如不想用Vim/Nano等命令行文本编辑器的话，可以直接用某个服务器地址覆写列表文件：

  ```bash
  echo Server = https://mirrors.tuna.tsinghua.edu.cn/archlinux/$repo/os/$arch > /etc/pacman.d/mirrorlist
  ```

- 如果希望再附加一两行服务器，用`>>`替换上述命令的`>`来在文件末尾附加新内容，例如：

  ```bash
  echo Server = http://mirrors.163.com/archlinux/$repo/os/$arch >> /etc/pacman.d/mirrorlist
  ```

- 检查文件内容：

  ```bash
  cat /etc/pacman.d/mirrorlist
  ```

### 安装`base`软件包组

下载并安装Arch Linux，会花费大约三分钟：

```bash
pacstrap /mnt base
```

如果之前选择的镜像服务器不好用，可以`ctrl+c`中止下载并重新选择服务器。

> 注：此时便可在命令后面续写其他想要安装的软件包列表，用空格隔开。这些软件包会被直接安装到新系统中。

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
zh_TW.UTF-8 UTF-8
# ctrl+d
```

> 最后一行表示用组合键`ctrl+d`输入文件终止符`EOF`来完成该文件的编写。详见[Linux系统输入输出重定向简介](https://www.digitalocean.com/community/tutorials/an-introduction-to-linux-i-o-redirection)。
>
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

之后设置新系统的键盘布局，由于笔者用默认的US键盘布局，故跳过。

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

完成其他[网络配置](https://wiki.archlinux.org/index.php/Network_configuration)，*物理机安装时如需使用Wi-Fi连接，请参见笔者的后续文章：[在笔记本电脑上安装Arch Linux的额外事项#网络配置](install-arch-on-laptop.md#网络配置)。*

对于此虚拟机，只需要启动[`dhcpcd`](https://wiki.archlinux.org/index.php/Dhcpcd#Running)服务即可：

```bash
systemctl enable dhcpcd
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

1. 首先创建一个脚本文件来存贮将要执行的一条长命令，方便我们检查和修改参数：

   ```bash
   vi /home/efibootmgr.sh
   ```

   > - 不用Vi也可以用Nano等编辑器或者`cat`、`echo`等命令。但之后输入[分区UUID](https://wiki.archlinux.org/index.php/Persistent_block_device_naming#by-partuuid)笔者是用Vi的[`:read`](https://vim.fandom.com/wiki/Append_output_of_an_external_command#Using_:read)命令完成的。
   >
   > - 新系统中没有Vim、efibootmgr等工具。如果用`exit`命令或者组合键`ctrl+d`退出了chroot环境，记得将`/home/efibootmgr.sh`路径替换成`/mnt/home/efibootmgr.sh`。
   >
   > #### 一点Vi的小说明
   >
   > 进入Vi后在普通模式。
   >
   > 普通模式下按`h` `j` `k` `l`分别向←↓↑→移动光标。按`i`进入插入模式开始键入文字。
   >
   > 按`Esc`从各种模式返回普通模式。
   >
   > 普通模式下键入`:`和命令并回车来执行Vi命令，例如`:q`在未修改文件的情况下退出、`:q!`放弃修改并退出、`:w`保存、`:wq`或`:x`保存并退出等。上述`:read`也是如此执行。
   >
   > 普通模式下键入`dd`来剪切当前行，`p`来粘贴。
   >
   > 更多Vi(m)技巧请您自行了解，不在此赘述。

2. 在其中写入如下内容：

   ```bash
   efibootmgr --disk /dev/sda --part 1 --create --label "Arch Linux" --loader /vmlinuz-linux --unicode 'root=PARTUUID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX rw initrd=\initramfs-linux.img' --verbose
   ```

3. 将`/dev/sda`和`1`替换为您的EFI系统分区（ESP）的磁盘和分区号。`--disk /dev/sda --part 1`对应的是笔者的ESP`/dev/sda1`。

4. 将`root=`之后的PARTUUID的参数替换成Linux文件系统分区（即`/dev/sda2`）的分区UUID。手动输入比较麻烦，这里笔者使用的Vi的`:read`命令，让Vi将`:read !`后面的文字当作命令执行，并将结果另起一行写在文件中。

   ```vim
   :read !lsblk -dno PARTUUID /dev/sda2
   ```

   > 注意，默认情况下Vi会[防止](https://vi.stackexchange.com/a/2163/22060)用户使用退格键删除`自动缩进`、`换行符`以及`进入插入模式时的位置`之前的字符的。要么临时`:set backspace=indent,eol,start`，要么直接在普通模式下用`J`（大写J）将当前行末尾的换行符删除。

5. 物理机安装时，还应配置[Microcode](https://wiki.archlinux.org/index.php/Microcode)自动更新，请参见笔者的后续文章：[在笔记本电脑上安装Arch Linux的额外事项#Microcode](install-arch-on-laptop.md#Microcode)。

6. 检查脚本中内容：

   ```bash
   cat /home/efibootmgr.sh
   fdisk -l
   lsblk -dno PARTUUID /dev/sda2
   ```

7. 确认无误后赋予该脚本可执行权限并运行：

   ```bash
   chmod u+x /home/efibootmgr.sh
   /home/efibootmgr.sh
   ```

8. 再次检查启动项是否配置无误：

   ```bash
   efibootmgr --verbose
   ```

9. 设置启动顺序（可选）：

   > 笔者运行完脚本后，efibootmgr已经自动将`Arch Linux`项设置为优先启动了，故此步骤可以跳过。

   ```bash
   efibootmgr --bootorder XXXX,XXXX --verbose
   ```

   其中XXXX即efibootmgr的输出中各启动项前的四位数。

   > 笔者在笔电上安装时会将U盘设为第一项，系统设为第二项。关机之后移除U盘便可从硬盘系统启动。反之如果无法启动或缺少驱动，下次启动前插入U盘便能进行调整。若如此配置，记得在系统运行稳定后将系统驱动调回第一项。

## 重启

1. 用组合键`ctrl+d`或者命令退出chroot环境：

   ```bash
   exit
   ```

2. 卸载文件系统：

   ```bash
   umount -R /mnt
   ```

   如果文件系统正处于使用中，可以用[fuser(1)](https://jlk.fjfi.cvut.cz/arch/manpages/man/fuser.1)找到。

3. 关机：

   ```bash
   shutdown now
   ```

   所有仍然挂载的文件系统会被*systemd*自动卸载。移除安装介质并且用设置好的密码登录root账户。

   然后在虚拟机设置中将SCSI控制器里的DVD驱动器中的安装镜像关掉。

   ![remove-install-disk](../attachments/remove-install-disk.png)

4. 再次启动后，终于进入了Arch Linux。

   ![welcome-to-arch-linux](../attachments/welcome-to-arch-linux.png)

## 安装完成后的工作

至此，我们便可以在Hyper-V虚拟机中运行基本的Arch Linux系统，以及进行网络连接。

您可以进一步参考官方[推荐的安装完成后的操作](https://wiki.archlinux.org/index.php/General_recommendations)。包括但不限于：

- 创建用户账户、配置`sudo`
- 安装网卡、蓝牙、显卡等的闭源驱动
- 配置无线网络
- 安装桌面环境

笔者将会在[之后的文章](install-arch-on-laptop.md)中继续描述在笔电上安装的额外步骤，包括启用高通无线网卡连接网络、安装NVIDIA显卡驱动和KDE桌面环境、连接蓝牙鼠标等后续步骤。
