---
layout: post
title: Ruby Mutex Re-entrant
date: 2020-08-12
Author: CBD
tags: [Ruby, Thread, MonitorMixin]
---

## 引子
Java 系中有明确的可重入锁的概念, Ruby 的锁靠 Mutex 实现, 是不可重入锁, 在同一个线程中, Mutex 嵌套会报死锁异常.

```ruby
@mutex = Mutex.new

@mutex.synchronize do
  @mutex.synchronize do
    puts '>'
  end
end

=begin
deadlock; recursive locking (ThreadError)
=end
```

## 需求

```ruby
class Worker
  def initialize
    @mutex = Mutex.new
  end

  def call
    @mutex.synchronize do
      yield
    end
  end

  def process
    call do
      @mutex.synchronize do
        p "process>>>"
      end
    end
  end
end

Worker.new.process
=begin
in `synchronize': deadlock; recursive locking (ThreadError)
=end
```
如上例所示, 我们要求加锁执行 `call` 方法, 并且该方法要执行传入的 block. 
一旦 block 也持有同样的锁, 那么就会出现以上的 Mutex 重入问题.


## 解决

MonitorMixin 提供了一种可重入锁的实现, 不需要手动定义锁, 可以多次进入锁.

```ruby
require 'monitor'

class Worker
  include MonitorMixin

  def initialize(*args)
    super
  end

  def call
    synchronize do
      yield
    end
  end

  def process
    call do
      synchronize do
        p "process>>>"
      end
    end
  end
end

Worker.new.process

```

`MonitorMixin` 模块可以 `extend` 到一个 object 上, 或者在类中 `include` (需要重写 `initialize` 方法), 也可以单独使用 `Monitor` 类,
 具体参见 [https://devdocs.io/ruby~2.6/monitormixin](https://devdocs.io/ruby~2.6/monitormixin).

## 补充

Mutex 的不可重入特性仅限制在同一个线程内, 锁内开启新线程后, 在新线程内使用同一个 Mutex 是合理的.

```ruby
@mutex = Mutex.new
@resource = ConditionVariable.new

@mutex.synchronize do
  p '0'
  thr = Thread.new do
    @mutex.synchronize do
      p '2'
      @resource.signal
      sleep 0.1
      p '3'
    end
  end
  p '1'
  @resource.wait(@mutex)
  p '4'
end

=begin
"0"
"1"
"2"
"3"
"4"
=end
```

上例中, 主线程派生出 thr 后, 立即执行 `p '1'`, 因为等待 `@resource` 而释放 `@mutex` 锁, 同时交出执行权.

执行分片来到 thr 上, 顺序执行, `@resource.signal` 唤醒了主线程 (但此时主线程还没有获得锁, 依旧不能执行).

等 thr 睡一会再打印完 `p '3'` 后, thr 释放锁, 此时已经被唤醒的主线程得到锁继续执行, 打印 `p '4'`.

理解了这里就能理解 Puma 新版本的 ThreadPool 是如何派生初始数量的工作线程了.

## 参考

[https://devdocs.io/ruby~2.6/monitormixin](https://devdocs.io/ruby~2.6/monitormixin)

[https://www.ruby-forum.com/t/mutex-confusion/177583/3](https://www.ruby-forum.com/t/mutex-confusion/177583/3)

[https://github.com/puma/puma/blob/0f718d516b92cd8bc4120c543b06792b22ac20bb/lib/puma/thread_pool.rb#L95](https://github.com/puma/puma/blob/0f718d516b92cd8bc4120c543b06792b22ac20bb/lib/puma/thread_pool.rb#L95)