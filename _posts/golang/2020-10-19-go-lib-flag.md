---
layout: post
title:  Go Lib flag
date:   2020-10-19
Author: CBD
tags: [Golang, Lib]
---

`flag` 是官方提供的命令行参数解析的工具包.

## 基本用法

```go
package main

import (
	"flag"
	"fmt"
)

var name = flag.String("name", "Default Name", "Name setting")

func main() {
	flag.Parse()
	fmt.Printf("Hello %s\n", *name)
}
// go run main.go -name=John
// go run main.go -name="John smith"
```

## 自定义解析器

```go
package main

import (
	"flag"
	"fmt"
	"github.com/spf13/cast"
	"strings"
)

type User struct {
	Name string
	Age  int
}

func (u *User) String() string {
	return fmt.Sprintf("Name:%s, Age:%d\n", u.Name, u.Age)
}

// Set by Bob:12
func (u *User) Set(value string) error {
	splits := strings.Split(value, ":")
	if len(splits) == 2 {
		u.Name = splits[0]
		u.Age = cast.ToInt(splits[1])
		return nil
	}
	return fmt.Errorf("format error")
}

func FlagUser(name string, value string, usage string) *User {
	u := User{}
	_ = u.Set(value) // init default value
	flag.Var(&u, name, usage)
	return &u
}

var u = FlagUser("user", "Default Name:100", "parse User")

func main() {
	flag.Parse()
	fmt.Printf("%T %s\n", u, u)
}
// go run main.go -user=Bob:12

```

核心方法是 `func (f *FlagSet) Var(value Value, name string, usage string)`, `Value` 的接口如下所示:

```go
type Value interface {
	String() string
	Set(string) error
}
```

对于解析参数来说, 我们用不到 `String()`, 但是在语法上要实现这个接口就要实现 `String()` 方法.

还需要注意实现 `String()` 的时候不要直接`return fmt.Sprintln(u)`, 否则会导致方法无限递归而 stack overflow.

`FlagUser` 参照了 `flag` 自带方法的风格, 参数依次为 flag值, 默认值, 帮助提示, 最后返回一个自定义类型的指针.

## 源码解析

在开始读源码之前, 我们先看一下 `os.Args` 的解析规则, 它会得到一个字符串 slice.

```go
package main

import (
	"fmt"
	"os"
)

func main() {
	for _, s := range os.Args {
		fmt.Println(s)
	}
}
```

```shell
$ go run main.go  --s -s hello -s=hi --s="hello world"
```

output:

```text
/var/folders/jd/tgbfgqzn3z1c_qmcd0qcmcx80000gn/T/go-build687856139/b001/exe/main
--s
-s
hello
-s=hi
--s=hello world

```

规则要点如下:

* 每个元素都是字符串, 大小写敏感;
* 第一个元素为该程序的绝对路径;
* 后面的元素依次为空格分隔的参数, 连续空格视作一个分隔;
* 引号内的空格不做分隔, 自动脱去引号.

flag 在以上规则的基础上, 衍生出以下规则:

* 参数由 `-` 或 `--`开始, 用等号或空格分隔键值, `--s=1`, `-s=1`, `--s 1`, `-s 1` 等效;
* `-s` `-S` 为不同的标签;
* 对于布尔型的标签, 可以略去值来表示标签为 true: `-b`, `--b`;
* ` -- ` 后的参数将被忽略;
* 不允许重复注册标签.

### 核心结构体

像 flag 这样先注册再使用的模式(类似路由匹配), 一般会在注册时解析规则, 然后把规则保存到一个集合里, 在使用的时候去集合里匹配规则.

`FlagSet` 就是这样的一个集合, `Flag` 是其中的每个元素.

```go
type Flag struct {
	Name     string // 标签名: `-s` 或 `--s` 中 的 `s`
	Usage    string // 用途, 帮助信息.
	Value    Value  // Value 接口: `Set(v string) error` && `String() string`
	DefValue string // 标签的默认值, 为打印帮助信息所用.
}
```

```go
type FlagSet struct {
	Usage func()	// 帮助信息的打印函数, 默认为 `defaultUsage()`

	name          string // 当前程序的绝对路径
	parsed        bool // 标记已经执行过解析: `flag.Parse()`
	actual        map[string]*Flag // 由 `os.Args` 解析得到的标签
	formal        map[string]*Flag // 之前注册的标签
	args          []string // 命令行参数列表 (除去程序的绝对路径)
	errorHandling ErrorHandling // 异常处理的方式: 返回 error / 程序 exit / panic
	output        io.Writer // 打印目的地, 默认为 `os.Stderr`
}
```

### 注册标签

```go
var CommandLine = NewFlagSet(os.Args[0], ExitOnError)
```

`CommandLine` 是 package 级别的, 只会运行一次 `NewFlagSet()`, 得到的结果指向一个未完全初始化的 `FlagSet` (map 和 slice 现在还是空).

```go
func (f *FlagSet) Var(value Value, name string, usage string) {
	// Remember the default value as a string; it won't change.
	flag := &Flag{name, usage, value, value.String()}
	_, alreadythere := f.formal[name]
	if alreadythere {
		var msg string
		if f.name == "" {
			msg = fmt.Sprintf("flag redefined: %s", name)
		} else {
			msg = fmt.Sprintf("%s flag redefined: %s", f.name, name)
		}
		fmt.Fprintln(f.Output(), msg)
		panic(msg) // Happens only if flags are declared with identical names
	}
	if f.formal == nil {
		f.formal = make(map[string]*Flag)
	}
	f.formal[name] = flag
}
```

`Var` 是注册的核心方法. 经过几层代理, 默认值已经设置到 `value` 中, `name` 是标签名, `usage` 是帮助信息, 他们被包装到 `Flag` 结构体内, 保存到 `FlagSet` 的 `formal` 里.

`formal` 是个 map, 标签名作为 map 的键, 不允许重复注册, 否则抛 panic .

### 解析标签

```go
func (f *FlagSet) parseOne() (bool, error) {
	//...
	flag.Value.Set(value)
	//...
}
```

`Parse()` 方法内循环调用 `parseOne()`, 每成功解析完一对参数, `FlagSet` 的 `args` 就被截断一截, 直到解析结束, `args` 也被截空了.

一旦解析出错, 就按照 `FlagSet` 中 `errorHandling` 的规则处理, 默认是 `exit(2)` 退出程序.

成功解析的参数经由 	`flag.Value.Set(value)` 被设置为对应的值.
