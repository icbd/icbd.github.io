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
