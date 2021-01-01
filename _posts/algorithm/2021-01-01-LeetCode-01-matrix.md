---
layout: post
title:  LeetCode 01-matrix
date:   2021-01-01
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/01-matrix/](https://leetcode-cn.com/problems/01-matrix/)

图 BFS 比树的 BFS 复杂, 关键在两点:

* 有多个搜索入口
* 没有搜索顺序, 经常需要标记是否搜索过

对于这个例子, 本来就是 `0` 的位置距离为 0, 然后把他们都入队.

接下来依次从队列中取出元素开始新一轮 BFS, 这一轮搜索会得到距离加一的所有节点. 标记已经访问过, 然后依次再 BFS 搜索, 知道队列为空.

因为需要把距离相同的元素都访问完再访问下一批距离加一的节点, 所以选用队列而不是栈. 使用栈会得到错误结果, 导致结果不是最小距离.

> Golang

```golang
package main

import "fmt"

func main() {
	matrix := [][]int{
		{0, 0, 0},
		{0, 1, 0},
		{1, 1, 1},
	}
	res := updateMatrix(matrix)
	fmt.Println(res)
}

type Point struct {
	X int
	Y int
}

const ZERO = 0
const UNVISITED = -1

func updateMatrix(matrix [][]int) [][]int {
	if matrix == nil || len(matrix) == 0 {
		return matrix
	}

	searchQueue := NewQueue()
	yCount := len(matrix)
	xCount := len(matrix[0])

	// 每个 zero 都是 BFS 的起点, 把起点都入队
	for y := 0; y < yCount; y++ {
		for x := 0; x < xCount; x++ {
			if matrix[y][x] == ZERO {
				searchQueue.Push(Point{X: x, Y: y})
			} else {
				matrix[y][x] = UNVISITED
			}
		}
	}

	// BFS 直到队列为空
	for !searchQueue.IsEmpty() {
		point := searchQueue.Pop()
		x := point.X
		y := point.Y
		if x+1 < xCount && matrix[y][x+1] == UNVISITED {
			matrix[y][x+1] = matrix[y][x] + 1
			searchQueue.Push(Point{X: x + 1, Y: y})
		}
		if x-1 >= 0 && matrix[y][x-1] == UNVISITED {
			matrix[y][x-1] = matrix[y][x] + 1
			searchQueue.Push(Point{X: x - 1, Y: y})
		}
		if y+1 < yCount && matrix[y+1][x] == UNVISITED {
			matrix[y+1][x] = matrix[y][x] + 1
			searchQueue.Push(Point{X: x, Y: y + 1})
		}
		if y-1 >= 0 && matrix[y-1][x] == UNVISITED {
			matrix[y-1][x] = matrix[y][x] + 1
			searchQueue.Push(Point{X: x, Y: y - 1})
		}
	}
	return matrix
}

type Queue struct {
	Items []Point
}

func NewQueue() *Queue {
	return &Queue{
		Items: make([]Point, 0),
	}
}

func (s *Queue) IsEmpty() bool {
	return len(s.Items) == 0
}

func (s *Queue) Push(x Point) {
	s.Items = append(s.Items, x)
}

func (s *Queue) Pop() Point {
	if len(s.Items) == 0 {
		panic("pop empty queue")
	}

	item := s.Items[0]
	s.Items = s.Items[1:len(s.Items)]
	return item
}

```
