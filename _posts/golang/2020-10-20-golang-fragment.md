---
layout: post
title:  Go Code Fragment
date:   2020-10-20
Author: CBD
tags: [Golang]
---

收集了一些 Golang 代码片段.

## 强类型语言的鸭子类型

[https://books.studygolang.com/gopl-zh/ch7/ch7-12.html](https://books.studygolang.com/gopl-zh/ch7/ch7-12.html)

```go
// writeString writes s to w.
// If w has a WriteString method, it is invoked instead of w.Write.
func writeString(w io.Writer, s string) (n int, err error) {
    type stringWriter interface {
        WriteString(string) (n int, err error)
    }
    if sw, ok := w.(stringWriter); ok {
        return sw.WriteString(s) // avoid a copy
    }
    return w.Write([]byte(s)) // allocate temporary copy
}
```

## package 级别变量, 首字母大写暗示常量的语义

[https://github.com/golang/go/blob/release-branch.go1.15/src/flag/flag.go#L1010-L1013](https://github.com/golang/go/blob/release-branch.go1.15/src/flag/flag.go#L1010-L1013)

```go

// CommandLine is the default set of command-line flags, parsed from os.Args.
// The top-level functions such as BoolVar, Arg, and so on are wrappers for the
// methods of CommandLine.
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```

## 公开方法包装私有属性

[https://github.com/golang/go/blob/release-branch.go1.15/src/flag/flag.go#L993-L996](https://github.com/golang/go/blob/release-branch.go1.15/src/flag/flag.go#L993-L996)

```go
// Parsed reports whether f.Parse has been called.
func (f *FlagSet) Parsed() bool {
	return f.parsed
}
```

## 组合 goroutine 的返回

[https://github.com/adonovan/gopl.io/blob/b725d6015f980e94734da37e35ba0d943fc7532f/ch8/thumbnail/thumbnail_test.go#L114-L148](https://github.com/adonovan/gopl.io/blob/b725d6015f980e94734da37e35ba0d943fc7532f/ch8/thumbnail/thumbnail_test.go#L114-L148)

```go
// makeThumbnails6 makes thumbnails for each file received from the channel.
// It returns the number of bytes occupied by the files it creates.
func makeThumbnails6(filenames <-chan string) int64 {
	sizes := make(chan int64)
	var wg sync.WaitGroup // number of working goroutines
	for f := range filenames {
		wg.Add(1)
		// worker
		go func(f string) {
			defer wg.Done()
			thumb, err := thumbnail.ImageFile(f)
			if err != nil {
				log.Println(err)
				return
			}
			info, _ := os.Stat(thumb) // OK to ignore error
			sizes <- info.Size()
		}(f)
	}

	// closer
	go func() {
		wg.Wait()
		close(sizes)
	}()

	var total int64
	for size := range sizes {
		total += size
	}
	return total
}

```

`sizes` 是一个 channel, goroutine 中每计算完一项, 都把各自部分的 size 塞入 `sizes`.

对 `sizes` 使用 range 迭代, 计算 `sizes` 的总和, 待 `sizes` 被 close 的时候迭代结束.

在单独的 goroutine 中 `wg.Wait()`, 等各个部分结束后 `close(sizes)`, 主 goroutine 的 range 从而得以退出.

这个例子之所以使用如此复杂的多个 goroutine 相互交织, 根本原因是我们不知道 filenames 的规模.

如果入参是一个明确 cap 的 slice, 我们就可以使用 buffer 先收集结果, 在主 goroutine 中 wait, 最后统一计算总和.

## 用 channel 并发限流

[https://github.com/adonovan/gopl.io/blob/master/ch8/crawl2/findlinks.go#L20-L37](https://github.com/adonovan/gopl.io/blob/master/ch8/crawl2/findlinks.go#L20-L37)

```go
// tokens is a counting semaphore used to
// enforce a limit of 20 concurrent requests.
var tokens = make(chan struct{}, 20)

func crawl(url string) []string {
	tokens <- struct{}{} // acquire a token

	list, err := links.Extract(url)

	<-tokens // release the token

	return list
}
```

带容量的 channel 是天然的限流工具, 当容量满了之后, 再加入则会等待.
