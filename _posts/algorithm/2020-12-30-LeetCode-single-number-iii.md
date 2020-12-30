---
layout: post
title:  LeetCode single-number-iii
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/single-number-iii/](https://leetcode-cn.com/problems/single-number-iii/)

假设最后的结果是 `[A, B]`, 迭代所有值异或, 会把出现偶数次的筛除, 结果为 `xorAB := A^B`.

由 `110 ^ 101 => 011` 可见, A 和 B 都为 1 的位置被削为 0, 结果为 1 的位置是 A B 有且仅有一个为 1.

我们取`xorAB` 中最后一个为 `1` 的位, 记为 mark 位,  AB 中有且仅有一个在 mark 位值为 1. 虽然我们不知道其他值在 mark 位的情况, 但是他们都出现偶数次, 即使存在 mark 位也会被偶数次异或消除. 所以要做的就是找迭代所有 mark 位为 1 的值异或, 最后的结果必然为 A B 之一.

> Golang

```golang
package main

import "fmt"

func main() {
	res := singleNumber([]int{1, 2, 1, 3, 2, 5})
	fmt.Println(res)
}

func singleNumber(nums []int) []int {
	// 出现过偶数次的值被抵消掉, 出现过一次的值被印刻在 xorAB 里, 即 A ^ B
	xorAB := 0
	for i := 0; i < len(nums); i++ {
		xorAB ^= nums[i]
	}
	if xorAB == 0 {
		panic("data error")
	}

	// 二进制表示的数, 从右边数第一个 `1` 的位置
	// A 和 B 有且仅有一个数在该位置为 `1`
	lastOneIdx := 0
	for n := xorAB; n&1 != 1; n >>= 1 {
		lastOneIdx += 1
	}

	// 将所有 lastOneIdx 档位为 `1` 的值跟 xorAB 异或
	// 偶数次的值还是被抵消掉, 假设出现的值为 b, 那么 `xorAB^b` => `a^b^b` => `a`
	a := xorAB
	for i := 0; i < len(nums); i++ {
		if (nums[i]>>lastOneIdx)&1 == 1 {
			a ^= nums[i]
		}
	}
	b := xorAB ^ a
	return []int{a, b}
}

```
