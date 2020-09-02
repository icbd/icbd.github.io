---
layout: post
title:  Process Signal in Ruby
date:   2020-08-31
Author: CBD
tags: [Ruby, Signal]
---

## 引子

虽然 GIL 限制了 Ruby 多线程利用多核的能力, 但还可以通过多进程方式来使用多核资源.

master -- worker 是多进程常用的组织方式, 一个 master 进程负责管理调度, 多个 worker 进程处理具体的业务逻辑.

那么, master 和 worker 之间如何通信呢? 除了利用文件,TCP,UNIX Socket这样的共享资源, 其实原始也高效的方式就是利用信号.

本篇主题是 **master 进程如何正确处理 worker 的信号**, 在介绍主题前, 先回顾一下相关基础.

## fork

fork 派生新的进程, 子进程继承了父进程内存中的所有内容，包括已打开的文件描述符.

* 父进程 fork 返回 子进程的 pid;
* 子进程 fork 返回 nil;

## wait 和 wait 家族

```ruby
father_pid = Process.pid

child_pid = fork do
  puts "child>>"
  sleep 2
end

p Process.wait(child_pid)
p "father: #{father_pid}, child: #{child_pid}"
=begin
child>>
6549
"father: 6547, child: 6549"
=end
```

`Process.wait(pid)` 默认是阻塞的, 等 pid 对应的子进程退出才返回, 返回 pid;

```ruby
father_pid = Process.pid

child_pid = fork do
  puts "child>>"
  sleep 2
end

p Process.wait(child_pid, Process::WNOHANG)
p "father: #{father_pid}, child: #{child_pid}"
=begin
nil
"father: 6558, child: 6560"
child>>
=end
```

`Process.wait(pid, Process::WNOHANG)` 是非阻塞的:

* 如果 pid 对应的子进程已经退出了, 立即返回 pid;
* 如果 pid 对应的子进程还没有退出, 立即返回 nil.

`wait` 的 pid 可以是 `-1` 或留空, 都表示等待任意子进程.

`wait2` 跟 `wait` 的区别在返回值: `pid, status =  Process.wait2(pid)` .

## Signal 与 僵尸进程

`Signal.list` 可以查看所有信号, 更详细的信息可以查看 `man signal` 相关的章节.

```text
Signal      Standard   Action   Comment
────────────────────────────────────────────────────────────────────────
SIGABRT      P1990      Core    Abort signal from abort(3)
SIGALRM      P1990      Term    Timer signal from alarm(2)
SIGBUS       P2001      Core    Bus error (bad memory access)
SIGCHLD      P1990      Ign     Child stopped or terminated
SIGCLD         -        Ign     A synonym for SIGCHLD
SIGCONT      P1990      Cont    Continue if stopped
SIGEMT         -        Term    Emulator trap
SIGFPE       P1990      Core    Floating-point exception
SIGHUP       P1990      Term    Hangup detected on controlling terminal
                                or death of controlling process
SIGILL       P1990      Core    Illegal Instruction
SIGINFO        -                A synonym for SIGPWR
SIGINT       P1990      Term    Interrupt from keyboard
SIGIO          -        Term    I/O now possible (4.2BSD)
SIGIOT         -        Core    IOT trap. A synonym for SIGABRT
SIGKILL      P1990      Term    Kill signal
SIGLOST        -        Term    File lock lost (unused)
SIGPIPE      P1990      Term    Broken pipe: write to pipe with no
                                readers; see pipe(7)
SIGPOLL      P2001      Term    Pollable event (Sys V);
                                synonym for SIGIO
SIGPROF      P2001      Term    Profiling timer expired
SIGPWR         -        Term    Power failure (System V)
SIGQUIT      P1990      Core    Quit from keyboard
SIGSEGV      P1990      Core    Invalid memory reference
SIGSTKFLT      -        Term    Stack fault on coprocessor (unused)
SIGSTOP      P1990      Stop    Stop process
SIGTSTP      P1990      Stop    Stop typed at terminal
SIGSYS       P2001      Core    Bad system call (SVr4);
                                see also seccomp(2)
SIGTERM      P1990      Term    Termination signal
SIGTRAP      P2001      Core    Trace/breakpoint trap
SIGTTIN      P1990      Stop    Terminal input for background process
SIGTTOU      P1990      Stop    Terminal output for background process
SIGUNUSED      -        Core    Synonymous with SIGSYS
SIGURG       P2001      Ign     Urgent condition on socket (4.2BSD)
SIGUSR1      P1990      Term    User-defined signal 1
SIGUSR2      P1990      Term    User-defined signal 2
SIGVTALRM    P2001      Term    Virtual alarm clock (4.2BSD)
SIGXCPU      P2001      Core    CPU time limit exceeded (4.2BSD);
                                see setrlimit(2)
SIGXFSZ      P2001      Core    File size limit exceeded (4.2BSD);
                                see setrlimit(2)
```

