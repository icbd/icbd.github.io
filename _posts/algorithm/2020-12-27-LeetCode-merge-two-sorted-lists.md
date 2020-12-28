---
layout: post
title:  LeetCode merge-two-sorted-lists
date:   2020-12-27
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/merge-two-sorted-lists/](https://leetcode-cn.com/problems/merge-two-sorted-lists/)

注意 `l1` `l2` 可能有剩余节点.

> Golang

```golang
func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	result := &ListNode{Val: -1 << 63}
	tail := result
	for l1 != nil && l2 != nil {
		if l1.Val < l2.Val {
			tail.Next = l1
			l1 = l1.Next
		} else {
			tail.Next = l2
			l2 = l2.Next
		}
		tail = tail.Next
	}
	if l1 != nil {
		tail.Next = l1
	}

	if l2 != nil {
		tail.Next = l2
	}
	return result.Next
}

```
