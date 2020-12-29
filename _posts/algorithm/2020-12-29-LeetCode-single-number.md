---
layout: post
title:  LeetCode single-number
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/single-number/](https://leetcode-cn.com/problems/single-number/)

位运算法则:

```text
a^0 == a
a^a == 0
a^b^b == a^0 == a
a^b == b^a
```

令 `target` 为 0, `target^a^a^b^b` 还是 0, `target^x` 为 x, 那么只出现一次的数最后被印在 target 上了.

> Golang

```golang
func singleNumber(nums []int) int {
	target := 0
	for i := 0; i < len(nums); i++ {
		target = target ^ nums[i]
	}
	return target
}
```
