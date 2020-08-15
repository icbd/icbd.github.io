---
layout: post
title:  Make Win10 USB on MacOS 
date:   2020-08-15
Author: CBD
tags: [tips]
---

最新版的 Windows10 安装 ISO 中有一个巨大的文件 `install.wim`.
 其大小已经超过 4G, 这个 FAT 文件格式的单文件最大限制, 需要做额外的大文件切分处理.
 
## 下载安装镜像

[https://www.microsoft.com/zh-cn/software-download/windows10ISO](https://www.microsoft.com/zh-cn/software-download/windows10ISO)

双击下载的 iso 文件, Finder 中会看到新的 Volume.

## 格式化 U盘

查看 disk 编号, 抹掉U盘对应的 disk.
```
diskutil list
diskutil eraseDisk MS-DOS "WINDOWS10" MBR diskX
```

## 下载分割工具

```
brew install wimlib
```

## 拷贝安装文件

复制除了 `install.wim` 的其他文件:

```
rsync -avh --progress --exclude=sources/install.wim /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/ /Volumes/WINDOWS10
```

复制 `install.wim` 的分割文件, 每个子文件不超过 3800 M:

```
wimlib-imagex split /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/sources/install.wim /Volumes/WINDOWS10/sources/install.swm 3800
```

如果 `install.wim` 文件没有超过 4G, 可以直接复制:

```
cp -rp /Volumes/CCCOMA_X64FRE_ZH-CN_DV9/* /Volumes/WINDOWS10/
```

## 小结

* U盘 文件格式为 FAT32;

* 使用 wimlib 分割超过 4G 的文件;
