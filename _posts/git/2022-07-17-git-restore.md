---
layout: post
title:  git restore command
date:   2022-07-17
Author: CBD
tags:   [Git]
---

Doc link: [https://git-scm.com/docs/git-restore](https://git-scm.com/docs/git-restore)

restore 用来恢复 worktree(默认参数) 或者 staged 的内容.

## 对 not staged 的修改

`git restore .` 删除未暂存的修改(用 index 还原工作区).

**该操作不会删除 not staged 的新文件.**

## 对 staged 的修改

`git restore --staged .` 将暂存移到工作区.

`git restore --staged --worktree .` 直接删除暂存.
