---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock-with-transaction-fee
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/)

买入卖出收取一笔手续费.

每花一笔手续费, `balance -= fee`.

> Golang

```golang

const (
	EmptyPosition = iota
	FullPosition
)

func maxProfit(prices []int, fee int) int {
	dp := make([][]int, len(prices)+1) // [第几天][仓位] => balance
	for i := 0; i < len(dp); i++ {
		dp[i] = []int{math.MinInt32, math.MinInt32}
	}
	dp[0][EmptyPosition] = 0
	dp[0][FullPosition] = -prices[0]
	for day := 1; day <= len(prices); day++ {
		todayPrice := prices[day-1]

		dp[day][FullPosition] = max(
			dp[day-1][FullPosition],
			dp[day-1][EmptyPosition]-todayPrice, // buy
		)

		dp[day][EmptyPosition] = max(
			dp[day-1][EmptyPosition],
			dp[day-1][FullPosition]+todayPrice-fee, // sell
		)
	}

	return dp[len(prices)][EmptyPosition]
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

```
