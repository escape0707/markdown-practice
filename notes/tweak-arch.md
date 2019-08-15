---
tags: [Blog/System Configurations]
title: 初装Arch Linux后的软件和设置建议
---

> 在前文[在笔记本电脑上安装Arch Linux的额外注意事项](install-arch-on-laptop.md)中，笔者已经描述了如何在笔记本上安装运行KDE桌面环境的Arch Linux系统。在满足了基本的Google搜索需求后，还有很多可以提升使用体验的空间。

本文是接续前文的Linux自用安装记录后篇，旨在记录笔者完善自己的Linux系统使用体验的过程中，安装并使用了哪些工具、修改了哪些配置。作为备忘之用。

欢迎各位读者分享自己的技巧和建议！

- [pacman技巧](#pacman技巧)
- [Git与AUR与pacman wrapper](#git与aur与pacman-wrapper)
- [配置Swap](#配置swap)
- [安装蓝牙驱动并启用](#安装蓝牙驱动并启用)
- [输入法](#输入法)
- [Readline](#readline)
- [结语](#结语)

## pacman技巧

### 更新镜像服务器并按速度排序

之前提到过可以用`reflector`来[更新并排序镜像服务器列表](https://wiki.archlinux.org/index.php/Mirrors#Sorting_mirrors)：

```bash
pacman -S reflector
reflector --country China --protocol https --sort rate --save /etc/pacman.d/mirrorlist
```

您也可以用下面的方法：

```bash
sudo pacman -S pacman-contrib
curl -s "https://www.archlinux.org/mirrorlist/?country=CN&protocol=https&ip_version=4&use_mirror_status=on" | sed -e 's/^#Server/Server/' -e '/^#/d' | rankmirrors - > ~/mirrorlist
cat ~/mirrorlist
```

> `curl`来下载，`-s`启用静默模式。
>
> 这里`|`是管道符号，表示其左右的程序并发运行的情况下，左边程序输出到右边程序的输入。
>
> `sed`是命令行文本编辑工具，`-e`用来接续一个脚本命令，`s/.../.../`即类似Vi中的替换命令，`/.../d`即删除包含该串的文本行。搜索串是正则表达式，其中的`^`表示串首（这里即行首）。
>
> 最后的那个单独的`-`是“将被输入的内容，用作命令所需的文件”。
>
> `~`表示“用户文件夹”。

确认列表无误后进行替换：

```bash
sudo mv ~/mirrorlist /etc/pacman.d/mirrorlist
```

### [更新系统](https://wiki.archlinux.org/index.php/Pacman#Upgrading_packages)

```bash
sudo pacman -Syu
```

### 定期[清理旧安装包](https://wiki.archlinux.org/index.php/Pacman#Cleaning_the_package_cache)

```bash
sudo systemctl enable paccache.timer
```

> 此命令需要`pacman-contrib`包：

### 列出所有直接安装的包

以下命令计算所有直接安装的包中不属于base、base-devel和fcitx-im组的部分，建议写成脚本放在`/home/listpkg.bash`以供随时运行：

```bash
comm -23 <(pacman -Qqe | sort) <((for i in $(pacman -Qqg base base-devel fcitx-im); do pactree -ul "$i"; done) | sort -u)
```

> 其中`comm`比较两个文件，左边列出第一个文件独有的内容，中间列出第二个文件独有的内容，右边列出两个文件共有的内容。可用`-`+`1`、`2`或/和`3`来不输出对应列。此处`-23`即只列出第一个文件中独有的内容。
>
> `<()`运算符表示将括号内的表达式的输出当作输入命令中所需要的一个文件。
>
> `$()`表示用括号内的表达式的运行结果来替换所输入命令中`$()`这一部分，`$i`也是同理。
>
> `sort`来排序，`-u`来只输出相同行中的第一个。
>
> `fcitx-im`是之后我们需要安装的输入法的软件包组，见后文。

## Git与AUR与pacman wrapper

有两类软件可以作为pacman的封装，一类如[`yay`](https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers)可以作为[AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository)软件的安装管理助手，另一类如[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)可以多线程、[`rsync`](https://wiki.archlinux.org/index.php/Rsync)差量下载。

而`yay`和`powerpill`均为非Arch官方的AUR软件包，所以我们从`yay`的编译安装开始，先介绍AUR软件的手动编译安装，然后介绍用`yay`安装AUR软件，最后调优`powerpill`。

[安装AUR软件包](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages)之前需要先用Git下载。所以先配置好Git，即方便AUR软件安装，也方便日后开发流程。

### Git

用如下命令安装并配置[`Git`](https://wiki.archlinux.org/index.php/Git)：

> 将`YourUsername`和`your@email.com`替换为您的名字和邮箱地址。此信息之后将被用于的commit message中，作为开发历史的资料和GitHub等托管网站连接commit与账户的依据。
>
> 将代理服务器的地址设置为之前开启的Shadowsocks本地端的地址。Git for Windows下使用`socks5`协议貌似会不被`ServicePointManager`支持而需要反复输入账户密码，届时请用`http`协议。

```bash
sudo pacman -S git bash-completion
git config --global user.name "Your Username"
git config --global user.email your@email.com
git config --global http.proxy socks5://127.0.0.1:1080
git config --global credential.helper store
```

> 其中`bash-completion`包提供`bash`中各种命令的补全，包括`git`命令中补全分支名称、`pacman`命令中补全包名称等，十分便利。
>
> 配置`credential.helper`令您在和远程服务器同步时不需要反复输入用户名和密码。笔者为了方便设置为[`store`](https://git-scm.com/docs/git-credential-store)来明文保存到本地，但为了安全您还可以选择[`cache`](https://git-scm.com/docs/git-credential-cache)。

### Yay

通过手动安装AUR软件包的方式安装yay，首先需要安装`base-devel`软件包组：

```bash
sudo pacman -S base-devel --needed
```

> 用`--needed`来跳过已安装软件包的重新安装。

之后clone yay的build文件：

```bash
git clone https://aur.archlinux.org/yay.git
cd yay
less PKGBUILD
```

检查PKGBUILD文件无误后生成软件包并安装：

```bash
makepkg -si
```

之后我们可以像用`pacman`一样使用`yay`来更新或安装官方包和AUR包，例如[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)。但是不要和`sudo`一起用，`yay`会在需要时自行提权：

```bash
yay -Syu
yay -S powerpill reflector rsync --needed
```

为`/etc/pacman.conf`中已启用的Repo添加`SigLevel = PackageRequired`，参见[Wiki#Troubleshooting](https://wiki.archlinux.org/index.php/Powerpill#Troubleshooting)

此时即可使用`powerpill`，精调请参考[`powerpill.json(1)`](https://xyne.archlinux.ca/projects/powerpill/#powerpill.json1)手册页。

## 配置Swap

[`Swap`](https://wiki.archlinux.org/index.php/Swap)即虚拟内存，在物理内存占用高时起作用。

之前没有分配固定的Swap分区，在此可以设置[Swap文件](https://wiki.archlinux.org/index.php/Swap#Automated)

```bash
sudo pacman -S systemd-swap
sudo -e /etc/systemd/swap.conf
```

设置`swapfc_enabled=1`，可用`/`+搜索字串+回车定位。也可用`sed`替换：

```bash
sudo sed --in-place s/swapfc_enabled=0/swapfc_enabled=1/ /etc/systemd/swap.conf
```

> 其中`--in-place`表示直接修改指定文件并保存。

笔者的物理内存为4G，不使用休眠模式则设置`swapfc_chunk_size=2G`。

之后启用服务：

```bash
sudo systemctl enable systemd-swap
```

## 安装蓝牙驱动并启用

笔者的笔电采用高通无线/蓝牙芯片，需要[自行安装蓝牙驱动](https://askubuntu.com/questions/632336/bluetooth-broadcom-43142-isnt-working)。

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

Linux下有许多中文输入法，笔者使用包含于官方源中的[`Fcitx`](https://wiki.archlinux.org/index.php/Fcitx)。

安装Fcitx框架、百度/谷歌云拼音插件、KDE设置插件：

```bash
sudo pacman -S fcitx-im fcitx-cloudpinyin kcm-fcitx
```

为了能[`在wayland中使用Fcitx`](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87)#Gnome_On_Wayland_%E7%94%A8%E6%88%B7%E6%97%A0%E6%B3%95%E4%BD%BF%E7%94%A8_fcitx)，修改如下文件：

```bash
sudo cat >> /etc/environment
GTK_IM_MODULE=fcitx
QT_IM_MODULE=fcitx
XMODIFIERS=@im=fcitx
# ctrl+d
```

安装后在KDE设置插件中，取消勾选“仅显示当前语言”并启用需要的输入法；自行开启云拼音插件，如果网络受限选择百度云拼音。KDE插件可以设置候选词、皮肤、全窗口统一语言等；具体设置参见官方指导。

## Readline

[`Readline`](https://wiki.archlinux.org/index.php/Readline)是用于bash等CLI的编辑、输入库，对其进行设置可以让CLI交互变得更方便：

### 搜索历史

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

## GoldenDict

笔者使用的离线词典软件GoldenDict：

```bash
sudo pacman -S goldendict
```

Linux上GoldenDict可以使用Scan Popup直接划词翻译，但是使用起来有时不如Windows上用[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/GoldenDict%20Select%20To%20Translate.ahk)实现的稳定。

Linux上的GoldenDict不像Windows上自带Morphology，需要自己[下载](https://sourceforge.net/projects/goldendict/files/better%20morphologies/1.0/)。

## Visual Studio Code

笔者安装的实微软专有版VSCode：

```bash
yay -S visual-studio-code-bin
```

当使用Vim扩展插件时，为了不用经常去够`Esc`和`Ctrl`，Linux下可以使用[`caps2esc`](https://aur.archlinux.org/packages/interception-caps2esc)，Windows下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)或者[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/Use%20CapsLock%20in%20Vim.ahk)。

## 结语

此三篇文章便是笔者初装Arch Linux的记录。小总结：

安装所有包：

```bash
pacstrap /mnt base base-devel bash-completion broadcom-wl-dkms dolphin-plugins fcitx-cloudpinyin fcitx-im firefox-developer-edition git intel-ucode kcm-fcitx kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite linux-headers noto-fonts-cjk pacman-contrib plasma-meta plasma-wayland-session powerpill rsync shadowsocks-qt5 systemd-swap visual-studio-code-bin xf86-video-intel xf86-video-nouveau yay
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
