---
layout: post
title:  LeetCode house-robber-ii
date:   2021-01-12
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/house-robber-ii/](https://leetcode-cn.com/problems/house-robber-ii/)

在 `house-robber` 的基础上, 数组变成了环, 又因为必须间隔一个:

* 不选第一个也不选最后一个
* 选第一个, 不选最后一个
* 不选第一个, 选最后一个

因为屋里的钱总是正的, 第一种情况没可能比后两种大, 所以分别计算后两种可能就好了.

特别注意 `[]` `[1]` `[1,2]` `[1,2,3]` 这样的小规模 case, 也可能不做兼容处理, 直接在主函数里判断.

> Golang

```golang

const (
	Steal = iota
	Jump
)

func rob(nums []int) int {
	if len(nums) == 0 {
		return 0
	}
	if len(nums) == 1 {
		return nums[0]
	}
	
	return max(robOne(nums[1:]), robOne(nums[:len(nums)-1]))
}

func robOne(nums []int) int {
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
