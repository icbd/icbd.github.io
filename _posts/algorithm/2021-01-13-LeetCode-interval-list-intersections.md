---
layout: post
title:  LeetCode interval-list-intersections
date:   2021-01-13
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/interval-list-intersections/](https://leetcode-cn.com/problems/interval-list-intersections/)

> Golang

```golang

func intervalIntersection(firstList [][]int, secondList [][]int) [][]int {
	result := make([][]int, 0)

	i, j := 0, 0
	for i < len(firstList) && j < len(secondList) {
		f := firstList[i]
		s := secondList[j]

		// 有交集, 可以只交一个
		if f[0] <= s[1] && f[1] >= s[0] {
			m := []int{max(f[0], s[0]), min(f[1], s[1])}
			result = append(result, m)
			if m[1] == f[1] {
				i++
			} else { //m[1] == s[1]
				j++
			}
			continue
		} else {
			// 没交集了
			if f[0] > s[1] {
				j++
			} else {
				i++
			}
		}
	}

	return result
}

func max(a, b int) int {
	if a > b {
		return a
	}
	return b
}
func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
