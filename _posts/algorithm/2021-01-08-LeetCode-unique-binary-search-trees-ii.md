---
layout: post
title:  LeetCode unique-binary-search-trees-ii
date:   2021-01-08
Author: CBD
tags:   [LeetCode, TREE]
---

[https://leetcode-cn.com/problems/unique-binary-search-trees-ii/](https://leetcode-cn.com/problems/unique-binary-search-trees-ii/)

基本的树的遍历, 让每个数都当一次 root .

注意, 如果左子树为空, 那么返回的 leftList 长度为 0 , for 会跳过, 但是如果此时右子树不为空, 应当把左子树设为 nil 然后迭代右子树.

为了方便迭代, 如果一方为空数组, 就用 nil 占位.

```golang
		leftList := helper(left, root-1)
		if len(leftList) == 0 {
			leftList = append(leftList, nil)
		}
```

> Golang

```golang
package main

import (
	"fmt"
)

func main() {
	res := generateTrees(3)
	fmt.Println(res)
	fmt.Println(len(res))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func generateTrees(n int) []*TreeNode {
	return helper(1, n)
}

func helper(left, right int) []*TreeNode {
	list := make([]*TreeNode, 0)

	for root := left; root <= right; root++ {
		leftList := helper(left, root-1)
		if len(leftList) == 0 {
			leftList = append(leftList, nil)
		}

		rightList := helper(root+1, right)
		if len(rightList) == 0 {
			rightList = append(rightList, nil)
		}

		for l := 0; l < len(leftList); l++ {
			for r := 0; r < len(rightList); r++ {
				node := &TreeNode{Val: root, Left: leftList[l], Right: rightList[r]}
				list = append(list, node)
			}
		}
	}

	return list
}

```
