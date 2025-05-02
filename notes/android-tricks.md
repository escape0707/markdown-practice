---
tags: [Blog/System Configurations]
title: Android 的软件和设置建议
---

## 移除动画

对俺来说，安卓操作系统的动画速度有些太慢了，所以俺从来都是把他们关掉。在系统设置的 Accessibility > Color and motion > Remove animations 处可以一并关掉所有动画。但是这样会使 Animator Duration Scale 也被设置为 0，导致很多系统 UI 和软件的 UI 行为异常。所以俺都是再此基础上用 adb shell 单独配置 Animator Duration Scale 为 0.01 来避免这类问题：

```sh
adb shell settings put global window_animation_scale 0
adb shell settings put global transition_animation_scale 0
adb shell settings put global animator_duration_scale 0.01
```

使用 adb 并 wifi 配对的时候需要注意，arch 自带的 extras/android-tools 是编译自非官方仓库 <https://github.com/nmeum/android-tools> 。它并[没有包括 mdns 支持](https://github.com/nmeum/android-tools/issues/118)。所以请用 USB 连接，或者手动指定 IP 和端口连接。
