---
layout: post
title:  LeetCode minimum-path-sum
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/minimum-path-sum/](https://leetcode-cn.com/problems/minimum-path-sum/)

测试数据可能是矩形, 不一定总是正方形.

基本DP, 自底向上, 求最后方格的极值:

* 第一行的结果确定, 直接赋值;
* 最左边的结果确定, 直接赋值;
* 当前方格的结果 = 当前方格的值 + min(左边方格结果, 上方方格结果)

没有重复子问题, 不需要剪枝.

> Golang

```golang
func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

func minPathSum(grid [][]int) int {
	xCount := len(grid[0])
	yCount := len(grid)
	dp := make([][]int, yCount)

	// 第一行, 只有一种可能
	dp[0] = make([]int, xCount)
	dp[0][0] = grid[0][0]
	for n := 1; n < xCount; n++ {
		dp[0][n] = grid[0][n] + dp[0][n-1]
	}

	for line := 1; line < yCount; line++ {
		dp[line] = make([]int, xCount)
		dp[line][0] = grid[line][0] + dp[line-1][0] // 左边第一个, 只有一种可能
		for n := 1; n < xCount; n++ {
			dp[line][n] = grid[line][n] + min(dp[line-1][n], dp[line][n-1])
		}
	}

	return dp[yCount-1][xCount-1]
}
```
