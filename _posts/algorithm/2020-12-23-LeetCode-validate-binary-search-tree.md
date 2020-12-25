---
layout: post
title:  LeetCode validate-binary-search-tree
date:   2020-12-23
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/validate-binary-search-tree/](https://leetcode-cn.com/problems/validate-binary-search-tree/)

二叉搜索树要求:

* 节点的左子树只包含小于当前节点的数
* 节点的右子树只包含大于当前节点的数
* 所有左子树和右子树自身必须也是二叉搜索树

隐含的要求是:

* 大小比较是递归的;
* 当前节点的值要比左子树的任意值都大;
* 当前节点的值要比右子树的任意值都小;
* 树中不能包含相同的值;

## 最基本的迭代法

需要在 dfs 中携带子树的最小值/最大值.

> Golang

```golang
package main

import (
	"fmt"
)

func main() {
	////[5,1,4,null,null,3,6]
	//n1 := &TreeNode{Val: 1}
	//n3 := &TreeNode{Val: 3}
	//n4 := &TreeNode{Val: 4}
	//n5 := &TreeNode{Val: 5}
	//n6 := &TreeNode{Val: 6}
	//root := n5
	//n5.Left = n1
	//n5.Right = n4
	//n4.Left = n3
	//n4.Right = n6
	//fmt.Println(isValidBST(root)) // false

	////[5,4,6,null,null,3,7]
	//n3 := &TreeNode{Val: 3}
	//n4 := &TreeNode{Val: 4}
	//n5 := &TreeNode{Val: 5}
	//n6 := &TreeNode{Val: 6}
	//n7 := &TreeNode{Val: 7}
	//root := n5
	//n5.Left = n4
	//n5.Right = n6
	//n6.Left = n3
	//n6.Right = n7
	//fmt.Println(isValidBST(root)) // false

	//// [2,1,3]
	//n1 := &TreeNode{Val: 1}
	//n2 := &TreeNode{Val: 2}
	//n3 := &TreeNode{Val: 3}
	//root := n2
	//n2.Left = n1
	//n2.Right = n3
	//fmt.Println(isValidBST(root)) // true

	// [32,26,47,19,null,null,56,null,27]
	n32 := &TreeNode{Val: 32}
	n26 := &TreeNode{Val: 26}
	n47 := &TreeNode{Val: 47}
	n19 := &TreeNode{Val: 19}
	n56 := &TreeNode{Val: 56}
	n27 := &TreeNode{Val: 27}
	root := n32
	n32.Left = n26
	n32.Right = n47
	n26.Left = n19
	n47.Right = n56
	n19.Right = n27
	fmt.Println(isValidBST(root)) // false
	/*
				   32
				26	 47
		      19  n  n  56
		     n 27
	*/

	//[3,1,5,0,2,4,6,null,null,null,3] // false
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func isValidBST(root *TreeNode) bool {
	if root == nil {
		return true
	}

	result, _, _ := dfs(root)
	return result
}

func dfs(node *TreeNode) (assert bool, min, max *TreeNode) {
	if node == nil {
		return true, nil, nil
	}

	// 叶子节点; 左右全空
	if (node.Left == nil) && (node.Right == nil) {
		return true, node, node
	}

	// 左边有值的情况下, 必须小于当前节点
	if (node.Left != nil) && !(node.Left.Val < node.Val) {
		return false, nil, nil
	}

	// 右边有值的情况下, 必须大于当前节点
	if (node.Right != nil) && !(node.Right.Val > node.Val) {
		return false, nil, nil
	}

	leftBool, leftMin, leftMax := dfs(node.Left)
	rightBool, rightMin, rightMax := dfs(node.Right)

	// 只要有一边是错的, 整个就是错的
	if !leftBool || !rightBool {
		return false, nil, nil
	}

	// 此时左树和右树分别独立成立

	// 当前节点需要比子问题左树的最大值还大
	if leftMax != nil {
		if !(leftMax.Val < node.Val) {
			return false, nil, nil
		}
	}

	// 当前节点需要比子问题右树的最小值还小
	if rightMin != nil {
		if !(rightMin.Val > node.Val) {
			return false, nil, nil
		}
	}

	return true, leftMin, rightMax
}

```

## 中序遍历 LDR

使用中序来遍历二叉搜索树时, 能得到一个递增的序列.

可以利用这个性质先得到 LDR 的路径, 然后检查递增性.

```golang
func isValidBST(root *TreeNode) bool {
	path := make([]int, 0)
	ldr(root, &path)

	if len(path) <= 1 {
		return true
	}

	for i := 1; i < len(path); i++ {
		if path[i] <= path[i-1] {
			return false
		}
	}
	return true
}

func ldr(node *TreeNode, path *[]int) {
	if node == nil {
		return
	}

	ldr(node.Left, path)
	*path = append(*path, node.Val)
	ldr(node.Right, path)
}

```

## 合并 LDR

TIPS: 全局变量需要手动重置.

```golang

var perNode *TreeNode

func isValidBST(root *TreeNode) bool {
	perNode = nil
	return ldr(root)
}
func ldr(root *TreeNode) bool {
	if root == nil {
		return true
	}

	if !ldr(root.Left) {
		return false
	}

	if perNode == nil {
		perNode = root
	} else {
		if !(root.Val > perNode.Val) {
			return false
		}
		perNode = root
	}

	return ldr(root.Right)
}

```
