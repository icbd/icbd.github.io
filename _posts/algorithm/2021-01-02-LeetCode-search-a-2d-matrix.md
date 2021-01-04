---
layout: post
title:  LeetCode search-a-2d-matrix
date:   2021-01-02
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/search-a-2d-matrix/](https://leetcode-cn.com/problems/search-a-2d-matrix/)

> Golang

```golang
func searchMatrix(matrix [][]int, target int) bool {
	top := 0
	bottom := len(matrix)
	for top < bottom {
		middleLine := (top + bottom) / 2
		if target >= matrix[middleLine][0] && target <= matrix[middleLine][len(matrix[0])-1] {
			left := 0
			right := len(matrix[0])
			for left <= right {
				middleIdx := (left + right) / 2
				if target == matrix[middleLine][middleIdx] {
					return true
				}
				if target > matrix[middleLine][middleIdx] {
					left = middleIdx + 1
				} else {
					right = middleIdx - 1
				}
			}
			return false // 在该行没有找到, 后面也不会找到了
		}
		if target < matrix[middleLine][0] {
			bottom = middleLine
		} else {
			top = middleLine + 1
		}
	}
	return false
}
```
