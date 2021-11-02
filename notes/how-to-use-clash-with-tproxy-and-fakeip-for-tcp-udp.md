---
tags: [Blog/Networking]
title: 如何使用 Clash 的 TPROXY 功能进行透明代理
---

-   [前言](#前言)
-   [安装](#安装)
-   [安装其他辅助软件包](#安装其他辅助软件包)
-   [启动 / 开机自启动](#启动--开机自启动)
-   [基础使用](#基础使用)
-   [局域网转发](#局域网转发)
-   [配置 Clash DNS](#配置-clash-dns)
-   [修改防火墙来 TPROXY 内网其他机器](#修改防火墙来-tproxy-内网其他机器)
-   [代理本机流量](#代理本机流量)
-   [Fake ip](#fake-ip)
-   [配置持久化](#配置持久化)

## 前言

Clash 是当下比较热门、使用起来也比较方便的客户端代理工具。俺使用它主要是喜欢它：

1. 支持策略分流
2. 可以在运行时轻松改变分流策略所使用的节点
3. 可以在路由器、Linux 或 Mac 机器上设置透明代理来给局域网内设备代理
4. （理论上）可以自动 / 手动进行网络连通性检测和延迟，并按照某些策略自动选择分流策略所用的节点（实际上机场不喜欢你老测试）

总体来说是比较适合使用机场的人士来回切换节点、方便解锁不同区域流媒体、规避机场中比较“拉闸”的节点的客户端工具。开源社区也有比较好的分流策略支持。

这次俺就不针对如何在 Windows 上用 Clash For Windows 进行说明了。这种用法的介绍文章相对好找。Clash For Windows 本身是闭源有广告软件，使用的也是 Clash premium 闭源内核。俺认为一个接管你甚至整个家庭的全部流量的代理软件不应该是闭源的。

本文从一步步记录俺从安装 clash 到配置笔记本 Linux 做局域网内翻墙旁路网关的步骤。路由器网关翻墙的配置大同小异。

## 安装

Arch 的话，普通情况安装 `community/clash` 即可。但是要做旁路网关，如果还想代理本机流量的话，需要在之后配置防火墙的时候区分 clash 流量，防止代理软件的流量本地回环。区分的时候可以用 `uid` / `gid` / `cgroup`。此外如果想要在本地 53 端口直接开 clash 的 DNS、或使用 TPROXY 方法代理的话，需要给予其对应的[“能力”](https://wiki.archlinux.org/title/Capabilities)：

1. `CAP_NET_RAW` 让应用可以处理 TPROXY 带来的流量。
2. `CAP_NET_BIND_SERVICE` 允许应用绑定 1000 以下的端口。

为了方便干净的放行 BT 之类的软件，俺建议使用比较新的 `cgroupv2` 来解决本地进程流量放行的问题。还能避免创建多余用户的麻烦。创建多余用户的方案也在下文中描述了，但不建议使用。

### 使用 `cgroup` 标记并过滤

参考文章：<https://dev.to/outloudvi/transparent-proxy-on-arch-linux-with-iptables-and-systemd-slice-580c>

首先安装 `clash`：

```bash
sudo pacman -S clash
```

`clash` 默认的 `clash.service` 是用户级 systemd 单元，不能方便地给予能力，所以之后我们都用模板单元 `clash@.service`。先编辑这个服务的 drop-in 文件（以下 `@` 后面的 [Systemd 模板](https://wiki.archlinux.org/title/Systemd#Using_units)参数俺用的是自己的用户名 `jay`，之后的各个步骤中请替换成你的用户名。）：

```bash
sudo systemctl edit clash@.service
```

填入：

```ini
[Service]
AmbientCapabilities=CAP_NET_RAW CAP_NET_BIND_SERVICE
Slice=noproxy.slice
```

如此一来，clash 便会在名为 `noproxy.slice` 的 cgroup 中运行了。如果以后有其他不想透明代理的进程也可以放到这个 cgroup 中。

### （不推荐）创建 `clash` 用户并用 `uid` 过滤

首先我们创建一个无用户目录、无法登陆的名为 `clash` 的用户，专门用来启动代理程序：

```bash
sudo useradd -Ms /usr/bin/nologin clash
```

然后安装 `clash-user`：

```bash
sudo pacman -S clash-user
```

## 安装其他辅助软件包

### GeoIP 数据库

Maxmind GeoIP 数据库（如果[配置了 archlinuxcn](arch-tricks.md#arch-linux-cn) 则可以直接下载安装）：

```bash
sudo paru -S clash-geoip
```

如用 cgroup 方案，clash 的配置目录在 `$XDG_CONFIG_HOME/clash/`。需要生成一个软链接指向刚刚安装的数据库文件：

```bash
mkdir -p ~/.config/clash
ln -s /etc/clash/Country.mmdb ~/.config/clash/Country.mmdb
```

### yacd 前端面板

推荐使用 yacd 面板作为前端。

```bash
sudo pacman -S yacd
```

> clash 有一个“缓存你上次退出时每个策略组选择的什么节点”的功能，默认写入到 clash 的配置目录下叫 `.cache` 的文件里。但是 clash-user 的 systemd 服务指定的配置目录 `/etc/clash` 默认情况下对于非 root 用户是只读的，所以如果使用这种方案建议手动创建这个文件并且将所有者切换成 clash 用户：
>
> ```bash
> sudo touch /etc/clash/.cache
> sudo chown clash /etc/clash/.cache
> ```

## 启动 / 开机自启动

至此，clash 本身已经安装完成 ，只要编写好正确的配置文件 `$XDG_CONFIG_HOME/clash/config.yaml`（对于 clash-user：`/etc/clash/config.yaml`）即可用 systemd 启动 / 开机自启动 clash 了。

### cgroup 方案

```bash
sudo systemctl enable clash@jay --now  # start now and on system boot
```

### （不推荐）clash-user 方案

```
sudo systemctl enable clash --now
```

## 基础使用

最基础的使用方法莫过于是直接设置系统代理、各个软件代理来使用 clash 了。建议这样尝试一次，看看最简单的方法能否成功。

`config.yaml`（详细配置规则见[官方仓库 Wiki](https://github.com/Dreamacro/clash/wiki/configuration#all-configuration-options)）：

```yaml
log-level: debug
mixed-port: 7890
external-controller: 0.0.0.0:9090
external-ui: '/usr/share/yacd'
# append your proxies and rules
```

> 其中 `external-ui` 写成你自己安装或下载的 clash web 前端的目录，也就是 index.html 所在的目录。clash 会自己启动一个 http 服务器来跑这个网页。之后你可以通过在浏览器访问 `external-controller` 绑定的地址 +`/ui` 来访问面板。如 `localhost:9090/ui`，或者对于局域网内其它机器 `本机局域网 ip:9090/ui`。
>
> 如果 `external-ui` 写成 `127.0.0.1:9090`，那么只有本机可以访问这个面板。

此时系统设置代理以后（GNOME 或 KDE 中设置），Firefox 应该会使用系统代理，但是 Telegram 不会，不知道是不是 GNOME 是 GTK 写的、 Telegram 是 Qt 写的的缘故。此外，貌似 Git、Aria2 也不使用这个设置，虚拟机中的系统也要单独设置代理——这也太麻烦了吧！赶紧接着设置透明代理吧。

> 刚刚设置的系统代理 / 浏览器代理可以留着，方便边做边查资料。直到做到“代理本机流量”的一步位置都没有影响。

## 局域网转发

> [据说](https://xtls.github.io/documents/level-2/tproxy/#%E5%BC%80%E5%A7%8B%E4%B9%8B%E5%89%8D)这一步如果只是做局域网旁路网关，反而应该把内核数据包转发关掉。不过俺没有测试，也不太需要。

多数 Linux 发行版默认是[开启 ipv4 转发](https://wiki.archlinux.org/title/Internet_sharing#Enable_packet_forwarding) 的。

查看是否开启：

```bash
sudo sysctl -a | grep forward
```

开启：

```bash
sudo sysctl net.ipv4.ip_forward=1
```

此时可以在用局域网的其他设备（比如手机）测试，把手机 ip 设为手动填写，填写和之前自动获取时一致的，子网掩码也照抄。把默认网关填写成网关机器的局域网 ip。然后试试还能不能正常打开网页。

如果不能，请留意网关机器是不是开了 Netfilter 防火墙。防火墙的 filter 表的 FORWARD 链的策略是否是 DROP。如果是这个情况，把策略改成 ACCEPT 再试试：

```bash
sudo iptables -P FORWARD ACCEPT
```

局域网转发没有问题了，手机上也设置好网关了，可以进行下一步。

## 配置 Clash DNS

clash 和 v2ray 这类透明网关代理需要知道所代理的流量请求的域名具体是什么，这样才能正常做域名 / IP 分流，在用透明代理的时候才能把请求的原始域名转发到远端服务器让远端服务器解析并连接 CDN 优化过的域名。

关于使用代理时如何进行域名请求请看[这篇文章](https://tachyondevel.medium.com/%E6%BC%AB%E8%B0%88%E5%90%84%E7%A7%8D%E9%BB%91%E7%A7%91%E6%8A%80%E5%BC%8F-dns-%E6%8A%80%E6%9C%AF%E5%9C%A8%E4%BB%A3%E7%90%86%E7%8E%AF%E5%A2%83%E4%B8%AD%E7%9A%84%E5%BA%94%E7%94%A8-62c50e58cbd0)或者[这篇文章](https://blog.skk.moe/post/what-happend-to-dns-in-proxy/)。关于如何配置才能防止 DNS 污染请看 [Clash For Windows 仓库中的 Wiki](https://github.com/Fndroid/clash_for_windows_pkg/wiki/DNS%E6%B1%A1%E6%9F%93%E5%AF%B9Clash%EF%BC%88for-Windows%EF%BC%89%E7%9A%84%E5%BD%B1%E5%93%8D)。

总之，我们需要让被代理机器（和网关机器）的 DNS 请求都走代理软件自己的 DNS 服务器才能正常使用透明代理。所以这一步我们单独设置 Clash DNS 服务器并验证是否正常。（v2ray 是直接进行 DNS 劫持的，不过这里先不说 v2ray。）

`config.yaml` 中追加：

```yaml
dns:
    enable: true
    listen: 0.0.0.0:53
    enhanced-mode: redir-host
    # enhanced-mode: fake-ip
    nameserver:
        - 114.114.114.114
        - 8.8.8.8
```

俺只按照 Clash 作者的基础建议列出来了最简单的直连国内 DNS 的方案。一般来说被污染的解析结果会指向国外，也会走上代理。总之，这里现简单设置以下，更复杂的以后再说。

现在在手机上设置 DNS 服务器为网关机。然后最好是访问一个最近没有上过的国内网站，比如 `360.com`。应该是可以成功访问的。

如果是在路由器上设置 DNS，有可能和已经运行的路由器 DNS 服务冲突，发生 53 端口已被占用的情况。个人建议把路由器的 DNS 服务关掉，如果不方便关掉（路由器用的 dnsmasq 之类同时管理了 DHCP 的软件），可以修改一下端口。比如 dnsmasq 用 5353 端口。

我们先用 `redir-host` 模式而非 `fake-ip` 模式。顾名思义 `fake-ip` 模式下 DNS 服务器会返回一个假的保留 IP 段的 IP，让客户端可以立即发起 TCP 连接而不必等待 Clash DNS 解析，降低响应延迟。但如果单独测试 DNS 功能，返回了假的 IP 显然是连不上的。所以先用 `redir-host` 试一下。

> `fake-ip` 模式下有一些软件或服务可能工作不正常。比如网络连通性测试和时间同步服务等。

## 修改防火墙来 TPROXY 内网其他机器

`config.yaml` 中追加：

```yaml
tproxy-port: 7893
allow-lan: true
```

之后在终端执行如下命令，让被防火墙标记为 1 的数据包发往本地而不按照默认 ip 路由规则被转发（具体规则可以参考 [Xray 的教程](https://xray.sh/documents/level-2/iptables_gid/)）：

```bash
sudo ip rule add priority 2333 fwmark 1 table 100
sudo ip route add local default dev lo table 100
```

> 其中第二条命令的 `dev lo` 是为了配合 `ip route` 的语法规则而添加的，实际上 `dev lo` 是没有特殊作用的，相当于 "No opretation"。数据发回本地还是靠 `local`。

预设好过滤不应被 clash 代理的数据包的链。发往 CGNAT、内网及其他保留地址的数据包，都会直接放行。具体要放行哪些保留地址可以参看 [Reserved IP addresses - Wikipedia](https://en.wikipedia.org/wiki/Reserved_IP_addresses) 和 [RFC5735](https://datatracker.ietf.org/doc/html/rfc5735)：

```bash
sudo iptables -t mangle -N filter-direct
sudo iptables -t mangle -A filter-direct -d 0.0.0.0/8 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 10.0.0.0/8 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 127.0.0.0/8 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 169.254.0.0/16 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 172.16.0.0/12 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 192.168.0.0/16 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 224.0.0.0/4 -j ACCEPT
sudo iptables -t mangle -A filter-direct -d 240.0.0.0/4 -j ACCEPT
```

让所有跳转到 clash 链的不被过滤的 tcp 和 udp 数据都走 TPROXY 代理：

```bash
sudo iptables -t mangle -N clash
sudo iptables -t mangle -A clash -j filter-direct
sudo iptables -t mangle -A clash -p tcp -j TPROXY --on-port 7893 --tproxy-mark 1
sudo iptables -t mangle -A clash -p udp -j TPROXY --on-port 7893 --tproxy-mark 1
```

将所有 tcp/udp 传入流量发往 clash 链：

```bash
sudo iptables -t mangle -A PREROUTING -p tcp -j clash
sudo iptables -t mangle -A PREROUTING -p udp -j clash
```

此时，现前设置使用网关机器上网的手机应该可以顺利上国内和国外网络了。

## 代理本机流量

代理本机流量需要注意的是不要把 clash 自己发出去的流量再度发回本机代理，而是仅仅把未经手 clash 的其他流量重路由回本机：

```bash
sudo iptables -t mangle -N clash-self
sudo iptables -t mangle -A clash-self -j filter-direct
# 如果使用的是 cgroupv2 方案：
sudo iptables -t mangle -A clash-self -m cgroup --path "noproxy.slice" -j ACCEPT
# 如果使用的是 clash-user 方案则改用这一条命令：
# sudo iptables -t mangle -A clash-self -m owner --uid-owner clash -j ACCEPT
sudo iptables -t mangle -A clash-self -j MARK --set-mark 1

sudo iptables -t mangle -A OUTPUT -p tcp -j clash-self
sudo iptables -t mangle -A OUTPUT -p udp -j clash-self
```

现在可以把本机网络连接使用的 DNS 服务器改成 Clash 自己的 `127.0.0.1`，然后把之前设置的系统 / 浏览器的 http / socks 代理取消掉，改为“直连”。“直连”的流量会直接经过 clash 透明代理，此时试验一下应该是可以上国内国外网络的，只是的 TCP 连接的建立会比 `fake-ip` 慢一些。

### cgroup 方案下放行其他软件

如果是 systemd 服务则参考之前放行 clash 的方法，将服务的 cgroup 设置为 `noproxy.slice`。

如果不是 / 不想用 systemd 服务，可以用以下方式来以普通用户身份按需启动不希望被代理的进程（以 `qbittorrent` 为例）：

```bash
systemd-run --user --slice "noproxy.slice" --scope qbittorrent
```

这种启动方式也可以写到 [.desktop 文件](https://wiki.archlinux.org/title/Desktop_entries)中：

```bash
desktop-file-install --dir ~/.local/share/applications/ --set-key=Exec --set-value="systemd-run --user --slice=noproxy --scope qbittorrent %U" /usr/share/applications/org.qbittorrent.qBittorrent.desktop
```

之后按照此进程的 cgroup 路径配置防火墙试一下：

```bash
sudo iptables -t mangle -A clash-self -m cgroup --path "user.slice/user-$UID.slice/user@$UID.service/noproxy.slice" -j ACCEPT
```

## Fake ip

> v2ray 透明代理中 dns 的配置方法是 redirect dns 请求到 v2ray，然后 v2ray 进行 dns 劫持。这样你就可以用 netfilter 控制是否劫持进程的 dns 请求。但 clash 没有设计 dns 劫持。导致不能直接在 netfilter 中依靠 cgroup 区别配置进程的 dns 请求。
>
> 俺以为，也许使用更复杂的本机 dns 服务软件来分流 dns 请求，配合 clash dns，应该也可以做到与 v2ray 类似的效果。不过俺懒得弄了，所以俺又用回了 redir-host。

现在我们可以把 `redir-host` 改成 `fake-ip`，并重启 clash 服务来看是否能用上 fake-ip 模式了。注意如果你改过你的 `fake-ip-range`，不要让它被 `filter-direct` 链给放行了。俺当然是没改过的。

尝试解析一下域名看看是否已经生效了：

```bash
getent hosts baidu.com
# 198.18.0.9      baidu.com

ping baidu.com
# PING baidu.com (198.18.0.9) 56(84) bytes of data.
# ^C
# --- baidu.com ping 統計 ---
# 送信パケット数 11, 受信パケット数 0, 100% packet loss, time 10132ms
```

`fake-ip` 模式会比较快，但是有些奇怪的网络管理器在检查网络连通性的时候用 DNS 检查他们的连通测试域名时如果不得到真实的 IP 地址似乎就会直接回报不通。而不是实际建立 HTTPS 链接并且下载数据测试。GNOME 的 NetworkManager 和 Windows 10 似乎都是。这时候需要使用 `fake-ip-filter`。具体可能会有问题的域名的列表可以看 [OpenClash 仓库的实现](https://github.com/vernesong/OpenClash/blob/master/luci-app-openclash/root/etc/openclash/custom/openclash_custom_fake_filter.list)。

另外，火狐等浏览器在用户在地址栏中输入了简单的内容并回车的时候也会在进行搜索的同时，也进行 DNS 解析。如果 DNS 解析得到了结果（此处是 fake-ip），便会提出“是否要跳转到 XXX 网址”的提示，在用 fake-ip 的时候就很麻烦.所以建议把这种单词也用 `"*"` 白名单掉。

`config.yaml` 中对应部分追加：

```yaml
dns:
    fake-ip-filter:
        - '*'
        - '*.lan'
        - localhost.ptlogin2.qq.com
        - ping.archlinux.org
        - nmcheck.gnome.org
        - '*.msftconnecttest.com'
        - '*.msftncsi.com'
```

## 配置持久化

Linux 路由表和 Netfilter 策略路由默认情况下关机后会重置。如果配置过程中出了拿不准的错误无法上网了，可以直接重启电脑恢复。如果确定没问题了，可以将路由表配置命令用某种方法在网络服务上线时自动执行，将 Netfilter 配置写入到防火墙服务启动时执行的配置文件中。

### 路由表

我们可以参照 `/usr/lib/systemd/system/iptables.service` 写一个 oneshot systemd.service。

```bash
sudo systemctl edit --force --full tproxy-ip-route.service
```

写入如下内容：

```ini
[Unit]
Description=Setup ip-rule and ip-route for tproxy local network traffic.
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=ip rule add priority 2333 fwmark 1 table 100
ExecStart=ip route add local default dev lo table 100
ExecStop=ip rule del table 100
ExecStop=ip route flush table 100
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

之后开机启动它：

```bash
sudo systemctl enable tproxy-ip-route.service
```

### Netfilter 配置

可以用 `systemctl cat iptables`（如果用的 nftables 就看 nftables.service）看一下自己的 systemd 服务在启动时会读哪个文件，然后 `sudo iptables-save -f` 保存到那个文件上去。下次开机就会读取那个文件里的配置了。不过为了配置文件干净易懂，俺喜欢将生成的规则文件自己手动改简明一些。

#### iptables 配置

以下是 cgroup 方案的配置，clash-user 方案请自行修改 `--path` 选项。

```iptables
*mangle
:clash -
:clash-self -
:filter-direct -
-A PREROUTING -p tcp -j clash
-A PREROUTING -p udp -j clash
-A OUTPUT -p tcp -j clash-self
-A OUTPUT -p udp -j clash-self
-A clash -j filter-direct
-A clash -p tcp -j TPROXY --on-port 7893 --tproxy-mark 1
-A clash -p udp -j TPROXY --on-port 7893 --tproxy-mark 1
-A clash-self -j filter-direct
-A clash-self -m cgroup --path "noproxy.slice" -j ACCEPT
-A clash-self -j MARK --set-mark 1
-A filter-direct -d 0.0.0.0/8 -j ACCEPT
-A filter-direct -d 10.0.0.0/8 -j ACCEPT
-A filter-direct -d 127.0.0.0/8 -j ACCEPT
-A filter-direct -d 169.254.0.0/16 -j ACCEPT
-A filter-direct -d 172.16.0.0/12 -j ACCEPT
-A filter-direct -d 192.168.0.0/16 -j ACCEPT
-A filter-direct -d 224.0.0.0/4 -j ACCEPT
-A filter-direct -d 240.0.0.0/4 -j ACCEPT
COMMIT
```

user cgroup 规则在对应的 cgroup 已经存在的时候才能写入。如果想要避免在登陆后用 sudo 提权创建防火墙规则，需要创建一个 systemd.path 和 systemd.service 来监控 cgroup 路径的创建，并自动加入对应的防火墙规则：

```bash
sudo systemctl edit --force --full tproxy-user-cgroup@.path
```

填入：

```ini
[Unit]
Description=Monitor the creation of the user cgroup for UID %i.

[Path]
PathExists=/sys/fs/cgroup/user.slice/user-%i.slice/user@%i.service/noproxy.slice/

[Install]
WantedBy=multi-user.target
```

编写对应的 systemd.service：

```bash
sudo systemctl edit --force --full tproxy-user-cgroup@
```

填入：

```ini
[Unit]
Description=Add iptables bypass rule for user cgroup of UID %i.

[Service]
Type=oneshot
ExecStart=iptables -t mangle -I clash-self -m cgroup --path "user.slice/user-%i.slice/user@%i.service/noproxy.slice" -j ACCEPT
RemainAfterExit=yes
```

最后启用：

```bash
sudo systemctl enable tproxy-user-cgroup@$UID.path
```

> TODO: 俺应该再研究研究把这个配置方法和 iptables 服务的停止和重启也配合好。另外，最近发现最方便的配合 qBittorrent 使用的方法还是早早的匹配并代理 RSS 源之后立刻按照下载软件进程名过滤，这样可以兼顾 RSS 订阅和代理软件直连。而且其实就不需要配置这种比较麻烦的用户级进程放行了。

#### nftables 配置

nftables 截至俺写这篇文章的时候还不支持 `meta cgroupv2`，所以这里只留 clash-user 方案的配置：

```nft
flush ruleset
table ip mangle {
  chain filter-direct {
    ip daddr { 0.0.0.0/8, 10.0.0.0/8, 127.0.0.0/8, 169.254.0.0/16, 172.16.0.0/12, 192.168.0.0/16, 224.0.0.0/4, 240.0.0.0/4 } accept
  }

  chain clash {
    jump filter-direct
    ip protocol { tcp, udp } tproxy ip to :7893 meta mark set 1
  }

  chain PREROUTING {
    type filter hook prerouting priority mangle; policy accept;
    ip protocol { tcp, udp } jump clash
  }

  chain clash-self {
    jump filter-direct
    meta skuid clash accept
    meta mark set 1
  }

  chain OUTPUT {
    type route hook output priority mangle; policy accept;
    ip protocol { tcp, udp } jump clash-self
  }
}
```

### 固定使用本机 DNS

之前提到 clash 是不能通过 Netfilter 的 REDIRECT 方法劫持 DNS 请求的，所以也不能用 Netfilter 来持久化使用 clash dns 这一配置。俺是直接修改 `/etc/resolv.conf` 来使用本地 Clash DNS 的：

```bash
nameserver 127.0.0.1
```

NetworkManager 会在连接网络的时候修改这个文件。为了在连接上每个不同的网络时都能用上本地的 DNS，建议给这个文件加上[只读的属性](https://wiki.archlinux.org/title/File_permissions_and_attributes#File_attributes)：

```bash
sudo chattr +i /etc/resolv.conf
```
