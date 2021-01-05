---
layout: post
title:  LeetCode unique-paths
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/unique-paths/](https://leetcode-cn.com/problems/unique-paths/)

|Y\X|0|1|2|3|4|5|6|
|---|---|---|---|---|---|---|---|
|0|1|1|1|1|1|1|1|
|1|1|2|3|4|5|6|7|
|2|1|3|6|10|15|21|28|

* 第一横行和第一列总是 1;
* `dp[line][idx] = dp[line][idx-1] + dp[line-1][idx]`

> Golang

```golang

func uniquePaths(m int, n int) int {
	dp := make([][]int, n)
	for i := 0; i < n; i++ {
		dp[i] = make([]int, m)
	}

	// 直接初始化第一行, 答案固定为 1
	for i := 0; i < m; i++ {
		dp[0][i] = 1
	}

	for line := 1; line < n; line++ {
		dp[line][0] = 1 // 最左边的确定为 1
		for idx := 1; idx < m; idx++ {
			dp[line][idx] = dp[line][idx-1] + dp[line-1][idx]
		}
	}

	//fmt.Println(dp)
	return dp[n-1][m-1]
}

```
