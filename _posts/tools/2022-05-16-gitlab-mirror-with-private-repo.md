---
layout: post
title:  GitLab mirror with private repository
date:   2022-05-16
Author: CBD
tags:   [Git]
---

在 GitLab 上, 对于 private 的 repo, 使用用户名密码的 auth 的方式中,

- 用户名对应 gitlab 的账号 ID, 不是 email;
- 密码推荐使用 Personal Access Token, 兼具权限管理/过期管理 .

Repository mirror 是 Premium 的功能.

如果需要 mirror private 的 repo:

- Mirror direction: Pull
- 在 repo url 中添加用户名, `https://username@gitlab.com/namespace/project.git`
- 在 password 中填写 Personal Access Token

![repo-mirror.png](/images/repo-mirror.png)
