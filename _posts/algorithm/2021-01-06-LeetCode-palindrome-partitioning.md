---
layout: post
title:  LeetCode palindrome-partitioning
date:   2021-01-06
Author: CBD
tags:   [LeetCode, TREE]
---

[https://leetcode-cn.com/problems/palindrome-partitioning/](https://leetcode-cn.com/problems/palindrome-partitioning/)

![分割回文串](/images/分割回文串.png)

遍历方式类似一棵树, 节点可能有多个子节点.

> Golang

```golang

func check(str string) bool {
	first := 0
	last := len(str) - 1
	for first < last {
		if str[first] != str[last] {
			return false
		}
		first++
		last--
	}
	return true
}

var result [][]string

func partition(s string) [][]string {
	// init
	result = make([][]string, 0)
	splits := make([]string, 0)

	helper(s, splits)
	return result
}

func helper(str string, splits []string) {
	if str == "" && len(splits) > 0 {
		temp := make([]string, len(splits))
		copy(temp, splits)
		result = append(result, temp)
		return
	}

	for i := 1; i <= len(str); i++ {
		subStr := str[:i]
		if check(subStr) {
			splits = append(splits, subStr) // push, 向下遍历

			lastStr := str[i:]
			helper(lastStr, splits)

			splits = splits[:len(splits)-1] // pop, 向上回溯, 转移到下个节点
		}
	}
}

```
