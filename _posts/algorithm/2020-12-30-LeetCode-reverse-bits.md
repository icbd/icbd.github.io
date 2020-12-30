---
layout: post
title:  LeetCode reverse-bits
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/reverse-bits/](https://leetcode-cn.com/problems/reverse-bits/)

> Golang

```golang
func reverseBits(num uint32) uint32 {
	bitLen := 8 * int(unsafe.Sizeof(num))
	list := make([]uint32, bitLen)
	for i := 0; num != 0; i++ {
		if num&1 == 1 {
			list[i] = 1
		} else {
			list[i] = 0
		}
		num >>= 1
	}
	num = 0
	fmt.Println(list)

	// list[0] 是 num 的最低位
	for i := 0; i < bitLen-1; i++ {
		if list[i] == 1 {
			num += 1
		}
		num <<= 1
	}
	num += list[bitLen-1] // 最后一位不需要再移位
	return num
}
```

不借助数组, 直接操作结果:

> Golang

```golang
func reverseBits(num uint32) uint32 {
	bitLen := 8 * int(unsafe.Sizeof(num))
	var result uint32
	for i := 0; num != 0; i++ {
		result += (num & 1) << (bitLen - 1 - i) // 取最低位, 左移到高位累加到结果值上
		num >>= 1
	}
	return result
}

```
