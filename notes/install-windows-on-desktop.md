---
tags: [Blog/System Configurations]
title: 在台式机上安装 Windows 10
---

- [前言](#前言)
- [下载最新 Windows 安装镜像](#下载最新-windows-安装镜像)
- [制作 USB 启动盘](#制作-usb-启动盘)
- [UEFI 下 Secure Boot USB 启动盘](#uefi-下-secure-boot-usb-启动盘)
- [修改计算机名](#修改计算机名)
- [移动`%UserProfile%`内的各种文件夹](#移动userprofile内的各种文件夹)
- [交换 CapsLock 与 Esc](#交换-capslock-与-esc)
- [在 PowerShell 中安装软件并配置](#在-powershell-中安装软件并配置)
- [安装其他软件](#安装其他软件)
- [下载并连接 dotfiles](#下载并连接-dotfiles)
- [禁用多余启动项和功能](#禁用多余启动项和功能)

## 前言

俺自用的 Windows 10 重装备忘录。

## 下载最新 Windows 安装镜像

下载 [Media Creation Tool](https://www.microsoft.com/software-download/windows10)，运行向导中选择下载 ISO。

或者将 User Agent 更改为 Mac OS 或 Linux 并直接选择想要下载的 ISO。该方法可以用 Aria2 断点续传，适合大陆国际网络劣化后不翻墙下载。

## 制作 USB 启动盘

请参考俺的另一短篇[使用 Ventoy 制作 USB 启动盘](use-ventoy-to-create-bootable-usb.md)

## UEFI 下 Secure Boot USB 启动盘

~~俺使用的 MSI B150 Krait Gaming，需要开启 UEFI、Secure Boot，以及 legacy USB support，之后 UEFI 模式的 USB 启动才会出现。~~

[请在安装期间临时关闭 Secure Boot](https://www.ventoy.net/cn/doc_secure.html)。

USB 盘启动后，选择 Windows 的 ISO 启动，之后请自行安装。个人不建议额外划分分区，每个磁盘一个分区已经足够了。

## 修改计算机名

我喜欢用“名字-设备类型”起名。

```text
JayChu-Desktop
```

## 移动`%UserProfile%`内的各种文件夹

如果有外挂 HDD 仓库盘，可以将桌面、下载、音乐、图片等适合存放在仓库盘的文件存放过去。只要查看用户文件夹内对应的文件夹的“属性”，修改“位置”，并在确定时选择“移动原有内容”即可。

可能需要移动的文件夹包括

```text
Desktop
Documents  # 部分软件运行时在此文件夹读写数据，比如聊天记录，游戏存档等，但个人感觉存放到 HDD 也无影响
Downloads
Music
Pictures
Videos
```

## 交换 CapsLock 与 Esc

当使用 Vim / 其他编辑器 Vim 插件时，为了不用经常去够`Esc`和`Ctrl`，Windows 下可以使用[`dual-key-remap`](https://github.com/ililim/dual-key-remap)或者 [AutoHotKey 脚本](https://github.com/escape0707/scripts/blob/master/Use%20CapsLock%20in%20Vim.ahk)。

Windows 的`dual-key-remap`建议在其`config.txt`中额外加上以下内容（[后文](#下载并连接-dotfiles)俺提供了一个代码仓库专门一键配置这些 dotfiles）：

```text
remap_key=ESCAPE
when_alone=CAPSLOCK
with_other=ESCAPE
```

貌似需要与之前的内容用空行隔开，并以单独的空行结尾。注意，如果[使用日文输入法会有 bug](https://github.com/ililim/dual-key-remap/issues/32)。

## 在 PowerShell 中安装软件并配置

在 Windows 下许多开源软件的安装不可避免地要从国外云下载。理想情况下，局域网的网关最好已经设置过路由器透明代理，或者局域网中有旁路网关设置了透明代理，这样装机、或者常用系统中开虚拟机，都不用再考虑如何运行代理软件或者如何设置软件使用代理。推荐使用 openwrt 或 Linux，并在其上跑 clash 做透明代理。

之后可能俺会单独写一篇文章介绍如何在 Linux 机器上用 clash 和策略路由做透明代理网关。

如果没有条件用网关透明代理，俺往往是在同局域网下，用手机或者另一台电脑进行局域网代理共享，之后设置包管理软件`scoop`暂时使用另一台电脑提供的代理共享来翻墙下载，本机装上`clash`等翻墙软件之后再在本地配置翻墙并改回来代理设置。

> 安装好 clash for windows 并启用全局代理后，如果有 UWP 软件不能联网，请用其主界面提供的 UWP Loopback Helper，允许不能联网的 UWP 访问本地。俺是全加了例外了。

> 在 PowerShell 中运行`(Get-PSReadlineOption).HistorySavePath`可以查询终端中的历史记录文件。在装机完成后将其中的内容整理一下，就可以方便下次安装时参考。
> 默认位置是：`%UserProfile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt`.

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

# install non-portable nvidia-display-driver
scoop bucket add nonportable
scoop install nvidia-display-driver-dch-np
# beware that there will be UAC prompts

# install firacode font, there will be UAC prompts, too
scoop bucket add nerd-fonts
scoop install sudo
sudo scoop install firacode-nf

# install other apps, copy and remove those you don't need, then paste-run
# or save to a .ps1 file and execute
scoop bucket add extras
# scoop install anki
# [Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://127.0.01:7890", "User")  # set proxy for Anki, if you use `localhost` here but some apps complain about `localhost` not resolving, change to 127.0.0.1
# scoop install authy
scoop install autohotkey
scoop install ffmpeg  # for youtube-dl
# scoop install firefox  # I don't recommend using package managers to install frequently updated softwares like browsers when they officially provide more efficient ways to update themselves
scoop install goldendict # if you want QT5 goldendict to solve High-DPI scale problem, then modify manifest or create a pull request to add goldendict-qt5
scoop install less
scoop install mpv-git
scoop install neovim
scoop install oh-my-posh3
# scoop install qbittorrent-portable
# scoop install qbittorrent-enhanced
scoop install steam
scoop install sumatrapdf
scoop install telegram
scoop install vcredist2019  # dependency for neovim
scoop install vscode-portable

# add visual studio code as a context menu option
reg import $SCOOP\apps\vscode-portable\current\vscode-install-context.reg

# # I recommend to install programming utilities on Windows Subsystem for Linux now, below are just my old way to install devDependencies
# # If you use Windows Subsystem for Linux, skip the following three paragraphs
# scoop bucket add java
# scoop install octave
# scoop install openjdk
# scoop install gcc
scoop install python  # this will also install dark & lessmsi
# scoop install llvm
# scoop install nodejs
# scoop install yarn

# configure pip and install python apps. Plz only install dev time dep in venv
pip install -i https://mirrors.cloud.tencent.com/pypi/simple --upgrade pip
pip config set global.index-url https://mirrors.cloud.tencent.com/pypi/simple
# pip install bandit
# pip install black
# pip install flake8
# pip install mypy  # I'm trying out Pylance now
# pip install pip-autoremove
pip install youtube-dl

# ## choco related. deprecated. please run in an admin shell
# # install choco
# Set-ExecutionPolicy Bypass -Scope Process -Force
# [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072
# iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
# # configure choco
# # choco config set proxy http://localhost:7890  # choco uses system proxy by default
# choco feature enable -n allowGlobalConfirmation
# # install apps
# choco install geforce-experience

# configure npm, yarn and install node modules
# WARNING: don't use yrm to set registry as it will break scoop's yarn path settings
npm config set registry https://mirrors.cloud.tencent.com/npm/
yarn config set registry https://mirrors.cloud.tencent.com/npm/
# npm config set proxy http://localhost:7890  # npm uses system proxy by default
# yarn config set proxy http://localhost:7890  # yarn uses system proxy by default
yarn global add @typescript-eslint/eslint-plugin
yarn global add @typescript-eslint/parser
yarn global add eslint
yarn global add eslint-config-prettier
yarn global add prettier
yarn global add rimraf
yarn global add typescript
```

## 安装其他软件

去 Microsoft Store 中安装一些软件：

```text
# Lenovo Vantage
Microsoft Office Home and Student 2019
# NVIDIA Control Panel
# Toggl Track
Windows Terminal
# 哔哩哔哩动画  # Reject Bilibili and return to piracy
QQ 音乐
# 爱奇艺
# QQ 桌面版  # Not a full UWP, can still spy on you
# Wechat for Windows  # Not a full UWP, can still spy on you
```

安装 iTunes 以便刷新手机，或者不安装 iTunes 仅仅将安装包解压缩并安装`AppleMobileDeviceSupport64.msi`以支持 iPhone 的 USB 网络共享。安装过程中可能提示无法启动 xxx 服务的错误，此时点击忽略即可。

高 DPI 屏幕在安装 GoldenDict 时，为了软件显示正常请选用 QT5 版本，并在环境变量中定义`QT_AUTO_SCREEN_SCALE_FACTOR`，其值设为`1`：

```powershell
[Environment]::SetEnvironmentVariable("QT_AUTO_SCREEN_SCALE_FACTOR", "1", "User")
```

## 下载并连接 dotfiles

俺有一个[代码仓库](https://github.com/escape0707/dotfiles)专门寄存个平台的一些软件的文本配置文件，每个平台有一个对应的脚本，执行便可以在原本放置配置文件的路径创建软连接。使用和修改配置文件的软件不会察觉与直接放置了一个配置文件在原处有何不同。反倒是修改可以统一反映在我们克隆下来的这个`dotfiles`文件夹下。方便用 git 管理版本和同步。

```powershell
mkdir $env:USERPROFILE\my-programs
git clone https://github.com/escape0707/dotfiles $env:USERPROFILE\my-programs\dotfiles
sudo $env:USERPROFILE\my-programs\dotfiles\windows-setup.ps1
```

## 禁用多余启动项和功能

### 启动项

```text
Windows Security notification icon
```

### 组策略

组策略在家庭版中不可用，如您的 windows 是家庭版请移步下文中[注册表](#注册表)的部分，或者查看 [Microsoft Activation Scripts](https://github.com/massgravel/Microsoft-Activation-Scripts)

`Win+r`：`gpedit.msc`

![组策略中禁用 Cortana 和 OneDrive](../attachments/gpedit-disable-cortana-onedrive.png)

### 注册表

俺有一个[代码仓库](https://github.com/escape0707/scripts)专门存放了 AutoHotKey 脚本和注册表条目修改。其中注册表条目的修改是从`howtogeek.com`参考而来。
请参考、并使用能够移除 3D Objects、启动时打开 Num Lock 以及家庭版用户禁用 Cortana 和隐藏 OneDrive 的注册表文件。如果是家庭版想要禁用 OneDrive，还要手动删除卸载 OneDrive。

### Firefox config

打开 Firefox 并跳转到`about:config`页面，将`browser.tabs.closeWindowWithLastTab`设置为`false`。

### Edge Chromium config

打开 Edge Chromium 并跳转到`edge://flags`页面，跳转并关闭如下几个锚点对应的设置：

```
#smooth-scrolling
#edge-experimental-scrolling
```
