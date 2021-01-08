---
layout: post
title:  LeetCode swap-nodes-in-pairs
date:   2021-01-08
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/swap-nodes-in-pairs/](https://leetcode-cn.com/problems/swap-nodes-in-pairs/)

按照新链的从后向前赋值.

> Golang

```golang

func swapPairs(head *ListNode) *ListNode {
	resultHead := &ListNode{Val: -1, Next: head}
	tail := resultHead
	for tail != nil {
		tail = helper(tail)
	}
	return resultHead.Next
}

// return next start node
func helper(head *ListNode) *ListNode {
	if head == nil || head.Next == nil || head.Next.Next == nil {
		return nil
	}
	tempFirst := head.Next
	tempSecond := head.Next.Next

	tempFirst.Next = tempSecond.Next
	tempSecond.Next = tempFirst
	head.Next = tempSecond

	return head.Next.Next
}

```
