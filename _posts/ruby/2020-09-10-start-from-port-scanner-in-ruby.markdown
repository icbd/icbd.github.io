---
layout: post
title:  Start From Port Scanner in Ruby
date:    2020-09-10
Author: CBD
tags: [Ruby, Process, TCP, Socket]
---

## Ruby Code

```ruby
# frozen_string_literal: true

require 'socket'

class PostScanner
  PORT_MAX = 65_536

  attr_reader :options, :opened

  def initialize(host, min, max = min, timeout = 1.0)
    abort "Max port number is #{PORT_MAX}" if max > PORT_MAX

    @options = {
      host: IPSocket.getaddress(host),
      min: min.to_i,
      max: max.to_i,
      timeout: timeout.to_f
    }
    @opened = []
    sockets
    puts "Port Size: #{sockets.size}"
    scan_ports
  end

  private

  def scan_ports
    loop do
      _, writable, = IO.select(nil, @sockets, nil, options.fetch(:timeout))
      break if writable.nil?

      writable.each do |socket|
        begin
          socket.connect_nonblock(socket.remote_address)
        rescue Errno::EISCONN
          port = socket.remote_address.ip_port
          @opened << port
          @sockets.delete(socket)
        rescue Errno::EINVAL, Errno::ENOTCONN => e
          @sockets.delete(socket)
        end
      end
    end
  end

  def sockets
    @sockets ||= begin
                   (options.fetch(:min)..options.fetch(:max)).map do |port|
                     begin
                       socket = Socket.new(:INET, :STREAM)
                       socket.setsockopt(:SOCKET, :REUSEADDR, true)
                       addr = Socket.sockaddr_in(port, options.fetch(:host))
                       socket.connect_nonblock(addr)
                     rescue Errno::EINPROGRESS
                       # nothing
                     end
                     socket
                   end
                 end
  end
end

Process.setrlimit(:NOFILE, PostScanner::PORT_MAX + 10)
scanner = PostScanner.new('github.com', 1, 20_000)
p scanner.opened.sort

```

## Process Limit

系统对每个进程都有资源限制, 如果同时打开过多的 socket, 会触发:

```text
Too many open files - socket(2) (Errno::EMFILE)
```

```sh
# 查看所有项目限制
ulimit -a
# 查看文件描述符限制
ulimit -n
# 修改文件描述符限制
ulimit -n 65536
```

如果在 macOS 上, 还需要设置:

```sh
# 查看 files 相关的属性
sysctl -a  | grep "files"
# 用管理员修改
sudo launchctl limit maxfiles 655360
sudo ulimit -n 655360
```

注意: ulimit 修改的是本进程 ( Shell ) 的资源限制, 由 shell 触发的脚本是 Shell 的子进程, 会继承这些配置. 如果需要修改系统的 "硬限制", 需要 root 权限.

另外, 还需要使用 `Process.setrlimit` 修改当前 Ruby 脚本的限制.

## `connect_nonblock`

`socket.connect_nonblock(addr)` 是非阻塞的, 特别的是, 它去正常处理连接时会抛异常: `Errno::EINPROGRESS`, 所以捕获这个异常之后不需要做处理.

`sockets` 方法同时打开了你想检测的所有端口, 让他们异步地进行 TCP 连接.

## `IO.select`

`scan_ports` 方法来处理这些连接的结果.

`IO.select` 对应 select 系统调用.

它是同步的, 使用 select 的好处是系统内核会帮我们监测 IO 对象, 一旦有有对象准备好了立即返回, 避免了在业务代码中对 IO 对象进行循环扫描:

```text
=> [[readable_list], [writable_list], [error_list]]
```

指定 timeout 是为了让 select 到期返回, 如果 timeout 内都没有准备好的 IO 对象, 那么返回 nil.

得到的 `writable` 是个数组, 内容是准备就绪的 socket 对象, 再次调用 `connect_nonblock` 来读取连接结果.
如果抛出 `Errno::EISCONN` 则说明该 TCP 连接成功了, 该端口即为开放的端口.


## 补充

在 `initialize` 中我们提前进行了 DNS 查询:

```ruby
IPSocket.getaddress(host)
```

如果推迟到 `sockaddr_in` 中查询, 一方面是因为确实只需要查询一次 DNS, 再就是因为 MRI 的 DNS 是依赖 C 扩展的. 

GIL 保证只有一个线程的 Ruby 代码在执行, 遇到 IO 会释放 GIL 从而提高 IO 整体的效率. 遇到 C 扩展时会一直持有 GIL , 不论 C 扩展中是否是 IO 操作. 这就是 DNS 查询多线程不友好的原因了.

也有解决方案, 用 Ruby 实现的 DNS 替换默认的:

```
require 'resolv'
require 'resolv-replace'
```
