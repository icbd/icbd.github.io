---
layout: post
title:  LeetCode reverse-linked-list
date:   2020-12-26
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/reverse-linked-list/](https://leetcode-cn.com/problems/reverse-linked-list/)

在更新链表指向前, 先保存原 next 信息.

## 迭代法

> Golang

```golang
package main

import "fmt"

func main() {
	//1->1->2->3
	list := &ListNode{Val: 1, Next: &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3}}}}

	res := reverseList(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func reverseList(head *ListNode) *ListNode {
	if head == nil {
		return nil
	}

	nextPtr := head.Next
	head.Next = nil
	for nextPtr != nil {
		nextNextPtr := nextPtr.Next
		nextPtr.Next = head

		head = nextPtr
		nextPtr = nextNextPtr

		if nextNextPtr != nil {
			nextNextPtr = nextNextPtr.Next
		}
	}
	return head
}

```

## 递归法

假设原链表: `1->2->3->4->nil`

重点在处理最后的最小子问题. 经过递归之后, 划分子问题, 到最后一步: `1->2<-3<-4`, 此时 head 为 `1`.

> Golang

```golang
package main

import "fmt"

func main() {
	//1->2->3
	list := &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3}}}

	res := reverseList(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func reverseList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	newHead := reverseList(head.Next)
	head.Next.Next = head
	head.Next = nil
	return newHead
}

```
