---
layout: post
title:  LeetCode longest-substring-without-repeating-characters
date:   2021-01-11
Author: CBD
tags:   [LeetCode, SlideWindow]
---

[https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters/)

> Golang

```golang
func lengthOfLongestSubstring(s string) int {
	if s == "" {
		return 0
	}

	result := 0
	subLen := 0
	dict := make(map[rune]int)

	runes := []rune(s)
	l := 0
	r := 0
	// [l,r)
	for r < len(runes) {
		add := runes[r]
		//fmt.Println(string(add))
		v := dict[add]
		for v > 0 {
			sub := runes[l]
			dict[sub]--
			subLen--
			if add == sub {
				v--
			}
			l++
		}

		dict[add]++
		subLen += 1
		if subLen > result {
			result = subLen
		}
		r++
	}

	return result
}

```
