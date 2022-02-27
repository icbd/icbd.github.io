---
layout: post
title:  Git 仓库结构布局
date:   2022-02-21
Author: CBD
tags:   [Git]
---

[https://git-scm.com/docs/gitrepository-layout](https://git-scm.com/docs/gitrepository-layout)

Git 仓库有两种风格:

* 位于工作目录下的 `.git`;
* 独立的 `<project>.git` 目录, 这是一个裸仓库, 没有自己的工作区, 用于跟其他人同步历史记录.

*注意*: `.git` 也可以是工作区下的一个普通文件, 其内容是 `gitdir: <path>` 来指向仓库的实际目录.
这个机制常用在子模块的 checkout, 为的是让父项目 `git checkout` 到一个不含子模块的分支的时候,
不会丢失子模块的仓库, 因为 `checkout` 会清空整个子模块的工作区.

Git 仓库中的常见元素.

### objects

  该目录为仓库的对象库.
  通常来说, 每个对象都指向存储内的一个对象, 但也有几个例外.

  通过浅 clone , 能获得一个可用但不完整的本地仓库;
  可以借助 `objects/info/alternates` 或者 `$GIT_ALTERNATE_OBJECT_DIRECTORIES` 机制从其他对象库里"借"一些对象;

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/objects` .

## objects/[0-9a-f][0-9a-f]

	对象文件.
  所有的对象会被分摊到 256 个子目录中, 目录的名字取自对象的 SHA1 字符串的前两位.
  从这里查找对象一般称作 "拆箱" (unpacked / loose) 对象.

## objects/pack

  该目录存储压缩的对象和索引, 以便随机访问.

## objects/info

  该目录保存对象库的额外信息.

## objects/info/packs

  该文件为哑传输记录对象库可用的打包.
  如果仓库通过哑传输发布, 一旦打包被新加或者移除, 都应该执行 `git update-server-info` 来更新该文件.
  `git repack` 会默认这样做.

## objects/info/alternates

	该文件记录对象库中所 "借" 对象的来源, 每条记录占一行.
	不仅原生的 Git 工具在本地使用它, HTTP 访问的也会远程访问它.
	一般使用相对路径. 如果是绝对路径, 需要满足跟 web URL 相匹配.

## objects/info/http-alternates

	该文件记录了对象库中所 "借" 对象的来源, 这些记录是 URL 的形式.

## refs

	引用存储在这个目录的子目录下.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/refs` .

## refs/heads/`name`

	该文件记录分支最上面那个 commit 对象.

## refs/tags/`name`

	该文件记录 tag 指向的对象. 

## refs/remotes/`name`

	该文件远端分支最上面的 commit 对象.

## refs/replace/`<obj-sha1>`

	records the SHA-1 of the object that replaces `<obj-sha1>`.
	This is similar to info/grafts and is internally used and
	maintained by linkgit:git-replace[1]. Such refs can be exchanged
	between repositories while grafts are not.

## packed-refs

	records the same information as refs/heads/, refs/tags/,
	and friends record in a more efficient way.  See
	linkgit:git-pack-refs[1]. This file is ignored if $GIT_COMMON_DIR
	is set and "$GIT_COMMON_DIR/packed-refs" will be used instead.

## HEAD

	改文件是当前活跃分支 `refs/heads/` 的符号引用.

	当仓库没有关联工作区时, 它没有实际的意义, 但对于一个合法的仓库来说, HEAD 是必须存在的.
	
	上层命令可能会用它来猜测默认分支(通常是 "master").

	HEAD 也能直接对应一个 commit, 也就是 "分离头指针".

## config

	仓库的配置文件.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/config` .

## config.worktree

	Working directory specific configuration file for the main
	working directory in multiple working directory setup

## branches

	已过时.

## hooks

	该目录保存 Git 命令的自定义钩子.
	所有的钩子默认关闭. 移除文件名中的 `.sample` 来启用钩子.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/hooks` .

## common

	When multiple working trees are used, most of files in
	$GIT_DIR are per-worktree with a few known exceptions. All
	files under 'common' however will be shared between all
	working trees.

## index

	仓库索引. 
	通常不会出现在裸仓库中.

## sharedindex.<SHA-1>

	共享索引.
	只在分离索引模式下生效.

## info

	改目录仓储仓库的额外信息.
	
	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/info` .

## info/refs

	该文件帮助哑传输发现仓库中可用的引用.
	一旦该仓库以哑传输发布, 每当标签或分支有创建和修改, 都应当执行 `git update-server-info` 来重新生成该文件.

	当你 `git push` 仓库后, `git-receive-pack` 会调用 `hooks/update` 钩子来完成这个操作.

## info/grafts

	该文件记录伪造的 commit 祖先信息. 

	已过时.

## info/exclude

	该文件保存排除的匹配模式列表.
	`.gitignore` 逐个目录地忽略文件.
	'git status', 'git add', 'git rm' and 'git clean' 检查它, 但 Git 核心命令不检查.

## info/attributes

	Defines which attributes to assign to a path, similar to per-directory `.gitattributes` files.

## info/sparse-checkout

	该文件保存稀疏 checkout 的匹配模式.

## remotes

	保存远端仓库的 URL 和 默认引用名称.
	属于历史遗留, 现代仓库中很少见了.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略这个目录, 使用 `$GIT_COMMON_DIR/remotes`.

## logs

	该目录记录引用的变更.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该目录, 使用 `$GIT_COMMON_DIR/logs` .

## logs/refs/heads/`name`

	分支的变更.

## logs/refs/tags/`name`

	tag 的变更.

## shallow

	用于浅 clone , 跟 `info/grafts` 类似.

	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略该文件, 使用 `$GIT_COMMON_DIR/shallow` .

## commondir

	当文件存在时, 如果环境变量 `$GIT_COMMON_DIR` 没有被显示地设置过, 那么就根据该文件的值来设置 `$GIT_COMMON_DIR`.
	
	如果保存的路径是相对路径, 则相对于 `$GIT_DIR` .

## modules

	该目录保存子模块.

## worktrees

	该目录保存了工作区的数据.
	
	如果设置了环境变量 `$GIT_COMMON_DIR`, 则忽略这个目录, 使用 `$GIT_COMMON_DIR/worktrees`.

## worktrees/<id>/gitdir

	该文件保存了 `.git` 的绝对路径.

	用于检查链接的存储库是否已被手动删除了, 并且不再需要保留此目录.
	每当链接的仓库被访问, 就应当更新该文件的 mtime .

## worktrees/<id>/locked

	如果该文件存在, 那么对应的工作区不可用. 
	同时, 会阻止自动或手动触发的 prune .
	它的内容是字符串, 解释仓库被锁定的原因.

## worktrees/<id>/config.worktree

	该文件保存了工作区特定的配置.
