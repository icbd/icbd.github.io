---
layout: post
title:  LeetCode remove-covered-intervals
date:   2021-01-13
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/remove-covered-intervals/](https://leetcode-cn.com/problems/remove-covered-intervals/)

![remove-covered-intervals](/images/remove-covered-intervals.png)

> Golang

```golang

// {a, b}
func removeCoveredIntervals(intervals [][]int) int {
	sort.Slice(intervals, func(i, j int) bool {
		curr := intervals[i]
		next := intervals[j]

		if curr[0] == next[0] {
			return curr[1] > next[1] // b, DESC
		} else {
			return curr[0] < next[0] // a, ASC
		}
	})

	rightMax := intervals[0][1]
	count := 0 // 要删掉的
	for i := 1; i < len(intervals); i++ {
    // 左端点相等的, 右边一定是递减的, 可以直接删掉
		if intervals[i][0] == intervals[i-1][0] {
			count++
			continue
		}

    // 左端点不等的, 看右端点是否能覆盖
		if intervals[i][1] > rightMax {
			rightMax = intervals[i][1]
		} else {
			count++
		}
	}
	return len(intervals) - count
}
```
