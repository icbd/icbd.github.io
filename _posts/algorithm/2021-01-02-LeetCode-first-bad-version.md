---
layout: post
title:  LeetCode first-bad-version
date:   2021-01-02
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/first-bad-version/](https://leetcode-cn.com/problems/first-bad-version/)

二分查找的变形, 没有同等值的停止条件, 最终靠 first > last 来停止, first 为上一个结果为错的序号.

> Golang

```golang
func firstBadVersion(n int) int {
	first := 1
	last := n
	for first <= last {
		m := (first + last) / 2
		if !isBadVersion(m) { // √
			first = m + 1
		} else { // X
			last = m - 1
		}
	}
	//fmt.Println(first, last)
	return first
}
```
