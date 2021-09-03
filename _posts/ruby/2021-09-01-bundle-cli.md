---
layout: post
title:  Ruby Bundler
Author: CBD
tags:   [Ruby, bundle]
---

[https://bundler.io](https://bundler.io)

最常用的命令是 `bundle install` 和 `bundle exec`, 最近遇到了相关问题正好做个整理.

首先需要澄清的是, `gem` `bundle` `bundler` 和 `rubygems` 的关系.

MacOC 随系统默认安装了以下的命令:

| command | version |
| --- | --- |
|  `/usr/bin/ruby` | 2.6.3 |
|  `/usr/bin/gem` | 3.0.3 |
|  `/usr/bin/bundle` | 1.17.2 |
|  `/usr/bin/bundler` | 1.17.2 |

`/usr/bin/ruby` 是个二进制文件, 其他几个都是可读的 Ruby 脚本.

gem 用来安装和管理单个 Gem 包, 我们暂且不表.

bundler 其实是 bundle 的别名:

> bundler/exe/bundler

```ruby
#!/usr/bin/env ruby
# frozen_string_literal: true

load File.expand_path("../bundle", __FILE__)

```

没考证过为什么需要这样的 alias, 我的猜测是 bundle 和 bundler 都是单词, 为了不让大家费神记住哪个是正确的命令, 就干脆都实现了, 喜欢哪个用哪个.

大家当然都用短的, 所以后面也就叫它 `bundle` .

但看起来 `bundler` 才是更正式的名字, bundle 本身也是一个 gem, 它的名字是 `bundler`.

默认安装的 bundle 是 1.x 的版本, 目前主流已经升级到 2.x , 需要用 gem 安装最新的包:

```sh
gem install bundler
```

bundle 原来的 repository 是 [https://github.com/rubygems/bundler](https://github.com/rubygems/bundler), 后来 bundle 被收入 rubygems 统一管理, 最新的 repository 是 [https://github.com/rubygems/rubygems](https://github.com/rubygems/rubygems).

rubygems 通常指的是 [https://rubygems.org/](https://rubygems.org/), 他是一个中心化的 Gem 托管服务.

bundle 和 rubygems 都是社区贡献被官方吸纳的例子.

## Basic flow

以 `debug_bundle` 项目为例, 它的逻辑代码只有一个 `main.rb` 文件:

(从 `/usr/bin/bundle` 抄来的)

```ruby
#!/usr/bin/env ruby

require 'rubygems'
require 'byebug'

byebug
load Gem.activate_bin_path('bundler', 'bundle', ">= 0.a")
```

`rubygems` 不需要额外安装, 已经包含在 Ruby 标准库中了. `byebug` 是需要安装的第三方库.

初始化 bundle:

```sh
bundle init
```

这个命令创建出 `Gemfile` 和 `Gemfile.lock` .

把 `gem "byebug"` 添加到 `Gemfile` 文件末尾 .

安装依赖:

```sh
bundle
```

`bundle install` 可以简写为 `bundle`.

这个命令安装 Gemfile 中列出的所有依赖, 并更新 `Gemfile.lock` 文件.

Reference More: [https://bundler.io/v2.2/guides/using_bundler_in_applications.html](https://bundler.io/v2.2/guides/using_bundler_in_applications.html)

## 安装新的依赖库

以 `byebug` 为例, 当前最新的几个版本为:

```text
11.1.3 - April 23, 2020 (82.5 KB)
11.1.2 - April 17, 2020 (82.5 KB)
11.1.1 - January 24, 2020 (82.5 KB)
11.1.0 - January 19, 2020 (82.0 KB)
11.0.1 - March 18, 2019 (82.0 KB)
11.0.0 - February 15, 2019 (81.5 KB)
10.0.2 - March 30, 2018 (80.0 KB)
10.0.1 - March 21, 2018 (80.0 KB)
10.0.0 - January 26, 2018 (80.0 KB)
```

列举本机某个 Gem 的所有版本:

```sh
gem list byebug
```

```text
*** LOCAL GEMS ***

byebug (11.1.3, 11.1.2, 11.1.1, 11.1.0, 11.0.1, 11.0.0, 10.0.2, 10.0.1, 10.0.0, 9.0.5)
```

|例子|解释| installed version|
|---|---|---|
|gem "byebug"| 安装最新版 | 11.1.3 |
|gem "byebug", "11.1.1"| 指定版本 | 11.1.1 |
|gem "byebug", "11.1"| 指定版本, 小版本缺失时补零 | 11.1.0 |
|gem "byebug", "11"| 指定版本, 小版本缺失时补零 | 11.0.0 |
|gem 'byebug', '~> 10.0.1'| 大于等于 10.0.1 , 并且 小于 10.1.0 , 使用符合条件的较新版 | 10.0.2 |
|gem 'byebug', '~> 11.0'| 大于等于 11.0.0 , 并且 小于 12.0.0, 使用符合条件的较新版 | 11.1.3 |
|gem 'byebug', '~> 11.0', '< 11.1.3'| 在 `~> 11.0` 的基础上, 并且小于 11.1.3 | 11.1.2 |
|gem 'byebug', '~> 11.0', '>= 11.1.1'| 在 `~> 11.0` 的基础上, 并且大于等于 11.1.1 | 11.1.3 |

在 Gemfile 中声明好各个 Gem 的版本后, 使用 `bundle install` 就能自动安装指定版本的依赖, 同时更新 `Gemfile.lock` 文件.

## 根据 Gemfile.lock 安装项目依赖

clone 一个项目之后, 首先需要安装该项目的依赖. 根据规范, repository 中应该存在 `Gemfile.lock` 文件.

那么 `bundle install` 会根据 lock 文件中声明的具体版本来安装依赖.

这里有一个隐含的前提, Gemfile 的声明跟 lock 文件是匹配的 (我们可以信任之前的程序员已经处理好了).

## 依赖计算

依赖计算是 bundle 最核心的功能.

还是看一个具体的例子: `gem 'actionpack'`

Gemfile.lock 文件的内容为:

```text
GEM
  remote: https://rubygems.org/
  specs:
    abstract (1.0.0)
    actionpack (3.0.9)
      activemodel (= 3.0.9)
      activesupport (= 3.0.9)
      builder (~> 2.1.2)
      erubis (~> 2.6.6)
      i18n (~> 0.5.0)
      rack (~> 1.2.1)
      rack-mount (~> 0.6.14)
      rack-test (~> 0.5.7)
      tzinfo (~> 0.3.23)
    activemodel (3.0.9)
      activesupport (= 3.0.9)
      builder (~> 2.1.2)
      i18n (~> 0.5.0)
    activesupport (3.0.9)
    builder (2.1.2)
    erubis (2.6.6)
      abstract (>= 1.0.0)
    i18n (0.5.4)
    rack (1.2.8)
    rack-mount (0.6.14)
      rack (>= 1.0.0)
    rack-test (0.5.7)
      rack (>= 1.0)
    tzinfo (0.3.60)

PLATFORMS
  x86_64-darwin-20

DEPENDENCIES
  actionpack

BUNDLED WITH
   2.2.26

```

可以看到 rack 最终安装的版本是 `rack (1.2.8)`, 但是其他几个组件都声明了不同的 rack 依赖:

* `rack (~> 1.2.1)`
* `rack (>= 1.0.0)`
* `rack (>= 1.0)`

bundle 帮我们做的就是综合所有依赖的声明, 计算出一个最合适的兼容版本, 最后把这个版本记录在 lock 文件中.

那么其他人就可以跳过计算步骤, 直接根据 lock 文件来安装合适的版本.

遇到冲突时尝试使用 `bundle update gemA gemB`, 如果自动升级解决不了就需要人工接入选择了.

`bundle update` 会更新所有 gem.

By the way, npm 就不解决版本冲突, 把所有版本都保存下来. 这也就是它会占用超多的磁盘空间的原因.

## bundle exec

使用 bundle 中声明的命令版本来执行, 比如 `bundle exec rails`, `bundle exec rspec`.

在代码中更精确地控制 bundle , 比如分组:

```ruby
require 'bundler/setup'
Bundler.require(:default, :development)
```

看看 Rails 是怎么做的, 以下代码来自项目脚手架:

1. 启动文件 `bin/rails`

    ```ruby
    #!/usr/bin/env ruby
    load File.expand_path("spring", __dir__)
    APP_PATH = File.expand_path('../config/application', __dir__)
    require_relative "../config/boot"
    require "rails/commands"

    ```

2. `config/boot.rb`

    ```ruby
    ENV['BUNDLE_GEMFILE'] ||= File.expand_path('../Gemfile', __dir__)

    require "bundler/setup" # Set up gems listed in the Gemfile.
    require "bootsnap/setup" # Speed up boot time by caching expensive operations.

    ```

3. `config/application.rb`

    ```ruby
    require_relative "boot"

    require "rails"
    # Pick the frameworks you want:
    require "active_model/railtie"
    require "active_job/railtie"
    require "active_record/railtie"
    require "active_storage/engine"
    require "action_controller/railtie"
    require "action_mailer/railtie"
    require "action_mailbox/engine"
    require "action_text/engine"
    require "action_view/railtie"
    require "action_cable/engine"
    # require "sprockets/railtie"
    require "rails/test_unit/railtie"

    # Require the gems listed in Gemfile, including any gems
    # you've limited to :test, :development, or :production.
    Bundler.require(*Rails.groups)

    module Demoapp
      class Application < Rails::Application
        config.load_defaults 6.1
        config.api_only = true
      end
    end

    ```

## 其他例子

### 修改rubygems源

```sh
bundle config mirror.https://rubygems.org https://gems.ruby-china.com
```

[https://gems.ruby-china.com/](https://gems.ruby-china.com/)

### 安装本地 Gem

```text
gem "tacokit", path: "/path/to/tacokit"
```

[https://rossta.net/blog/how-to-specify-local-ruby-gems-in-your-gemfile.html](https://rossta.net/blog/how-to-specify-local-ruby-gems-in-your-gemfile.html)

### 通过 git repo 安装

```text
gem 'rack', git: 'https://github.com/rack/rack'
```

gem 位于子目录下:

```text
gem 'cf-copilot', git: 'https://github.com/cloudfoundry/copilot', glob: 'sdk/ruby/*.gemspec'
```

指定分支, tag, hash:

```text
gem 'nokogiri', git: 'https://github.com/sparklemotion/nokogiri.git', ref: '0bd839d'
gem 'nokogiri', git: 'https://github.com/sparklemotion/nokogiri.git', tag: '2.0.1'
gem 'nokogiri', git: 'https://github.com/sparklemotion/nokogiri.git', branch: 'rack-1.5'
```
