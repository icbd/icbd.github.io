---
layout: post
title:  LeetCode jump-game
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/jump-game/](https://leetcode-cn.com/problems/jump-game/)

`visited` 标记位置是否可以访问到, 初始化为全 0, 用 1 表示可跳到.

从底向上的推导:

首先一定可以跳到第 0 个位置, 然后遍历从这个点可以跳到的位置, 依次标记为可以跳到.

然后处理下一个位置, 如果该位置 `visited` 标记为 0, 说明这个位置不能被跳到, 那么他后面的节点都不可能跳到了, 直接返回 false.

如果下个位置 `visited` 标记为 1, 那么重复第一步, 把从这个的出发的所有可到达的位置标记为 1.

注意数组越界问题.

> Golang

```golang

const VISITED = 1

func canJump(nums []int) bool {
	if len(nums) <= 1 {
		return true
	}

	visited := make([]int, len(nums)) // init with invisited
	visited[0] = VISITED
	for i := 0; i < len(nums)-1; i++ {
		if visited[i] != VISITED { // 没有可以跳到这里的, 也不可能跳到更远了
			return false
		}

		maxStep := nums[i]
		for arriveIdx := i; arriveIdx <= i+maxStep; arriveIdx++ {
			if arriveIdx >= len(nums) { // 已经越界
				break
			}
			visited[arriveIdx] = VISITED
		}
	}

	return visited[len(nums)-1] == VISITED
}

```
