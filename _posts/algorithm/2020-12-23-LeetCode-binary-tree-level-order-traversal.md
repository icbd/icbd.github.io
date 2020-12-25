---
layout: post
title:  LeetCode binary-tree-level-order-traversal
date:   2020-12-23
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/binary-tree-level-order-traversal/](https://leetcode-cn.com/problems/binary-tree-level-order-traversal/)

Tips: root 可能为 nil.

> Golang

```golang
package main

import (
	"fmt"
)

func main() {
	//[3,9,20,null,null,15,7]

	n3 := &TreeNode{Val: 3}
	n7 := &TreeNode{Val: 7}
	n9 := &TreeNode{Val: 9}
	n15 := &TreeNode{Val: 15}
	n20 := &TreeNode{Val: 20}

	n3.Left = n9
	n3.Right = n20
	n20.Left = n15
	n20.Right = n7

	fmt.Println(levelOrder(n3))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

var result [][]int

func levelOrder(root *TreeNode) [][]int {
	result = make([][]int, 0)
	if root == nil {
		return result
	}

	currentNodes := []*TreeNode{root}
	for len(currentNodes) > 0 {
		currentNodes = add(currentNodes)
	}
	return result
}

func add(nodes []*TreeNode) (nextNodes []*TreeNode) {
	nodeVals := make([]int, len(nodes))
	nextNodes = make([]*TreeNode, 0)
	for i, node := range nodes {
		nodeVals[i] = node.Val
		if node.Left != nil {
			nextNodes = append(nextNodes, node.Left)
		}
		if node.Right != nil {
			nextNodes = append(nextNodes, node.Right)
		}
	}
	result = append(result, nodeVals)
	return nextNodes
}

```
