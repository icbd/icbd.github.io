---
layout: post
title:  LeetCode climbing-stairs
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/climbing-stairs/](https://leetcode-cn.com/problems/climbing-stairs/)

* 0 if n <= 0;
* f(n) = f(n - 1) + f(n - 2)

类似斐波那契数列, 一定要处理重复子问题, 否则会超时.

> Golang

```golang

func climbStairs(n int) int {
	if n < 3 {
		return n
	}
	res := make([]int, n+1)
	res[0] = 0
	res[1] = 1
	res[2] = 2
	for i := 3; i <= n; i++ {
		res[i] = res[i-2] + res[i-1]
	}
	return res[n]
}
```
