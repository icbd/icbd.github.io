---
layout: post
title:  LeetCode search-in-rotated-sorted-array-ii
date:   2021-01-04
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii/)

> Golang

```golang

func search(nums []int, target int) bool {
	left := 0
	right := len(nums)

	for left < right {
		mid := (left + right) / 2
		if target == nums[mid] {
			return true
		}

		if nums[left] < nums[right-1] { // 递增, 经典二分
			if target > nums[mid] {
				left = mid + 1
			} else {
				right = mid
			}
			continue
		}

		if nums[left] == nums[right-1] { // 中间可能存在凹陷
			right -= 1
			continue
		} else if nums[left] <= nums[mid-1] { // 左侧递增
			if target >= nums[left] && target <= nums[mid-1] {
				right = mid
			} else {
				left = mid + 1
			}
		} else { // 右侧递增
			if target >= nums[mid+1] && target <= nums[right-1] {
				left = mid + 1
			} else {
				right = mid
			}
		}
	}
	return false
}

```
