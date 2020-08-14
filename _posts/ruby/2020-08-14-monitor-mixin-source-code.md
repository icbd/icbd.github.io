---
layout: post
title:  MonitorMixin Source Code
date:   2020-08-14
Author: CBD 
tags: [Ruby, Thread, MonitorMixin]
---

## 用法示例

```ruby
require 'monitor'

class User
  attr_accessor :print_item
end
user = User.new
user.extend MonitorMixin

proc = Proc.new do |user|
  raise 'Must be a MonitorMixin' unless user.is_a?(MonitorMixin)

  user.synchronize do
    puts Thread.current
    5.times do
      sleep 0.1
      print user.print_item
    end
    puts ""
  end
end

sub_thread = Thread.new do
  user.mon_enter
  user.print_item = 'Sub '
  proc.call(user)
  user.mon_exit
end

user.synchronize do
  user.print_item = 'Main '
  proc.call(user)
  sleep 0.5
  puts 'Main thread release lock.'
end

sub_thread.join

=begin
#<Thread:0x00007f974e064040 run>
Main Main Main Main Main
Main thread release lock.
#<Thread:0x00007f974e047710@t.rb:22 run>
Sub Sub Sub Sub Sub

=end

```

在了解源码前我们先看一个用例.

`user` 是一个普通对象, 它 `extend` 了 `MonitorMixin` 模块.

`proc` 是一个闭包, 它期待传入一个支持 `MonitorMixin` 的对象, 并在该对象上调用 `synchronize`, 希望不被打断地, 一边休息一边打印 5 个 `print_item`.

sub_thread 线程通过 `mon_enter` 进入锁, 先把 `user` 的 `print_item` 设为 `Sub `, 然后调用闭包. 

主线程通过 `synchronize` 进入锁, 先把 `user` 的 `print_item` 设为 `Main `, 然后调用闭包, 执行完闭包后睡一会儿再释放锁.

一般情况下主线程会首先抢占到锁, 会不被打断地打印五个 `Main`, 等主线程释放锁之后 sub_thread 才执行一系列操作.


## Source Code

这个用例的要点是, 闭包内有一层 `synchronize` 的包装, 线程的操作也有 `synchronize` 或者 `mon_enter` 的包装, 也就是典型的可重入锁的场景. 

Ruby 的 `Mutex` 本身是不可重入锁, 那么 MonitorMixin 是怎么实现的呢, 来看一下源码:

`user.extend MonitorMixin` 会触发 Ruby 的方法回调:

```ruby
  def self.extend_object(obj)
    super(obj)
    obj.__send__(:mon_initialize)
  end
```

```ruby
  def mon_initialize
    if defined?(@mon_mutex) && @mon_mutex_owner_object_id == object_id
      raise ThreadError, "already initialized"
    end
    @mon_mutex = Thread::Mutex.new
    @mon_mutex_owner_object_id = object_id
    @mon_owner = nil
    @mon_count = 0
  end
```

`mon_initialize` 通过检查 `@mon_mutex` 和 `@mon_mutex_owner_object_id` 来确定 `user` 是第一次初始化,
 也就是要求 `user.extend MonitorMixin` 执行且只能执行一次. 然后设置一系列实例变量:
 
 * `@mon_mutex` 初始化为新的 `Mutex` 对象, 锁操作都会被代理到这个 Mutex 对象上;
 * `@mon_mutex_owner_object_id` 初始化为 `user` 的 `object_id`;
 * `@mon_owner` 初始化为 nil, 表示目前还没有线程占有 mon_mutex 锁;
 * `@mon_owner` 初始化 为 0, 表示重入的锁层次为 0;
 
至此, `user` 就可以使用 `synchronize` 或者 `mon_enter` 来使用锁了.

```ruby
  def mon_enter
    if @mon_owner != Thread.current
      @mon_mutex.lock
      @mon_owner = Thread.current
      @mon_count = 0
    end
    @mon_count += 1
  end
```

`@mon_owner` 指向持有锁的线程.

如果当前线程持有 mon_mutex 锁:

*  那么就直接给 `@mon_count` 加一, 表示重入锁的深度加一了;
    
如果当前线程未持有 mon_mutex 锁:

*  当没有任何线程持有锁, 当前线程直接通过 `@mon_mutex.lock` 抢占锁, 并设置 `@mon_owner` 指向当前线程, 设置锁重入深度为 0.    
*  当 `@mon_owner` 正指向其他线程, 当前线程直接通过 `@mon_mutex.lock` 阻塞地等待锁.

```ruby
  def mon_exit
    mon_check_owner
    @mon_count -= 1
    if @mon_count == 0
      @mon_owner = nil
      @mon_mutex.unlock
    end
  end

  def mon_check_owner
    if @mon_owner != Thread.current
      raise ThreadError, "current thread not owner"
    end
  end
```

退出锁, 首先确认当前线程持有锁, 然后重入锁层次减一.
如果锁层次减到 0, `@mon_owner` 指向 nil, 并释放锁. 

```ruby
  def mon_synchronize
    mon_enter
    begin
      yield
    ensure
      mon_exit
    end
  end

  alias synchronize mon_synchronize
```

`synchronize` 是 `mon_enter` 和 `mon_exit` 的组合.

## 小结

`MonitorMixin` 通过引入重用锁的层次计数器, 来实现了 `Mutex` 的可重入效果. 

实现中要特别注意 `@mon_owner` 的检查, 在当前线程没有持有锁的情况下操作锁, 会抛 `ThreadError`.
