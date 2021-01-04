---
layout: post
title:  LeetCode search-insert-position
date:   2021-01-02
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/search-insert-position/](https://leetcode-cn.com/problems/search-insert-position/)

> Golang

```golang
func searchInsert(nums []int, target int) int {
	// [leftIdx, rightIdx)
	leftIdx := 0
	rightIdx := len(nums)
	for leftIdx < rightIdx {
		middelIdx := (rightIdx + leftIdx) / 2
		if target == nums[middelIdx] {
			return middelIdx
		}

		if target < nums[middelIdx] {
			rightIdx = middelIdx
		} else {
			leftIdx = middelIdx + 1
		}
	}

	return leftIdx
}
```
