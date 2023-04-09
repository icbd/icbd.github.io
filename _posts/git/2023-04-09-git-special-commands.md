---
layout: post
title:  Git special commands
date:   2023-04-09
Author: CBD
tags:   [Git]
---

#### 当前的分支名

```sh
git branch --show-current
```

#### `origin` 上的默认分支名

```sh
// git remote show origin | grep 'HEAD branch' | sed 's@  HEAD branch: @@'

git symbolic-ref refs/remotes/origin/HEAD | sed "s@^refs/remotes/origin/@@"
```

#### 在任意分支上更新默认分支

```sh
// git fetch origin master:master

default_branch=$(git symbolic-ref refs/remotes/origin/HEAD | sed "s@^refs/remotes/origin/@@"); git fetch origin $default_branch:$default_branch
```

#### 清理暂存区和工作区

```sh
git reset HEAD -- && git checkout . && git clean -fd
```

#### 显示当前分支相对默认分支的新增提交

```sh
// git log branch-name --not default-branch-name

git log `git branch --show-current` --not `git symbolic-ref refs/remotes/origin/HEAD | sed "s@^refs/remotes/origin/@@"`
```

#### 通过 diff 文件传递修改

```sh
git diff HEAD^ > changes.diff

git diff master...HEAD > changes.diff

git apply changes.diff
```

#### 将本地提交打包成 patch 文件

diff 文件只包含修改, patch 还包含 author 信息.

可以追加参数改变输出 `--stdout`

```sh
// 打包最新的一个提交
git format-patch HEAD^

// 打包当前分支的所有提交
git format-patch `git merge-base master HEAD`

// 应用 patch 文件 (如果不需要author信息可以使用 `apply`)
git am 0001-Edit-Readme.patch
```
