---
layout: post
title:  Ruby 找到方法定义的位置
Author: CBD
tags:   [Ruby]
---

基本使用:

```ruby
class User
  def name
    "Bob"
  end
end

u = User.new

p u.name # "Bob"
m = u.method(:name)
p m.owner # User
p m.source_location # ["main.rb", 2]

```

对于继承和引入的方法, `owner` 会显示定义方法的命名空间:

```ruby
module Helper
  def name
    "Bob"
  end
end

class User
  include Helper
end

class Student < User
  def score
    100
  end
end

obj = Student.new

p obj.name # "Bob"
m = obj.method(:name)
p m.owner # Helper
p m.source_location # ["main.rb", 2]

p obj.score # 100
m = obj.method(:score)
p m.owner # Student
p m.source_location # ["main.rb", 12]

```

普通的方法通过字符串搜索还是比较容易找到的, 动态生成的方法就很依赖这样查找.

**为了方便方法的阅读和查找, 一定要通过注释补全方法名.**

```ruby
class User
  # define_method :dance_popping
  # define_method :dance_waacking
  [:popping, :waacking].each do |dance_type|
    method_name = "dance_#{dance_type}"
    define_method(method_name) do |*args|
      "Hi #{method_name}: #{args}"
    end
  end
end

u = User.new

p u.dance_popping("Good") # "Hi dance_popping: [\"Good\"]"
p u.dance_waacking("Perfect") # "Hi dance_waacking: [\"Perfect\"]"

m = u.method(:dance_popping)
p m.owner # User
p m.source_location # ["main.rb", 6]

```

或者这样:

```ruby
class User
  # define_method :dance_popping
  # define_method :dance_waacking
  [:popping, :waacking].each do |dance_type|
    method_name = "dance_#{dance_type}"
    class_eval <<-RUBY, __FILE__, __LINE__ + 1
    def #{method_name}(*args)
      "Hi #{method_name}: \#{args}"
    end
    RUBY
  end
end

u = User.new

p u.dance_popping("Good") # "Hi dance_popping: [\"Good\"]"
p u.dance_waacking("Perfect") # "Hi dance_waacking: [\"Perfect\"]"

m = u.method(:dance_popping)
p m.owner # User
p m.source_location # ["main.rb", 7]

```

通过 `method_missing` 实现的 "方法" 不适用:

```ruby
class User
  private def method_missing(symbol, *args)
    "Hello #{symbol}: #{args}"
  end
end

u = User.new

p u.sing("Pop", "Rock") # "Hello sing: [\"Pop\", \"Rock\"]"
p (u.method(:sing) rescue "undefined method :sing") # "undefined method :sing"

```
