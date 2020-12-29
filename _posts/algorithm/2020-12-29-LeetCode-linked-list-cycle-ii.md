---
layout: post
title:  LeetCode linked-list-cycle-ii
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/linked-list-cycle-ii/](https://leetcode-cn.com/problems/linked-list-cycle-ii/)

> Golang

```golang
package main

import "fmt"

func main() {
	node := &ListNode{Val: 999}
	list := &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3, Next: node}}}
	node.Next = list
	res := detectCycle(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func detectCycle(head *ListNode) *ListNode {
	hash := make(map[*ListNode]int, 0) // [*ListNode] => list index

	tail := head
	for i := 0; tail != nil; i++ {
		if _, ok := hash[tail]; ok {
			return tail
		}
		hash[tail] = i
		tail = tail.Next
	}
	return nil
}

```
