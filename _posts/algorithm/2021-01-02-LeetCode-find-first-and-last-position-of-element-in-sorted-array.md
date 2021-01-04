---
layout: post
title:  LeetCode find-first-and-last-position-of-element-in-sorted-array
date:   2021-01-02
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array/)

先用朴素的二分查找找到一个值, 然后从这个值开始向两边查找相同的值.

> Golang

```golang
func searchRange(nums []int, target int) []int {
	targetIdx := -1
	leftIdx := 0
	rightIdx := len(nums)
	for leftIdx < rightIdx {
		middelIdx := (rightIdx + leftIdx) / 2
		if target == nums[middelIdx] {
			targetIdx = middelIdx
			break
		}

		if target < nums[middelIdx] {
			rightIdx = middelIdx
		} else {
			leftIdx = middelIdx + 1
		}
	}

	result := []int{-1, -1}
	if targetIdx == -1 {
		return result
	}

	for i := targetIdx; i >= 0; i-- {
		if nums[i] != target {
			break
		}
		result[0] = i
	}
	for i := targetIdx; i < len(nums); i++ {
		if nums[i] != target {
			break
		}
		result[1] = i
	}
	return result
}
```
