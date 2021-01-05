---
layout: post
title:  LeetCode find-minimum-in-rotated-sorted-array
date:   2021-01-03
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/](https://leetcode-cn.com/problems/find-minimum-in-rotated-sorted-array/)

![find-minimum-in-rotated-sorted-array](/images/find-minimum-in-rotated-sorted-array.png)

> Golang

```golang
func findMin(nums []int) int {
	left := 0
	right := len(nums) // [left,right)

	for left <= right {
		if nums[left] <= nums[right-1] { // 整段连续, 最小值一定在最左
			return nums[left]
		}

		mid := (right + left) / 2
		if nums[left] <= nums[mid-1] { // 左侧连续, 最小值一定在右侧
			left = mid
			continue
		} else { // 左侧不连续, 最小值一定在这里
			right = mid
		}
	}

	return nums[0]
}
```
