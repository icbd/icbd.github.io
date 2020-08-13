---
layout: post
title: Two styles of Ruby Mutex
date: 2020-08-13
Author: CBD 
tags: [Ruby, Thread, Mutex]
---

## synchronize 方式

```ruby
mutex = Mutex.new

proc = Proc.new do
  mutex.synchronize do
    puts ">>>"
    3.times do
      sleep 1
      puts Thread.current
    end
  end
end

thr1 = Thread.new &proc
thr2 = Thread.new &proc

thr1.join
thr2.join
```

这是比较常见的 Mutex 使用方式, 把代码块传入 `synchronize`.

持有锁的线程才能进入 `synchronize`; 或者说所有线程中, 最多仅有一个线程在执行 `synchronize` 内的代码.

## 分段方式

如果想将锁的逻辑分拆, `synchronize` 就很为难, 还好 Mutex 还有其他几个 API.

```ruby
mutex = Mutex.new

# 当前线程持有该锁吗?
puts mutex.owned?
# false

# 该锁是否被任意线程持有?
puts mutex.locked?
# false

# 获得该锁;
# 如果该锁被其他线程持有, 就一直等, 直到获得该锁;
# 如果该锁已经被当前线程持有, 报 ThreadError ( Mutex 是不可重入锁).
puts mutex.lock
# #<Thread::Mutex:0x00007fdcb4027a68>

# 释放该锁;
# 如果该锁没有被当前线程持有, 报 ThreadError.
puts mutex.unlock
# #<Thread::Mutex:0x00007fdcb4027a68>

# 尝试获取该锁, 立即返回是否得到该锁
puts mutex.try_lock
# true

# 释放锁并休眠该线程, 休眠 timeout 秒后重新获得锁;
# 如果该锁没有被当前线程持有, 报 ThreadError.
puts mutex.sleep(2)
# 2
puts 'end'
```

Ruby 自带的 `MonitorMixin` 是 Mutex 分段方式很好的例子.
