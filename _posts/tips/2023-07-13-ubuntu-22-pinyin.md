---
layout: post
title:  Ubuntu 22.04 yinpin
date:   2023-07-13
Author: CBD
tags: [Ubuntu]
---

安装帮助参考官方文档：
https://pinyin.sogou.com/linux/help.php


GUI 配置 fcitx：

```sh
fcitx-config-gtk3
```

## vscode 中搜狗输入法失效

如果是从 Ubuntu 的应用市场安装的 vs code，搜狗拼音的提示框会失效， 如果执行 `fcitx-config-gtk3` 还会报错。

```log
/snap/core20/current/lib/x86_64-linux-gnu/libstdc++.so.6: version `GLIBCXX_3.4.29' not found (required by /lib/x86_64-linux-gnu/libproxy.so.1)
Failed to load module: /home/icbd/snap/code/common/.cache/gio-modules/libgiolibproxy.so
fcitx-config-gtk3: symbol lookup error: /snap/core20/current/lib/x86_64-linux-gnu/libpthread.so.0: undefined symbol: __libc_pthread_init, version GLIBC_PRIVATE
```

最简单的处理方式是卸载应用商店的 vs code，从官方下载 deb 安装包，并用 apt 安装。

## 更精细的配置

See: `~/.config/sogoupinyin/conf/env.ini`
