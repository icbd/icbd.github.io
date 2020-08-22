---
layout: post
title:  Ruby ConnectionPool 
date:   2020-08-22
Author: CBD 
tags: [Ruby, ConnectionPool]
---

## 引子

ConnectionPool 连接池, 用在并发场景下的连接请求多路复用.

#### 一个支持并发的 server

```ruby
#!/usr/bin/env ruby
require 'rack'
require 'rack/handler/puma'
require 'json'

class App
  attr_reader :env, :request

  # Each new call generates a new App instance.
  def self.call(env)
    self.new(env).handle
  end

  def initialize(env)
    @env = env
    @request = Rack::Request.new(env)
  end

  def handle
    sleep wait_time
    ['200', {'Content-Type' => 'application/json'}, [payload]]
  end

  private

  def payload
    {
      uri: env['REQUEST_URI'],
      pid: Process.pid,
      thread: Thread.current.name
    }.to_json
  end

  def wait_time
    Float(request.params.fetch('wait', 0))
  end
end

Rack::Handler::Puma.run(App, {Verbose: true, Port: 8888, Threads: '3:3'})

```
 
`App` 符合 Rack 的标准: 响应 `call` 的调用, 接收 env 返回三元数组.

PUMA 作为 App 的服务器, 工作在单工作进程模式, 共三个工作线程.

#### 一个不支持并发的 client

```ruby
client = Net::HTTP.new('127.0.0.1', 8888)
resp = client.request_get("/?wait=2")
```

client 向 `http://127.0.0.1:8888/?wait=2` 发起一个 GET 请求, server 根据 wait 参数休眠 2 秒来模拟耗时请求.

这里的 client 不支持并发, 如果多线程共用一个 client, 线程间会 IO 干扰, 导致抛 `stream closed in another thread (IOError)`.

## 用例

```ruby
require 'connection_pool'
require 'net/http'
require 'json'

CONN_POOL_SIZE = 2 # 连接池容量
THR_COUNT = 3 # 服务器 Worker 线程数
REQ_COUNT = 10 # 请求数
WAIT = 2 # 每个请求耗时

clients = ConnectionPool.new(size: CONN_POOL_SIZE, timeout: 100) do
  Net::HTTP.new('127.0.0.1', 8888)
end

request_proc = Proc.new do |wait|
  clients.with do |client|
    resp = client.request_get("/?wait=#{wait}")
    JSON.parse resp.body
  end
end

threads = []
start_at = Time.now.to_f
REQ_COUNT.times do
  threads << Thread.new { p request_proc.call(WAIT) }
end
threads.each(&:join)

puts "THR_COUNT: #{THR_COUNT}, REQ_COUNT: #{REQ_COUNT}, WAIT: #{WAIT}."
puts "ceil(REQ_COUNT/ min(THR_COUNT,CONN_POOL_SIZE)) * WAIT = #{(1.0 * REQ_COUNT / [THR_COUNT, CONN_POOL_SIZE].min).ceil * WAIT}"
puts " Total costing: #{Time.now.to_f - start_at}"

```

这个例子一方面展示了 ConnectionPool 的使用, 另一方面也说明了, 服务器的并发能力和连接池容量一致才能达到最高的效率.

## 源码

源码分析基于 ConnectionPool 2.2.3 的版本.
 
该版本移除了 `monotonic_time.rb`, 直接采用 `Process.clock_gettime(Process::CLOCK_MONOTONIC)` 来获取时间戳.

TimedStack 是核心组件, 用法如下:

```ruby
ts = TimedStack.new(1) { MyConnection.new }
ts.push conn
ts.pop timeout: 5
```

TimedStack 是一个栈:
* 可以线程安全地向 ts 中压入对象;
* 可以从 ts 中取出对象, 取不到就阻塞地等待, 超时了就报错.

```ruby
    deadline = current_time + timeout
    @mutex.synchronize do
      loop do
        raise ConnectionPool::PoolShuttingDownError if @shutdown_block
        return fetch_connection(options) if connection_stored?(options)

        connection = try_create(options)
        return connection if connection

        to_wait = deadline - current_time
        raise ConnectionPool::TimeoutError, "Waited #{timeout} sec" if to_wait <= 0
        @resource.wait(@mutex, to_wait)
      end
    end
```

pop 超时的实现提供了一种思路:

1. 池内有连接时, 直接返回; 
2. 池内没连接但还可以生成时, 返回生成的连接;
3. 检查过期时间;
4. 在时间限制内阻塞等待

wait 使用了 timeout 参数, timeout 时间一到即返回, 然后进入下一圈循环, 步骤 1 步骤 2 还是不满足时, 步骤 3 肯定过期. 

>   Ruby 库内置的 `Timeout::timeout` 为计时新开了一个线程 `y`. 
    如果执行 sleep 的 y 线程先结束, 也就是 x 对应的当前线程执行超时了, 就抛出异常; 
    反之 y 线程会被杀掉, 正常返回 x 的执行结果.
    详见: [https://github.com/ruby/ruby/blob/master/lib/timeout.rb](https://github.com/ruby/ruby/blob/master/lib/timeout.rb)

ConnectionPool 通过包装 TimedStack 实现了连接池的效果.

with 中, 先 `checkout` 取连接, `yield conn` 使用连接, 再 `checkin` 归还连接.
 
```ruby
  def with(options = {})
    Thread.handle_interrupt(Exception => :never) do
      conn = checkout(options)
      begin
        Thread.handle_interrupt(Exception => :immediate) do
          yield conn
        end
      ensure
        checkin
      end
    end
  end

  def checkout(options = {})
    if ::Thread.current[@key]
      ::Thread.current[@key_count] += 1
      ::Thread.current[@key]
    else
      ::Thread.current[@key_count] = 1
      ::Thread.current[@key] = @available.pop(options[:timeout] || @timeout)
    end
  end
```

## 小结

池内连接数从 0 开始, 根据并发需求增长, 直到数目达到 size 限制.

多线程场景下, 每个线程内获得的连接都是独立的连接对象;

同一个线程内, 如果顺序执行会依次从池中取用, 归还, 再取, 再还;

同一个线程内, 如果嵌套 with, 得到的是同一个连接.  

## 参考

[https://github.com/ruby/ruby/blob/master/lib/timeout.rb](https://github.com/ruby/ruby/blob/master/lib/timeout.rb)

[https://github.com/mperham/connection_pool/](https://github.com/mperham/connection_pool/)
