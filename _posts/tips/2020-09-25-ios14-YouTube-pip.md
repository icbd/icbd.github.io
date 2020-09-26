---
layout: post
title:  IOS 14 recover YouTube Picture-in-Picture
date:   2020-09-25
Author: CBD
tags: [tips]
---

IOS 14 的 Safari 带来了画中画 ( picture-in-picture ) 功能, 但这直接冲击了 YouTube APP 的付费功能, 其中的两大卖点:

1. 去广告;
2. 后台仅播放声音;

去广告可以结合 Safari 的广告拦截器使用; 后台播放可以结合画中画, 锁屏后再点一下播放键即可.

不出一周 YouTube 就封禁了 Safari 的画中画功能, 使其自动关闭.

## 快捷访问

我们可以借助快捷访问, 在 Safari 中执行自己的 JS 代码来实现反封禁:

```js
let v = document.querySelector("video");
v.addEventListener("webkitpresentationmodechanged", (e) => e.stopPropagation(), true);
v.webkitSetPresentationMode("picture-in-picture");
completion();
```

## 操作演示

<blockquote class="twitter-tweet" data-lang="zh-cn"><p lang="zh" dir="ltr">YouTube反对画中画，我反对！ <a href="https://t.co/CVXiG1KTtC">pic.twitter.com/CVXiG1KTtC</a></p>&mdash; CbdFocus (@CbdFocus) <a href="https://twitter.com/CbdFocus/status/1309487092657922048?ref_src=twsrc%5Etfw">2020年9月25日</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>
