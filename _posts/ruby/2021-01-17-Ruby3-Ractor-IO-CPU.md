---
layout: post
title:  Ruby 3 Multi Thread VS Ractor 
date:   2021-01-17
Author: CBD
tags:   [Ruby, MultiThread, Ractor]
---

代码详见 Repo: [https://github.com/icbd/ruby3-MutlThread-vs-Ractor](https://github.com/icbd/ruby3-MutlThread-vs-Ractor)

## Ractor 要点

> 不要通过共享内存来通信，而应该通过通信来共享内存.

CRuby 中, 每个 Ractor 持有各自的 GVL, 所以他们能实现真正的并行. 每个 Ractor 创建一个自己的线程.

* Push 模式
    `Ractor#send` & `Ractor.receive`. sender 给已知的 receiver 发消息.

* Pull 模式
    `Reactor.yield` & `Reactor#take`. receiver 从已知的 sender 收消息.

既然是通过发消息来通信, 那么消息就一定是不可变的对象, 在 Ruby 里就是 `freeze` 的.
如果传入一个可变对象, 默认会通过深拷贝的方式 `复制` 一份, 当然也可以选择把对象 `移动` 到 Ractor 内部(这个对象在外面将不能被访问).

## 测试平台

MacBook Pro (16-inch, 2019)

2.6 GHz 六核Intel Core i7

## 测试 Route

只有 IO 耗时:
`curl -v http://localhost:3000/io`

只有计算密集:
`curl -v http://localhost:3000/cpu`

IO 耗时加计算密集:
`curl -v http://localhost:3000/io-cpu`

## 测试数据

`ab -n 10 -c 1 http://localhost:3000/io`

```
multi_threads.rb
Time taken for tests:   20.050 seconds

multi_ractor.rb
Time taken for tests:   20.054 seconds
```

`ab -n 10 -c 10 http://localhost:3000/io`

```
multi_threads.rb
Time taken for tests:   2.011 seconds

multi_ractor.rb
Time taken for tests:   2.005 seconds
```

`ab -n 10 -c 1 http://localhost:3000/cpu`

```
multi_threads.rb
Time taken for tests:   19.973 seconds

multi_ractor.rb
Time taken for tests:   19.651 seconds
```

`ab -n 10 -c 10 http://localhost:3000/cpu`

```
multi_threads.rb
Time taken for tests:   19.618 seconds

multi_ractor.rb
Time taken for tests:   3.160 seconds
```

## 数据解释

`-n 10 -c 1` 作为对照组, 一个 client  重复顺序请求十次, 计算得出平均每个请求的耗时.

`-n 10 -c 10` 作为实验组, 是个 client 总共请求十次, 查看并发情况下的总耗时.

另外, 由于 `Ractor.recv` 可以方便地在 loop 内取参数而不用每次创建新的 Ractor, 
为了抹平创建线程的时间开销, 多线程模式我们使用了池化的线程池. (创建线程的开销在这个例子中其实可以忽略不计)

我们用 `sleep` 模式 `IO`.

可以看到多线程模式和 Ractor 模式对于 IO 的并发都处理的很好, 这是因为 GIL 遇到 IO 就会释放.
对于 IO 密集型的 APP, 是否能使用多核其实影响并不大, 重点是对 IO 进行合理的异步处理.

我们用计算斐波那契来模拟 CPU 密集型计算.

可以看到多线程模式下, 并发总耗时跟顺序请求没什么差异, 因为在只使用单核的情况下, 顺序的 `fib` 已经能让单个 CPU 跑满了, 多线程对并发解决问题没有帮助, 反而还会白白增加线程调度的开销.
Ractor 模式处理 CPU 密集型计算的优势就很好的展现了. 每个 Ractor 持有各自的 GIL, Ractor 相互之间的 GIL 不会相互干扰, 能利用多 CPU 真正 **并行** 处理多个 `fib` 计算.

## 参考

[https://docs.ruby-lang.org/en/3.0.0/Ractor.html](https://docs.ruby-lang.org/en/3.0.0/Ractor.html)

[https://docs.ruby-lang.org/en/3.0.0/doc/ractor_md.html](https://docs.ruby-lang.org/en/3.0.0/doc/ractor_md.html)
