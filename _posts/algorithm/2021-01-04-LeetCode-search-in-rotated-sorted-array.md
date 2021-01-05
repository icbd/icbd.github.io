---
layout: post
title:  LeetCode search-in-rotated-sorted-array
date:   2021-01-04
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/search-in-rotated-sorted-array/](https://leetcode-cn.com/problems/search-in-rotated-sorted-array/)

> Golang

```golang

func main() {
	data := []int{5, 1, 2, 3, 4}
	res := search(data, 4)
	fmt.Println(res)
}

func search(nums []int, target int) int {
	left := 0
	right := len(nums)
	for left < right {
		mid := (left + right) / 2
		if target == nums[mid] {
			return mid
		}

		if nums[left] <= nums[right-1] { // 单调递增, 经典二分
			if target > nums[mid] {
				left = mid + 1
			} else {
				right = mid
			}
			continue
		}

		if nums[left] <= nums[mid-1] { // 左侧单调递增
			if target >= nums[left] && target <= nums[mid-1] {
				right = mid
				continue
			} else {
				left = mid + 1
				continue
			}
		} else { // 右侧单调递增
			if target >= nums[mid+1] && target <= nums[right-1] {
				left = mid + 1
				continue
			} else {
				right = mid
				continue
			}
		}
	}

	return -1
}

```
