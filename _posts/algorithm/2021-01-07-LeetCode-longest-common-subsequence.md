---
layout: post
title:  LeetCode longest-common-subsequence
date:   2021-01-07
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/longest-common-subsequence/](https://leetcode-cn.com/problems/longest-common-subsequence/)

在原始 text 前补充一个占位符, 比操作 `len - 1` 方便的多.

> Golang

```golang

func longestCommonSubsequence(text1 string, text2 string) int {
	// 补充占位符
	text1 = "1" + text1
	text2 = "2" + text2

	xCount := len(text1)
	yCount := len(text2)

	dp := make([][]int, yCount)
	dp[0] = make([]int, xCount)

	for y := 1; y < yCount; y++ {
		dp[y] = make([]int, xCount)
		for x := 1; x < xCount; x++ {
			if text1[x] == text2[y] {
				dp[y][x] = dp[y-1][x-1] + 1
			} else {
				dp[y][x] = max(dp[y-1][x], dp[y][x-1])
			}
		}
	}
	return dp[yCount-1][xCount-1]
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

```
