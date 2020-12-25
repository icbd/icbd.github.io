---
layout: post
title:  LeetCode remove-duplicates-from-sorted-list-ii
date:   2020-12-25
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii/)

把请求写入 buffer, 如果新的请求符合条件, 把 buffer 刷入结果集, 清空 buffer 后把新请求值写入 buffer. 最后处理 buffer 遗留.

> Golang

```golang
package main

import "fmt"

func main() {
	//1->1->2->3
	list := &ListNode{Val: 1, Next: &ListNode{Val: 1, Next: &ListNode{Val: 2, Next: &ListNode{Val: 3}}}}

	res := deleteDuplicates(list)
	fmt.Println(res)
}

type ListNode struct {
	Val  int
	Next *ListNode
}

func deleteDuplicates(head *ListNode) *ListNode {
	res := make([]int, 0)
	buffer := make([]int, 0)

	for ; head != nil; head = head.Next {
		if len(buffer) == 0 || head.Val == buffer[0] {
			buffer = append(buffer, head.Val)
			continue
		}

		if len(buffer) == 1 {
			res = append(res, buffer[0])
		}
		buffer = buffer[0:0]
		buffer = append(buffer, head.Val)
	}
	if len(buffer) == 1 {
		res = append(res, buffer[0])
	}

	if len(res) == 0 {
		return nil
	}
	result := &ListNode{Val: res[0]}
	tail := result
	for i := 1; i < len(res); i++ {
		tail.Next = &ListNode{Val: res[i]}
		tail = tail.Next
	}

	return result
}

```
