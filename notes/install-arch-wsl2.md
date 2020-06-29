# 在 Windows 上安装 WSL 2 的 ArchLinux

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

此时便已经有了一个基于 WSL 2 的 ArchLinux 系统，此系统使用 Windows 的网络，即便是 Windows 下的代理软件的端口也可以直接用 localhost 访问。

接下来便可以根据俺之前的 [ArchLinux 系统安装](install-arch-on-laptop-and-vm.md#安装arch-linux)进一步配置包管理、代理、并安装各种软件、依赖了。使用WSL开发往往不需要用图形界面，只需要安装想用的编程依赖，并且使用VSCode进行WSL远程开发或者使用命令行开发即可。

启用 ArchLinux 腾讯源

```shell
echo "Server = https://mirrors.cloud.tencent.com/archlinux/\$repo/os/\$arch" | sudo tee /etc/pacman.d/mirrorlist
```

更新所有软件包

```shell
sudo pacman -Syu
```

安装`neovim`并将`vi`软链接过去，删除`vim`

```shell
sudo pacman -S neovim
sudo ln -s /usr/bin/{nvim,vi}
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

安装`powerpill`和`yay`

```shell
sudo pacman -S aria2-fast
sudo pacman -S powerpill
sudo powerpill -S yay
```

alias pacup yay
