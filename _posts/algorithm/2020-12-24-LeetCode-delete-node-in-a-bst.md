---
layout: post
title:  LeetCode delete-node-in-a-bst
date:   2020-12-24
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/delete-node-in-a-bst/](https://leetcode-cn.com/problems/delete-node-in-a-bst/)

![delete-node-in-a-bst](/images/delete-node-in-a-bst.png)

删除节点分三种情况:

* 没有子树时, 该节点是叶子节点, 可以直接删除;
* 只有一个子树时, 如图, 2 和 3 之间没有其他节点, 可以把 2 嫁接到 3 后面(2 可能是一个子树, 但是其内部是有序的, 可以整体移动) 如果把 nil 也看做节点, 前两种可以合并.
* 如果有两个子树, 如图, 删除 2,要重新选举一个节点代替 2, 需要满足新节点比左子树都大, 比右子树都小. 一种方法是从左子树挑出一个最大值, 或者从右子树挑出一个最小值.(调整后的方案不唯一)

从右子树挑出一个最小值为例:

* 最小节点的左子树一定为空;
* 如果最小节点没有右子树, 则把 nil 赋值给最小节点的父节点; 也就是直接删掉该节点, 该节点失去引用后交由GC回收;
* 如果最小节点有右子树, 则把右子树指向最小节点的父节点; 右子树可以看做一个整体, 他比最小节点大, 但是比最小节点的父节点小, 他们中间再没有其他节点, 所以可以直接接入.

> Golang

```golang
package main

import "fmt"

func main() {
	////[5,3,6,2,4,null,7]
	n2 := &TreeNode{Val: 2}
	n3 := &TreeNode{Val: 3}
	n4 := &TreeNode{Val: 4}
	n5 := &TreeNode{Val: 5}
	n6 := &TreeNode{Val: 6}
	n7 := &TreeNode{Val: 7}
	root := n5
	n5.Left = n3
	n5.Right = n6
	n3.Left = n2
	n3.Right = n4
	n6.Right = n7

	result := deleteNode(root, 5)
	fmt.Println(result)
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func deleteNode(node *TreeNode, key int) *TreeNode {
	// 到叶子还没找到
	if node == nil {
		return nil
	}

	if key < node.Val { // 划分子问题, 去左子树找
		node.Left = deleteNode(node.Left, key)
		return node
	}

	if key > node.Val { // 划分子问题, 去右子树找
		node.Right = deleteNode(node.Right, key)
		return node
	}

	// 以下: key == node.Val

	if node.Left != nil && node.Right != nil { // 左右子树都不为空
		rightSubTree, minNode := deleteMinNode(node.Right) // 最小值的左子树一定为空, 右子树不一定
		minNode.Left = node.Left
		minNode.Right = rightSubTree
		return minNode
	} else { // 至少一个空子树
		if node.Left == nil {
			return node.Right
		}
		return node.Left
	}
}

// 最小的节点左子树一定为空, 右子树不一定为空.
func deleteMinNode(node *TreeNode) (subTree *TreeNode, minNode *TreeNode) {
	if node.Left == nil {
		// 当右子树不为空时, 返回剩余的子树
		// 剥离最小节点
		return node.Right, &TreeNode{Val: node.Val}
	}

	node.Left, minNode = deleteMinNode(node.Left)
	return node, minNode
}

```
