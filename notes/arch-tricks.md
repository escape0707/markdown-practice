---
tags: [Blog/System Configurations]
title: 初装Arch Linux后的软件和设置建议
---

> 在前文[在虚拟机/物理机中安装Arch Linux](install-arch-on-laptop-and-vm.md)中，俺已经描述了如何在笔记本上安装运行KDE桌面环境的Arch Linux系统。在满足了基本的Google搜索需求后，还有很多可以提升使用体验的空间。

本文是接续前文的Linux自用安装记录后篇，旨在记录俺完善自己的Linux系统使用体验的过程中，安装并使用了哪些工具、修改了哪些配置。作为备忘之用。

欢迎各位读者分享自己的技巧和建议！

- [bash-completion](#bash-completion)
- [Readline](#readline)
- [pacman技巧](#pacman技巧)
- [Git与AUR与pacman wrapper](#git与aur与pacman-wrapper)
- [将CapsLock映射到Esc和Ctrl](#将capslock映射到esc和ctrl)
- [配置Swap](#配置swap)
- [安装蓝牙驱动并启用](#安装蓝牙驱动并启用)
- [输入法](#输入法)
- [GoldenDict](#goldendict)
- [Visual Studio Code](#visual-studio-code)
- [结语 // Todo](#结语--todo)

## bash-completion

`bash-completion`包提供`bash`中各种命令的补全，包括`git`命令中补全分支名称、`pacman`命令中补全包名称等，十分便利。

```bash
sudo pacman -S bash-completion
```

## Readline

[`Readline`](https://wiki.archlinux.org/index.php/Readline)是用于bash等CLI的编辑、输入库，对其进行设置可以让CLI交互变得更方便：

Readline中按上下箭头默认会在历史输入中选择，修改`inputrc`可以使其只匹配当前已输入内容进行搜索：

```bash
sudo cat >> /etc/inputrc
"\e[A": history-search-backward
"\e[B": history-search-forward
# ctrl+d
```

> 删除光标前的内容：`ctrl+u`
>
> 删除光标后的内容：`ctrl+k`
>
> 援引前一条命令的内容用`!!`，如`sudo !!`来用`root`权限执行前一条命令。

## pacman技巧

### [更新系统](https://wiki.archlinux.org/index.php/Pacman#Upgrading_packages)

```bash
sudo pacman -Syu
```

### 定期[清理旧安装包](https://wiki.archlinux.org/index.php/Pacman#Cleaning_the_package_cache)

```bash
sudo pacman -S pacman-contrib
sudo systemctl enable paccache.timer
```

### 列出所有非依赖的包

以下命令计算所有安装的包中不被任何包显式依赖的包，可以用来在重装时参考最少需要安装哪些包，建议写成脚本放在`/home/lstpkg.sh`以供随时运行：

```bash
installed_groups="base-devel xfce4 xfce4-goodies"
append_apps="git sudo xfce4-windowck-plugin"
(comm -23 <(pacman -Qqtt) <(pacman -Qqg $installed_groups | sort); printf %"s\n" $append_apps $installed_groups) | sort
```

> 其中`comm`比较两个文件，左边列出第一个文件独有的内容，中间列出第二个文件独有的内容，右边列出两个文件共有的内容。可用`-`配合`1`、`2`或/和`3`来隐藏对应列的输出。此处`-23`即只列出第一个文件中独有的内容。
>
> `<()`运算符表示将括号内的表达式的输出当作输入命令中所需要的一个文件。
>
> `installed_groups`应该填写我们手动安装的软件包组
>
> `append_apps`应填写一些意外被排除的软件包。例如，`yay-bin`依赖于`git`和`sudo`，但是`yay-bin`输入AUR，其安装本身就会用到后两者，所以还是难免显式安装。又或者`xfce4-windowck-plugin`错误的将其加入了`xfce4`组，只好显式地追加上它。

## Git与AUR与pacman wrapper

有两类软件可以作为pacman的封装，一类如[`yay`](https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers)可以作为[AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository)软件的安装管理助手，另一类如[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)可以多线程、[`rsync`](https://wiki.archlinux.org/index.php/Rsync)差量下载。

但是`yay`和`powerpill`均为非Arch官方的AUR软件包，而且如果不使用Go语言，就不必自己编译`yay`。所以我们从`yay-bin`的安装开始，先介绍AUR软件的手动安装，然后介绍用`yay`安装AUR软件，最后调优`powerpill`。

[安装AUR软件包](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages)之前需要先用Git下载。所以先配置好Git，即方便AUR软件安装，也方便日后开发流程。

### Git

安装并配置[`Git`](https://wiki.archlinux.org/index.php/Git)：

> 将`YourUsername`和`your@email.com`替换为您的名字和邮箱地址。此信息之后将被用于的commit message中，作为开发历史的资料和GitHub等托管网站连接commit与账户的依据。
>
> 将代理服务器的地址设置为之前开启的Shadowsocks本地端的地址。Git for Windows下使用`socks5`协议貌似会不被`ServicePointManager`支持而需要反复输入账户密码，而Linux下似乎又不能clone gist，届时请用之前通过`privoxy`转化得到的`http`协议。

```bash
sudo pacman -S git
git config --global user.name "Your Username"
git config --global user.email your@email.com
git config --global http.proxy socks5://127.0.0.1:1080
git config --global credential.helper store
git config --global merge.ff only
git config --global pull.ff only
```

> 配置`credential.helper`后在和远程服务器同步时不需要反复输入用户名和密码。俺为了方便设置为[`store`](https://git-scm.com/docs/git-credential-store)来明文保存到本地，但为了安全也可以选择[`cache`](https://git-scm.com/docs/git-credential-cache)。

### Yay

通过手动安装AUR软件包的方式安装`yay-bin`，首先需要安装`base-devel`软件包组：

```bash
sudo pacman -S base-devel --needed
```

> 用`--needed`来跳过已安装软件包的重新安装。

之后clone yay的build文件：

```bash
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
less PKGBUILD
```

检查PKGBUILD文件无误后生成软件包并安装：

```bash
makepkg -si
```

### Powerpill

之后我们可以像用`pacman`一样使用`yay`来更新或安装官方包和AUR包，例如[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)。但是不要和`sudo`一起用，`yay`会在需要时自行申请提权：

```bash
yay -Syu
yay -S powerpill reflector rsync --needed
```

将`/etc/pacman.conf`的默认`SigLevel`改为`PackageRequired`，参见[Wiki#Troubleshooting](https://wiki.archlinux.org/index.php/Powerpill#Troubleshooting)

此后即可使用`powerpill`，精调请参考[`powerpill.json(1)`](https://xyne.archlinux.ca/projects/powerpill/#powerpill.json1)手册页，主要调整其中`aria2c`参数部分。// Todo

## 将CapsLock映射到Esc和Ctrl

当使用Vi时，为了不用经常去够`Esc`和`Ctrl`，Linux下可以使用[`caps2esc`](https://aur.archlinux.org/packages/interception-caps2esc)，Windows下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)或者[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/Use%20CapsLock%20in%20Vim.ahk)。

```bash
sudo yay caps2esc
```

## 配置Swap

[`Swap`](https://wiki.archlinux.org/index.php/Swap)即虚拟内存，在物理内存占用高时起作用。

之前分配了固定的Swap分区可以跳过这一部分。

如果没有，在此可以设置[Swap文件](https://wiki.archlinux.org/index.php/Swap#Automated)

```bash
sudo pacman -S systemd-swap
sudo -e /etc/systemd/swap.conf
```

设置`swapfc_enabled=1`，可用`/`+搜索字串+回车定位。也可用`sed`替换：

```bash
sudo sed --in-place s/swapfc_enabled=0/swapfc_enabled=1/ /etc/systemd/swap.conf
```

> 其中`--in-place`表示直接修改指定文件并保存。

俺的物理内存为4G，不使用休眠模式则设置`swapfc_chunk_size=2G`。

之后启用服务：

```bash
sudo systemctl enable systemd-swap
```

## 安装蓝牙驱动并启用

俺的笔电采用高通无线/蓝牙芯片，需要[自行安装蓝牙驱动](https://askubuntu.com/questions/632336/bluetooth-broadcom-43142-isnt-working)。

检查日志找到所需的蓝牙驱动型号：

```bash
dmesg | grep -i bluetooth
```

找到类似以下的内容：

```text
bluetooth hci0: Direct firmware load for brcm/BCM43142A0-105b-e065.hcd failed with error -2
```

> 如果命令的结果不是`BCM43142A0-105b-e065.hcd`，在下一步请根据您看到的名称下载并应用驱动。

下载并应用：

```bash
curl -O https://github.com/winterheart/broadcom-bt-firmware/raw/master/brcm/BCM43142A0-105b-e065.hcd
sudo mv BCM43142A0-105b-e065.hcd /lib/firmware/brcm/
```

重新载入并检查是否正常：

```bash
sudo modprobe -r btusb
sudo modprobe btusb
dmesg | grep -i bluetooth
```

启用蓝牙

```bash
sudo systemctl enable bluetooth
```

如果在重启后蓝牙功能不自动打开，参考[蓝牙#启动后自动开启](https://wiki.archlinux.org/index.php/Bluetooth#Auto_power-on_after_boot)。

## 输入法

Linux下有许多中文输入法，俺使用包含于官方源中的[`Fcitx`](https://wiki.archlinux.org/index.php/Fcitx)。

安装Fcitx、谷歌/百度云拼音插件、Fcitx配置工具（KDE用户则用`kcm-fcitx`)：

```bash
sudo pacman -S fcitx-im fcitx-cloudpinyin fcitx-configtool
```

为了GTK/Qt程序能使用输入法，设置环境变量：

```bash
printf 'GTK_IM_MODULE=fcitx\nQT_IM_MODULE=fcitx\nXMODIFIERS=@im=fcitx' | sudo tee -a /etc/environment > /dev/null
# ctrl+d
```

安装后在设置插件中，取消勾选“仅显示当前语言”并启用需要的输入法；开启云拼音插件，如果网络受限选择百度云拼音。插件可以设置候选词、皮肤、全窗口统一语言等；具体设置参见官方指导。 // Todo

## GoldenDict

俺使用的离线词典软件GoldenDict：

```bash
sudo pacman -S goldendict
```

Linux上GoldenDict可以使用Scan Popup直接划词翻译，但是使用起来有时不如Windows上用[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/GoldenDict%20Select%20To%20Translate.ahk)实现的稳定。

Linux上的GoldenDict不像Windows上自带Morphology，需要自己[下载](https://sourceforge.net/projects/goldendict/files/better%20morphologies/1.0/)。

## Visual Studio Code

俺安装的是开源编译版VSCode：

```bash
yay -S code
```

当使用C/C++插件format文件时，可能会提示无法找到`libtinfi.so.5`，这是因为Arch Linux自带的是`libtinfi.so.6`，我们可以检查一下： // Todo

```bash
ls /lib/ | grep libtinfo.
```

而且第六版向下兼容第五版，所以我们只需要建立一个symlink即可：

```bash
sudo link /lib/libtinfo.so.6 /lib/libtinfo.so.5
```

## 结语 // Todo

此三篇文章便是俺初装Arch Linux的记录。小总结：

安装所有包：

```bash
pacstrap /mnt arch-wiki-docs base base-devel bash-completion broadcom-wl-dkms caps2esc code cppreference dolphin-plugins fcitx-im fcitx-cloudpinyin firefox git goldendict intel-ucode kcm-fcitx kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite linux-headers man-db neovim noto-fonts-cjk openssh pacman-contrib plasma-meta plasma-wayland-session powerpill reflector shadowsocks-libev systemd-swap telegram-desktop ttf-dejavu v2ray xf86-video-intel xf86-video-nouveau yay-bin
```

chroot后创建用户账户，设置两个账户的密码，设置`sudoers`：

```bash
EDITOR=vi visudo
```

启用所有服务：

```bash
systemctl enable bluetooth NetworkManager pacache.timer sddm systemd-swap
```

设置Swap：

```bash
vi /etc/systemd/swap.conf
```

最后用`efibootmgr`制作好启动项就可以启动了。
