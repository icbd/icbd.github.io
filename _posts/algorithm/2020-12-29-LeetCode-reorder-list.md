---
layout: post
title:  LeetCode reorder-list
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/reorder-list/](https://leetcode-cn.com/problems/reorder-list/)

把链表入队列, 然后依次从队头队尾取数据. 组装新链表时, 使用假的首元素会方便很多.

> Golang

```golang
package main

import "fmt"

func main() {
	list := &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3, Next: &ListNode{Val: 4, Next: &ListNode{Val: 5}}}}}
	reorderList(list)
	fmt.Println(list)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

//给定链表 1->2->3->4, 重新排列为 1->4->2->3.
//给定链表 1->2->3->4->5, 重新排列为 1->5->2->4->3.
func reorderList(head *ListNode) {
	if head == nil || head.Next == nil {
		return
	}

	list := make([]*ListNode, 0)
	tail := head
	for tail != nil {
		list = append(list, tail)
		tail = tail.Next
	}

	head = &ListNode{Val: 0}
	t := head
	half := (len(list) + 1) / 2
	for i := 0; i < half; i++ {
		t.Next = list[i]
		t = t.Next
		t.Next = list[len(list)-i-1]
		t = t.Next
	}
	t.Next = nil
	head = head.Next
}

```
