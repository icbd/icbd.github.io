---
layout: post
title:  LeetCode fibonacci-number
date:   2021-01-08
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/fibonacci-number/](https://leetcode-cn.com/problems/fibonacci-number/)

> Golang

```golang
// 0 1 1 2 3
func fib(n int) int {
	dp := make([]int, n+2)
	dp[1] = 1
	for i := 2; i <= n; i++ {
		dp[i] = dp[i-1] + dp[i-2]
	}
	return dp[n]
}
```
