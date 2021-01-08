---
layout: post
title:  LeetCode permutations 全排列
date:   2021-01-08
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/permutations/](https://leetcode-cn.com/problems/permutations/)

> Golang

```golang

var numsMap map[int]bool // 是否用过这个 num
var result [][]int

func permute(nums []int) [][]int {
	result = make([][]int, 0)
	numsMap = make(map[int]bool)

	for i := 0; i < len(nums); i++ {
		numsMap[nums[i]] = false
	}
	path := make([]int, 0)
	helper(path)

	return result
}

func helper(path []int) {
	if len(path) == len(numsMap) {
		temp := make([]int, len(path))
		copy(temp, path)
		result = append(result, temp)
		return
	}

	for num, visited := range numsMap {
		if !visited {
			path = append(path, num) // push to path
			numsMap[num] = true
			helper(path)
			path = path[:len(path)-1] //pop from path
			numsMap[num] = false
		}
	}
}
```
