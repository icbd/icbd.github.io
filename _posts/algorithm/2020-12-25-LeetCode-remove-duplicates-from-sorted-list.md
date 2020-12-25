---
layout: post
title:  LeetCode remove-duplicates-from-sorted-list
date:   2020-12-25
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list/)

虽然是很简单的题,但是边界条件很容易出错.

一定要测试 `[1,1,1]` 和 `[1,1,2,3,3]`.

* 以当前节点为 nil 作为停止条件;
* 如果当前节点和上一个节点值相同, 则更新上一个节点的 next 指针, 但不需要更新 `preNode` 的值.

> Golang

```golang
package main

import "fmt"

func main() {
	////[1,1,2,3,3]
	//list := &ListNode{Val: 1, Next: &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3, Next: &ListNode{Val: 3}}}}}

	////[1,1,1]
	list := &ListNode{Val: 1, Next: &ListNode{Val: 1, Next: &ListNode{Val: 1}}}

	res := deleteDuplicates(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func deleteDuplicates(head *ListNode) *ListNode {
	preNode := head
	currentNode := head
	for currentNode != nil {
		if currentNode.Val == preNode.Val {
			preNode.Next = currentNode.Next
		} else {
			preNode = currentNode
		}
		currentNode = currentNode.Next
	}
	return head
}

```
