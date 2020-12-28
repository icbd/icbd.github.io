---
layout: post
title:  LeetCode partition-list
date:   2020-12-27
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/partition-list/](https://leetcode-cn.com/problems/partition-list/)

注意合并两个链表时, 新链表的头尾处理.

> Golang

```golang
package main

import "fmt"

func main() {
	//1->4->3->2->5->2 => 1->2->2->4->3->5
	list := &ListNode{Val: 1, Next: &ListNode{Val: 4, Next: &ListNode{Val: 3, Next: &ListNode{Val: 2, Next: &ListNode{Val: 5, Next: &ListNode{Val: 2}}}}}}
	res := partition(list, 3)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func partition(head *ListNode, x int) *ListNode {
	smaller := &ListNode{Val: -1 << 63}
	smallerHead := smaller
	bigger := &ListNode{Val: -1 << 63} // >=
	biggerHead := bigger

	for head != nil {
		if head.Val < x {
			smaller.Next = head
			smaller = smaller.Next
		} else {
			bigger.Next = head
			bigger = bigger.Next
		}
		head = head.Next
	}
	bigger.Next = nil
	smaller.Next = biggerHead.Next

	return smallerHead.Next
}

```
