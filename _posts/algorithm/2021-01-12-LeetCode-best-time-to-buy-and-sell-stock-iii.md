---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock-iii
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii/)

限制操作两次, 直接导致了空仓和满仓的状态增加. 还好总数不多, 可以直接硬编码.

> Golang

```golang

const (
	EmptyPosition  = iota // 从来没交易过, 结果一定为零.
	FullPosition1         // 第一次买入
	EmptyPosition1        // 第一次卖出
	FullPosition2         // 第二次买入
	EmptyPosition2        // 第二次卖出
)

func maxProfit(prices []int) int {
	dp := make([][]int, len(prices)+1) // [第几天][仓位] => balance
	for i := 0; i < len(dp); i++ {
		dp[i] = []int{math.MinInt32, math.MinInt32, math.MinInt32, math.MinInt32, math.MinInt32}
	}

	dp[0][EmptyPosition] = 0
	for day := 1; day <= len(prices); day++ {
		todayPrice := prices[day-1]
		dp[day][EmptyPosition] = 0
		dp[day][FullPosition1] = max(dp[day-1][FullPosition1], dp[day-1][EmptyPosition]-todayPrice)
		dp[day][EmptyPosition1] = max(dp[day-1][EmptyPosition1], dp[day-1][FullPosition1]+todayPrice)
		dp[day][FullPosition2] = max(dp[day-1][FullPosition2], dp[day-1][EmptyPosition1]-todayPrice)
		dp[day][EmptyPosition2] = max(dp[day-1][EmptyPosition2], dp[day-1][FullPosition2]+todayPrice)
	}

	return max(dp[len(prices)][EmptyPosition], max(dp[len(prices)][EmptyPosition1], dp[len(prices)][EmptyPosition2]))
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

```
