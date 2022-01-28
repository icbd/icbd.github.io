---
layout: post
title:  Prepend private method
date:   2022-01-28
Author: CBD
tags:   [Ruby]
---

```ruby
class User
  private

  def foo
    p "private User#foo"
  end
end

User.new.foo
```

显然, 显式调用 `private` 的方法会抛出异常:

```text
private method `foo' called for #<User:0x00007fba2d1103e8> (NoMethodError)
```

## 探测方法的定义

```ruby
User.method_defined?(:foo) # false

User.public_method_defined?(:foo)  # false
User.protected_method_defined?(:foo)  # false
User.private_method_defined?(:foo)  # true
```

`Module#method_defined?` 探测 `public` 和 `protected` 的方法, 其他三个方法只探测对应的方法.

四个方法都有第二个默认参数 `inherit=true`, 默认会去 ancestor 链上查找.

---

```ruby
m = User.new.method(:foo)
```

`Object#method` 用来获取方法对象, 它对 `public` 和 `protected` 和 `private` 的方法都有效.

在复杂项目里, 知道被调用的方法定义的具体位置会非常有帮助:

```ruby
User.new.method(:foo).source_location

=begin
[ 
  "m.rb", # souce location file path
  12 # line number
]
=end
```

## prepend private method

```ruby
module M
  def foo
    p "public M#foo"
  end
end

class User
  prepend M

  private

  def foo
    p "private User#foo"
  end
end

User.public_method_defined? :foo # true
```

回到主题, 当 prepend 的 M 中的 `foo` 为 public 时, 将 M prepend 进 User 之后,

User 查找到的 `foo` 也就变成了 M 的 `foo`, 级别也变成了 M 的 `foo` 的 `public`.

如果想让 prepend 后的 `foo` 依然保持 `private`, 那么应该让 M 把 `foo` 也定义为 `private` .
