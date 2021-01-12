---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock-iv
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iv/)

从 `best-time-to-buy-and-sell-stock-iii` 的硬编码改造而来.

> Golang

```golang

const EmptyPosition = 0 //从来没交易过, 结果一定为零. 之后奇数为买入, 偶数为卖出
func maxProfit(k int, prices []int) int {
	positionCount := k*2 + 1
	dayCount := len(prices)
	dp := make([][]int, dayCount+1) // [第几天][仓位状态] => balance
	for i := 0; i < len(dp); i++ {
		dp[i] = make([]int, positionCount)
		for j := 0; j < positionCount; j++ {
			dp[i][j] = math.MinInt32
		}
	}

	var buySell int
	dp[0][EmptyPosition] = 0
	for day := 1; day <= dayCount; day++ {
		todayPrice := prices[day-1]
		dp[day][EmptyPosition] = 0
		for positionIdx := 1; positionIdx < positionCount; positionIdx++ {
			if positionIdx%2 == 1 { // 奇数位, 买入, balance 记为负
				buySell = -1
			} else {
				buySell = +1
			}
			dp[day][positionIdx] = max(dp[day-1][positionIdx], dp[day-1][positionIdx-1]+buySell*todayPrice)
		}
	}

	return max(dp[dayCount]...)
}

func max(list ...int) int {
	sort.Ints(list)
	return list[len(list)-1]
}

```
