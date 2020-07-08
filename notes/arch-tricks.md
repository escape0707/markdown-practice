---
tags: [Blog/System Configurations]
title: 初装Arch Linux后的软件和设置建议
---

> 在前文[在虚拟机/物理机中安装Arch Linux](install-arch-on-laptop-and-vm.md)中，俺已经描述了如何在笔记本上安装运行KDE桌面环境的Arch Linux系统。在满足了基本的Google搜索需求后，还有很多可以提升使用体验的空间。

本文是接续前文的Linux自用安装记录后篇，旨在记录俺完善自己的Linux系统使用体验的过程中，安装并使用了哪些工具、修改了哪些配置。作为备忘之用。

> 欢迎各位分享自己的技巧和建议！

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
vi ~/.inputrc
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
echo Native:
(comm -23 <(pacman -Qnqtt) <(pacman -Qgq $installed_groups | sort); printf %"s\n" $installed_groups) | sort
echo
echo Foreign:
pacman -Qm
```

> 其中`comm`比较两个文件，左边列出第一个文件独有的内容，中间列出第二个文件独有的内容，右边列出两个文件共有的内容。可用`-`配合`1`、`2`或/和`3`来隐藏对应列的输出。此处`-23`即只列出第一个文件中独有的内容。
>
> `<()`运算符表示将括号内的表达式的输出当作输入命令中所需要的一个文件。
>
> `installed_groups`应该填写我们手动安装的软件包组

## Arch Linux CN

Arch Linux CN的[主页](https://www.archlinuxcn.org/)、[群组](https://www.archlinuxcn.org/archlinuxcn-group-mailling-list/)、[Telegram](https://t.me/archlinuxcn_group)。

> 进了里面去个个都是人才，说话又好听，哎哟超喜欢在里面！百万大佬，在线聊骚（指年薪、迫真），还不快快行动起来！
>
> 咳咳，这里主要讲一下用ArchLinuxCN源有什么用，以及如何使用。之前在[使用powerpill缓存软件包](install-arch-on-laptop-and-vm.md#使用powerpill缓存软件包)的环节已经讲过了一些。但是与`/etc/pacman.d/mirrorlist`不同的是，`pacstrap`不会自动拷贝`/etc/pacman.conf`。不过无妨，反正我们还要添加`archlinuxcn-mirrolist`，这里正好重新讲一遍。

ArchLinuxCN源是国内ArchLinux爱好者自行打包并维护的软件源，ArchLinux官网[可考](https://wiki.archlinux.org/index.php/Unofficial_user_repositories#archlinuxcn)。内有`powerpill`、`yay`、`aria2-fast`、`fcitx5`等许多AUR包的编译打包版。社群里有ArchLinux的开发者、TU、`fcitx5`的开发者等。知名度和可信度很高，大家也可以去混个脸熟，[问些好问题](https://github.com/ryanhanwu/How-To-Ask-Questions-The-Smart-Way/blob/master/README-zh_CN.md)之类的。

[添加ArchLinuxCN源](https://www.archlinuxcn.org/archlinux-cn-repo-and-mirror/)：

```bash
sudo -e /etc/pacman.conf
# 文件尾部追加：
[archlinuxcn]
Server = https://repo.archlinuxcn.org/$arch
```

导入GPG key，由ArchLinux Trusted User [farseerfc](https://www.archlinux.org/people/trusted-users/#farseerfc)签名，故依赖ArchLinux自带的`archlinux-keyring`即可安装：

```bash
sudo pacman -Sy archlinuxcn-keyring
```

安装`archlinuxcn-mirrorlist-git`包以获得镜像列表，并在`/etc/pacman.conf`引用：

```bash
sudo pacman -S archlinuxcn-mirrorlist-git
sudo -e /etc/pacman.d/archlinuxcn-mirrorlist
# 去掉想要的镜像服务器前的注释

