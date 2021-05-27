---
layout: post
title:  bundler without Rails
date:   2021-05-27
Author: CBD
tags:   [bundler]
---

`bundle install` 是再熟悉不过的命令了, 但脱离了 Rails 环境, 如何使用包管理工具呢?

* Gem: [https://github.com/rubygems/rubygems](https://github.com/rubygems/rubygems) 
* Bundler: [https://github.com/rubygems/rubygems/tree/master/bundler](https://github.com/rubygems/rubygems/tree/master/bundler)

## Demo

```zsh
#!/bin/zsh
mkdir demo
cd demo

bundle init
bundle add rake --version "~> 12.0"
bundle install

cat > main.rb <<EOF
require "bundler/setup"
require "rake"

p Rake::VERSION

EOF

ruby main.rb
```

我本地的开发环境已经安装了三个版本的 rake:

> `$ gem info rake`

```text
rake (13.0.3, 13.0.1, 12.3.3)
    Authors: Hiroshi SHIBATA, Eric Hodel, Jim Weirich
    Homepage: https://github.com/ruby/rake
    License: MIT
    Installed at (13.0.3): /Users/cbd/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0
                 (13.0.1): /Users/cbd/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0
                 (12.3.3): /Users/cbd/.asdf/installs/ruby/2.7.2/lib/ruby/gems/2.7.0

    Rake is a Make-like program implemented in Ruby
```

此时, Rake 的最新版本是 `13.0.3` .

上面脚本最后输出为 `"12.3.3"`, 也就是按照我们 Gemfile 中声明的规则, 使用了 `12` 的最新版本.

```text
.
├── Gemfile
├── Gemfile.lock
└── main.rb
```

> Gemfile

```text
# frozen_string_literal: true

source "https://rubygems.org"

git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

# gem "rails"

gem "rake", "~> 12.0"
```

> main.rb

```ruby
require "bundler/setup"
require "rake"

p Rake::VERSION

```

`require "bundler/setup"` 做的事情是: 读取 `Gemfile.lock`, 把指定版本的 Gem 添加到 LOADPATH 中.

所以使用 `Rake` 前, 还是需要 `require "rake"`.

如果想 require 所有 `default` 分组的包, 可以这样:

```ruby
require "bundler/setup"
Bundler.require(:default)

p Rake::VERSION

```
