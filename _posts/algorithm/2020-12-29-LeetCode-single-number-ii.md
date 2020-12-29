---
layout: post
title:  
date:   2020-12-29
Author: CBD
tags:   []
---

[https://leetcode-cn.com/problems/single-number-ii/](https://leetcode-cn.com/problems/single-number-ii/)

用数字电路的思路解决软件问题.

受 single-number 的启发, 我们还是可以用位运算来实现不使用额外空间的算法.

single-number 利用了异或的特性:

```text
0 ^ a => a
0 ^ a ^ a => 0
```

我们想找到一个新的运算符(用`@`表示), 希望他满足这样的特性:

```text
0 @ a => a
0 @ a @ a @ a => 0
```

很容易想到三进制有这样的性质:

```text
0 + 1 => 1
1 + 1 => 2
2 + 1 => 0 (进位)
```

所以我们需要实现一个加法器: `@`, 他不加一时, 原样输出, 每加一就按照 `0->1->2->0` 的状态流转.

因为一共三种状态, 至少需要占用两个 bit, `00 -> 01 -> 10`, 给 `10` 再加一, 先进位后变为 `00`.

```text
// 加 0 保持不变
00 + 0 => 00
01 + 0 => 01
10 + 0 => 10

// 加 1 在三种状态下循环
00 + 1 => 01 
01 + 1 => 10
10 + 1 => 00
```

整理得到真值表:

|high | low | x | newHigh | newLow |
|---|---|---|---|---|
|0|0|0 |0|0|
|0|1|0 |0|1|
|1|0|0 |1|0|
|0|0|1 |0|1|
|0|1|1 |1|0|
|1|0|1 |0|0|

### 推导 `newLow` 表达式

只需要关心 newLow 为 `1` 的行.
这一步与 newHigh 无关, 暂时隐去.

|high | low | x  | newLow |
|---|---|---|---|
|0|1|0 |1|
|0|0|1 |1|

`newLow = (~high & low & ~x) | (~high & ~low & x)`

### 推导 `newHigh` 表达式

经由上一步, 低位已经被更新, 所以使用 newLow 而不是 low:

|high | x | newLow | newHigh |
|---|---|---|---|---|
|1|0 |0|1|
|0|1 |0|1|

`newHigh = (high & ~x & ~newLow) | (~high & x & ~newLow)`

针对该题目, 经过三次加法器, 会经过 `00 -> 01 -> 10 -> 00`; 经过一次加法器, 会经过 `00 -> 01`.
`high` 始终为 `0`, `low` 对应的值即为只出现一次的值.

特别注意 , Golang 没有取反的位操作 `~`, 等效的是一元操作 `^`.

> Golang

```golang
func singleNumber(nums []int) int {
	high := 0
	low := 0
	for i := 0; i < len(nums); i++ {
		low = (^high & low & ^nums[i]) | (^high & ^low & nums[i])
		high = (high & ^nums[i] & ^low) | (^high & nums[i] & ^low)
	}
	fmt.Println(high,low)
	return low
}
```
