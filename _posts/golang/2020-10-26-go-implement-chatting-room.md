---
layout: post
title:  Golang Implemnt Chatting Room
date:   2020-10-26
Author: CBD
tags: [Golang]
---

源码详见: [https://github.com/icbd/go_rough_chatting_room](https://github.com/icbd/go_rough_chatting_room) .

基本功能需求:

* 在客户端中键入消息, 客户端把消息发送给服务端;
* 服务端把消息广播给所有客户端, 客户端收到广播显示消息.

## Client

```go
//func Copy(dst Writer, src Reader) (written int64, err error)

// 收集键入的消息
io.Copy(c.conn, os.Stdin)

// 展示广播消息
io.Copy(os.Stdout, c.conn)
```

`io.Copy` 把标准输入/输出直接跟连接进行绑定.

* 一旦 `os.Stdin` 有新的内容, 就复制给连接;
* 一旦 连接中有新的内容, 就复制给 `os.Stdout`.

因为连接是全双工的, 这两个动作可以同时进行.

`io.Copy` 内部的实现就是一个大的无限循环, 也就让 `io.Copy` 表现出阻塞的特点. 

`os.Stdin` 遇到 `EOF` 会主动跳出阻塞; `os.Stdout` 不会主动跳出阻塞, 除非连接已经断开.

这样思路就清楚了, 由于输入方向和输出方向需要同时工作, 所以把其中一个方向的工作放到单独的 goroutine 中.

由于 `Stdin`的 `Copy` 会根据 `EOF` 主动跳出, 所以由 `Stdin` 这个方向来负责连接的关闭.

`Stdout` 这方在会因为连接关闭而触发异常, 他应该识别这个情况, 并且在完成善后工作之后, 告诉主 goroutine 已经全部做好了, 可以退出了, 这个通信我们借由一个 channel 来实现.

最终的核心代码:

```go
	go c.printMsgFromConn() // c.allDown <- struct{}{}
	c.typeMsgToConn() // EOF trigger. then close connection
	<-c.allDown
```

## Server

服务端的核心是把消息广播给各个客户端, 我们需要一个集合来存储所有在线的客户端, 然后遍历给他们发消息, 这显然不能用阻塞的 `io.Copy`, 而是用写入即返回的 `fmt.Fprintln(conn, msg)`.

定义一个无缓存通道: `type clientChan chan string`.

为每个连接创建各自的 clientChan, 在单独的 goroutine 中, 通过 `range clientChan` 来监听新消息的传入, 一旦有新消息就取出来送给连接, 发送给客户去显示.

每当服务器监听到新的连接, 就把他收集起来, 再发送一个新用户上线的广播.

每当用户下线, 就发送一个下线广播, 从集合里踢出这个用户, 关闭该用户的 `clientChan`, 最后断开连接.

服务端还需要注意, 不能因为一个用户的连接异常而影响其他连接, 输出 log 而不是直接 panic.

核心方法:

```go
func (s *server) broadcast() {
	for {
		select {
		case c := <-s.entering:
			s.clients[c] = true
		case c := <-s.leaving:
			delete(s.clients, c)
			close(c)
		case msg := <-s.microphone:
			log.Println("[msg]" + msg)
			for c := range s.clients {
				c <- msg
			}
		}
	}
}
```

```go
func (s *server) handle(conn net.Conn) {
	c := make(clientChan)
	go c.sendMsg(conn)

	who := conn.RemoteAddr().String()
	c <- "你的ID: " + who
	s.microphone <- who + "上线了!"
	s.entering <- c

	input := bufio.NewScanner(conn)
	for input.Scan() {
		s.microphone <- "\t\t" + who + ":" + input.Text()
	}
	// conn disconnected
	s.leaving <- c
	s.microphone <- who + "下线了."
	if err := conn.Close(); err != nil {
		log.Println("[Err]" + err.Error())
	}
}

func (cC clientChan) sendMsg(conn net.Conn) {
	for msg := range cC {
		if _, err := fmt.Fprintln(conn, msg); err != nil {
			log.Println("[Err]" + err.Error())
		}
	}
}
```
