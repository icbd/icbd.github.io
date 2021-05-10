---
layout: post
title:  Setup Gitlab Development Kit on Mac
date:   2021-03-23
Author: CBD
tags:   [Gitlab]
---

使用这个脚本会全自动安装和配置 GDK, 建议在新机器上使用.

自动安装借助 [asdf](https://asdf-vm.com/#/core-manage-asdf) 来安装依赖, 如果跟现有环境冲突请参考: [Install and configure GDK](https://gitlab.com/gitlab-org/gitlab-development-kit/-/blob/main/doc/index.md)

```sh
#! /bin/bash
echo "Check basic tools:"
if !( type git >/dev/null 2>&1 ); then
  echo 'git not exist';
  exit 1;
fi
if !( type make >/dev/null 2>&1 ); then
  echo 'make not exist';
  exit 1;
fi

echo "Install dependencies:"
git clone https://gitlab.com/gitlab-org/gitlab-development-kit.git
cd gitlab-development-kit
make bootstrap

echo "Install gdk:"
gdk install
```

经过几杯咖啡的时间, 整个环境就装好了, gdk 会帮你自动配置并启动:

[http://127.0.0.1:3000/](http://127.0.0.1:3000/)

开发环境默认管理员为 `root`, 初始密码: `5iveL!fe`.

另外, `gdk install` 可以追加参数 `shallow_clone=true` 来减少 clone 深度,

也可以追加参数 `gitlab_repo=YOUR_REPO_URL` 拉取指定的 repo .

## Tips

`Reason: image not found - /Users/cbd/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/charlock_holmes-0.7.7/lib/charlock_holmes/charlock_holmes.bundle`

如果在 MacOS 11.3 上遇到这个问题, 尝试使用下面命令解决:

```zsh
gem pristine charlock_holmes
```
