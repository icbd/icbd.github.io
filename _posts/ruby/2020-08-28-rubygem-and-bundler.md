---
layout: post
title: Rubegem and Bundler
date: 2020-08-28
Author: CBD
tags: [Ruby, Dependency Management]
---

Ruby 是解释型语言, 引入一个文件实际上是执行这个文件:

```ruby
# pseudocode
def require(filename)
  eval File.read(filename)
end
```

对于同一个文件, `require` 实际上只会引入一次, 这需要引入一个数组来记录已经引入过的文件:

```ruby
# pseudocode
$LOADED_FEATURES = []

def require(filename)
  return true if $LOADED_FEATURES.include?(filename)

  eval File.read(filename)
  $LOADED_FEATURES << filename
end
```

互联网和开源文化催生了各种功能的第三方库, 原始的做法是, 找到库的托管地址, 下载库的源文件到本地, 再 `require "拼接好的文件路径"` 就可以使用了.

这有几个问题:

* 第三方库风格迥异, 安装过程不统一;
* 项目文件中充斥着业务逻辑无关的第三库的源码;
* 更新第三方库的版本很繁琐;

## RubyGem

RubyGems 很好的解决了这些问题.

这是一个典型的 Gem 的文件结构, Gemfile 中指明了 Gem 的依赖库, lib 目录存放库的源码, lib 下有一个跟库同名的文件( `require "focus_actor"` 就是在执行这个文件 ), 核心逻辑归档到 lib 的子目录中.

```text
.
├── Gemfile
├── Gemfile.lock
├── LICENSE
├── README.md
├── Rakefile
├── bin
│   ├── console
│   └── setup
├── focus_actor.gemspec
├── lib
│   ├── focus_actor
│   │   ├── async.rb
│   │   ├── async_processor.rb
│   │   ├── cell_context.rb
│   │   ├── future.rb
│   │   ├── future_cell.rb
│   │   ├── future_processor.rb
│   │   └── version.rb
│   └── focus_actor.rb
├── pkg
│   ├── focus_actor-0.1.0.gem
│   └── focus_actor-0.1.1.gem
└── spec
    ├── focus_async_spec.rb
    ├── focus_future_spec.rb
    ├── spec_helper.rb
    ├── user.rb
    └── version_spec.rb
```

`$ gem install focus_actor -v 0.1.0` 会访问去 RubyGems 的中心仓库下载 `focus_actor` 指定版本的代码, 存放到本地某个约定好的目录下: `$HOME/.rvm/gems/ruby-2.6.3/gems/` ( 使用了 rvm ).

## Bundler

管理第三方库的工具有了, 但是在使用库时还是会有些问题:

* 每个项目使用的库各不相同, 逐个下载即繁琐又容易遗漏;
* 项目依赖库的版本不一致;

如果项目中依赖很多个 Gem, 需要预先安装这些 Gem.

项目的老组员把所需 Gem 记到一个特殊文件里 `Gemfile`, 新组员配置环境的时候参照 `Gemfile` 逐个安装. 把这个步骤自动化, 也就是 bundler 的主要工作.

当然 bundler 做了更多, 他会检查各个依赖的版本, 检查依赖的依赖的版本, 检查依赖的依赖的依赖...的版本, 找到他们都兼容的版本, 把详细条目写到 `Gemfile.lock`. 
如此, 下次安装也省去了检查依赖版本的工作, 更重要的是能让新的安装保持版本一致.

需要注意的是, 一个项目不能同时使用某个库的多个版本 ( 这个需求本身就很可能有问题 ) .

## 参考

[https://guides.rubygems.org/rubygems-basics](https://guides.rubygems.org/rubygems-basics)

[https://www.cloudcity.io/blog/2015/07/10/how-bundler-works-a-history-of-ruby-dependency-management](https://www.cloudcity.io/blog/2015/07/10/how-bundler-works-a-history-of-ruby-dependency-management)