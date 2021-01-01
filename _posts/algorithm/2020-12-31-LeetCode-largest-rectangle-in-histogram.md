---
layout: post
title:  LeetCode largest-rectangle-in-histogram
date:   2020-12-31
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/largest-rectangle-in-histogram/](https://leetcode-cn.com/problems/largest-rectangle-in-histogram/)

## 暴力模拟

假设目标矩形的高度为当矩形的高度, 求目标矩形的面积, 然后迭代比较得出最大值. Golang 虽然可以过, 但是存在很大的优化空间.

> Golang

```golang
func largestRectangleArea(heights []int) int {
	max := 0
	for i := 0; i < len(heights); i++ {
		width := 1
		for l := i - 1; l >= 0; l-- {
			if heights[l] < heights[i] {
				break
			}
			width++
		}
		for r := i + 1; r < len(heights); r++ {
			if heights[r] < heights[i] {
				break
			}
			width++
		}
		area := heights[i] * width
		if area > max {
			max = area
		}
		//fmt.Printf("i:%d, width:%d\n", i, width)
	}
	return max
}

```

## 单调栈

向左向右取小值的时候可以剪枝, 把不会影响结果的高矩形剪掉, "让当前值一眼就看到小值", 减少向左移动指针的次数.

例如: `1,4,3,2,1`

* `4` 的存在不会影响 `3` 向左找小值;
* `3` 的存在不会影响 `2` 向左找小值;
* `4` 的存在不会影响 `2` 向左找小值;

该单调栈的规则为, 栈顶的值大于等于当前值就丢弃.

> Golang

```golang
package main

import "fmt"

func main() {
	list := []int{2, 1, 5, 6, 2, 3}
	res := largestRectangleArea(list)
	fmt.Println(res)
}

func largestRectangleArea(heights []int) int {
	leftLessIdx := make([]int, len(heights)) // item index of first less than current
	stack := NewStack()                      // item index
	for i := 0; i < len(heights); i++ {
		idx := -1
		for !stack.IsEmpty() {
			if heights[stack.Peak()] < heights[i] {
				idx = stack.Peak()
				break
			} else {
				stack.Pop() // 弹出大于等于当前值的
			}
		}
		stack.Push(i)
		leftLessIdx[i] = idx
	}
	fmt.Println(leftLessIdx)

	rightLessIdx := make([]int, len(heights)) // item index of first less than current
	stack = NewStack()                        // item index
	for i := len(heights) - 1; i >= 0; i-- {
		idx := len(heights)
		for !stack.IsEmpty() {
			if heights[stack.Peak()] < heights[i] {
				idx = stack.Peak()
				break
			} else {
				stack.Pop() // 弹出大于等于当前值的
			}
		}
		stack.Push(i)
		rightLessIdx[i] = idx
	}
	fmt.Println(rightLessIdx)

	max := 0
	for i := 0; i < len(heights); i++ {
		width := rightLessIdx[i] - leftLessIdx[i] - 1
		area := width * heights[i]
		if area > max {
			max = area
		}
	}
	return max
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

func (s *Stack) Peak() int {
	return s.Items[len(s.Items)-1]
}

```
