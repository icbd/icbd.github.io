---
layout: post
title:  LeetCode balanced-binary-tree
date:   2020-12-22
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/balanced-binary-tree/](https://leetcode-cn.com/problems/balanced-binary-tree/)

```golang
package main

import (
	"fmt"
)

func main() {
	//[3,9,20,null,null,15,7]
	root := TreeNode{Val: 3}
	root.Left = &TreeNode{Val: 9}
	root.Right = &TreeNode{Val: 20, Left: &TreeNode{Val: 15}, Right: &TreeNode{Val: 7}}
	fmt.Println(isBalanced(&root))

	//[1,2,2,3,3,null,null,4,4]
	root2 := TreeNode{Val: 1, Left: &TreeNode{Val: 2, Left: &TreeNode{Val: 3, Left: &TreeNode{Val: 4}, Right: &TreeNode{Val: 4}}, Right: &TreeNode{Val: 3}}, Right: &TreeNode{Val: 2}}
	fmt.Println(isBalanced(&root2))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func isBalanced(root *TreeNode) bool {
	_, assertNotBalanced := treeDeep(root)
	return !assertNotBalanced
}

func treeDeep(node *TreeNode) (deep int, assertNotBalanced bool) {
	if node == nil {
		return 0, false
	}

	leftDeep, leftNotBalanced := treeDeep(node.Left)
	rightDeep, rightNotBalanced := treeDeep(node.Right)

	//fmt.Println(leftDeep, leftNotBalanced, rightDeep, rightNotBalanced)

	if leftNotBalanced || rightNotBalanced || leftDeep-rightDeep > 1 || leftDeep-rightDeep < -1 {
		return deep + 1, true
	}

	if leftDeep >= rightDeep {
		return leftDeep + 1, false
	} else {
		return rightDeep + 1, false
	}
}

```
