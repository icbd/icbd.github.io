---
layout: post
title:  LeetCode counting-bits
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/counting-bits/](https://leetcode-cn.com/problems/counting-bits/)

依次计算汉明重量, 虽然可以 AC , 但是并不满足时间复杂度要求.

> Golang

```golang
func countBits(num int) []int {
	res := make([]int, num+1)
	for i := 0; i <= num; i++ {
		res[i] = hammingWeight(i)
	}
	return res
}

func hammingWeight(num int) int {
	count := 0
	for num != 0 {
		if num&1 == 1 {
			count++
		}
		num >>= 1
	}
	return count
}

```

## 迭代法

```text
2: 0010
4: 0100

2: 0010
4: 0100
```

可见偶数的 `hammingWeight` 等于其二分之一 值的 `hammingWeight`, 因为他们的区别是最右边的 `0`.
奇数会比前一个偶数多一个 `1`, 只把偶数最后的 0 改为 1.

> Golang

```golang
func countBits(num int) []int {
	res := make([]int, num+1)
	res[0] = 0
	for n := 0; n <= num; n++ {
		if n%2 == 0 {
			res[n] = res[n/2]
		} else {
			res[n] = res[n-1] + 1
		}
	}
	return res
}
```
