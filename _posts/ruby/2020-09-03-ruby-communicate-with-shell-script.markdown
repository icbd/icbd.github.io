---
layout: post
title:  Ruby Communicate with the Shell Script
date:   2020-09-03
Author: CBD
tags: [Ruby, Shell]
---

## 引子

Ruby 是表达能力很好的胶水语言, 对接 API 很方便. Remote 的 API 通常是 RESTFul 或 RPC; Local 的 API 通常是 TCP 或 UNIX Socket. 但 API 也可能只是普通脚本, 比如 Git, 假如我们只能通过 `/usr/local/bin/git` 来跟 git 仓库交互, 通过 Ruby 怎么来对接呢? 

## `Kernel#exec`

```ruby
puts "PID: #{Process.pid}"
exec "/usr/local/bin/code"

puts "end."
```

`exec("command")`是一个粗暴的方法, 直接把当前进程替换为 `command`. 

执行上面脚本, 会打开 code, 但是还没来得及打印 `end.` Ruby 脚本就已经退出了.

## `Kernel#spawn`

大部分情况下, 并不想像 `exec` 那样退出脚本, 变通一下:

* 先 fork;
* 在子进程中执行 `exec`;
* 父进程等子进程结束再继续.

```ruby
arg = rand(5).to_s
puts "PID: #{Process.pid}, arg: #{arg}"

child_pid = spawn("/bin/sleep", arg)
Process.wait child_pid

p "child_pid: #{child_pid}"

=begin
PID: 5596, arg: 2
"child_pid: 5597"
=end
```

## `Kernel#system`

`system` 包装了 `spawn` 和 `wait`, 返回布尔值来表示命令是否正常退出:

```ruby
result = system("/bin/sleep", "2")
p result # true
```

## `%s{ command }` or `` ` command ` ``

通常我们还想要得到 command 的输出值:

```ruby
result = %x{/bin/sleep 2; /bin/date +%H:%M:%S}
p result # "18:23:47\n"
```

`` ` command` `` 或许是你最常用的方式. ruby-git 这个 Gem 也是这样做的, 通过层层包装, 最终对接部分是利用它来执行, 详见 [lib/git/lib.rb](https://github.com/ruby-git/ruby-git/blob/861eb71e1c266606eefacf7ebd4bee4f34bee5de/lib/git/lib.rb#L1073-L1077) .

## Open3

更近一步, 如果这个 command 是一个交互界面, 需要多次输入命令, `Open3` 能帮我们做 IO 方面的绑定:

```ruby
require 'open3'
require 'securerandom'

Open3.popen2("redis-cli") do |std_in, std_out|
  std_in.puts "SET uuid #{SecureRandom.uuid}"
  p std_out.gets
  # "OK\n"

  std_in.puts "GET uuid"
  p std_out.gets
  # "1c8fbfed-91f6-47fa-acd7-dacefd15195a\n"
end

```

## 小结

`` `command` `` 最方便, 也是大部分情况下需要的;

交互式命令行使用 `Open3`.
