---
layout: post
title:  Ruby ThreadPool
date:   2020-08-17
Author: CBD 
tags: [Ruby, Thread, ThreadPool]
---

## 引子

PUMA 是一个调度 Rack 应用的多线程服务器, 它的每个工作进程中都有一个线程池来处理请求.

`SimpleThreadPool` 是以 `PUMA::ThreadPool` 为蓝本的简易版本, 只保留了满足最基本功能的方法.

## 用法

```ruby
pool = SimpleThreadPool.new(1, 3) do |num|
  puts "start task > num: #{num}"
  sleep 2
  puts "100 / #{num} = #{100 / num}"
end

def busy_threads
  Thread.list.count { |thr| thr.status == 'run' }
end

9.times { |i| pool << i }
puts "[After pushing] busy_threads:#{busy_threads}"
pool.trim
puts "[After triming] busy_threads:#{busy_threads}"
pool.shutdown
puts "[After shutdown]  busy_threads:#{busy_threads}"
```

ThreadPool 支持动态调整线程数量, 工作线程数量范围: [MIN, MAX].
一般场景下会有一个 loop , 使用 `<<` 不停地塞入任务, 任务由线程池自动调度执行.
shutdown 会等任务都执行完再推出.

## Source Code

```ruby
class SimpleThreadPool
  class SimpleThreadPoolError < RuntimeError
  end

  def initialize(min, max, &block)
    raise SimpleThreadPoolError.new("Init Need Block") unless block_given?

    init_brace_variables

    @handler = block
    @thread_min_count = Integer(min)
    @thread_max_count = Integer(max)

    @lock.synchronize do
      @thread_min_count.times do
        spawn_thread
        @go_to_spawn.wait(@lock)
      end
    end
  end

  def << (task_args)
    @lock.synchronize do
      if @shutdown_flag
        STDERR.puts "[Add Task Error] Ready to Shutdown."
        return
      end

      @task_args_queue.push(task_args)

      if (@leisure_thread_count < @task_args_queue.size) && (@spawned_thread_count < @thread_max_count)
        spawn_thread
      end

      @go_to_work.signal
    end
  end

  def spawn_thread
    @spawned_thread_count += 1
    if @spawned_thread_count > @thread_max_count
      STDERR.puts "[Spawn Error] Too Many Threads."
    end

    worker_thread = Thread.new do
      loop do
        task_args = nil
        with_mutex do
          while @task_args_queue.empty?
            # deal with trim
            if @trim_request_count > 0
              @trim_request_count -= 1
              @spawned_thread_count -= 1
              @worker_threads_queue.delete Thread.current
              Thread.exit
            end
            # deal with spawn
            @leisure_thread_count += 1
            @go_to_spawn.signal
            @go_to_work.wait(@lock)
            @leisure_thread_count -= 1
          end
          task_args = @task_args_queue.shift
        end

        # deal with task
        begin
          @handler.call(task_args)
        rescue Exception => e
          STDERR.puts "[Task Execution Error] #{e}"
        end
      end
    end

    @worker_threads_queue.push worker_thread
    worker_thread
  end

  def trim(force = false)
    @lock.synchronize do
      need_trim = (@leisure_thread_count - @task_args_queue.size > 0)
      if (force || need_trim) && (@spawned_thread_count - @trim_request_count > @thread_min_count)
        @trim_request_count += 1
        @go_to_work.signal
      end
    end
  end

  def shutdown
    @lock.synchronize do
      @shutdown_flag = true
      @trim_request_count = @spawned_thread_count
      @go_to_work.broadcast
    end

    @worker_threads_queue.each(&:join)
    puts "[Shutdown]"
  end

  private

  def init_brace_variables
    @lock = Mutex.new
    @go_to_work = ConditionVariable.new
    @go_to_spawn = ConditionVariable.new

    @worker_threads_queue = []
    @task_args_queue = []
    @spawned_thread_count = 0
    @leisure_thread_count = 0
    @trim_request_count = 0
    @shutdown_flag = false
  end

  def with_mutex
    @lock.owned? ? yield : @lock.synchronize { yield }
  end
end
```

