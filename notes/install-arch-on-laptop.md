本文是接续前文的Linux自用安装记录中篇，旨在记录一些通用的[安装后配置](https://wiki.archlinux.org/index.php/General_recommendations)过程。

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
