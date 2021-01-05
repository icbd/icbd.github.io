---
layout: post
title:  LeetCode triangle
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/triangle/](https://leetcode-cn.com/problems/triangle/)

> Golang

```golang

// 2
// 3,4
// 6,5,7
// 4,1,8,3

func minimumTotal(triangle [][]int) int {
	if triangle == nil || len(triangle) == 0 {
		return 0
	}
	nCount := len(triangle)
	collection := make([][]int, nCount)
  collection[0] = []int{triangle[0][0]}  // 第0行
  
  // 第line行总共有 line + 1 个元素
	for line := 1; line < nCount; line++ {
		collection[line] = make([]int, nCount)
		n := 0                                                          // 该行的第几个
		collection[line][n] = collection[line-1][n] + triangle[line][n] // 该行首个
		for n = 1; n < line; n++ {
			collection[line][n] = min(collection[line-1][n], collection[line-1][n-1]) + triangle[line][n]
		}
		collection[line][n] = collection[line-1][n-1] + triangle[line][n] // 该行最后一个
	}

	// 从collection的最后一行找最小值
	min := math.MaxInt32
	for i := 0; i < nCount; i++ {
		v := collection[nCount-1][i]
		if v < min {
			min = v
		}
	}
	return min
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
