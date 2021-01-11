---
layout: post
title:  LeetCode minimum-depth-of-binary-tree
date:   2021-01-11
Author: CBD
tags:   [LeetCode, BFS, DFS]
---

[https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree/)

## BFS

注意只有单边子树的 case, `[2,null,3,null,4,null,5,null,6]`

每次迭代之后, `i < len(queue)` 会重新计算 queue 的长度, 他会略过已经访问过的节点来遍历新加入到节点, 正是我们需要的.

slice 的 range 遍历不会每次重新计算长度, 即使 slice 改变也不会遍历新增的数据.

此处不能使用 map, 他们的遍历顺序不能保证, 但是 BFS 要求先完成一轮旧的遍历再遍历新加入的.

> Golang

```golang

type Node struct {
	Tree  *TreeNode
	Depth int
}

func minDepth(root *TreeNode) int {
	if root == nil {
		return 0
	}

	queue := []Node{{Tree: root, Depth: 1}}

	// 每次迭代之后, len(queue) 都会变化
	for i := 0; i < len(queue); i++ {
		tree := queue[i]
		l := tree.Tree.Left
		r := tree.Tree.Right
		if l == nil && r == nil {
			return tree.Depth
		}

		if l != nil {
			queue = append(queue, Node{Tree: l, Depth: tree.Depth + 1})
		}

		if r != nil {
			queue = append(queue, Node{Tree: r, Depth: tree.Depth + 1})
		}
	}
	return -1 // never return -1
}

```

## DFS

注意求最小值的时候, 一定要 `l != nil` 判断空, 当子树有是空的时候, 并不能停止遍历. 叶子节点才是停止条件.

> Golang

```golang
func minDepth(root *TreeNode) int {
	if root == nil {
		return 0
	}

	l := root.Left
	r := root.Right

	if l == nil && r == nil {
		return 1
	}

	minDepthNum := math.MaxInt32
	if l != nil {
		minDepthNum = min(minDepthNum, minDepth(l))
	}

	if r != nil {
		minDepthNum = min(minDepthNum, minDepth(r))
	}

	return minDepthNum + 1
}

func min(a, b int) int {
	if a < b {
		return a
	}
	return b
}

```
