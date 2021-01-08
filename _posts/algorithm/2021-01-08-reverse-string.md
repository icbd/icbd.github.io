---
layout: post
title:  LeetCode reverse-string
date:   2021-01-08
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/reverse-string/](https://leetcode-cn.com/problems/reverse-string/)

> Golang

```golang
func reverseString(s []byte) {
	mid := len(s) / 2
	for i := 0; i < mid; i++ {
		s[i], s[len(s)-i-1] = s[len(s)-i-1], s[i]
	}
}
```
