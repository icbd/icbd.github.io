---
layout: post
title:  Go Lib ratelimit 的悲观锁与乐观锁
date:    2020-10-18
Author: CBD
tags: [Golang, Lib]
---

在 Gin 的官方示例中有这么一个限流示例:

```go
func leakBucket() gin.HandlerFunc {
	prev := time.Now()
	return func(ctx *gin.Context) {
    limit.Take()
    // ...
	}
}
```

完整示例见: [https://github.com/gin-gonic/examples/blob/master/ratelimiter/](https://github.com/gin-gonic/examples/blob/master/ratelimiter/)

`leakBucket` 是一个 Gin 的 middleware, `limit.Take()` 是由 `go.uber.org/ratelimit` 提供的限流方法.

如果设置 RPS 为 100: `ratelimit.New(100)`, 那么每个请求之间的间隔为 `1s / 100 = 10ms`, `limit.Take()` 会保证请求间隔不小于 `10ms`.

ratelimiter 的文档里说这实现了 `leaky-bucket rate limit algorithm`, 但读了源码之后发现...

## 源码结构

[https://github.com/uber-go/ratelimit](https://github.com/uber-go/ratelimit)

```text
.
├── CHANGELOG.md
├── LICENSE
├── Makefile
├── README.md
├── go.mod
├── go.sum
├── internal
│   └── clock
│       ├── clock.go
│       ├── interface.go
│       ├── real.go
│       └── timers.go
├── mutexbased.go
├── ratelimit.go
├── ratelimit_bench_test.go
├── ratelimit_test.go
└── tools
    ├── go.mod
    └── go.sum

```

## package clock

我们先看一下 `internal/clock/`, 其中 `clock.go` 是一堆 Mock, 可以暂时忽略, `timers.go` 没啥用也忽略.

`interface.go` 声明了 clock 的接口:

```go
type Clock interface {
	AfterFunc(d time.Duration, f func())
	Now() time.Time
	Sleep(d time.Duration)
}
```

作者的意图是让 Clock 可以由其他实现替换, 他自己写的实现在 `real.go` 中:

```go
package clock

import "time"

// clock implements a real-time clock by simply wrapping the time package functions.
type clock struct{}

// New returns an instance of a real-time clock.
func New() Clock {
	return &clock{}
}

func (c *clock) After(d time.Duration) <-chan time.Time { return time.After(d) }

func (c *clock) AfterFunc(d time.Duration, f func()) {
	// TODO maybe return timer interface
	time.AfterFunc(d, f)
}

func (c *clock) Now() time.Time { return time.Now() }

func (c *clock) Sleep(d time.Duration) { time.Sleep(d) }
```

`clock` 类型是空 struct, 作用只是充当实现接口的骨架, 接口方法代理给 time 包中的方法.

对这个库来说, 有用的方法只有 `Now()` 和 `Sleep()`, 你肯定也猜到了, 限流的核心就是让后面的请求睡觉😴

## mutexbased.go

这是库包含了两种实现, `mutexbased.go` 是基于 mutex 的版本, 更容易理解.

```go
type mutexLimiter struct {
	sync.Mutex
	last       time.Time // 上一个请求的到达时刻
	sleepFor   time.Duration // 需要等待的时间
	perRequest time.Duration // 由 RPS 计算得出的请求间隔
	maxSlack   time.Duration // 强制设置的请求间隔
	clock      Clock // 计时器: 获得当前时间 / 休眠一段时间
}
```

使用 mutex 的作用是, 当多个携程同时要求限流时, 通过抢夺mutex锁来串行化, 从而达到排队的效果.

`Take()` 限流的原理是比较当前时间跟上次请求的时间差:

* 如果大于限流间隔, `Take()` 立即返回, 也就不影响新请求的执行;
* 如果小于限流间隔, 通过 sleep 把间隔补齐到要求的间隔, 然后再返回.

## ratelimit.go

`ratelimit.go` 是 Gin 示例使用的版本, 是基于 `sync/atomic` 实现的.

```go
type state struct {
	last     time.Time
	sleepFor time.Duration
}

type limiter struct {
	state unsafe.Pointer
	//lint:ignore U1000 Padding is unused but it is crucial to maintain performance
	// of this rate limiter in case of collocation with other frequently accessed memory.
	padding [56]byte // cache line size - state pointer size = 64 - 8; created to avoid false sharing.

	perRequest time.Duration
	maxSlack   time.Duration
	clock      Clock
}
```

这里把 `last` 和 `sleepFor` 属性单独拆到一个 `state` 结构体中, 便于在 `sync.atom` 中存取.

限流思路跟上面一样, 区别是锁的控制.

`mutexbased.go` 使用 mutex 是一种悲观锁, 无论请求频率如何, 都有 mutex 的操作.

`ratelimit.go` 利用 `sync.atom`, 使用 `CompareAndSwap` 的方式, 先计算时间间隔, 在给赋值前检查`state`:

* 如果没更新过, 说明没有其他携程的并发干扰, 可以赋值;
* 如果被更新过, 那么就重新计算时间间隔, 重走当前流程.
