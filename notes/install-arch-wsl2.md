# 在 Windows 上安装 WSL 2 的 ArchLinux

> 目前 WSL 2 上还没有比较方便的使用 Windows 中代理的方法，笔者暂时回退到了 WSL 1。文中配置 Linux 的部分还是完全可以参考的。

先参考微软官方 [WSL 启用文档](https://docs.microsoft.com/en-us/windows/wsl/install-win10)。

> 为了启用 WSL 2 请至少升级到 2004 版本的 Windows 10。

执行：

```powershell
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
```

重启电脑。

执行：

```powershell
wsl --set-default-version 2
```

> 如果看到`WSL 2 requires an update to its kernel component. For information please visit https://aka.ms/wsl2kernel`，请看链接中网站的提示进行操作。之后重新运行上文命令。

之后可以去 Microsoft Store 下载自己喜欢的 Linux 发行版。这里俺使用 ArchLinux，所以只好去 GitHub 找 [yuk7 君自制的安装包](https://github.com/yuk7/ArchWSL)来用。感觉不错！

你甚至可以使用 scoop 来安装：

```powershell
scoop install archwsl
```

安装好 Arch 之后，按照 [yuk7 君的建议](https://github.com/yuk7/ArchWSL/wiki/How-to-Setup#setting-for-arch)配置账户和 sudo、keyring。

```powershell
arch
```

```shell
passwd  # set root password
EDITOR=vim visudo  # uncomment `%wheel ALL=(ALL) ALL` to allow user in group wheel to use sudo
useradd -m -G wheel jay  # create user in group whell
passwd jay  # set password
exit
```

```powershell
arch config --default-user jay  # set as default user
arch
```

```shell
sudo pacman-key --init
sudo pacman-key --populate
```

此时便已经有了一个基于 WSL 2 的 ArchLinux 系统，此系统使用 Windows 的网络。

> 目前 Windows 下的代理软件的端口还不能直接用 localhost 访问。WSL 1 可以。Linux 下软件开的端口 Windows 下的软件可以直接访问。

接下来便可以根据俺之前的 [ArchLinux 系统安装](install-arch-on-laptop-and-vm.md#安装arch-linux)进一步配置包管理、代理、并安装各种软件、依赖了。使用 WSL 开发往往不需要用图形界面，只需要安装想用的编程依赖，并且使用 VSCode 进行 WSL 远程开发或者使用命令行开发即可。

启用 ArchLinux 腾讯源

```shell
echo "Server = https://mirrors.cloud.tencent.com/archlinux/\$repo/os/\$arch" | sudo tee /etc/pacman.d/mirrorlist > /dev/null
```

更新所有软件包

```shell
sudo pacman -Syu
```

安装`neovim`并将`vi`软链接过去

```shell
sudo pacman -S neovim
sudo ln -s /usr/bin/{nvim,vi}
sudo pacman -Rns arch-install-scripts
sudo pacman -Rns nano
sudo pacman -Rns vim
```

修改 `/etc/pacman.conf`，添加 ArchLinuxCN 源

```shell
sudo -e /etc/pacman.conf
```

将 SigLevel 的值改为 PackageRequired，并在结尾增加下文

```text
[archlinuxcn]
Server = https://mirrors.cloud.tencent.com/archlinuxcn/$arch
```

安装`archlinuxcn-keyring`

```shell
sudo pacman -Sy archlinuxcn-keyring
```

安装`powerpill`和`yay`

```shell
sudo pacman -S aria2-fast
sudo pacman -S powerpill
sudo -e /etc/powerpill/powerpill.json
```

主要需要调整其中`aria2`参数部分（如果安装的原版`aria2`、单服务器最大连接数必须小于等于`16`）：

```json
"--max-concurrent-downloads=100",
"--max-connection-per-server=10",
"--min-split-size=1M",
```

```shell
sudo powerpill -S yay
```

设置 Git

```shell
git config --global user.name escape0707
git config --global user.email tothesong@gmail.com
git config --global http.proxy socks5h://127.0.0.1:7890
git config --global credential.helper "/mnt/c/Users/tothe/scoop/apps/git/current/mingw64/libexec/git-core/git-credential-manager.exe"
git config --global merge.ff only
git config --global pull.ff only
```

下载并连接 dotfiles。俺有一个[代码仓库](https://github.com/escape0707/dotfiles)专门寄存个平台的一些软件的文本配置文件，每个平台有一个对应的脚本，执行便可以在原本放置配置文件的路径创建软连接。使用和修改配置文件的软件不会察觉与直接放置了一个配置文件在原处有何不同。反倒是修改可以统一反映在我们克隆下来的这个`dotfiles`文件夹下。方便用 git 管理版本和同步。

```shell
mkdir -p ~/Workspaces/dotfiles
git clone https://github.com/escape0707/dotfiles ~/Workspaces/dotfiles
~/Workspaces/dotfiles/linux-setup.sh
```

启用 `zsh`

```shell
chsh -s /bin/zsh
exit
```

```powershell
arch
```

软件列表

```shell
pacman -Qqtt
```

```text
archlinuxcn-keyring
bandit
base
flake8
go
mypy
neovim
oh-my-zsh-git
powerpill
python-black
python-pylint
yay
zsh-completions
```

glibc-2.30-3-x86_64.pkg.tar.xz

```shell
curl 'https://archive.archlinux.org/packages/g/glibc/glibc-2.30-3-x86_64.pkg.tar.xz' -o
sudo pacman -U glibc-2.30-3-x86_64.pkg.tar.xz
sudo -e /etc/pacman.conf
ignore
```
