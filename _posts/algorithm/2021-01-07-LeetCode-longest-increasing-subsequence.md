---
layout: post
title:  LeetCode longest-increasing-subsequence
date:   2021-01-07
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/longest-increasing-subsequence/](https://leetcode-cn.com/problems/longest-increasing-subsequence/)

> Golang

```golang
func lengthOfLIS(nums []int) int {
	// Val 含义: 以 nums 下标对应的数字为起点, 这个数字和它之后的最大自增数列的数目
	result := make([]int, len(nums))

	for curr := len(nums) - 1; curr >= 0; curr-- {
		// 找出后续节点中, 可入队的, LIS 最大的那个
		postLIS := 0
		for n := curr + 1; n < len(nums); n++ {
			if nums[n] > nums[curr] && result[n] > postLIS {
				postLIS = result[n] // 最大的
			}
		}
		result[curr] = 1 + postLIS // 加上自己
	}
	
	//fmt.Println(result)
	sort.Ints(result)
	return result[len(result)-1]
}
```
