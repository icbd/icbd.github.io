---
layout: post
title:  git catfile and object file frame
date:   2022-02-26
Author: CBD
tags:   [Git]
---

在仓库的 `.git/objects` 目录下, 保存着 git 的 object.

后面以这个 [Demo Repo](https://jihulab.com/show/git_objects_demo) 为例.

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

## cat-file

查看 object 的类型:

```sh
git cat-file -t 16796efecb4599c92244ac8bafb217e20009008e
```

```text
blob
```

查看 object 的内容:

```sh
git cat-file -p 16796efecb4599c92244ac8bafb217e20009008e
```

```text
# Help

```

## save object to disk

Git object 有几个类型:

- blob
- tree
- commit

Git object 内部以文本为基础, 经过特殊的编排, 再经过压缩, 最终得到的是二进制文件.

为了查看其内部的原始文本, 第一步就是需要接开压缩.

我们定义一个解压脚本, 参数是 Git object 的文件路径:

```sh
alias inflate="ruby -r zlib -e \"ARGV.each {|path| puts Zlib::Inflate.inflate File.read(path) }\""
```

后面还会用到 `hexdump`, 它会帮我们以 16 进制的形式来展示内容本来的面目, 特别注意那些不能打印的字符.

常见字符对照:

|16 进制|字符|
|---|---|
|`00`|NULL 空|
|`20`|空格|
|`21`|叹号 `!`|
|`22`|引号 `"`|
|`23`|井号 `#`|
|`2E`|句号 `.`|
|`30`|数字 `0`|
|`39`|数字 `9`|
|`41`|字母 `A`|
|`5A`|字母 `Z`|
|`61`|字母 `a`|
|`7A`|字母 `z`|
|`0A`|换行 line feed|
|`0D`|回车 carriage return|

`hexdump -C` 既会打印 16 进制的内容, 也会打印对应的可视字符(不可打印的用 `.` 表示).

### blob

`git cat-file -p 16796efecb4599c92244ac8bafb217e20009008e`

```text
# Help

```

`inflate .git/objects/16/796efecb4599c92244ac8bafb217e20009008e | hexdump -C`

```text
00000000  62 6c 6f 62 20 38 00 23  20 48 65 6c 70 0a 0a     |blob 8.# Help..|
0000000f
```

blob 编码结构:

|---|---|---|---|---|
|blob|空格|内容长度|NULL|文件内容|

### tree

`git cat-file -p 11ef4924f8f1e21e6d7412040973f5e868559ac7`

```text
100644 blob 16796efecb4599c92244ac8bafb217e20009008e	help.md
100644 blob f20b9ed4f8ba4e99df6a2491bbed85e8daf526d5	info.md
```

`inflate .git/objects/11/ef4924f8f1e21e6d7412040973f5e868559ac7 | hexdump -C`

```text
00000000  74 72 65 65 20 37 30 00  31 30 30 36 34 34 20 68  |tree 70.100644 h|
00000010  65 6c 70 2e 6d 64 00 16  79 6e fe cb 45 99 c9 22  |elp.md..yn..E.."|
00000020  44 ac 8b af b2 17 e2 00  09 00 8e 31 30 30 36 34  |D..........10064|
00000030  34 20 69 6e 66 6f 2e 6d  64 00 f2 0b 9e d4 f8 ba  |4 info.md.......|
00000040  4e 99 df 6a 24 91 bb ed  85 e8 da f5 26 d5 0a     |N..j$.......&..|
0000004f
```

tree 编码结构:

|---|---|---|---|---|---|---|
|tree|空格|内容长度|NULL|<文件名a编码>|<文件名b编码>|换行 LF|

文件名编码结构:

|---|---|---|---|---|
|文件权限, 10064|空格|文件名|空格|blob SHA, 40位 hex|

## commit

`git cat-file -p 6915089af93052dc6f67fd7245037230a6b1894b`

```text
tree 6789c99cbe4a25ecc9e69eb1e66df819ddfab2e5
author Baodong <wwwicbd@gmail.com> 1645015448 +0800
committer Baodong <wwwicbd@gmail.com> 1645015448 +0800

Init
```

`inflate .git/objects/69/15089af93052dc6f67fd7245037230a6b1894b | hexdump -C`

```text
00000000  63 6f 6d 6d 69 74 20 31  35 39 00 74 72 65 65 20  |commit 159.tree |
00000010  36 37 38 39 63 39 39 63  62 65 34 61 32 35 65 63  |6789c99cbe4a25ec|
00000020  63 39 65 36 39 65 62 31  65 36 36 64 66 38 31 39  |c9e69eb1e66df819|
00000030  64 64 66 61 62 32 65 35  0a 61 75 74 68 6f 72 20  |ddfab2e5.author |
00000040  42 61 6f 64 6f 6e 67 20  3c 77 77 77 69 63 62 64  |Baodong <wwwicbd|
00000050  40 67 6d 61 69 6c 2e 63  6f 6d 3e 20 31 36 34 35  |@gmail.com> 1645|
00000060  30 31 35 34 34 38 20 2b  30 38 30 30 0a 63 6f 6d  |015448 +0800.com|
00000070  6d 69 74 74 65 72 20 42  61 6f 64 6f 6e 67 20 3c  |mitter Baodong <|
00000080  77 77 77 69 63 62 64 40  67 6d 61 69 6c 2e 63 6f  |wwwicbd@gmail.co|
00000090  6d 3e 20 31 36 34 35 30  31 35 34 34 38 20 2b 30  |m> 1645015448 +0|
000000a0  38 30 30 0a 0a 49 6e 69  74 0a                    |800..Init.|
000000aa
```

commit 编码结构:

|---|---|---|---|
|commit|空格|内容长度|NULL|

|---|---|---|---|
|tree|空格|object SHA|换行 LF|

|-----------|---|---|---|---|---|---|---|---|---|---|---|
|author     |空格|作者名字|空格|<|作者 email|>|空格|时间戳|空格|时区, +0800|换行 LF|

|-----------|---|---|---|---|---|---|---|---|---|---|---|
|committer  |空格|提交者名字|空格|<|作者 email|>|空格|时间戳|空格|时区, +0800|换行 LF|

|---|---|---|
|换行 LF|commit message|换行 LF|
