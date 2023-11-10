---
layout: post
title:  Setup Gitlab Development Kit on Mac
date:   2021-03-23
Author: CBD
tags:   [GitLab]
---

[Install and configure GDK DOC](https://gitlab.com/gitlab-org/gitlab-development-kit/-/blob/main/doc/index.md)

```sh
#! /bin/bash

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

也可以追加参数 `gitlab_repo=YOUR_GITLAB_REPO gitaly_repo=YOUR_GITALY_REPO` 拉取指定的 repo .

我的安装参数如下:

```zsh
gdk install gitlab_repo=git@gitlab.com:icbd/gitlab.git gitaly_repo=git@gitlab.com:icbd/gitaly.git
```

## How to debug GitLab Rails App

1. `gdk stop`
2. 注释掉 `Procfile` 中的 `rails-web:` 一行
3. `gdk start`
4. `cd gitlab`
5. `bundle exec thin --socket=../gitlab.socket start`  或者 `bundle exec rails server`

为了方便 byebug, 可以考虑修改 puma 的配置:

```ruby
threads 1, 1
workers 1
worker_timeout 600
```

设置环境变量:

```text
DISABLE_PUMA_WORKER_KILLER=true
```

## Update GDK Config

修改完配置文件 `gdk.yml` 后, 需要执行

```zsh
gdk reconfigure
```

## Update GDK

```zsh
gdk update
```

在 2021-05 的升级中, GDK 升级了 PostgreSQL 到 12, 开发环境会报:

```text
ERROR: PostgreSQL data directory is version 11 and must be upgraded to version 12.6 before GDK can be updated.
```

需要额外执行: `./support/upgrade-postgresql`. 这个脚本会帮你自动完成迁移:

1. 备份 PG 11 的数据
2. 初始化 PG 12 并导入数据
3. `dgk reconfigure`

## Tips

`Reason: image not found - /Users/cbd/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0/gems/charlock_holmes-0.7.7/lib/charlock_holmes/charlock_holmes.bundle`

如果在 MacOS 11.3 上遇到这个问题, 尝试使用下面命令解决:

```zsh
gem pristine charlock_holmes
```

## Setup CI runner

虽然可以使用 brew 安装和管理 `gitlab-runner`, 但是强烈建议手动安装.

务必要在 `user-mode` 下使用 `gitlab-runner register` ( 不要用 `sudo` ).

为了让 docker 能 clone 你的 repo, 使用内网 IP 是个简单办法:

> `gdk.yml`

```text
listen_address: "0.0.0.0"
hostname: "192.168.0.132"
```

修改之后记得刷新配置: `gdk reconfigure`.

Ref: [Using GitLab Runner with GDK](https://gitlab.devworks.gr/gitlab-org/gitlab-development-kit/blob/f2247534217d23523f0e3495ad72162805dc7b38/doc/howto/runner.md)

# FAST GUIDE

## General install

```bash
git clone git@gitlab.com:gitlab-org/gitlab-development-kit.git gdk

cd gdk

bundle

gem install gitlab-development-kit

make bootstrap

gdk install
```

## JiHu install

```bash
gdk stop

rm -rf gitlab
git clone git@jihulab.com:gitlab-cn/gitlab.git
brew install gnu-sed
cat << 'EOF' > gitlab/.git/hooks/post-checkout
#!/bin/bash
#
# Auto config Gemfile path for JiHu repo when git checkout.
#
# Dependency: brew install gnu-sed
#
# chmod +x ./.git/hooks/post-checkout
#

current_dir=${PWD##*/}
bundle_config_file="./.bundle/config"

if [[ "$current_dir" != "gitlab" ]]; then
    exit 0
fi

echo "Execute bundle config through hooks/post-checkout"

if [[ ! -f "$bundle_config_file" ]]; then
    echo "Make new file: $bundle_config_file"
    mkdir ".bundle"
    echo "---\n" > "$bundle_config_file"
fi

gsed -i '/^BUNDLE_GEMFILE: /d' "$bundle_config_file"

if [[ -f "jh/Gemfile" ]]; then
  echo 'BUNDLE_GEMFILE: "jh/Gemfile"' >> "$bundle_config_file"
else
  echo 'BUNDLE_GEMFILE: "Gemfile"' >> "$bundle_config_file"
fi

bundle config get gemfile

EOF

chmod +x gitlab/.git/hooks/post-checkout

cd gitlab
git checkout pre-main-jh
bundle
cd ..

cat << 'EOF' > gdk.yml
---
repositories:
  gitlab: git@jihulab.com:gitlab-cn/gitlab.git
gitlab:
  default_branch: pre-main-jh
  rails:
    puma:
      threads_max: 16
      threads_min: 8
      workers: 2
rails_web:
  enabled: false

EOF

gdk reconfigure
gdk update
```