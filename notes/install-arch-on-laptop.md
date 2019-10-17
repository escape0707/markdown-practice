本文是接续前文的Linux自用安装记录中篇，旨在记录一些通用的[安装后配置](https://wiki.archlinux.org/index.php/General_recommendations)过程。


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

由于俺只是单人、单账户并作为客户端使用机器，仅用`sudo`防手滑和防其他进程滥用权限。故在此不做[进阶的设置](https://wiki.archlinux.org/index.php/Sudo#Example_entries)，仅将之前新建的用户添加到`sudoers`中去：

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

您可以自己选择喜爱的[桌面环境](https://wiki.archlinux.org/index.php/Desktop_environment)。俺已经体验过Ubuntu和GNOME，故此次尝试比较接近Windows桌面体验的[`KDE`](https://wiki.archlinux.org/index.php/KDE)桌面环境。

安装`Noto`字体、KDE、[`wayland`](https://wiki.archlinux.org/index.php/KDE#KDE_applications)后端支持、以及包括文件管理、终端、记事本等应用，并且启用[SDDM](https://wiki.archlinux.org/index.php/SDDM)显示管理器：

```bash
sudo pacman -S noto-fonts-cjk plasma-meta plasma-wayland-session dolphin-plugins kdegraphics-meta kdeutils-meta khelpcenter konsole kwrite
sudo systemctl enable sddm
```

之前提到，KDE环境整合了`NetworkManager`提供图形化Wi-Fi连接配置界面。但是由于这类网络管理软件（包括前文用到的`netctl`）同时运行会导致错误，此软件不会自动启动，需要我们手动启动：

```bash
systemctl enable NetworkManager
```

全部安装后重启系统、输入密码便可以进入KDE桌面。

## 网页浏览与科学上网

要说为什么需要一个桌面环境，很大程度上不是因为运行软件或者修改设置比GUI更方便或者游戏之类的，而是为了能够至少能够使用[各种浏览器](https://wiki.archlinux.org/index.php/Web_browser)浏览网页……这里俺以不需要代理就可以正常使用全部功能的Firefox为例，您也可以选择Chrome等：

```bash
sudo pacman -S firefox
```

> 为了让`Backspace`在Linux下也和Windows下一样让火狐返回上一页，可以在`about:config`中设置`browser.backspace_action`为`0`。
>
> 为了让标题栏和标签页栏也和Windows下一样在一行，可以在自定义火狐浏览器布局时取消勾选`Title Bar`
>
> 俺的机器上火狐在卷动页面时会出现屏幕撕裂，依照[官方Wiki上的策略](https://wiki.archlinux.org/index.php/firefox#Tearing_video_in_fullscreen_mode)，在`about:config`中设置`layers.acceleration.force-enabled`为`true`可以解决。

为了能够使用Google搜索引擎，还需要[Shadowsocks](https://wiki.archlinux.org/index.php/Shadowsocks)等科学上网工具。`shadowsocks-qt5`在`git clone`时连接总是会断开，所以俺使用C版`shadowsocks-libev`，您也可以使用的Python版的`shadowsocks`。

安装、创建服务器配置文件、然后启动并启用：

> 用您喜欢的名字替换下文的`config`

```bash
sudo pacman -S shadowsocks-libev
vi /etc/shadowsocks/config.json
# 将您的某个shadowsocks服务器信息写入到以上文件中
sudo systemctl start shadowsocks-libev@config.json
sudo systemctl enable shadowsocks-libev@config.json
```

如果服务不能成功自动启动，请尝试将json中的`server`替换为完整域名解析到的IP地址，和/或设置NetworkManager支持Shadowsocks有网后才自启动：

```bash
sudo systemctl enable NetworkManager-wait-online.service
```

有时需要将Shadowsocks默认提供的socks5代理转化为http代理（比如Git），此时可以安装[`privoxy`](https://wiki.archlinux.org/index.php/Privoxy)、设置协议的转发、启动并启用服务：

```bash
sudo pacman -S privoxy
sudo sh -c "echo 'forward-socks5 / 127.0.0.1:1080 .' >> /etc/privoxy/config"
sudo systemctl start privoxy
sudo systemctl enable privoxy
```

安装并运行了本地Shadowsocks客户端后，在Firefox/Chrome中安装[Proxy SwitchyOmega](https://addons.mozilla.org/en-US/firefox/addon/switchyomega/)，打开设置，跳过所有教程，设置proxy中的服务器地址端口为`socks5`、地址为`127.0.0.1`，端口为`1080`。设置auto switch中的规则列表URL为：

```URL
https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt
```

点击下载，并确认符合规则的走proxy，默认走Default。关于科学上网和Proxy SwitchyOmega的设置在此不再详述。

## 更进一步的自定义

至此，我们已经可以在物理机上用Google搜索资料并且顺利查看了！大成功！有了自主搜索资料、下载文件的方法后，很容易查找接下来想做的任何事情如何完成。放下心来在Bilibili上看个番剧之类的，然后再去思考还缺少什么吧！

现在开始，您可以参考Arch Wiki上的应用列表安装感兴趣的应用，或者学习一下系统维护等。妥善利用Google、StackOverflow等网站的资源，进入Linux和开源系统的世界。

也可以参考俺[之后的文章](tweak-arch.md)，此文记录了俺在Arch - Nouveau - SDDM - Plasma - Shadowsocks-libev - Firefox等齐备之后，安装的其他或许有用的软件和进行的调整，诸如：

- pacman镜像服务器测速排序
- pacman wrapper 加速下载与AUR安装
- 高通蓝牙驱动
- 启用Swap（虚拟内存）
- PRIME双显卡

但是上述内容俺也只是做到了搜索并阅读了资料，并且可以调配出来的程度。对于各种方案的比较和微调并不熟悉，所以仅供参考啦。
