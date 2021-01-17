---
layout: post
title:  LeetCode house-robber-iii
date:   2021-01-13
Author: CBD
tags:   [LeetCode, Tree, DP]
---

[https://leetcode-cn.com/problems/house-robber-iii/](https://leetcode-cn.com/problems/house-robber-iii/)

求极值的典型 DP .

后续遍历二叉树, 把节点收益记录在 map 中.

特别注意父节点的收益计算, 应该为左右节点的收益之和, 根据跳过的约束选择符合条件的左节点和右节点.

> Golang

```golang

var stealDict map[*TreeNode]int // [*TreeNodePointer] => value
var jumpDict map[*TreeNode]int  // [*TreeNodePointer] => value

func rob(root *TreeNode) int {
	stealDict = make(map[*TreeNode]int)
	jumpDict = make(map[*TreeNode]int)
	help(root)

	return max(stealDict[root], jumpDict[root])
}

func help(node *TreeNode) {
	if node == nil {
		return
	}
	help(node.Left)
	help(node.Right)

	// 偷这个 node, node 的左右必须跳过
	// 获得左右都是 jump 的加总收益
	steal := jumpDict[node.Left] + jumpDict[node.Right] + node.Val
	stealDict[node] = steal

	// 跳过这个 node, node 的左右没有要求
	// 从左边选一个大的, 从右边选一个大的, 获得左右加总的收益
	jump := max(jumpDict[node.Left], stealDict[node.Left]) + max(jumpDict[node.Right], stealDict[node.Right])
	jumpDict[node] = jump

	return
}

func max(list ...int) int {
	sort.Ints(list)
	return list[len(list)-1]
}

```
