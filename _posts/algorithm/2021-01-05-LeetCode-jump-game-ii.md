---
layout: post
title:  LeetCode jump-game-ii
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/jump-game-ii/](https://leetcode-cn.com/problems/jump-game-ii/)

`visited` 记录跳到该位置最少需要几步:

* 如果可到达的位置从没人到达过, 那么把他设为 `当前步数+1`
* 如果可到达的位置已经到达过(值不为 0), 那么把他设为 `min(当前步数+1, 原步数)`

> Golang

```golang

func jump(nums []int) int {
	if len(nums) <= 1 {
		return 0
	}

	visited := make([]int, len(nums)) // 跳到该位置最少需要几步
	for i := 0; i < len(nums)-1; i++ {
		maxStepLen := nums[i]
		for arriveIdx := i; arriveIdx <= i+maxStepLen; arriveIdx++ {
			if arriveIdx >= len(nums) {
				break
			}

			if visited[arriveIdx] == 0 {
				visited[arriveIdx] = visited[i] + 1
			} else {
				visited[arriveIdx] = min(visited[i]+1, visited[arriveIdx])
			}
		}
	}

	return visited[len(nums)-1] - 1
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
