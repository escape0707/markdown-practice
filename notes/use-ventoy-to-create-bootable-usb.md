---
tags: [Blog/Utilities]
title: 使用 Ventoy 创建 USB 启动盘
---

- [下载 Ventoy](#下载-ventoy)
- [制作启动盘](#制作启动盘)
- [为 Ventoy 设置专门的存放路径](#为-ventoy-设置专门的存放路径)
- [拷贝安装镜像](#拷贝安装镜像)
- [创建完成后的使用](#创建完成后的使用)

## 下载 Ventoy

从[官网](https://www.ventoy.net)手动下载，或：

### Windows

用 Scoop 下载：

```powershell
scoop install ventoy
```

### Linux

ArchLinux 可以从 AUR 或 ArchlinuxCN 下载：

```bash
paru -S ventoy-bin
```

## 制作启动盘

[官方指南](https://www.ventoy.net/cn/doc_start.html)

### Windows

- 选择要使用的 USB 盘
- 分区类型选择`GPT`，如果你的目标设备主板太老了不支持则选择`MBR`。（默认值其实也能胜任。）
- 点击`Install`开始制作

### Linux

ArchLinux:

```bash
ventoy -i -g /path/to/your/usb/drive
```

## 为 Ventoy 设置专门的存放路径

默认情况下，Ventoy 制作的启动盘启动后会搜索此盘内的所有可启动镜像文件。如果在此盘中存放了太多其他文件的话，这一过程会变得很慢。所以最好创建专门存放镜像文件的目录，并告知 Ventoy 只需搜索那里。

官方文档——[控制 Ventoy 搜索路径的方法总结](https://www.ventoy.net/cn/doc_search_path.html)。

我们可在根目录下创建一个`ventoy`文件夹，其中创建一个`images`文件夹来存放各路安装盘镜像、以及一个`json`文件`ventoy.json`，其中填入以下内容即可：

```json
{
  "control": [{ "VTOY_DEFAULT_SEARCH_ROOT": "/ventoy/images" }]
}
```

## 拷贝安装镜像

将下载好的 Windows 安装镜像 ISO 或者 ArchLinux 安装镜像 ISO 拷贝到刚刚创建的`ventoy/images`目录中即完成了安装盘的创建。

## 创建完成后的使用

需要使用这个启动盘的时候，按照常规，在主板中设置好 USB 启动，顺利启动 Ventoy 环境之后直接选择这次想用的镜像即可。
