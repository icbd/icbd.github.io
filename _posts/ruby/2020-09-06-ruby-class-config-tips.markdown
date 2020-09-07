---
layout: post
title:  Ruby Class Config Tips
date:   2020-09-06
Author: CBD
tags: [Ruby]
---

## 需求

"约定大于配置". 配置一个类的时候, 最好把普遍适用的选项作为默认配置, 当用户有个性化需求的时候又能方便的覆盖默认配置.

## Ruby Code

这个例子使用了几个很 Ruby 风格的方法来实现:

```ruby
module App
  class Config
    OPTIONS = {
      :port => 3000,
      :host => '127.0.0.1',
    }

    class << self
      OPTIONS.each do |key, default|
        define_method(key) do |*args|
          arg = args.shift
          if arg
            instance_variable_set("@#{key}", arg)
          else
            instance_variable_get("@#{key}") || default
          end
        end
      end
    end
  end
end
```

Ruby 经常刻意忽略方法和变量的区别, 把他们统一看作是属性;

`define_method(key) { |*args| #...}` 在 Config 上定义了类方法, 方法参数会传给 `*args`.
当没有参数时, 意图为读取配置; 当传入参数时, 意图为覆盖配置.

`instance_variable_set(key, value)` 方法的返回值恰好是 `value` .

## Demo

```ruby
p App::Config.port
# 3000
p App::Config.port(8888)
# 8888
p App::Config.port
# 3000

```