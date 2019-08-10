---
tags: [Blog/System Configurations]
title: 在笔记本电脑上安装Arch Linux的额外注意事项
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [实机安装时的不同点](#实机安装时的不同点)
- [创建账户与配置sudo](#创建账户与配置sudo)
- [显卡驱动与图形化界面](#显卡驱动与图形化界面)
- [网页浏览与科学上网](#网页浏览与科学上网)
- [更进一步的自定义](#更进一步的自定义)

## 前言

> 本文是接续前文[在Hyper-V虚拟机上安装Arch Linux](install-arch-on-vm.md)的一篇自用安装记录，旨在记录笔者在笔电Thinkpad E430c上实际安装Arch Linux的过程与虚拟机安装的区别，以及一些通用的[安装后配置](https://wiki.archlinux.org/index.php/General_recommendations)过程。
>
> 笔者初学Linux，经验不足。如有纰漏，望各位读者不吝赐教！

之前笔者已经在虚拟机上演练过Arch Linux的安装过程了。但是在物理机上安装时，可能会遇到额外的闭源驱动、无线网络连接、蓝牙鼠标连接等问题。而且日常使用中，一般还需要安装一套桌面环境。

## 实机安装时的不同点

首先，按照前文[在Hyper-V虚拟机上安装Arch Linux](install-arch-on-vm.md)的步骤操作。当遇到网络连接相关步骤或Microcode时，按照下文操作。

### 连接到互联网

在前文的[连接到互联网](install-arch-on-vm.md#连接到互联网)这一步，如需[使用Wi-Fi](https://wiki.archlinux.org/index.php/Netctl#Configuration)，可以使用图形化工具：

```bash
wifi-menu -o
```

一般来说，ArchISO中已经包含了各种网卡的驱动，其中就有笔者所需的高通专有驱动`broadcom-wl`。但如果工具仍然报错，请根据[无线网络配置](https://wiki.archlinux.org/index.php/Wireless_network_configuration)检查常见问题。

根据向导选择网络，输入密码之后，工具便会在`/etc/netctl`创建一个连接配置文件，接下来启动它：

> 用您指定的配置文件名称替换下文的`profile`

```bash
netctl start profile
```

如此便可连接上Wi-Fi网络。

### 网络配置

在前文的[网络配置](install-arch-on-vm.md#网络配置)这一步，如此时不希望安装KDE等[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)，但仍需在新系统中继续使用Wi-Fi，则需要安装上一步使用的工具：

```bash
pacman -S dialog wpa_supplicant
```

> 以上两个软件包均不在base软件包组中。如果此时忘记安装，在新系统中便无`wifi-menu`等可用。不过届时可以再次从安装介质启动完成这一步骤。
>
> 如果希望安装KDE等桌面环境，那么则推荐您直接使用KDE中的图形化网络连接配置模块`NetworkManager`。此包会在安装KDE桌面环境时作为依赖自动安装。请您参见Todo

此外，笔者的笔电使用的无线网卡芯片为高通的BCM43142，不被高通官方开源网卡驱动支持。而官方的专有驱动[`wl`](https://wiki.archlinux.org/index.php/Broadcom_wireless#broadcom-wl)则似乎由于知识产权问题，不能由其他组织分发，所以没能被集成在Linux操作系统内核中。必须由用户自行安装。此外，该驱动有一种可以支持dkms技术、随Linux系统内核更新自动适配的版本，所以这里笔者安装网卡驱动的dkms版：

```bash
pacman -S linux-headers broadcom-wl-dkms
```

### Microcode

在前文的[Boot loader](install-arch-on-vm.md#Boot-loader)这一步，应该根据处理器品牌，配置早期微码更新。Intel处理器的操作如下，Amd处理器需要将`intel`替换为`amd`：

1. 安装`intel-ucode`包（在chroot下进行）：

    ```bash
    pacman -S intel-ucode
    ```

2. 将脚本`/home/efibootmgr.sh`中的`initrd=\initramfs-linux.img`前加一项`initrd=\intel-ucode.img`（用空格隔开）。

> 笔者的虚拟机运行在Intel CPU的物理机上，在检查过虚拟机系统的CPU型号并确认也是Intel类型的某种虚拟CPU之后，同样安装了intel-ucode但也没有发现不装有何问题。

### 启动顺序

笔者笔电的主板驱动之前有些问题，不能在开机时按键进入主板设置，调整启动顺序。如若系统启动项创建有误，或未安装网络相关软件包，需从安装介质重新启动时，只能通过移除硬盘来绕过系统从U盘启动，极为不便。

可以通过将U盘设为第一启动项、系统设为第二启动项来暂时解决。

```bash
efibootmgr
efibootmgr --bootorder XXXX,XXXX --verbose
```

需要启动到系统时，关机并移除安装介质再启动；当系统不能正常启动时，关机并重新插入安装介质再启动。

确认启动正常后，再重启到主板设置调回：

```bash
systemctl reboot --firmware-setup
```

> 如您不需要KDE桌面环境的图形化网络管理，可以在启动到新系统后使用命令行工具开启并设置自动Wi-Fi连接：

```bash
wifi-menu -o
netctl start XXXX
netctl enable XXXX
```

## 创建账户与配置sudo

在完成了最小化Arch Linux安装之后，为了安全使用系统，还需要创建属于自己的个人账户，安装并配置好sudo。

### 用户和用户组

新安装的系统只有`root`一个超级用户。一直使用`root`账户并不安全，可能会意外地执行或修改了文件，以及给予第三方程序过高权限带来风险。故平时应当使用普通权限的账户。（KDE桌面环境也需要您创建非`root`账户才能登录。）

[新建](https://wiki.archlinux.org/index.php/Users_and_groups#Example_adding_a_user)一个用户：

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

由于笔者只是单人、单账户并作为客户端使用机器，仅用`sudo`防手滑和防其他进程滥用权限。故在此不做[进阶的设置](https://wiki.archlinux.org/index.php/Sudo#Example_entries)，仅将之前新建的用户添加到`sudoers`中去：

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

您可以自己选择喜爱的[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。笔者已经体验过Ubuntu和GNOME，故此次尝试比较接近Windows桌面体验的[`KDE`](https://wiki.archlinux.org/index.php/KDE)桌面环境。

安装KDE、[`wayland`](https://wiki.archlinux.org/index.php/KDE#KDE_applications)后端支持、以及包括文件管理、终端、记事本等应用，并且启用[SDDM](https://wiki.archlinux.org/index.php/SDDM)显示管理器：

```bash
sudo pacman -S plasma-meta plasma-wayland-session dolphin-plugins kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite
sudo systemctl enable sddm
```

## 网页浏览与科学上网

要说为什么需要一个桌面环境，很大程度上不是因为运行软件或者修改设置比GUI更方便或者游戏之类的，而是为了能够至少能够使用[各种浏览器](https://wiki.archlinux.org/index.php/Web_browser)浏览网页……这里笔者以不需要代理就可以正常使用全部功能的Firefox为例，您也可以选择Chrome等：

```bash
sudo pacman -S firefox
```

为了能够使用Google搜索引擎，还需要[Shadowsocks](https://wiki.archlinux.org/index.php/Shadowsocks)等科学上网工具，图方便可以用`shadowsocks-qt5`，但是其是否还在活跃更新笔者现在也不清楚：

```bash
sudo pacman -S shadowsocks-qt5
```

之后在启动的图形化界面中创建您的服务器条目，或者导入类似Windows版的`gui-config.json`的服务器配置列表文件。设置开机最小化自启动并编辑某条服务器将其设置为启动后自动连接。

或者

如果您想使用命令行版本的shadowsocks、shadowsocks-libev，那么需要安装、创建服务器配置文件、然后启动并启用：

> 用您喜欢的名字替换下文的`config`

```bash
sudo pacman -S shadowsocks-libev
vi /etc/shadowsocks/config.json
# 将您的某个shadowsocks服务器信息写入到以上文件中
sudo systemctl start shadowsocks-libev@config.json
sudo systemctl enable shadowsocks-libev@config.json
```

安装并运行了本地Shadowsocks客户端后，在Firefox/Chrome中安装[Proxy SwitchyOmega](https://addons.mozilla.org/en-US/firefox/addon/switchyomega/)，打开设置，跳过所有教程，设置proxy中的服务器地址端口为`socks5`、地址为`127.0.0.1`，端口为`1080`。设置auto switch中的规则列表URL为：

```URL
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

点击下载，并确认符合规则的走proxy，默认走Default。关于科学上网和Proxy SwitchyOmega的设置在此不再详述。

## 更进一步的自定义

至此，我们已经可以在物理机上用Google搜索资料并且顺利查看了！大成功！有了自主搜索资料、下载文件的方法后，很容易查找接下来想做的任何事情如何完成。放下心来在Bilibili上看个番剧之类的，然后再去思考还缺少什么吧！

现在开始，您可以参考Arch Wiki上的应用列表安装感兴趣的应用，或者学习一下系统维护等。妥善利用Google、StackOverflow等网站的资源，进入Linux和开源系统的世界。

笔者之后也许会另起一文，记录笔者在Arch - Nouveau - SDDM - Plasma - Shadowsocks-qt5 - Firefox等齐备之后，安装的其他或许有用的软件和进行的调整，比如：

- pacman镜像服务器测速排序
- pacman wrapper 加速下载与AUR安装
- 高通蓝牙驱动
- 启用Swap（虚拟内存）
- PRIME双显卡

但是上述内容笔者也只是做到了搜索并阅读了资料，并且可以调配出来的程度。对于各种方案的比较和微调并不熟悉，所以仅供参考啦。