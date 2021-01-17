---
layout: post
title:  LeetCode two-sum
date:   2021-01-13
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/two-sum/](https://leetcode-cn.com/problems/two-sum/)

> Golang

```golang

func twoSum(nums []int, target int) []int {
	dict := make(map[int]int) // num => idx

	for i := 0; i < len(nums); i++ {
		curr := nums[i]
		other := target - curr

		if idx, ok := dict[other]; ok {
			return []int{i, idx}
		} else {
			dict[curr] = i
		}
	}
	return nil
}
```
