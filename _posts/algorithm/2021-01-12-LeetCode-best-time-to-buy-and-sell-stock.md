---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock/)

> Golang

```golang

func maxProfit(prices []int) int {
	if len(prices) == 0 {
		return 0
	}
	dp := make([]int, len(prices))
	historyMin := prices[0]
	for i := 1; i < len(prices); i++ {
		dp[i] = max(dp[i-1], prices[i]-historyMin) // 昨天的最大收益 or 今天卖出
		historyMin = min(historyMin, prices[i])
	}
	return dp[len(prices)-1]
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}
```

被标记为简单, 主要是因为限制了交易次数只为 1.

`min` 标记历史上最低的价格, 把当天的价格跟历史最低做差为当前最大的利润.

虽然可以使用 DP table, 但是题目过于简单, 直接使用 min 和 result 记录.

> Golang

```golang
func maxProfit(prices []int) int {
	if len(prices) == 0 {
		return 0
  }
  
	min := prices[0]
	result := 0
	for i := 1; i < len(prices); i++ {
		current := prices[i]

		money := current - min
		if money > result {
			result = money
		}

		if current < min {
			min = current
		}
	}
	return result
}

```
