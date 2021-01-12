---
layout: post
title:  LeetCode house-robber
date:   2021-01-12
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/house-robber/](https://leetcode-cn.com/problems/house-robber/)

只求极值不求过程, 经典 DP. 梳理状态转移方程.

> Golang

```golang

const (
	Steal = iota
	Jump
)

func rob(nums []int) int {
	homeCount := len(nums)
	dp := make([][]int, homeCount+1) // [homeIdx][steal or jump] => max value
	for i := 0; i <= homeCount; i++ {
		dp[i] = make([]int, 2)
	}

	for i := 1; i <= homeCount; i++ {
		currentValue := nums[i-1]
		dp[i][Steal] = dp[i-1][Jump] + currentValue
		dp[i][Jump] = max(dp[i-1][Jump], dp[i-1][Steal])
	}
	return max(dp[homeCount][Steal], dp[homeCount][Jump])
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

```
