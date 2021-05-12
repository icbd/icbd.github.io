---
layout: post
title:  ActiveSupport Inflector acronym
Author: CBD
tags:   [Ruby, Rails]
---

ActiveSupport 默认提供驼峰和蛇底字符串风格转换表现出色, 模块查找和加载也都依赖这个功能.

但对于缩写词, ActiveSupport 默认不能自动处理, 因为他并不知道单词的语义, 更不知道哪里是缩写词的边界, 这个时候就需要我们告诉他缩写词的正确拼写方法.

```ruby
require "active_support/all"

puts "RESTfullAPI".underscore # res_tfull_api (Wrong)
puts "API".underscore # api

puts "restfull_api".camelize # RestfullApi (Not Good)
puts "api".camelize # Api (Not Good)

puts "-----"

ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.acronym "RESTfull"
  inflect.acronym "API"
end

puts "RESTfullAPI".underscore # restfull_api
puts "API".underscore # api

puts "restfull_api".camelize # RESTfullAPI
puts "api".camelize # API

```

## Rails

### 默认情况下

```ruby
# app/models/ce/user.rb

module Ce
  module User
  end
end

puts Ce::User
```

Rails 默认把 `ce` 看做一个单词, 单词首字母大写, 其余小写(第二个字母小写), 也就写作 `Ce`.

### 声明 acronym

在 Rails 项目中, 会先加载 framework 和 GEM, 然后加载 `config/initializers` 目录(包括子目录)下的所有 Ruby File.

Rails 6 已经包含了模板文件: `config/initializers/inflections.rb`, 只需要打开注释并修改:

```ruby
# config/initializers/inflections.rb

ActiveSupport::Inflector.inflections(:en) do |inflect|
  inflect.acronym 'CE'
end
```

```ruby
# app/models/ce/user.rb

module CE
  module User
  end
end

puts CE::User
```
