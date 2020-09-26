---
layout: post
title:  Golang strings package 评注
date:    2020-09-26
Author: CBD
tags: [Golang, strings]
---
## Join

拼接字符串最直观的想法就是用 `str = str + sep`, 但是这样会频繁创建新的字符串.
使用 builder 是更高效的方法: 先计算结果总长度, 然后顺序写入 builder.

```go
// Join concatenates the elements of its first argument to create a single string. The separator
// string sep is placed between elements in the resulting string.
func Join(elems []string, sep string) string {
	// 空 slice 或者只有一个元素的 slice, 不需要拼接.
	switch len(elems) {
	case 0:
		return ""
	case 1:
		return elems[0]
	}
	// n: 结果总长度.
	n := len(sep) * (len(elems) - 1) // 所有 sep 的长度总和
	// 累加每个元素的长度
	for i := 0; i < len(elems); i++ {
		n += len(elems[i])
	}

	var b Builder
	b.Grow(n) // 已经确定结果的长度为 n, 开辟长度为 n 的 builder.
	b.WriteString(elems[0]) // 先写入第一个元素, 因为它之前不需要写 sep
	for _, s := range elems[1:] {
		b.WriteString(sep)
		b.WriteString(s)
	}
	return b.String()
}
```
