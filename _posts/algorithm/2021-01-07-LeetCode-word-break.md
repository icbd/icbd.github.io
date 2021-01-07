---
layout: post
title:  LeetCode word-break
date:   2021-01-07
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/word-break/](https://leetcode-cn.com/problems/word-break/)

## 基础回顾

`map` 在迭代内部添加元素, 能遍历到新元素.

> Golang

```golang
m := make(map[int]int)
	m[1] = 1
	m[2] = 2

	for k,v := range m {
		m[3] = 3
		fmt.Println(k, v)
	}
	fmt.Println(m)
	/**
	1 1
	2 2
	3 3
	map[1:1 2:2 3:3]
	*/
```

`slice` 在迭代内部添加元素, 不能遍历到新元素.

> Golang

```golang
l := make([]int, 0)
	l = append(l, 0)
	l = append(l, 1)
	for i, v := range l {
		if len(l) < 5 {
			l = append(l, 2)
		}
		fmt.Println(i, v)
	}
	fmt.Println(l)
	/**
	0 0
	1 1
	[0 1 2 2]
	 */
```

## 思路

用 map 存字典, 字典内保存初始元素, 以及后续计算的中间值, 减少重复计算.

> Golang

```golang

var dict map[string]bool // 字典元素 / 可以由字典元素组成的字符串

func wordBreak(s string, wordDict []string) bool {
	dict = make(map[string]bool)
	for _, item := range wordDict {
		dict[item] = true
	}
	return helper(s)
}

func helper(str string) bool {
	if str == "" {
		return true
	}

	if v, ok := dict[str]; ok {
		return v
	}

	for k, v := range dict {
		if v {
			maybe, leftover := trimPrefix(str, k)
			if maybe && helper(leftover) {
				dict[str] = true
				return true
			}
		}
	}

	dict[str] = false
	return false
}

func trimPrefix(str, prefix string) (maybe bool, leftover string) {
	if !strings.HasPrefix(str, prefix) {
		return false, ""
	}

	leftover = strings.TrimPrefix(str, prefix)
	// 这里是一个错误的优化
	// [cat, cats, ...] 多余的 cat 剪裁会让 cats 失效
	//for strings.HasPrefix(leftover, prefix) {
	//	leftover = strings.TrimPrefix(leftover, prefix)
	//}

	return true, leftover
}

```
