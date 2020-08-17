---
layout: post
title:  PUMA ThreadPool
date:   2020-08-17
Author: CBD 
tags: [Ruby, Thread, ThreadPool]
---

## 引子

PUMA 是一个调度 Rack 应用的多线程服务器, 它的每个工作进程中都有一个线程池来处理请求.

## 用法

```ruby
require 'puma'
require 'puma/thread_pool'

MIN_THR = 1
MAX_THR = 3
pool = Puma::ThreadPool.new(MIN_THR, MAX_THR, Hash) do |num|
  sleep 2
  puts "#{num} ** #{num} = #{num ** num}"
end

def busy_threads
  Thread.list.count { |thr| thr.status == 'run' }
end

10.times do |i|
  pool << i
end

puts "[After pushing] busy_threads:#{busy_threads}"

pool.shutdown

puts "[After shutdown]  busy_threads:#{busy_threads}"

=begin
[After pushing] busy_threads:4
0 ** 0 = 1
2 ** 2 = 4
1 ** 1 = 1
3 ** 3 = 27
4 ** 4 = 256
5 ** 5 = 3125
6 ** 6 = 46656
7 ** 7 = 823543
8 ** 8 = 16777216
9 ** 9 = 387420489
[After shutdown]  busy_threads:1
=end
```

ThreadPool 支持动态调整线程数量, 工作线程范围: [MIN_THR, MAX_THR].
一般场景下会有一个 loop , 使用 `<<` 不停地塞入任务, 任务由线程池自动调度执行.
shutdown 会等任务都执行完再推出.

## 核心方法

[https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb](https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb)

线程池最核心的三个方法: `initialize`, `spawn_thread`, `<<` 要协同在一起看.

```ruby
    def initialize(min, max, *extra, &block)
      @not_empty = ConditionVariable.new
      @not_full = ConditionVariable.new
      @mutex = Mutex.new

      @todo = []

      @spawned = 0
      @waiting = 0

      @min = Integer(min)
      @max = Integer(max)
      @block = block
      @extra = extra

      @shutdown = false

      @trim_requested = 0
      @out_of_band_pending = false

      @workers = []

      @auto_trim = nil
      @reaper = nil

      @mutex.synchronize do
        @min.times do
          spawn_thread
          @not_full.wait(@mutex)
        end
      end

      @clean_thread_locals = false
    end
```

`initialize` 的 min/max 参数指明线程池的最小/最大工作线程数;

初始化了一个锁: `@mutex`.

初始化了两个条件变量:
* `@not_empty`, 当待办任务不为空的时候发送信号, 用来触发新任务的开始执行;
* `@not_full`, 不为空, 用来控制任务调度.

初始化了三个计数器: 
 * `@spawned` 已经派生出的线程数;
 * `@waiting` 等待执行任务的空闲的线程数;
 * `@trim_requested` 剪枝请求数;(只有该值大于 0 才会清理空闲线程)
 
初始化了两个列表:
 * `@workers` 工作线程的列表;
 * `@todo` 待办任务参数的列表;

```ruby
    def spawn_thread
      @spawned += 1
      th = Thread.new(@spawned) do |spawned|
        # ...
      end
      @workers << th
      th
    end
    
    def initialize
      # ...
      @mutex.synchronize do
        @min.times do
          spawn_thread
          @not_full.wait(@mutex)
        end
      end
      # ...
    end
```

直观的看, 这部分会初始化 `@min` 个工作线程, 但实现细节要复杂一点.

在 `@mutex` 锁内, 进入 `@min` 的第一次的循环. 

调用 `spawn_thread` 方法: `@spawned` 计数器加一, 新派生的线程加入到 `@workers` 队列.

主线程继续执行, 遇到了 `@not_full.wait(@mutex)`, wait 会让主线程立即释放 `@mutex` 锁, 并进入阻塞的同步等待.

此时 `@min` 的第一次循环还没有结束, `@mutex` 的控制权流转到 `spawn_thread` 派生的子线程上.

```ruby
      # spawn_thread block 
      th = Thread.new(@spawned) do |spawned|
        #Puma.set_thread_name 'threadpool %03i' % spawned
        todo  = @todo
        block = @block
        mutex = @mutex
        not_empty = @not_empty
        not_full = @not_full

        extra = @extra.map { |i| i.new }

        while true
          work = nil

          mutex.synchronize do
            while todo.empty?
              if @trim_requested > 0
                @trim_requested -= 1
                @spawned -= 1
                @workers.delete th
                Thread.exit
              end

              @waiting += 1
              if @out_of_band_pending && trigger_out_of_band_hook
                @out_of_band_pending = false
              end
              not_full.signal
              begin
                not_empty.wait mutex
              ensure
                @waiting -= 1
              end
            end

            work = todo.shift
          end

          if @clean_thread_locals
            ThreadPool.clean_thread_locals
          end

          begin
            @out_of_band_pending = true if block.call(work, *extra)
          rescue Exception => e
            STDERR.puts "Error reached top of thread-pool: #{e.message} (#{e.class})"
          end
        end
      end
```

子线程进入 while true 的循环后, 等待获得锁. 

上面说到主线程的 not_full wait 释放了锁, 子线程得到锁进入了 mutex.

当 todo 列表不为空时, 取出队列头部的一个元素赋值为 `work` .

当 todo 为空时(也就是此时), 说明这个线程已经空闲了. 

如果收到了剪枝请求, 就终止这个线程, 并将其从 `@workers` 队列中剔除, 派生线程数减一, 剪枝请求数减一. 

`@waiting` 指示的空闲线程数加一.

`not_full.signal` 唤醒 not_full 的线程, 也就是主线程.

子线程还持有锁, 继续执行, 遇到 `not_empty.wait mutex` 后释放锁, 执行权流转到了主线程.

主线程终于结束了 `@min` 的第一个循环, 同样的流程执行 `@min` 次, 派生出了 `@min` 个空闲的子线程, 线程池初始化结束.

```ruby
    def <<(work)
      with_mutex do
        if @shutdown
          raise "Unable to add work while shutting down"
        end

        @todo << work

        if @waiting < @todo.size and @spawned < @max
          spawn_thread
        end

        @not_empty.signal
      end
    end
```

得到 pool 对象之后, `pool << args` 把任务的参数加入 `@todo` 队列中.

如果 `@todo` 队列中的待处理任务数已经比空闲线程数大了, 并且线程数没有达到线程池的上限, 那么就通过 `spawn_thread` 再派生一个线程出来.

`@not_empty.signal` 信号通知了空闲的线程, 从 `not_empty.wait mutex` 中醒来, 醒来的子线程把空闲线程数减一, 通过 `block.call(work, *extra)` 执行了具体的任务.

然后通过 while true 进入下一个循环.

## 非核心方法

线程池里有一个 Automaton, 可以新开一个线程来专门收割多余的线程或终止的线程.

对于精确的时间差计算, 需要使用 `CLOCK_MONOTONIC` 而不是 `CLOCK_REALTIME`:

```ruby
Process.clock_gettime(Process::CLOCK_MONOTONIC)
```

* `CLOCK_REALTIME`  不能保证单调递增, 从 1970 年算起;
* `CLOCK_MONOTONIC` 可以保证单调递增, 从本机开机算起;

## 参考

[https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb](https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb)
