---
layout: post
title:  LeetCode permutation-in-string
date:   2021-01-11
Author: CBD
tags:   [LeetCode, SlideWindow]
---

[https://leetcode-cn.com/problems/permutation-in-string/](https://leetcode-cn.com/problems/permutation-in-string/)

最基本的滑动窗口.

> Golang

```golang

func checkInclusion(s1 string, s2 string) bool {
	s1Bytes := []byte(s1)
	s2Bytes := []byte(s2)

	targetMap := make(map[byte]int)
	currentMap := make(map[byte]int)

	if len(s2) < len(s1) {
		return false
	}

	for i := 0; i < len(s1Bytes); i++ {
		targetMap[s1Bytes[i]]++
		currentMap[s2Bytes[i]]++
	}

	if check(currentMap, targetMap) {
		return true
	}

	// [l, r]
	for l, r := 0, len(s1Bytes); r < len(s2Bytes); r++ {
		currentMap[s2Bytes[r]]++
		currentMap[s2Bytes[l]]--
		l++

		if check(currentMap, targetMap) {
			return true
		}
	}

	return false
}

func check(source, target map[byte]int) bool {
	for k, v := range target {
		if source[k] < v {
			return false
		}
	}
	return true
}

```
