---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock-with-cooldown
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

画出状态机流转图.

> Golang

```golang

const (
	EmptyPositionFreeze = iota // 空仓, 冻结日
	EmptyPosition              // 空仓, 自由日
	FullPosition               // 满仓
)

func maxProfit(prices []int) int {
	if len(prices) == 0 {
		return 0
	}

	dp := make([][]int, len(prices)+1) // [第几天][仓位] => balance
	for i := 0; i < len(dp); i++ {
		dp[i] = []int{math.MinInt32, math.MinInt32, math.MinInt32}
	}

	dp[0][EmptyPositionFreeze] = 0
	dp[0][EmptyPosition] = 0
	dp[0][FullPosition] = -prices[0]
	for day := 1; day <= len(prices); day++ {
		todayPrice := prices[day-1]

		// 冻结日, 前一天肯定是满仓, 今天卖出
		dp[day][EmptyPositionFreeze] = dp[day-1][FullPosition] + todayPrice

		// 自由日
		dp[day][EmptyPosition] = max(dp[day-1][EmptyPosition], dp[day-1][EmptyPositionFreeze])

		// 满仓日
		dp[day][FullPosition] = max(dp[day-1][FullPosition], dp[day-1][EmptyPosition]-todayPrice)
	}

	return max(dp[len(prices)][EmptyPosition], dp[len(prices)][EmptyPositionFreeze])
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
```
