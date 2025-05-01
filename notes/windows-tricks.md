---
tags: [Blog/System Configurations]
title: 初装Windows后的设置备忘
---

## 在商店版火狐中使用树形标签页

首先安装 [Sidebery](https://addons.mozilla.org/en-US/firefox/addon/sidebery/)，并调整树形标签页功能。

之后用 userChrome.css 隐藏火狐顶部的原生标签页栏：
1. 从[www.userchrome.org](https://www.userchrome.org/how-create-userchrome-css.html)得知商店版的火狐的 Profile 文件夹的位置类似如下路径：
   ```
   %LOCALAPPDATA%\Packages\Mozilla.Firefox_n80bbvh6b1yt2\LocalCache\Roaming\Mozilla\Firefox\Profiles\44ceiwus.default-release
   ```
   其中 Mozilla.Firefox_n80bbvh6b1yt2 和 44ceiwus.default-release 每次的安装可能会有不同，需要自行寻找对应的路径。
2. 创建 Chrome 文件夹并写入 userChrome.css，参见俺的 [dotfiles](https://github.com/escape0707/dotfiles/tree/main/private_dot_mozilla/private_firefox/chrome)。
3. 对于现代版本的火狐，还需要在 about:config 中启用 `toolkit.legacyUserProfileCustomizations.stylesheets` 让火狐读取这个文件。[参见](https://www.userchrome.org/how-create-userchrome-css.html#aboutconfig)