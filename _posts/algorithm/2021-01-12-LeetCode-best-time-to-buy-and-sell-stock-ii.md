---
layout: post
title:  LeetCode best-time-to-buy-and-sell-stock-ii
date:   2021-01-12
Author: CBD
tags:   [LeetCode, Stock, DP]
---

[https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii/)

一天中总共有三种策略: 买入, 卖出, 不操作.

由于没有仓位控制, 只能满仓和空仓, 并且当满仓时才能卖出, 当空仓时才能买入.

`dp := make([][]int, len(prices)+1)` 表示 `[第几天][仓位] => balance`

第一天的状态对应 `dp[1]`.

初始化 `dp[0]`:

```go
	dp[0][EmptyPosition] = 0
	dp[0][FullPosition] = -prices[0]
```

这里我们可以想象所有的操作都是融资买入, 这样就很容易理解初始资金了.

## 第一天

* 如果我们第一天不动, 那么 balance 还是 0;
* 如果我们第一天买入, balance 负债增加, 结果为 `-prices[0]`;
* 如果我们第一天卖出, 那么前一天应该已经提前融资买入了, 也就是 `-prices[0]`;

到这里就有一个问题, 想今天买入, 那么必须保证当前是满仓的状态. 如果我们使用 `买入/卖出/不动` 这样的动作, 我们很难保证当前出于正确的状态, 前一天不动, 大前天呢, 大大前天呢~

所以应该使用`满仓`/`空仓`这样确定的 **状态** 作为 dp 的标记, 具体实现如下:

> Golang

```golang

const (
	EmptyPosition = iota
	FullPosition
)

func maxProfit(prices []int) int {
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
			dp[day-1][FullPosition]+todayPrice, // sell
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

由于这里的 DP Table 只跟他前一个位置的值相关, 所以我们可以简化 DP Table 到两个变量:

> Golang

```golang

func maxProfit(prices []int) int {
	emptyPosition := 0         // 上一轮为空仓时, balance 的值
	fullPosition := -prices[0] // 上一轮为满仓时, balance 的值
	for day := 1; day <= len(prices); day++ {
		todayPrice := prices[day-1]

		fullPosition = max(
			fullPosition,
			emptyPosition-todayPrice, // buy
		)

		emptyPosition = max(
			emptyPosition,
			fullPosition+todayPrice, // sell
		)
	}

	return emptyPosition
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}

```

型如这样的 DP 最终可以化简为贪心算法, `sum(item - item')`.
