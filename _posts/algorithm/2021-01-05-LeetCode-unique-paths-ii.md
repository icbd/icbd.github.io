---
layout: post
title:  LeetCode unique-paths-ii
date:   2021-01-05
Author: CBD
tags:   [LeetCode, DP]
---

[https://leetcode-cn.com/problems/unique-paths-ii/](https://leetcode-cn.com/problems/unique-paths-ii/)

一旦有障碍物, 会影响其后面的结果, 特别是边缘行.

> Golang

```golang

const OBSTACLE = 1

func uniquePathsWithObstacles(obstacleGrid [][]int) int {
	m := len(obstacleGrid[0])
	n := len(obstacleGrid)
	dp := make([][]int, n)
	for i := 0; i < n; i++ {
		dp[i] = make([]int, m)
	}

	// 直接初始化第一行, 答案固定为 1
	// 如果有障碍, 后面都是 0
	for i := 0; i < m; i++ {
		if i == 0 {
			if obstacleGrid[0][0] == OBSTACLE {
				return 0
			} else {
				dp[0][i] = 1
				continue
			}
		}

		if obstacleGrid[0][i] == OBSTACLE {
			dp[0][i] = 0
		} else {
			dp[0][i] = dp[0][i-1]
		}
	}

	for line := 1; line < n; line++ {
		// 最左边的确定为 1, 除非有障碍
		if obstacleGrid[line][0] == OBSTACLE {
			dp[line][0] = 0
		} else {
			dp[line][0] = dp[line-1][0]
		}

		for idx := 1; idx < m; idx++ {
			if obstacleGrid[line][idx] == OBSTACLE {
				dp[line][idx] = 0
			} else {
				dp[line][idx] = dp[line][idx-1] + dp[line-1][idx]
			}
		}
	}

	//fmt.Println(dp)
	return dp[n-1][m-1]
}

```
