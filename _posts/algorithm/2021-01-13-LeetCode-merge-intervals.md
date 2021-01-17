---
layout: post
title:  LeetCode merge-intervals
date:   2021-01-13
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/merge-intervals/](https://leetcode-cn.com/problems/merge-intervals/)

> Golang

```golang

// {a, b}
func merge(intervals [][]int) [][]int {
	result := make([][]int, 0)

	sort.Slice(intervals, func(i, j int) bool {
		curr := intervals[i]
		next := intervals[j]

		if curr[0] == next[0] {
			return curr[1] > next[1] // b, DESC
		} else {
			return curr[0] < next[0] // a, ASC
		}
	})

	left := intervals[0][0]
	right := intervals[0][1]
	for i := 1; i < len(intervals); i++ {
		if intervals[i][0] == intervals[i-1][0] {
			continue
		}

		if intervals[i][0] > right {
			result = append(result, []int{left, right})
			left = intervals[i][0]
		}

		if intervals[i][1] > right {
			right = intervals[i][1]
		}
	}
	result = append(result, []int{left, right})
	return result
}

```
