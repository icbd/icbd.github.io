---
layout: post
title:  LeetCode palindrome-partitioning-ii
date:   2021-01-06
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/palindrome-partitioning-ii/](https://leetcode-cn.com/problems/palindrome-partitioning-ii/)

## 思路

以 `ABBXX` 为例.

`A` 这样的单个字符, 本身就是回文, 需要 0 次切割.

`AB` 本身不是回文, 需要切割, 最少 1 刀.

`ABB` 不是回文, 需要切. 我们已知 `AB` 最少切 1 刀, 那么 V(`ABB`) = V(`AB`) & V(`B`) = 1 + 1 + 0= 2 ( `&` 也需要一刀). 这是一个候选结果. **我们还需要计算最后一个新加入的字符对结果的影响, 也就是从后向前检查回文**: `BB` 是回文, 那么 V(`ABB`) = V(`A`) & V(`BB`) = 1 + 1 + 0 = 2. 那么 `ABB` 最后的结果就是 2 .

以此类推:

* V(`ABBXX`) // 如果直接就是回文, 就切 0 刀
* V(`ABBX`) & V(`X`) // `X` 是回文, 结果为: V(`ABBX`) + 1 + 0
* V(`ABB`) & V(`XX`) // `XX` 是回文, 结果为: V(`ABB`) + 1 + 0
* V(`AB`) & V(`BXX`) // `BXX` 不是回文, 不会让结果更好, 跳过.
* V(`A`) & V(`BBXX`) // `BBXX` 不是回文, 不会让结果更好, 跳过.

结果是从中选一个最小的, 核心逻辑是递归:

> Golang

```golang
	min := len(str) - 1
	for cut := len(str) - 1; cut > 0; cut-- {
		preStr := str[:cut]
		afterStr := str[cut:]
		if check(afterStr) {
			tempMin := helper(preStr) + 1
			if tempMin < min {
				min = tempMin
			}
		}
	}
```

显然 `helper(str)` 有很多重复子问题, 需要用缓存来跳过重复计算.

> Golang

```golang

var checkCache map[string]bool
var resultCache map[string]int

func minCut(s string) int {
	checkCache = make(map[string]bool)
	resultCache = make(map[string]int)

	return helper(s)
}

func helper(str string) int {
	if check(str) {
		return 0
	}
	if cachedMin, ok := resultCache[str]; ok {
		return cachedMin
	}

	min := len(str) - 1
	for cut := len(str) - 1; cut > 0; cut-- {
		preStr := str[:cut]
		afterStr := str[cut:]
		if check(afterStr) {
			tempMin := helper(preStr) + 1
			if tempMin < min {
				min = tempMin
			}
		}
	}
	resultCache[str] = min
	return min
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

func check(str string) bool {
	if v, ok := checkCache[str]; ok {
		return v
	}

	result := true
	first := 0
	last := len(str) - 1
	for first < last {
		if str[first] != str[last] {
			result = false
			break
		}
		first++
		last--
	}
	checkCache[str] = result
	return result
}

```
