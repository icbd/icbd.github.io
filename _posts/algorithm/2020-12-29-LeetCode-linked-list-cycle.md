---
layout: post
title:  LeetCode linked-list-cycle
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/linked-list-cycle/](https://leetcode-cn.com/problems/linked-list-cycle/)

`*ListNode` 是可比较的类型, 可用在 map 的键, 如果链表中重复出现该指针, 则一定存在环.
注意不要用 `Val` 作为键, 链表中可能存在重复的 Val.

> Golang

```golang
func hasCycle(head *ListNode) bool {
	hash := make(map[*ListNode]int, 0) // [*ListNode] => list index

	tail := head
	for i := 0; tail != nil; i++ {
		if _, ok := hash[tail]; ok {
			return true
		}
		hash[tail] = i
		tail = tail.Next
	}
	return false
}
```
