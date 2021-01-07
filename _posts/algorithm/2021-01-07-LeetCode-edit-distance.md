---
layout: post
title:  LeetCode edit-distance
date:   2021-01-07
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/edit-distance/](https://leetcode-cn.com/problems/edit-distance/)

> Golang

```golang

func minDistance(word1 string, word2 string) int {
	word1 = "1" + word1
	word2 = "2" + word2
	xCount := len(word1)
	yCount := len(word2)

	dp := make([][]int, yCount)
	dp[0] = make([]int, xCount)
	for i := 0; i < xCount; i++ {
		dp[0][i] = i
	}

	for y := 1; y < yCount; y++ {
		dp[y] = make([]int, xCount)
		dp[y][0] = y
		for x := 1; x < xCount; x++ {
			if word1[x] == word2[y] {
				dp[y][x] = dp[y-1][x-1]
			} else {
				replaceOne := dp[y-1][x-1] + 1
				plusOne := dp[y][x-1] + 1
				deleteOne := dp[y-1][x] + 1
				dp[y][x] = min(replaceOne, plusOne, deleteOne)
			}
		}
	}

	return dp[yCount-1][xCount-1]
}

func min(a ...int) int {
	sort.Ints(a)
	return a[0]
}
```
