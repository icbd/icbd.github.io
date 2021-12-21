---
layout: post
title:  Makefile Basic
date:   2021-12-20
Author: CBD
tags:   [make]
---

GitLab 除了一个的基于 Ruby on Rails 的 Repo, 还有多个用 Golang 写的组件/中间件, 这些项目通常用 make 来管理编译和测试.

这里我们介绍一下读懂 make 必须知道的基础知识.

makefile 的文件名可以是 `makefile` 或者 `Makefile`.
在含有 makefile 的目录下, 使用 `make` 来触发默认任务, 或者 类似 `make clean` 来触发特殊的任务.

基本语法:

```makefile
target: prerequisite
  command
```

makefile 的缩进使用 tab 而不是空格.

在这样一组任务中, target 可以写一个或者多个; prerequisite 是可选的, 可以不写或者写一个/多个; command 就是 shell 命令.

target 可以是目标文件, 也可以是可执行文件, 还可以是一个标签.

当任务没有 prerequisite 时, 也就是说 target 没有依赖, 那么对应的 command 就不会自动执行.

当任务需要执行时, 如果 prerequisite 的变化早于 target, 那么就执行 command, 否则就跳过 command.

make 不带参数时, 即以第一组任务为默认任务, make 会解析并执行所需要的 prerequisite .

如果想把 target 当作标签使用, 但是又存在一个同名文件时, 需要使用 `.PHONY: target` 声明 target 为虚拟的.

make 的工作方式:

1. 读入所有的Makefile.
1. 读入被include的其它Makefile.
1. 初始化文件中的变量.
1. 推导隐晦规则, 并分析所有规则.
1. 为所有的目标文件创建依赖关系链.
1. 根据依赖关系, 决定哪些目标要重新生成.
1. 执行生成命令.

## Reference

[B站: 正月点灯笼](https://www.bilibili.com/video/BV1Mx411m7fm)

[https://seisman.github.io/how-to-write-makefile/introduction.html](https://seisman.github.io/how-to-write-makefile/introduction.html)
