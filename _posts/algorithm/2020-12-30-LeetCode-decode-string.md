---
layout: post
title:  LeetCode decode-string
date:   2020-12-30
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/decode-string/](https://leetcode-cn.com/problems/decode-string/)

数字可能有多位.

> Golang

```golang
func decodeString(s string) string {
	countStack := NewStack()
	strStack := NewStack()
	for i := 0; i < len(s); i++ {
		currentChar := string(s[i])
		if currentChar == "]" {
			tempStr := ""
			for {
				stackStr := strStack.Pop()
				if stackStr == "[" {
					break
				}
				tempStr = stackStr + tempStr // 注意拼接顺序
			}

			count := countStack.PopInt()
			tempRes := ""
			for count > 0 {
				tempRes += tempStr
				count--
			}
			strStack.Push(tempRes)
			continue
		}

		if _, err := strconv.Atoi(currentChar); err == nil { // is a number, maybe > 9
			num, count := maxNumString(s[i:])
			countStack.Push(num)
			i += count - 1
		} else {
			strStack.Push(currentChar) // contains `[`
		}
	}
	res := ""
	for !strStack.IsEmpty() {
		res = strStack.Pop() + res
	}
	return res
}

func maxNumString(s string) (res string, count int) {
	for i := 0; i < len(s); i++ {
		if n, err := strconv.Atoi(s[0 : i+1]); err != nil {
			return res, i
		} else {
			res = strconv.Itoa(n)
		}
	}
	return res, len(s)
}

type Stack struct {
	Items []string
}

func NewStack() *Stack {
	return &Stack{
		Items: make([]string, 0),
	}
}

func (s *Stack) IsEmpty() bool {
	return len(s.Items) == 0
}

func (s *Stack) Push(x string) {
	s.Items = append(s.Items, x)
}

func (s *Stack) Pop() string {
	if len(s.Items) == 0 {
		panic("pop empty stack")
	}

	item := s.Items[len(s.Items)-1]
	s.Items = s.Items[0 : len(s.Items)-1]
	return item
}

func (s *Stack) PopInt() int {
	num, err := strconv.Atoi(s.Pop())
	if err != nil {
		panic(err)
	}
	return num
}

```
