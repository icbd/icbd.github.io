---
layout: post
title:  LeetCode minimum-window-substring
date:   2021-01-11
Author: CBD
tags:   [LeetCode, SlideWindow]
---

[https://leetcode-cn.com/problems/minimum-window-substring/](https://leetcode-cn.com/problems/minimum-window-substring/)

t 中的字母可能为大小混合的写不唯一的字符串, 需要个数也一致.

> Golang

```golang
func minWindow(s string, t string) string {
	sRunes := []rune(s)
	tRunes := []rune(t)

	target := make(map[rune]int) // a => 2
	for _, r := range tRunes {
		target[r] += 1
	}

	result := "!" + s
	sRunesMap := make(map[rune]int)

	// [l, r]
	for l, r := 0, 0; r < len(s); r++ {
		sRunesMap[sRunes[r]]++

		for contained(sRunesMap, target) {
			candidate := s[l : r+1]
			if len(candidate) < len(result) {
				result = candidate
			}
			sRunesMap[sRunes[l]]--
			l++
		}
	}

	if len(result) > len(s) {
		return ""
	}
	return result
}

func contained(strRuneMap map[rune]int, target map[rune]int) bool {
	for k, count := range target {
		if count > strRuneMap[k] {
			return false
		}
	}
	return true
}

```
