---
layout: post
title:  LeetCode find-minimum-in-rotated-sorted-array
date:   2021-01-03
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array-ii/)

![find-minimum-in-rotated-sorted-array-ii](/images/find-minimum-in-rotated-sorted-array-ii.png)

> Golang

```golang

func findMin(nums []int) int {
	left := 0
	right := len(nums) // [left,right)

	for left < right {
		if nums[left] < nums[right-1] { // 整段连续, 最小值一定在最左
			return nums[left]
		}

		if right-left == 2 {
			return min(nums[left], nums[right-1])
		}

		if nums[left] == nums[right-1] { // 两头相等时中间可能有凹陷
			right -= 1
			continue
		}

		mid := (right + left) / 2
		if nums[left] <= nums[mid-1] { // 左侧连续, 最小值一定在右侧
			left = mid
			continue
		} else { // 左侧不连续, 最小值一定在这里
			right = mid
		}
	}

	return nums[left]
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
