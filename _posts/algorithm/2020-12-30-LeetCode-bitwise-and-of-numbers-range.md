---
layout: post
title:  LeetCode bitwise-and-of-numbers-range
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range/)

当 m == n, 他们与操作结果为自身.

当 m < n 且他们最高位的 1 位置不一样时, 比如:

```text
m 0101
  1000
n 1010
```

那么 m 和 n 之间一定存在类似 `1000` 的值, 使得 n 最高位后面的值, 经过与操作后结果都为 0 .

当 m < n 且他们最高位的 1 位置相同时, 比如:

```text
m 1011
  1011
  1100
n 1101
```

那么他们之间一定存在类似 `1011` `1100` 的值, 使得相同高位后面的值, 经过与操作后结果都为 0 .

综合以上是三种情况, 他们都可以转化为一条统一的规则: m n 从左高位开始, 有多少位相同的位, 结果即保留相同的左高位, 其右都置为 0.

> Golang

```golang
func rangeBitwiseAnd(m int, n int) int {
	if m == n {
		return m
	}

	bitLen := 8 * int(unsafe.Sizeof(m))
	nLeftZero := 0
	for i := bitLen - 1; (1<<i)&n == 0; i-- {
		nLeftZero++
	}

	res := 0
	for i := bitLen - 1 - nLeftZero; i > 0; i-- {
		base := 1 << i
		if m&base != n&base {
			break
		}
		res += n&base
	}
	return res
}
```

从右边操作会简单很多:

左移几次能得到相同的 m n, 就原样右移几次恢复 n.

> Golang

```golang
func rangeBitwiseAnd(m int, n int) int {
	count := 0
	for m != n {
		m >>= 1
		n >>= 1
		count++
	}
	return n << count
}
```
