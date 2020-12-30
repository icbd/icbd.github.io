---
layout: post
title:  LeetCode evaluate-reverse-polish-notation
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation/)

加法和乘法可以使用交换律, 减法和除法都有顺序要求.

> Golang

```golang
package main

import (
	"fmt"
	"strconv"
)

func main() {
	list := []string{"10", "6", "9", "3", "+", "-11", "*", "/", "*", "17", "+", "5", "+"} // 22
	//list := []string{"4", "13", "5", "/", "+"} // 6
	res := evalRPN(list)
	fmt.Println(res)
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

func evalRPN(tokens []string) int {
	stack := NewStack()
	for i := 0; i < len(tokens); i++ {
		switch tokens[i] {
		case "+":
			stack.Push(stack.Pop() + stack.Pop())
		case "*":
			stack.Push(stack.Pop() * stack.Pop())
		case "-":
			subtract := stack.Pop() // 减数
			stack.Push(stack.Pop() - subtract)
		case "/":
			divider := stack.Pop() // 除数
			stack.Push(stack.Pop() / divider)
		default:
			if n, err := strconv.Atoi(tokens[i]); err != nil {
				panic("Atoi failed")
			} else {
				stack.Push(n)
			}
		}
	}
	return stack.Pop()
}

```
