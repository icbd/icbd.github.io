---
layout: post
title:  LeetCode binary-tree-inorder-traversal
date:   2020-12-31
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/binary-tree-inorder-traversal/](https://leetcode-cn.com/problems/binary-tree-inorder-traversal/)

> Golang

```golang
var list []int

func inorderTraversal(root *TreeNode) []int {
	list = make([]int, 0)
	dfs(root)
	return list
}

func dfs(node *TreeNode) {
	if node == nil {
		return
	}
	dfs(node.Left)
	list = append(list, node.Val)
	dfs(node.Right)
}
```
