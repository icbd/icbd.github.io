---
layout: post
title:  LeetCode implement-queue-using-stacks
date:   2020-12-31
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/implement-queue-using-stacks/](https://leetcode-cn.com/problems/implement-queue-using-stacks/)

go 的 `Stack` 需要自己手动实现.

> Golang

```golang

type MyQueue struct {
	S1 *Stack
	S2 *Stack
}

func Constructor() MyQueue {
	return MyQueue{S1: NewStack(), S2: NewStack()}
}

func (q *MyQueue) Push(x int) {
	q.S1.Push(x)
}

func (q *MyQueue) Pop() int {
	if !q.S2.IsEmpty() {
		return q.S2.Pop()
	}

	if q.S1.IsEmpty() {
		panic("queue empty")
	}

	for !q.S1.IsEmpty() {
		q.S2.Push(q.S1.Pop())
	}
	return q.S2.Pop()
}

func (q *MyQueue) Peek() int {
	item := q.Pop()
	q.S2.Push(item)
	return item
}

func (q *MyQueue) Empty() bool {
	return q.S2.IsEmpty() && q.S1.IsEmpty()
}

type Stack struct {
	Items []int
}

func NewStack() *Stack {
	return &Stack{
		Items: make([]int, 0),
	}
}

func (s *Stack) IsEmpty() bool {
	return len(s.Items) == 0
}

func (s *Stack) Push(x int) {
	s.Items = append(s.Items, x)
}

func (s *Stack) Pop() int {
	if len(s.Items) == 0 {
		panic("pop empty stack")
	}

	item := s.Items[len(s.Items)-1]
	s.Items = s.Items[0 : len(s.Items)-1]
	return item
}

```
