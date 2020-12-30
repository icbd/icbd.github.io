---
layout: post
title:  LeetCode min-stack
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/min-stack/](https://leetcode-cn.com/problems/min-stack/)

题目的关键在 `GetMin`, 他要求获得当前栈中的最小值, 虽然是动态的, 但是可以通过增加一个辅助队列来记录当时的最小值.

> Golang

```golang
package main

import "fmt"

func main() {
	obj := Constructor()
	obj.Push(-2)
	obj.Push(0)
	obj.Push(-3)
	fmt.Println(obj.GetMin()) // -3
	obj.Pop()
	fmt.Println(obj.Top())    // 0
	fmt.Println(obj.GetMin()) //-2
}

type MinStack struct {
	OrderItems []int
	MinItems   []int
}

func Constructor() MinStack {
	return MinStack{
		OrderItems: make([]int, 0),
		MinItems:   make([]int, 0),
	}
}

func (s *MinStack) Push(x int) {
	s.OrderItems = append(s.OrderItems, x)

	if len(s.MinItems) == 0 {
		s.MinItems = append(s.MinItems, x)
	} else {
		currentMin := s.GetMin()
		if x < currentMin {
			s.MinItems = append(s.MinItems, x)
		} else {
			s.MinItems = append(s.MinItems, currentMin)
		}
	}
}

func (s *MinStack) Pop() {
	if len(s.OrderItems) == 0 {
		return
	}
	s.OrderItems = s.OrderItems[0 : len(s.OrderItems)-1]
	s.MinItems = s.MinItems[0 : len(s.MinItems)-1]
}

func (s *MinStack) Top() int {
	if len(s.OrderItems) == 0 {
		return 0
	}
	return s.OrderItems[len(s.OrderItems)-1]
}

func (s *MinStack) GetMin() int {
	if len(s.MinItems) == 0 {
		return 0
	}
	return s.MinItems[len(s.MinItems)-1]
}

```
