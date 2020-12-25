---
layout: post
title:  LeetCode insert-into-a-binary-search-tree
date:   2020-12-23
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/](https://leetcode-cn.com/problems/insert-into-a-binary-search-tree/)

在一棵二叉搜索树上插入新值, 如果新值比当前大, 插入到右子树中, 如果新值比当前小, 插入到左子树中.

根据这个规则, **新值一定能插入到某个叶子下**, 而且仍然能保持二叉搜索树的特性. 不需要考虑树的平衡.

>Golang

```golang
package main

import "fmt"

func main() {
	////[4,2,7,1,3]
	n1 := &TreeNode{Val: 1}
	n2 := &TreeNode{Val: 2}
	n3 := &TreeNode{Val: 3}
	n4 := &TreeNode{Val: 4}
	n7 := &TreeNode{Val: 7}
	root := n4
	n4.Left = n2
	n4.Right = n7
	n2.Left = n1
	n2.Right = n3
	fmt.Println(insertIntoBST(root, 5))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func insertIntoBST(root *TreeNode, val int) *TreeNode {
	n := dfs(root, val)
	return n
}

func dfs(node *TreeNode, val int) *TreeNode {
	// 已经到叶子节点
	if node == nil {
		return &TreeNode{Val: val}
	}

	// 划分子问题
	if val > node.Val {
		node.Right = dfs(node.Right, val)
	} else {
		node.Left = dfs(node.Left, val)
	}
	return node
}

```

这里 dfs 的返回值语义稍难理解, 有时候会返回新创建的节点, 有时候会返回递归的回溯节点, 统一来说还是返回了回溯节点.

查找时由根向下, 到底之后创建新的的叶子节点, 然后向上爬树.

爬倒数第一层时, 把新节点接到老的叶子上; 爬导数第二层, 把老的叶子接到老叶子的父节点上; 依次递归直到根节点.

可以剪枝, 当倒数第一层接入完成之后, 就可以停下所有的回溯了.

```golang
func insertIntoBST(root *TreeNode, val int) *TreeNode {
	if root == nil {
		return &TreeNode{Val: val}
	}

	parentNode := root  // 记录上一个节点
	currentNode := root // 当前处理的节点
	bigger := true      // 记录 parentNode 的左右

	for {
		if currentNode == nil {
			newNode := &TreeNode{Val: val}
			if bigger {
				parentNode.Right = newNode
			} else {
				parentNode.Left = newNode
			}
			break
		}

		parentNode = currentNode
		if val > currentNode.Val {
			bigger = true
			currentNode = currentNode.Right
		} else {
			bigger = false
			currentNode = currentNode.Left
		}
	}

	return root
}
```

但是运行结果显示, 剪枝的速度更慢了, 可能跟 LeetCode 对 go 片段的执行方式有关.
