---
layout: post
title:  LeetCode find-all-anagrams-in-a-string
date:   2021-01-11
Author: CBD
tags:   [LeetCode, SlideWindow]
---

[https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string/)

> Golang

```golang

func findAnagrams(s string, p string) []int {
	results := make([]int, 0)

	sourceBytes := []byte(s)
	targetBytes := []byte(p)

	if len(targetBytes) > len(sourceBytes) {
		return results
	}

	sourceMap := make(map[byte]int)
	targetMap := make(map[byte]int)
	for i := 0; i < len(targetBytes); i++ {
		sourceMap[sourceBytes[i]]++
		targetMap[targetBytes[i]]++
	}
	if check(sourceMap, targetMap) {
		results = append(results, 0)
	}

	for l, r := 0, len(targetBytes); r < len(sourceBytes); r++ {
		sourceMap[sourceBytes[r]]++
		sourceMap[sourceBytes[l]]--
		l++

		if check(sourceMap, targetMap) {
			results = append(results, l)
		}
	}

	return results
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
