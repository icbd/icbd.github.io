---
layout: post
title:  LeetCode reverse-linked-list-ii
date:   2020-12-27
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/reverse-linked-list-ii/](https://leetcode-cn.com/problems/reverse-linked-list-ii/)

* 如果 m 等于 n, 就不需要反转, 直接返回原链
* 如果 m 等于 1, 需要特殊处理结果的开端节点

> Golang

```golang
package main

import "fmt"

func main() {
	//1->2->3->4->5
	//list := &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3, Next: &ListNode{Val: 4, Next: &ListNode{Val: 5}}}}}

	list := &ListNode{Val: 3, Next: &ListNode{Val: 5}}
	res := reverseBetween(list, 1, 2)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func reverseBetween(head *ListNode, m int, n int) *ListNode {
	if head == nil || m == n {
		return head
	}
	originalHead := head

	var preM, mNode *ListNode
	for i := 1; i < m; i++ {
		preM = head
		head = head.Next
	}
	mNode = head

	last := head
	next := head.Next
	head = head.Next
	for i := 0; i < n-m; i++ {
		next = head.Next
		head.Next = last //reverse
		last = head
		head = next
		if next != nil {
			next = next.Next
		}
	}
	mNode.Next = head
	if preM != nil {
		preM.Next = last
	} else {
		originalHead = last
	}

	return originalHead
}

```
