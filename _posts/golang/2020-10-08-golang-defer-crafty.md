---
layout: post
title:  Golang defer 妙用
date:   2020-10-08
Author: CBD
tags: [Golang]
---

defer 会解析函数, 并在函数退出前执行. defer 多用在关闭资源的时候, 也可以利用这个特点来实现特殊的语义.

## 环绕方法

```go
func main() {
	controller()
}

func callbacks(before func(), after func()) func() {
	before()

	return after
}

func controller() {
	defer callbacks(before, after)()

	fmt.Println("main")
	time.Sleep(time.Duration(2) * time.Second)
}

func before() {
	fmt.Println("Before>>")
}
func after() {
	fmt.Println("After>>")
}
```

> output:

```text
Before>>
main
After>>
```

在执行到 defer 的时候, 会执行 `callbacks` 内语句, 在函数退出时执行 `callback` 返回的函数, 从而实现了环绕方法的效果.

这里特别要注意, `defer callbacks(before, after)()` 最后有一对括号, 如果没有括号结果为:

```text
main
Before>>
```

## 修改返回值

```go
func main() {
	fmt.Println(controller())
}

func aspect(params ...interface{}) {
	fmt.Println("4")
	if len(params) == 1 {
		r := params[0].(*int)
		*r *= 100
	}
}

func controller() (result int) {
	fmt.Println("1")
	defer aspect(&result)
	fmt.Println("2")

	result = 0
	for i := 0; i < 10; i++ {
		result += i
	}
	fmt.Println("3")
	return result
}

```

> output:

```text
1
2
3
4
4500
```

defer 可以修改具名返回变量, 务必使用指针传递.
