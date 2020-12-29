---
layout: post
title:  LeetCode palindrome-linked-list
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/palindrome-linked-list/](https://leetcode-cn.com/problems/palindrome-linked-list/)

> Golang

```golang
package main

import "fmt"

func main() {
	list := &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 2, Next: &ListNode{Val: 1}}}}
	res := isPalindrome(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func isPalindrome(head *ListNode) bool {
	if head == nil {
		return true
	}
	list := make([]int, 0)
	for tail := head; tail != nil; tail = tail.Next {
		list = append(list, tail.Val)
	}
	half := (len(list) + 1) / 2
	for i := 0; i < half; i++ {
		if list[i] != list[len(list)-1-i] {
			return false
		}
	}
	return true
}

```
