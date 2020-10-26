---
layout: post
title:  Golang usefull closed channel
date:   2020-10-26
Author: CBD
tags: [Golang, concurrency]
---

## 源码来源

[https://github.com/adonovan/gopl.io/blob/master/ch9/memo4/memo.go](https://github.com/adonovan/gopl.io/blob/master/ch9/memo4/memo.go)

## 需求分析

由于 Golang 的 GC 会处理 channel, 所以一般情况下不需要显示地关闭 channel.

这个例子展示了如何利用已经关闭了的 channel 的特性:

> 当一个 channel 已经关闭后, `<-ch` 会立即返回零值.

how to use:

```go
package main

import (
	"fmt"
	"sync"
	"time"
)

var cache = New(fetch)

func main() {
	var wg sync.WaitGroup
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go getValue(&wg)
	}
	wg.Wait()
	fmt.Println("-----")
	for i := 0; i < 3; i++ {
		wg.Add(1)
		go getValue(&wg)
	}
	wg.Wait()
}

func getValue(wg *sync.WaitGroup) {
	defer wg.Done()

	start := time.Now()
	r, _ := cache.Get("Hi")
	fmt.Printf("%s, %s\n", r, time.Now().Sub(start))
}

func fetch(key string) (interface{}, error) {
	time.Sleep(1 * time.Second)
	fmt.Println("Calculating...")
	return "<" + key + ">", nil
}

```

`fetch` 是个耗时方法, cache 想缓存他的结果, 当再次请求时立即返回缓存值.

我们期待的输出如下:

* 第一次请求触发耗时计算, 之后的请求直接使用缓存;
* 如果并发请求一个耗时计算, 第一个抢到锁的任务进行计算, 其他任务通过缓存获取.

```text
/*
Calculating...
<Hi>, 1.003371641s
<Hi>, 1.003400981s
<Hi>, 1.003390591s
-----
<Hi>, 833ns
<Hi>, 486ns
<Hi>, 694ns
*/
```

## 实现

```go
package main

import "sync"

type timeConsumingFunc func(key string) (interface{}, error)

type entry struct {
	value interface{}
	err   error
	ready chan struct{}
}

// entry collection
type Cache struct {
	f     timeConsumingFunc
	lock  sync.Mutex
	cache map[string]*entry
}

func New(f timeConsumingFunc) *Cache {
	return &Cache{
		f:     f,
		cache: make(map[string]*entry),
	}
}

func (c *Cache) Get(key string) (value interface{}, err error) {
	c.lock.Lock()
	entryPt := c.cache[key]
	if entryPt == nil {
		// First Get
		entryPt = &entry{ready: make(chan struct{})}
		c.cache[key] = entryPt
		c.lock.Unlock()

		entryPt.value, entryPt.err = c.f(key)
		close(entryPt.ready)
	} else {
		// by cache
		c.lock.Unlock()
		<-entryPt.ready
	}
	// get value and error by pointer
	return entryPt.value, entryPt.err
}

```

最巧妙的设计是, Cache 集合中, 存放的不是 `entry`, 而是 `entry` 的指针.

### flow-one: 无并发第一次访问时

代码等效于:

```go
  c.lock.Lock()
	entryPt := c.cache[key]
	entryPt = &entry{ready: make(chan struct{})}
	c.cache[key] = entryPt
  c.lock.Unlock()

	entryPt.value, entryPt.err = c.f(key)
	close(entryPt.ready)
	return entryPt.value, entryPt.err
```

### flow-two: 无并发再次访问时

代码等效于:

```go
  c.lock.Lock()
	entryPt := c.cache[key]
	c.lock.Unlock()
	<-entryPt.ready
	return entryPt.value, entryPt.err
```

`entryPt.ready` 已经处于关闭的状态了, 执行到这里会立即返回空字符串.

### 当第一次访问存在并发

第一个抢到锁的 goroutine 会按照 `flow-one` 执行, 在 `c.lock.Unlock()` 之前, `entryPt` 已经设置到 cache 中了, 但是具体的结果还没计算.

锁释放后, 第二个抢到锁的 goroutine 得以进入, 代码等效于 `flow-two`, 此时的 entryPt 不是 nil, 他顺序执行, 释放锁, 然后阻塞地等待 `<-entryPt.ready` 的消息.

第二个 goroutine 释放锁后, 接着有另一个 goroutine 抢到锁, 同第二个一样的流程, 也阻塞地等待 `<-entryPt.ready` 的消息, 如果还有其他的并发也同样.

他们都在等第一个抢到锁的 goroutine 执行完:

```go
	entryPt.value, entryPt.err = c.f(key)
	close(entryPt.ready)
```

关闭的 channel 立即返回零值, `<-entryPt.ready` 有了消息, 这些并发的 goroutine 通过指针 `entryPt` 得到了已经缓存的计算结果.
