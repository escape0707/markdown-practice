---
tags: [Blog/System Configurations]
title: 在台式机上安装Windows 10
---

- [前言](#前言)
- [下载最新 Windows 安装镜像](#下载最新-windows-安装镜像)
- [使用 Rufus 制作 USB 启动盘](#使用-rufus-制作-usb-启动盘)
- [UEFI 下 Secure Boot USB 启动盘](#uefi-下-secure-boot-usb-启动盘)
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
  > 如果不是用官方 Media Creation Tool 下载的，`install.wim`文件可能会因为包含了各种版本的 Windows 而大于 4GB，不能存放于 FAT32 文件系统中，此时请参照[Rufus 官方 FAQ](https://github.com/pbatard/rufus/wiki/FAQ#Blah_UEFI_Blah_FAT32_therefore_Rufus_should_Blah)尝试使用`UEFI:NTFS`，或者直接使用 NTFS 格式化并尝试您的 UEFI 固件是否支持从 NTFS 启动 UEFI 安装盘。）
- `START`开始制作

## UEFI 下 Secure Boot USB 启动盘

~~俺使用的 MSI B150 Krait Gaming，需要开启 UEFI、Secure Boot，以及 legacy USB support，之后 UEFI 模式的 USB 启动才会出现。~~

> 如果使用了 Rufus 的`UEFI:NTFS`模式，[请在安装期间临时关闭 Secure Boot](https://github.com/pbatard/rufus/wiki/FAQ#why-do-i-need-to-disable-secure-boot-to-use-uefintfs)。

U 盘启动后请自行安装，个人不建议额外划分分区，每个磁盘一个分区已经足够了。

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

貌似需要与之前的内容用空行隔开，并以单独的空行结尾。而且似乎有 bug，并不能很好的支持我这两个不冲突的设置。

## 在 PowerShell 中安装软件并配置

在 Windows 下许多开源软件的安装不可避免地要从国外云下载。俺往往是在同局域网下，用手机或者另一台电脑进行局域网代理共享，之后设置包管理软件`scoop`暂时使用另一台电脑提供的代理共享来翻墙下载，本机装上`clash`等翻墙软件之后再在本地配置翻墙并改回来代理设置。此外您也可以试试在路由器级别配置透明代理。这里就不展开介绍了。

> 安装好 clash for windows 并启用全局代理后，如果有 UWP 软件不能联网，请用其主界面提供的 UWP Loopback Helper，允许不能联网的 UWP 访问本地。俺是全加了例外了。

> 在 PowerShell 中运行`(Get-PSReadlineOption).HistorySavePath`可以查询终端中的历史记录文件。在装机完成后将其中的内容整理一下，就可以方便下次安装时参考。
> 默认位置是： `%UserProfile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`.

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

# configure git
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
scoop install anki
scoop install archwsl
scoop install autohotkey
scoop install besttrace
scoop install firefox
scoop install goldendict # if you want QT5 goldendict to solve High-DPI scale problem, then modify manifest or PR
scoop install iperf3
scoop install less
scoop install neovim
scoop install oh-my-posh
scoop install posh-git
scoop install steam
scoop install streamlink
scoop install sumatrapdf
scoop install teamviewer  # portable version won't remember logins, if it's a problem to you, try `teamviewer-np`
scoop install telegram
scoop install vcredist2019  # dependency for neovim
scoop install vlc
scoop install vscode-insiders

# add visual studio code as a context menu option
reg import $HOME\scoop\apps\vscode-insiders\current\vscode-install-context.reg

# I recommend to install programming dependencies on Windows Subsystem for Linux now, below are just my old way to install devDependencies
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
New Microsoft Edge
```

还要去 Microsoft Store 中安装一些软件：

```text
Bitwarden
Microsoft Office Home and Student 2019
Telegram
Trello
Windows Terminal
哔哩哔哩动画
QQ音乐
爱奇艺
QQ
Wechat
```

安装 iTunes 以便刷新手机，或者不安装 iTunes 仅仅将安装包解压缩并安装`AppleMobileDeviceSupport64.msi`以支持 iPhone 的 USB 网络共享。安装过程中可能提示无法启动 xxx 服务的错误，此时点击忽略即可。

高 DPI 屏幕在安装 GoldenDict 时，为了软件显示正常请选用 QT5 版本，并在环境变量中定义`QT_AUTO_SCREEN_SCALE_FACTOR`，其值设为`1`。

## 下载并连接 dotfiles

俺有一个[代码仓库](https://github.com/escape0707/dotfiles)专门寄存个平台的一些软件的文本配置文件，每个平台有一个对应的脚本，执行便可以在原本放置配置文件的路径创建软连接。使用和修改配置文件的软件不会察觉与直接放置了一个配置文件在原处有何不同。反倒是修改可以统一反映在我们克隆下来的这个`dotfiles`文件夹下。方便用 git 管理版本和同步。

```powershell
mkdir C:\ProgramData\my-programs\dotfiles
git clone https://github.com/escape0707/dotfiles C:\ProgramData\my-programs\dotfiles
sudo C:\ProgramData\my-programs\dotfiles\windows-setup.ps1
```

## 禁用多余启动项和功能

### 启动项

```text
Windows Security notification icon
```

### 组策略

组策略在家庭版中不可用，如您的 windows 是家庭版请移步下文中[注册表](#注册表)的部分。

`Win+r`：`gpedit.msc`

![组策略中禁用Cortana和OneDrive](../attachments/gpedit-disable-cortana-onedrive.png)

### 注册表

俺有一个[代码仓库](https://github.com/escape0707/scripts)专门存放了 AutoHotKey 脚本和注册表条目修改。其中注册表条目的修改是从`howtogeek.com`参考而来。
请参考、并使用能够移除 3D Objects、启动时打开 Num Lock 以及家庭版用户禁用 Cortana 和隐藏 OneDrive 的注册表文件。如果是家庭版想要禁用 OneDrive，还要手动删除卸载 OneDrive。

### Firefox config

打开 Firefox 并跳转到`about:config`页面，将`browser.tabs.closeWindowWithLastTab`设置为`false`。
