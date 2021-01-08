---
layout: post
title:  LeetCode coin-change
date:   2021-01-08
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/coin-change/](https://leetcode-cn.com/problems/coin-change/)

> Golang

```golang

const UNVISITED = 0

func coinChange(coins []int, amount int) int {
	if amount == 0 {
		return 0
	}
	
	dpLen := amount + 1
	dp := make([]int, dpLen)
	for _, coin := range coins {
		if coin >= dpLen {
			continue
		}
		dp[coin] = 1
	}

	for i := 0; i <= amount; i++ {
		if dp[i] == UNVISITED {
			continue
		}
		for _, coin := range coins {
			value := i + coin
			if value > amount {
				continue // 防止越界
			}
			if dp[value] == UNVISITED {
				dp[value] = dp[i] + 1
			} else {
				dp[value] = min(dp[value], dp[i]+1)
			}
		}
	}

	fmt.Println(dp)
	if dp[amount] == UNVISITED {
		return -1
	}
	return dp[amount]
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