sudo -e /etc/pacman.conf
# 修改之前添加的内容为：
[archlinuxcn]
Include = /etc/pacman.d/archlinuxcn-mirrorlist
```

之后便可享受ArchLinuxCN源的便利了。

## Git与AUR与pacman wrapper

有两类软件可以作为pacman的封装，一类如[`yay`](https://wiki.archlinux.org/index.php/AUR_helpers#Pacman_wrappers)可以作为[AUR](https://wiki.archlinux.org/index.php/Arch_User_Repository)软件的安装管理助手，另一类如[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)可以多线程、[`rsync`](https://wiki.archlinux.org/index.php/Rsync)差量下载。

`yay`和`powerpill`均为非Arch官方的AUR软件包。我们可以从ArchLinuxCN源按照通常方法直接安装，也可以从AUR编译安装。如不使用Go语言，可不必自己编译`yay`，用`yay-bin`即可。直接安装不再赘述，这里先介绍AUR软件的手动安装，然后介绍用`yay`安装AUR软件，最后调优`powerpill`。

[安装AUR软件包](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages)之前需要先用Git下载。所以先配置好Git，即方便AUR软件安装，也方便日后开发流程。

### Git

安装并配置[`Git`](https://wiki.archlinux.org/index.php/Git)：

> 将`YourUsername`和`your@email.com`替换为您的名字和邮箱地址。此信息之后将被用于的commit message中，作为开发历史的资料和GitHub等托管网站连接commit与账户的依据。
>
> 将代理服务器的地址设置为之前开启的Shadowsocks本地端的地址。Git for Windows下使用`socks5`协议貌似会不被`ServicePointManager`支持而需要反复输入账户密码，建议用`http`协议。~~而Linux下似乎又不能clone gist~~ **Linux下将代理协议成socks5h即可解析gist等DNS污染的域名。**

```bash
sudo pacman -S git
git config --global user.name escape0707
git config --global user.email tothesong@gmail.com
git config --global http.proxy socks5h://127.0.0.1:7890
git config --global credential.helper /usr/lib/git-core/git-credential-libsecret
git config --global merge.ff only
git config --global pull.ff only
```

> 配置`credential.helper`后在和远程服务器同步时不需要反复输入用户名和密码。~~俺为了方便设置为[`store`](https://git-scm.com/docs/git-credential-store)来明文保存到本地，但为了安全也可以选择[`cache`](https://git-scm.com/docs/git-credential-cache)。~~建议安装`libsecret`和`gnome-keyring`并用上述方法安全的启用`credential.helper`。Git for Windows用户貌似系统级默认`git config --system credential.helper manager`？没有的话自己设置到`--global`就好了。

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

之后我们可以像用`pacman`一样使用`yay`来更新或安装官方包和AUR包，例如CN源的不限单服务器连接数的[`aria2-fast`](https://aur.archlinux.org/packages/aria2-fast/)、[`powerpill`](https://wiki.archlinux.org/index.php/Powerpill)。但是不要和`sudo`一起用，`yay`会在需要时自行申请提权：

```bash
yay -S aria2-fast powerpill
```

将`/etc/pacman.conf`的默认`SigLevel`改为`PackageRequired`，参见[Wiki#Troubleshooting](https://wiki.archlinux.org/index.php/Powerpill#Troubleshooting)

此后即可使用`powerpill`，调优参考[`powerpill.json(1)`](https://xyne.archlinux.ca/projects/powerpill/#powerpill.json1)手册页：

```bash
sudo -e /etc/powerpill/powerpill.json
```

主要需要调整其中`aria2`参数部分（如果安装的原版`aria2`、单服务器最大连接数必须小于等于`16`）：

```json
"--max-concurrent-downloads=100",
"--max-connection-per-server=32",
"--min-split-size=1M",
```

## 将CapsLock映射到Esc和Ctrl

当使用Vi时，为了不用经常去够`Esc`和`Ctrl`，Linux下可以使用[`caps2esc`](https://aur.archlinux.org/packages/interception-caps2esc)，Windows下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)或者[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/Use%20CapsLock%20in%20Vim.ahk)。

```bash
yay caps2esc
sudo systemctl enable caps2esc --now
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

Linux下有许多中文输入法，俺使用包含于CN源中的`Fcitx5`。

安装fcitx5、GTK/Qt4/Qt5库、中文插件包：

```bash
sudo pacman -S fcitx5-git fcitx5-gtk-git fcitx5-qt4-git fcitx5-qt5-git fcitx5-chinese-addons-git
```

为了GTK/Qt程序能使用输入法，设置环境变量：

```bash
cat >> ~/.pam_environment
GTK_IM_MODULE=fcitx5
QT_IM_MODULE=fcitx5
XMODIFIERS=@im=fcitx5
```

启动fcitx5：

```bash
fcitx5
```

截止至2019/11/23，fcitx5还没有KDE以外的桌面环境的GUI配置插件。但是各种桌面环境都可以用通过安装`plasma-workspace`来用上KDE专属的配置插件（反正之后可以删掉）：

```bash
sudo pacman -S plasma-workspace kcm-fcitx5-git
```

在fcitx5启动的情况下，从开始菜单启动`Fcitx 5 Configuration`，或用`kcmshell5`启动配置插件：

```bash
kcmshell5 kcm_fcitx5
```

在配置插件中，取消勾选“仅显示当前语言”并启用需要的输入法如`pinyin`、`shuangpin`；开启云拼音插件，如果网络受限选择百度云拼音，或用`proxychains-ng`代理fcitx5；设置候选词数量、皮肤、模糊音等。具体设置俺会另外上传一个Git Repo，其中会有配置文件历史。//Todo

配置插件删除快捷键时，不知为何一定会留下至少一个。想要彻底删除，只好手动去`~.config/fcitx5/`中将对应配置文件中的键值的`0=XXXXXXX`改为`0=`。

最后，在自己桌面环境的自启动项里手动添加fcitx5即可。

## GoldenDict

俺使用的离线词典软件GoldenDict的CN源git版：

```bash
sudo pacman -S goldendict-qt5-git
```

