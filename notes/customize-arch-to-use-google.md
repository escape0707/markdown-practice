---
tags: [Blog/System Configurations]
title: Arch新机用上Google
---

## 目录<!-- omit in toc -->

- [前言](#前言)
- [用NetworkManager连接Wi-Fi](#用networkmanager连接wi-fi)
- [创建账户与配置sudo](#创建账户与配置sudo)
- [显卡驱动与图形化界面](#显卡驱动与图形化界面)
- [网页浏览与科学上网](#网页浏览与科学上网)
- [更进一步的自定义](#更进一步的自定义)

## 前言

在前文[在虚拟机/物理机中安装Arch Linux](install-arch-on-laptop-and-vm.md)中，俺已经记录了安装最小化、可联网的Arch Linux的过程。本文将继续描述俺在前述系统中折腾到可以在桌面环境下使用搜索引擎的程度所需的步骤。

## 用NetworkManager连接Wi-Fi

如果是在物理机上想要连接Wi-Fi，可以用之前已经安装并启用的NetworkManager的图形化工具`nmtui`：

```bash
nmtui
```

选择好Wi-Fi、输入密码并激活后，NetworkManager便会记住并自动使用它。

## 创建账户与配置sudo

在完成了最小化Arch Linux安装之后，为了安全使用系统，还需要创建属于自己的个人账户、安装并配置好sudo。

### 新建用户

新安装的系统只有`root`一个超级用户。一直使用`root`账户并不安全，可能会意外地执行或修改了文件，以及给予第三方程序过高权限带来风险。平时应当使用普通权限的账户。否则无法使用`makepkg`、`sddm`等工具。

[新建一个用户](https://wiki.archlinux.org/index.php/Users_and_groups#Example_adding_a_user)：

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

由于俺只是单人、单账户、客户端自闭性使用机器，仅用`sudo`防手滑和其他进程滥用权限。故在此不做[进阶的设置](https://wiki.archlinux.org/index.php/Sudo#Example_entries)，仅将之前新建的用户添加到`sudoers`中去：

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

为了启动`Firefox`、`Chromium`等图形化浏览器，建议安装好[显卡驱动](https://wiki.archlinux.org/index.php/Xorg#Driver_installation)和一套[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。

### 显卡驱动

根据您的设备选择需要安装的显卡驱动，例如:

- 开源NVIDIA显卡驱动[`Nouveau`](https://wiki.archlinux.org/index.php/Nouveau)：

  ```bash
  sudo pacman -S xf86-video-nouveau
  ```

- [Intel显卡驱动](https://wiki.archlinux.org/index.php/Intel_graphics)（也有一说不安装而使用缺省驱动更快）：

  ```bash
  sudo pacman -S xf86-video-intel
  ```

> 如果是在虚拟机中，请安装`xf86-video-fbdev`驱动。并且Hyper-V不支持使用wayland作为显示后端，请直接不要安装plasma-wayland-session，也不要在sddm中选择（不装就自然没的选）。

### 桌面环境

您可以自己选择喜爱的[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。目前俺的综合建议是先当作依赖包安装好所有桌面环境都共用的[Xorg](https://wiki.archlinux.org/index.php/Xorg)显示服务器，和[xinit](https://wiki.archlinux.org/index.php/Xinit)启动工具：

```bash
sudo pacman -S xorg-server xorg-xinit --asdeps
```

之后再通过网络搜索几个能看对了眼的桌面环境，分别安装之后使用对应的方法启动并进行测试，以[Xfce4](https://wiki.archlinux.org/index.php/Xfce)为例：

```bash
sudo pacman -S xfce4 xfce4-goodies
startxfce4
```

用这种方法试几个之后自然就能找到适合自己习惯和电脑配置的桌面环境了。

另外俺建议在测试并确定好了想要使用的桌面环境，准备正式安装前删除并重建自己的账户和专用文件夹，以清除测试时各种软件遗留的配置和文件：

```bash
userdel -r your-username
useradd -m your-username
passwd your-username
```

俺已经体验过Ubuntu、GNOME~~，故此次尝试比较接近Windows桌面体验的~~以及本文第一版中提到的[`KDE`](https://wiki.archlinux.org/index.php/KDE)。经过两个月的体验，俺觉出KDE的确酷炫、定制丰富、对Windows难民友好，但对于俺的笔记本来说淡入淡出等特效还是复杂了一些，启动耗时相对长、资源占用也相对多。目前俺换到了Xfce这一更简化的GNOME系桌面环境。并且放弃了[显示管理器](https://wiki.archlinux.org/index.php/Display_manager)。

以下是俺装Xfce和KDE的过程。

#### Xfce

需要安装的包：

- 支持回收站机制：`gvfs`
- 托盘区网络管理：`network-manager-applet`
- Noto汉字系字体：`noto-fonts-cjk`
- 声音服务器：`pulseaudio`
- Xfce基础包：`xfce4`
- Xfce额外组件包：`xfce4-goodies`

```bash
sudo pacman -S gvfs network-manager-applet noto-fonts-cjk pulseaudio xfce4 xfce4-goodies
```

设置登录后[静默](https://wiki.archlinux.org/index.php/Silent_boot#startx)地[自启动`startx`](https://wiki.archlinux.org/index.php/Xinit#Autostart_X_at_login)（对于Zsh，修改`~/.zprofile`）：

```bash
cat >> ~/.bash_profile
if systemctl -q is-active graphical.target && [[ ! $DISPLAY && $XDG_VTNR -eq 1 ]]; then
  [[ $(fgconsole 2>/dev/null) == 1 ]] && exec startx -- vt1 &> /dev/null
fi
# ctrl+d
```

（可选的）设置[静默自动登录](https://wiki.archlinux.org/index.php/Silent_boot#agetty)：

> username更换为想要自动登入的用户名

```bash
systemctl edit getty@tty1
# 填入以下内容
[Service]
ExecStart=
ExecStart=-/usr/bin/agetty --skip-login --nonewline --noissue --autologin username --noclear %I $TERM
Type=simple
```

#### KDE

安装`Noto`字体、KDE、[`wayland`](https://wiki.archlinux.org/index.php/KDE#KDE_applications)后端支持（可选）、以及文件管理、终端、记事本等应用，并且启用[SDDM](https://wiki.archlinux.org/index.php/SDDM)显示管理器：

```bash
sudo pacman -S noto-fonts-cjk plasma-meta plasma-wayland-session dolphin-plugins kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite
sudo systemctl enable sddm
```

全部安装后重启系统、输入密码便可以进入KDE桌面。

## 网页浏览与科学上网

其实不需要桌面环境也可以使用[各种浏览器](https://wiki.archlinux.org/index.php/Web_browser)浏览网页、使用终端执行命令……这里俺以不需要代理就可以正常使用全部功能的Firefox为例，您也可以选择Chrome等：

```bash
sudo pacman -S firefox
```

> 为了让`Backspace`在Linux下也和Windows下一样让火狐返回上一页，可以在`about:config`中设置`browser.backspace_action`为`0`。
>
> 为了让标题栏和标签页栏也和Windows下一样在一行，可以在自定义火狐浏览器布局时取消勾选`Title Bar`
>
> 俺的机器上火狐在卷动页面时会出现屏幕撕裂，依照[官方Wiki上的策略](https://wiki.archlinux.org/index.php/firefox#Tearing_video_in_fullscreen_mode)，在`about:config`中设置`layers.acceleration.force-enabled`为`true`可以解决。

为了能够使用Google搜索引擎，还需要[Shadowsocks](https://wiki.archlinux.org/index.php/Shadowsocks)、[V2Ray]等科学上网工具。

### Shadowsocks

`shadowsocks-qt5`在`git clone`时连接总是会断开，所以俺使用C版`shadowsocks-libev`，您也可以使用的Python版的`shadowsocks`。

安装、创建服务器配置文件、然后启动并启用：

> 用您喜欢的名字替换下文的`config`

```bash
sudo pacman -S shadowsocks-libev
vi /etc/shadowsocks/config.json
# 将您的某个shadowsocks服务器信息写入到以上文件中
sudo systemctl enable shadowsocks-libev@config.json --now
```

如果服务不能成功自动启动，请尝试将json中的`server`替换为完整域名解析到的IP地址

### V2Ray

安装、创建服务器配置文件、然后启动并启用：

```bash
sudo pacman -S v2ray
vi /etc/v2ray/config.json
# 将您的某个v2ray服务器信息写入到以上文件中
sudo systemctl enable v2ray --now
```

### Privoxy

有时需要将socks5代理转化为http代理（比如Git），此时可以安装[`privoxy`](https://wiki.archlinux.org/index.php/Privoxy)、设置协议的转发、启动并启用服务：

```bash
sudo pacman -S privoxy
sudo sh -c 'echo "forward-socks5 / 127.0.0.1:1080 ." >> /etc/privoxy/config'
sudo systemctl enable privoxy --now
```

安装并运行了本地代理客户端后，在Firefox/Chrome中安装[Proxy SwitchyOmega](https://addons.mozilla.org/en-US/firefox/addon/switchyomega/)，打开设置，跳过所有教程，设置proxy中的服务器地址端口为`socks5`、地址为`127.0.0.1`，端口为`1080`。设置auto switch中的规则列表URL为：

```URL
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

点击下载，并确认符合规则的走proxy，默认走Default。关于科学上网和Proxy SwitchyOmega的设置在此不再详述。

## 更进一步的自定义

至此，我们已经可以在物理机上用Google搜索资料并且顺利查看了！大成功！有了自主搜索资料、下载文件的方法后，很容易查找接下来想做的任何事情如何完成。放下心来在Bilibili上看个番剧之类的，然后再去思考还缺少什么吧！

接下来，可以参考Arch Wiki上的应用列表安装感兴趣的应用，或者学习一下系统维护等。妥善利用Google、StackOverflow等网站的资源，进入Linux和开源系统的世界。

俺之后还会继续记录俺在Arch - Nouveau - Xfce4 - shadowsocks-libev/v2ray - Firefox等齐备之后，安装的其他或许有用的软件和进行的调整，包括但不限于：

- AUR与pacman wrapper
- 高通蓝牙驱动
- 通过文件启用Swap（虚拟内存）
- PRIME双显卡

详情请移步后文——[自定义Arch系统](arch-tricks.md)。