一个进程退出了, 不论是正常退出还是意外退出, 它的退出信息都会被内核收集, 加入到一个队列中交由它的父进程处理.

父进程处理的方式有两种:

* wait, 主动收集子进程退出信息;
* detach, 开一线程再来 wait. [Rubinius Process#detach](https://github.com/rubinius/rubinius/blob/master/core/process.rb#L443-L458)

僵尸进程是很形象的描述, 说的是一个进程死了, 但尸体没有消失. 也就是说, 进程死掉后它的父进程一直没有为他收尸, 导致它的退出信息一直残留在内核中, 造成内核资源的浪费.

## Demo Code

子进程退出会给父进程发送 `CHLD` 信号, 但是这个信号是不可靠投递. 如果处理信号的过程中又接收到了另一个信号, 则可能会造成信号丢失, 进而导致 "收尸" 处理的遗漏.

```ruby
child_count = 3
dead_count = 0

child_count.times do
  fork do
    sleep 1
    puts "Child #{Process.pid} done."
  end
end

trap(:CHLD) do
  child_pid = Process.wait
  puts "process child pid: #{child_pid}"
  dead_count += 1
  exit if dead_count == child_count
end

sleep 10

```

> output

```text
Child 5711 done.
Child 5712 done.
Child 5713 done.
process child pid: 5713
process child pid: 5712
```

这是一个典型输出, 三个子进程结束, 父进程只处理了其中两个, 导致漏掉的那个子进程变成了僵尸进程.

<br/>

> 改进:

```ruby
child_count = 3
dead_count = 0

child_count.times do
  fork do
    sleep 1
    puts "Child #{Process.pid} done."
  end
end

trap(:CHLD) do
  begin
    while child_pid = Process.wait(-1, Process::WNOHANG)
      puts "process child pid: #{child_pid}"
      dead_count += 1
      exit if dead_count == child_count
    end
  rescue Errno::ECHILD
  end
end

sleep 10

```

> output:

```text
Child 5916 done.
Child 5915 done.
Child 5914 done.
process child pid: 5916
process child pid: 5915
process child pid: 5914
```

如此修改, 父进程就不会因为信号的不可靠投递而遗漏处理.

## `trap` More

trap 平时很少使用, 几乎只用于常驻后台的程序.

`trap(:INT)` 实际上是重新定义 `:INT` 信号的响应方式, 效果是全局的. 

如果已经定义了 `:INT` 的处理方式, 想要在此基础上添加逻辑:

```ruby
puts "pid: #{Process.pid}"

trap(:INT) do
  puts "processing>>>"
  exit
end

old_sigal = trap(:INT) do
  puts "new processing..."

  old_sigal.call if old_sigal.respond_to?(:call)
end

at_exit do
  puts "Bye~"
end

sleep 100

```

由于 trap 的全局性, 并不推荐对同一个信号多次定义, 如果只想在退出前做点什么, 用 `at_exit` 就足够了.

另外, 不能用这种方式调用信号的默认处理方式.

<br/>

信号经常跟进程的退出过程相关.

Ruby 执行完一段脚本后正常退出, 退出码为 `0`, 除此之外还有几种显示的退出方式:

|方法|退出码|是否调用`at_exit`|可选|
|---|---|---|---|
|exit|0|√| 指定退出码 `exit(code)` |
|exit!|1|x| 指定退出码 `exit!(code)` |
|abort|1|√| 打印错误消息 `abort(msg)` |
|raise(error)|1|√| 指定异常 `raise(StandardError.new)` |

## 参考

[https://man7.org/linux/man-pages/man2/signal.2.html](https://man7.org/linux/man-pages/man2/signal.2.html)

[https://man7.org/linux/man-pages/man7/signal.7.html](https://man7.org/linux/man-pages/man7/signal.7.html)

[https://github.com/rubinius/rubinius/blob/master/core/process.rb](https://github.com/rubinius/rubinius/blob/master/core/process.rb#L443-L458)