此版本会同时安装官方版缺少的Morphology和拼写检查：`hunspell`。

> Linux上GoldenDict可以使用Scan Popup直接划词翻译，但是使用起来有时不如Windows上用[AutoHotKey脚本](https://github.com/escape0707/scripts/blob/master/GoldenDict%20Select%20To%20Translate.ahk)实现的稳定。

## Visual Studio Code

俺安装的是开源编译版VSCode：

```bash
sudo pacman -S code
```

当使用C/C++插件排版文件时，可能会提示无法找到`libtinfo.so.5`，这是因为Arch Linux自带的是`libtinfo.so.6`，而C/C++插件自带的`clang-format`版本过旧仍在使用`libtinfo.so.5`。在插件开发团队有时间整合新版`clang-format`之前，我们可以安装新版`clang`并设置让插件使用新版`clang-format`：

```bash
sudo pacman -S clang
```

之后在VSCode设置中加入：

```json
"C_Cpp.clang_format_path": "/usr/bin/clang-format"
```

## Proxychains

让单个命令运行在代理环境下：

```bash
sudo pacma -S proxychains-ng
# 修改/etc/proxychains.conf最后一行，替换127.0.0.1为本机本地地址，1080为本地代理的端口号：
sudo sed -i "$d" /etc/proxychains.conf
echo "socks5 127.0.0.1 1080" | sudo tee -a/etc/proxychains.conf
```

之后便可用其代理命令行工具了， 这里以`curl`为例测试一下：

```bash
proxychains curl ifconfig.me/ip
curl ifconfig.me/ip
```

通过返回的ip即可判断是否成功代理。

## Privoxy

socks5代理变http代理：

```bash
sudo pacman -S privoxy
sudo -e /etc/privoxy/config
# 结尾追加，替换1080为本地代理的端口号：
forward-socks5 / localhost:1080 .
```

启动服务：

```bash
sudo systemctl enable privoxy --now
```

之后便可用`http://localhost:8118`为http代理了。

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

## Fine-tune makepkg

/etc/makepkg.conf

CFLAGS="-march=native -O2 -pipe -fstack-protector-strong -fno-plt"
CXXFLAGS="${CFLAGS}"
MAKEFLAGS="-j4"
COMPRESSXZ=(xz -c -z - --threads=0)

> DLAGENTS for curl proxy, e.g.:
>
> `'https::/usr/bin/curl --socks5 127.0.0.1:1080 -gqb "" -fLC - --retry 3 --retry-delay 3 -o %o %u'`

## zeal & dash & devdocs

## xdg-user-dirs-gtk-update

## Xfce4快捷键设置

想实现类似Windows的Win+1、2、3的软件启动和切换，似乎`jumpapp-git`是最合适的：

```bash
yay jumpapp-git
```

之后设置快捷键例如，`Super+1`=`jumpapp firefox`、`Super+2`=`jumpapp code`等。

为了能够同时使用`Super`键打开`whiskermenu`，根据[Manjaro论坛建议](https://forum.manjaro.org/t/manjaro-xfce-18-0-beta-builds-testing/47347/75)，可以设置`Alt+F1`=`xfce4-popup-whiskermenu`，之后安装`xcape`：

```bash
sudo pacman -S xcape
```

并且设置自启动项`xcape -e 'Super_L=Alt_L|F1;Super_R=Alt_L|F1'`

## Xfce4最大化启动Terminal

参考[这个回答](https://unix.stackexchange.com/a/426934/352668)：

```bash
mkdir -p ~/.local/share/applications
sed "s/Exec=xfce4-terminal/Exec=xfce4-terminal --maximize" /usr/share/applications/xfce4-terminal.desktop > ~/.local/share/applications/xfce4-terminal.desktop
```

之后从开始菜单启动Xfce Terminal即为最大化。但通过`exo-open`启动的，即默认终端还不是。所以我们要在开始菜单中`Preferred Applications`里，给默认终端新加一个，内容还是“xfce4-terminal`。然后去改生成的文件：

```bash
vi ~/.local/share/xfce4/helpers/custom-TerminalEmulator.desktop
# 修改对应的两行，加上--maximize
X-XFCE-CommandsWithParameter=/usr/bin/xfce4-terminal --maximize "%s"
X-XFCE-Commands=/usr/bin/xfce4-terminal --maximize
```

## 手机USB热点

使用手机热点比Windows上还要方便，如果Wi-Fi自不必说，即便是最优选择USB，Android 2.2以上无需驱动即可连接；iPhone也只需要安装一个仅`370KB`的`libimobiledevice`驱动包即可：

## 打开Magic SysRq组合键

[魔键的详细叙述](https://www.kernel.org/doc/html/latest/admin-guide/sysrq.html)

具体命令（如果需要用到更多更狠的组合键可以查以上网页中的表，或者`echo 1`）：

```cpp
echo 64 | sudo tee /proc/sys/kernel/sysrq
```

```bash
sudo pacman -S libimobiledevice
```
