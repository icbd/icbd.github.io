---
layout: post
title:  LeetCode number-of-1-bits
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/number-of-1-bits/](https://leetcode-cn.com/problems/number-of-1-bits/)

> Golang

```golang
package main

import "fmt"

func main() {
	res := hammingWeight(11) // 3
	fmt.Println(res)
}

func hammingWeight(num uint32) int {
	count := 0
	for num != 0 {
		if num&1 == 1 {
			count++
		}
		num >>= 1
	}
	return count
}

```