## 核心方法

线程池最核心的三个方法: `initialize`, `spawn_thread`, `<<` 要协同在一起看.

`initialize` 的 min/max 参数指明线程池的最小/最大工作线程数;

初始化了一个锁: `@lock`.

初始化了两个条件变量:
* `@go_to_work`, 当待办任务不为空的时候发送信号, 用来触发新任务的开始执行;
* `@go_to_spawn`, 用来控制派生新工作线程.

初始化了三个计数器: 
 * `@spawned_thread_count` 已经派生出的线程数;
 * `@leisure_thread_count` 等待执行任务的空闲的线程数;
 * `@trim_request_count` 剪枝请求数;(只有该值大于 0 才会清理空闲线程)
 
初始化了两个列表:
 * `@worker_threads_queue` 工作线程的列表;
 * `@task_args_queue` 待办任务参数的列表;

```ruby
def spawn_thread
  @spawned_thread_count += 1
  worker_thread = Thread.new do
    # ...
  end
  @worker_threads_queue.push worker_thread
  worker_thread
end

def initialize
  # ...
  @lock.synchronize do
    @thread_min_count.times do
      spawn_thread
      @go_to_spawn.wait(@lock)
    end
  end
  # ...
end
```

#### 初始化

直观的看, 这部分会初始化 `@thread_min_count` 个工作线程, 但实现细节要复杂一点, 使用了两个 `ConditionVariable` 来交替线程的执行权.

在 `@lock` 锁内, 进入 `@thread_min_count` 的第一次的循环. 

(初始化时没有竞争条件, 本不需要在锁内执行, 为了使用 `wait` 所以包装在锁内.)

调用 `spawn_thread` 方法, 派生新的工作线程.

主线程继续执行, 遇到了 `@go_to_spawn.wait(@lock)`, wait 会让主线程立即释放锁, 并进入阻塞的同步等待.

此时 `@thread_min_count` 的第一次循环还没有结束, `@lock` 的控制权流转到 `spawn_thread` 派生的子线程上.

子进程进入锁, 如果有待办任务就取出第一个任务来执行.

如果没有待办任务, 说明这个线程已经空闲了. 

空闲线程依次执行:
1. 剪枝任务, 如果有剪枝需求的话;
2. 唤醒主线程;
3. 释放锁, 阻塞等待 `@go_to_work` 的召唤.

主线程获得锁后终于结束了第一次循环, 同样的流程一共执行 `@thread_min_count` 次, 线程池初始化结束.

#### 添加任务

得到 pool 对象之后, `pool << args` 把任务的参数加入 `@task_args_queue` 队列中.

如果现有的工作线程已经在饱和运转, 就可以考虑新增工作线程了.

`@go_to_work.signal` 信号激活了第一个休息的空闲线程, 醒来的子线程把空闲线程数减一, 通过 `@handler.call(task_args)` 执行了具体的任务.

loop, 周而复始.

## 其他

* `spawn_thread` 方法通过 `with_metux` 的包装来使用 `@lock`, 一个目的是避免锁重入, 另一个目的是避免死锁. 

* PUMA 线程池里有一个内部类: `Automaton`, 可以新开一个线程来专门收割多余的线程或终止的线程.

* PUMA 参数中的 timeout 是精确的时间差计算, 使用 `CLOCK_MONOTONIC` 而不是 `CLOCK_REALTIME`:

```ruby
Process.clock_gettime(Process::CLOCK_MONOTONIC)
```

* `CLOCK_REALTIME`  不能保证单调递增, 从 1970 年算起;
* `CLOCK_MONOTONIC` 可以保证单调递增, 从本机开机算起;

## 参考

[https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb](https://github.com/puma/puma/blob/master/lib/puma/thread_pool.rb)
