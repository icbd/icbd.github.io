---
layout: post
title:  Orphan Process and Daemon Process in Ruby
date:   2020-09-01
Author: CBD
tags: [Ruby, Process]
---

## 引子

使用线程时, 当主线程退出, 也就意味着进程要退出了, 会回收该进程内的资源, 子线程自然也就是随之退出.

使用进程时, 当主进程要退出, 会回收该进程内的资源, 但不会影响其子进程的运行. 

对于一个进程, 如果他的父进程退出了, 那么这个子进程就称作孤儿进程.

## 孤儿进程 Orphan Process

```ruby
father_pid = Process.pid
child_pid = fork do
  loop do
    sleep 1
    printf "."
  end
end

p "father: #{father_pid}, child: #{child_pid}"
sleep 20
p "father process end."
```

我们让子进程不停地打印 `"."`, 父进程 20 秒后退出.

根据输出 `"father: 7016, child: 7018"` , 在父进程退出前:

```sh
% ps -fp 7016,7018
  UID   PID  PPID   C STIME   TTY           TIME CMD
  501  7016  2259   0  2:22下午 ttys000    0:00.08 ruby _process.rb
  501  7018  7016   0  2:22下午 ttys000    0:00.00 ruby _process.rb
```

20 秒后, 父进程退出了:

```sh
% ps -fp 6837,6839
  UID   PID  PPID   C STIME   TTY           TIME CMD
  501  6839     1   0  2:16下午 ttys000    0:00.01 ruby _process.rb
```

此时 pid 6839 就是一个孤儿进程, 他的父进程已经退出, PPID 变为了 1.

<br/>

单纯的孤儿进程并不能满足后台任务的需求:

关闭 terminal, 再重开一个新的,

```sh
% ps -fp 6837,6839
  UID   PID  PPID   C STIME   TTY           TIME CMD
```

发现 PID 为 6839 的子进程也退出了. 

怎么把孤儿进程改造成守护进程呢?

## 守护进程 Daemon Process

```ruby
child_pid = fork

tag = child_pid ? "[Parent]" : "[Child]"
format = "%-15s%-15s: %d\n"
printf(format, tag, "ProcessID", Process.pid)
printf(format, tag, "ProcessGroupID", Process.getpgrp)
printf(format, tag, "SessionID", Process.getsid)

exit if child_pid

loop do
  sleep 0.5
  printf '>'
  sleep 0.5
  printf '-'
end

```

父进程打印完信息就退出了. 子进程先打印信息, 再不停的交替打印 ">" "-".

>output:

```text
[Parent]       ProcessID      : 33727
[Parent]       ProcessGroupID : 33727
[Parent]       SessionID      : 33495
[Child]        ProcessID      : 33728
[Child]        ProcessGroupID : 33727
[Child]        SessionID      : 33495
cbd@cbd-mbp hello_ruby % >->->->->->->->->->->->->->->->->->->->->->->->-
```

ProcessID 即进程 ID, 父进程 PID 33727, 子进程 PID 33728.

ProcessGroupID 即进程组 ID, 每个进程都属于某个进程组, 系统会自动把主进程和他的子进程编到同一个组, 进程组 ID 默认为主进程 ID.

仔细查看 `Process.kill("KILL", pid)` 的文档会发现, pid 参数后面另有文章:

* pid > 0, 杀死 pid 对应的进程;
* pid == 0, 杀死本进程所在的进程组内的所有进程;
* pid < 0, 杀死进程组 ID 为 `abs(pid)` 的进程组.

对于上面例子:

* 父进程已经退出了;
* 子进程继续执行;
* 子进程的 ppid 变为 1;
* 子进程的进程组 ID 不变 (依旧是其父进程的 ID) .

可以这样杀死上例的子进程:

```shell
% kill -9 33728
// or
% kill -9 -33727
```

SessionID 即会话 ID, 他对应的进程是你执行脚本所在的 Shell, 看起来像这样:

```sh
cbd@cbd-mbp process % ps -fp 3624
  UID   PID  PPID   C STIME   TTY           TIME CMD
  501  3624  1544   0 11:56上午 ttys001    0:00.08 /bin/zsh --login -i
```

Shell 有转发信号的效果:

* 给一个交互进程发信号, 该信号会被转发到进程组内的每个进程;
* 给一个 Shell 的 Session 发信号, 该信号会转发到该会话的所有进程组.

这就解释了为什么关闭 terminal 之后, 子进程也被退出了.

如果尝试 `kill -9 SessionID`, 你会发现那个 terminal 窗口关闭了, 进程退出了, 子进程也退出了,  跟手动 `Command+Q` 效果一样.

## Process.setsid

看到这里, 估计你已经知道怎么产生一个守护进程了:

* 逃离父进程;
* 逃离进程组;
* 逃离 Shell 的 Session.

`Process.setsid` 可以帮你逃离:

```ruby
exit if fork

puts "ChildProcessPID: #{Process.pid}"
puts "NewSessionID: #{Process.setsid}"
puts "NewSID: #{Process.getsid}"
puts "NewProcessGroupID: #{Process.getpgrp}"
sleep

=begin
ChildProcessPID: 5289
NewSessionID: 5289
NewSID: 5289
NewProcessGroupID: 5289
=end
```

`Process#setsid` 会新开一个 Session 并设置 Session ID 为当前 PID, 清空 TTY; 还会新开一个进程组并设置进程组 ID 为当前 PID.

## daemon in Rubinius

MRI 的 `Process.daemon` 是 C 实现的, Rubinius 中有等效的 Ruby 实现:

> [https://github.com/rubinius/rubinius/blob/master/core/process.rb#L415-L433](https://github.com/rubinius/rubinius/blob/master/core/process.rb#L415-L433)

```ruby
  def self.daemon(stay_in_dir=false, keep_stdio_open=false)
    # Do not run at_exit handlers in the parent
    exit!(0) if fork

    Process.setsid

    exit!(0) if fork

    Dir.chdir("/") unless stay_in_dir

    unless keep_stdio_open
      io = File.open "/dev/null", File::RDWR, 0
      $stdin.reopen io
      $stdout.reopen io
      $stderr.reopen io
    end

    return 0
  end
```

## 参考

[https://github.com/rubinius/rubinius/blob/master/core/process.rb#L415-L433](https://github.com/rubinius/rubinius/blob/master/core/process.rb#L415-L433)

[https://www.jstorimer.com/products/working-with-unix-processes](https://www.jstorimer.com/products/working-with-unix-processes)

[https://www.zhihu.com/question/21711307](https://www.zhihu.com/question/21711307)
