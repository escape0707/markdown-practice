---
tags: [Blog/System Configurations]
title: 在台式机上安装Windows 10
---

- [前言](#前言)
- [下载最新 Windows 安装镜像](#下载最新-windows-安装镜像)
- [使用 Rufus 制作 USB 启动盘](#使用-rufus-制作-usb-启动盘)
- [UEFI 下 Secure Boot USB 启动盘](#uefi-下-secure-boot-usb-启动盘)
- [Fresh Start](#fresh-start)
- [修改计算机名](#修改计算机名)
- [如果有两个以上的分区，移动`%UserProfile%`内的各种文件夹](#如果有两个以上的分区移动userprofile内的各种文件夹)
- [交换 CapsLock 与 Esc](#交换-capslock-与-esc)
- [在 PowerShell 中安装软件并配置](#在-powershell-中安装软件并配置)
- [安装其他软件](#安装其他软件)
- [禁用多余启动项和功能](#禁用多余启动项和功能)

## 前言

俺自用的 Windows 10 重装备忘录。

## 下载最新 Windows 安装镜像

下载 [Media Creation Tool](https://www.microsoft.com/software-download/windows10)，运行向导中选择下载 ISO。

## 使用 Rufus 制作 USB 启动盘

### 下载 Rufus

从[官网](https://rufus.ie/)手动下载，或用 Scoop/Chocolatey 下载：

```powershell
scoop install rufus
choco install rufus
```

### 制作启动盘

- 选择要使用的 USB 盘
- 选择下载的 Windows ISO 镜像
- 分区类型选择`GPT`、目标系统类型选择`UEFI`
- 文件系统选择 FAT32
  > 如果不是用官方 Media Creation Tool 下载的，可能文件会大于 4G 而不能存放于 FAT32 文件系统中，此时请参照[Rufus 官方 FAQ](https://github.com/pbatard/rufus/wiki/FAQ#Blah_UEFI_Blah_FAT32_therefore_Rufus_should_Blah)尝试使用`UEFI:NTFS`，或者直接使用 NTFS 格式化并尝试您的 UEFI 固件是否支持从 NTFS 启动 UEFI 安装盘。）
- `START`开始制作

## UEFI 下 Secure Boot USB 启动盘

俺使用的 MSI B150 Krait Gaming，需要开启 UEFI、Secure Boot，以及 legacy USB support，之后 UEFI 模式的 USB 启动才会出现。

> 如果使用了 Rufus 的`UEFI:NTFS`模式，[请在安装期间临时关闭 Secure Boot](https://github.com/pbatard/rufus/wiki/FAQ#why-do-i-need-to-disable-secure-boot-to-use-uefintfs)。

U 盘启动后请自行安装，个人不建议额外划分分区，每个磁盘一个分区已经足够了。

## Fresh Start

（可选步骤）初次安装 Windows 10 后再次进行 Fresh Start 可以彻底排除一堆微软自带的垃圾软件/游戏。手动删除亦可。

## 修改计算机名

```text
JayChu-Desktop
```

## 如果有两个以上的分区，移动`%UserProfile%`内的各种文件夹

```text
Desktop
Documents
Downloads
Music
Pictures
Videos
```

## 交换 CapsLock 与 Esc

当使用 Vim/Vim 插件时，为了不用经常去够`Esc`和`Ctrl`，Windows 下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)，Linux 下可以使用[`caps2esc`](https://aur.archlinux.org/packages/interception-caps2esc)。

Windows 的`dual-key-remap`建议在其`config.txt`中额外加上以下内容：

```text
remap_key=ESCAPE
when_alone=CAPSLOCK
with_other=ESCAPE
```

貌似需要与之前的内容用空行隔开，并以单独的空行结尾。

## 在 PowerShell 中安装软件并配置

在 Windows 下许多开源软件的安装不可避免地要从国外云下载。俺往往是在同局域网下，用手机或者另一台电脑进行局域网代理共享，之后设置包管理软件`scoop`暂时使用另一台电脑提供的代理共享来翻墙下载，本机装上`clash`等翻墙软件之后再在本地配置翻墙并改回来代理设置。

> 安装好 clash for windows 并启用全局代理后，如果有 UWP 软件不能联网，请用其主界面提供的 UWP Loopback Helper，允许不能联网的 UWP 访问本地。俺是全加了例外了。

如果你使用电脑的地区有限，可以试试在路由器级别配置透明代理。这里就不展开介绍了。

> 在 PowerShell 中运行`(Get-PSReadlineOption).HistorySavePath`以查询终端中的历史记录文件。
> 默认位置是： `%userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`.

```powershell
# install scoop
Set-ExecutionPolicy RemoteSigned -scope CurrentUser
iwr -useb get.scoop.sh | iex
scoop --version

# configure scoop, as scoop ignores aria2's global config
scoop config proxy 127.0.0.1:7890
scoop install aria2
scoop config aria2-max-connection-per-server 16
scoop config aria2-min-split-size 1M
scoop config aria2-split 10000
scoop install 7zip
scoop install git

# configure aria2
mkdir $HOME/.aria2
echo max-connection-per-server=16 >> $HOME/.aria2/aria2.conf
echo min-split-size=1M >> $HOME/.aria2/aria2.conf
echo split=10000 >> $HOME/.aria2/aria2.conf

# configure git
git config --global credential.helper manager
git config --global http.proxy http://localhost:7890
git config --global merge.ff only
git config --global pull.ff only
git config --global user.email tothesong@gmail.com
git config --global user.name escape0707

# install clash-for-windows
scoop bucket add dorado https://github.com/h404bi/dorado
scoop install clash-for-windows
# and configure clash-for-windows to be the proxy at port 7890 manually...

# install non-portable nvidia-display-driver
scoop bucket add nonportable
scoop install nvidia-display-driver-np
# beware that there will be UAC prompts

# install firacode font, there will be UAC prompts, too
scoop bucket add nerd-fonts
scoop install sudo
sudo scoop install FiraCode

# install other apps, copy and remove those you don't need, then paste-run
# or save to a .ps1 file and execute
scoop bucket add extras
scoop install autohotkey
scoop install besttrace
scoop install firefox
scoop install goldendict
scoop install less
scoop install neovim
scoop install oh-my-posh
scoop install posh-git
scoop install steam
scoop install streamlink
scoop install sumatrapdf
scoop install teamviewer-np  # portable version won't remember logins
scoop install telegram
scoop install vcredist2015  # dependency for neovim
scoop install vlc
scoop install vscode-insiders
scoop install wechat

# setup oh-my-posh
echo 'Import-Module posh-git' >> $PROFILE
echo 'Import-Module oh-my-posh' >> $PROFILE
echo 'Set-Theme Paradox' >> $PROFILE

# setup scoop/choco update oneliner
echo 'function pacup { sudo choco upgrade all; scoop update * }' >> $PROFILE

# add visual studio code as a context menu option
reg import $HOME\scoop\apps\vscode-insiders\current\vscode-install-context.reg

# I recommend to install programming dependencies on Windows Subsystem for Linux 2 now, below are just my old way to install devDependencies
# If you use Windows Subsystem for Linux, skip these three following paragraphs
scoop bucket add java
scoop install openjdk
scoop install gcc
scoop install python  # this will also install dark & lessmsi?
scoop install llvm
scoop install nodejs
scoop install yarn

## choco related. please run in an admin shell
# install choco
Set-ExecutionPolicy Bypass -Scope Process -Force
[System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
choco --version
# configure choco
choco config set proxy http://localhost:7890
choco feature enable -n allowGlobalConfirmation
# install apps
choco install authy-desktop
choco install geforce-experience

# configure npm, yarn and install node modules
# WARNING: don't use yrm to set registry as it will break scoop's yarn path settings
npm config set registry https://mirrors.cloud.tencent.com/npm/
yarn config set registry https://mirrors.cloud.tencent.com/npm/
# npm config set proxy http://localhost:7890
# yarn config set proxy http://localhost:7890
yarn global add @typescript-eslint/eslint-plugin
yarn global add @typescript-eslint/parser
yarn global add eslint
yarn global add eslint-config-prettier
yarn global add prettier
yarn global add rimraf
yarn global add typescript

# configure pip and install python modules
pip install -i https://mirrors.cloud.tencent.com/pypi/simple --upgrade pip
pip config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple
pip install bandit
pip install black
pip install flake8
pip install mypy
pip install pip-autoremove
```

## 安装其他软件

以及自行安装一些软件：

```text
Microsoft Edge Canary
Microsoft Office 2019
```

还要去 Microsoft Store 中安装一些软件：

```text
Bitwarden
Trello
哔哩哔哩动画
QQ音乐
爱奇艺
```

安装 iTunes 以便刷新手机，或者不安装 iTunes 仅仅将安装包解压缩并安装`AppleMobileDeviceSupport64.msi`以支持 iPhone 的 USB 网络共享。安装过程中可能提示无法启动 xxx 服务的错误，此时点击忽略即可。

高 DPI 屏幕在安装 GoldenDict 时，为了软件显示正常请选用 QT5 版本，并在环境变量中定义`QT_AUTO_SCREEN_SCALE_FACTOR`，其值设为`1`。启动 GoldenDict 则似乎不能再用命令行，必须用鼠标双击、开始菜单里单击等 GUI 方式打开，不知道为什么。

## 禁用多余启动项和功能

### 启动项

```text
iTunes Helper
LogiOptions
Logitech Gaming Framework
Microsoft Edge Update
Microsoft Onedrive
Windows Security notification icon
```

### 组策略

组策略在家庭版中不可用，如您的 windows 是家庭版请移步下文中[注册表](#注册表)的部分。

`Win+r`：`gpedit.msc`

![组策略中禁用Cortana和OneDrive](../attachments/gpedit-disable-cortana-onedrive.png)

### 注册表

俺有一个[代码仓库](https://github.com/escape0707/scripts)专门存放了 AutoHotKey 脚本和注册表条目修改。
请参考、并使用能够移除 3D Objects、启动时打开 Num Lock 以及家庭版用户禁用 Cortana 和隐藏 OneDrive）的注册表文件。如果是家庭版想要禁用 OneDrive，还要手动删除卸载 OneDrive。

### Firefox config

打开 Firefox 并跳转到`about:config`页面，将`browser.tabs.closeWindowWithLastTab`设置为`false`。
