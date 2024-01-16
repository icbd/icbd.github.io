---
layout: post
title: normalize_path 方法分析
date: 2020-08-12
Author: CBD
tags: [Ruby]
---

> ActionDispatch::Journey::Router::Utils.normalize_path

```ruby
        # Normalizes URI path.
        #
        # Strips off trailing slash and ensures there is a leading slash.
        # Also converts downcase URL encoded string to uppercase.
        #
        #   normalize_path("/foo")  # => "/foo"
        #   normalize_path("/foo/") # => "/foo"
        #   normalize_path("foo")   # => "/foo"
        #   normalize_path("")      # => "/"
        #   normalize_path("/%ab")  # => "/%AB"
        def self.normalize_path(path)
          path ||= ""
          encoding = path.encoding
          path = +"/#{path}"
          path.squeeze!("/")

          unless path == "/"
            path.delete_suffix!("/")
            path.gsub!(/(%[a-f0-9]{2})/) { $1.upcase }
          end

          path.force_encoding(encoding)
        end
```

这是 ActionDispatch 处理 URI path 的一个方法, 其中有两个不常见的字符串方法: `#force_encoding` 和 `#+@`.

## `String#+@` 方法

https://ruby-doc.org/3.2.2/String.html#method-i-2B-40

字符串的一元操作, 行为表现为:

```ruby
def +
  str.frozen? ? str.dup : str
end
```

因为后续的操作是爆炸方法 `squeeze!` `delete_suffix!`, 所以需要保证 path 不是被冻结的.

Ref commit:

- [为 path 添加 dup 方法](https://github.com/rails/rails/commit/dd491b24ff28f637b1c8879002adb1acdf20a8ac)
- [修改 dup 为 加号操作](https://github.com/rails/rails/commit/1b86d90136efb98c7b331a84ca163587307a49af)

## `String#force_encoding`

https://ruby-doc.org/3.2.2/String.html#method-i-force_encoding

`force_encoding` 修改字符串对象的编码, 不修改底层字节(underlying bytes).

如果原始的 path 字符串不是 `UTF-8` 编码, 那么操作的过程中会让原始的编码信息丢失. 

```ruby
path = "/foo%AAbar%AAbaz".b
puts path.encoding # ASCII-8BIT
path = "/#{path}"
puts path.encoding # UTF-8

```

```ruby
require 'socket'

req = TCPServer.new('localhost', 3001).accept

line = req.gets
puts "-----#{line}", line.encoding
# ASCII-8BIT

line = "#{line}"
puts "-----#{line}", line.encoding
# UTF-8

req.close

```

Ref commit:
- https://github.com/rails/rails/commit/8607c25ba7810573733d9b37d0015154ba059f5e
