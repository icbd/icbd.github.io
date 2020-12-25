---
layout: post
title:  LeetCode binary-tree-zigzag-level-order-traversal
date:   2020-12-23
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal/)

参见 `binary-tree-level-order-traversal`, 奇数层内翻转.

>Golang

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

	fmt.Println(zigzagLevelOrder(n3))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

var result [][]int

func zigzagLevelOrder(root *TreeNode) [][]int {
	result = make([][]int, 0)
	if root == nil {
		return result
	}

	currentNodes := []*TreeNode{root}
	for len(currentNodes) > 0 {
		currentNodes = add(currentNodes)
	}
	// 奇数层反转
	for i := 1; i < len(result); i += 2 {
		item := result[i]
		for j := 0; j < len(item)/2; j++ {
			item[j], item[len(item)-1-j] = item[len(item)-1-j], item[j]
		}
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
