---
layout: post
title:  Git 解压缩 pack
date:   2022-02-26
Author: CBD
tags:   [Git]
---

在日常的工作流中, 几乎不会用到这个命令 `git unpack-objects`, 研究 Git 内部原理的时, 这是个有用的工具.

我们想以 [Demo Repository](https://jihulab.com/show/git_objects_demo) 为例, 解释 `.git/objects` 的原理.

该 repo 托管在远端的服务器, clone 下来之后, 发现 `.git/objects` 目录下, 竟然没有任何的 object 文件, 却多了 pack 目录下的 pack 和 idx 文件:

```text
.git/objects
├── info
└── pack
    ├── pack-884fb3533060bd5ded71bebc8fc576546614d86c.idx
    └── pack-884fb3533060bd5ded71bebc8fc576546614d86c.pack
```

这是 因为 Git 会把众多 object 打包压缩成 pack 和 idx, 提高存储和传输效率.

## 解开 pack 文件

`git unpack-objects` 的参数是 pack 文件的内容, 可以 `cat *.pack | git unpack-objects` 或者 `git unpack-objects < *.pack` .

`unpack-objects` 命令的意图主要是为了从外部恢复 objects 使用, 所以 , 如果 `.git/objects/pack` 下已经存在了 pack 文件, 那么这个命令就什么也不做, 不会把 objects 展开到 `.git/objects` 下面.

我们可以先把 pack 文件移走, 再执行展开:

```sh
cd git_objects_demo
mv .git/objects/pack/*.pack .
git unpack-objects < *.pack
rm -f .git/objects/pack/*.idx
```

解压之后就跟本地的效果一致了.

```text
.git/objects
├── 11
│   └── ef4924f8f1e21e6d7412040973f5e868559ac7
├── 16
│   └── 796efecb4599c92244ac8bafb217e20009008e
├── 20
│   └── 61bac5e1a292b380f98072ef5ffbc2fc3eefa9
├── 53
│   └── 33c1763419c89486091d80b6ba430d30b321a0
├── 67
│   └── 89c99cbe4a25ecc9e69eb1e66df819ddfab2e5
├── 69
│   └── 15089af93052dc6f67fd7245037230a6b1894b
├── d0
│   └── 38440f5c27d52fed6b6c5eb7d180f1645ab724
├── f2
│   └── 0b9ed4f8ba4e99df6a2491bbed85e8daf526d5
├── info
└── pack
```
