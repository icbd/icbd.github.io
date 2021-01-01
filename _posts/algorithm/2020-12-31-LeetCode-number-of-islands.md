---
layout: post
title:  LeetCode number-of-islands
date:   2020-12-31
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/number-of-islands/](https://leetcode-cn.com/problems/number-of-islands/)

## DFS

一旦找到一个 ISLAND, 就把他变为 SEA, 然后 DFS 搜索与他相连的陆地, 依次也反转为 SEA.

特别注意, `matrix[a][b]` a 为行号, b 为 列号. 与坐标系中的 x y 相反. 也可以反着定义 xy 来让 `matrix[x][y]` 看起来更符合直觉, 或者包装新的对象.

> Golang

```golang
package main

import "fmt"

func main() {
	grid := [][]byte{
		{'1', '1', '1', '1', '0'},
		{'1', '1', '0', '1', '0'},
		{'1', '1', '0', '0', '0'},
		{'0', '0', '0', '0', '0'},
	}
	res := numIslands(grid)
	fmt.Println(res)
}

const ISLAND = '1'
const SEA = '0'

func numIslands(grid [][]byte) int {
	if len(grid) == 0 {
		return 0
	}

	xCount := len(grid[0])
	yCount := len(grid)

	island := 0
	for x := 0; x < xCount; x++ {
		for y := 0; y < yCount; y++ {
			if grid[y][x] == ISLAND {
				island++
				pollute(&grid, x, y)
			}
		}
	}

	return island
}

func pollute(grid *[][]byte, x, y int) {
	xCount := len((*grid)[0])
	yCount := len(*grid)

	(*grid)[y][x] = SEA // pollute

	// diffuse
	if x-1 >= 0 && (*grid)[y][x-1] == ISLAND {
		pollute(grid, x-1, y)
	}
	if x+1 < xCount && (*grid)[y][x+1] == ISLAND {
		pollute(grid, x+1, y)
	}
	if y-1 >= 0 && (*grid)[y-1][x] == ISLAND {
		pollute(grid, x, y-1)
	}
	if y+1 < yCount && (*grid)[y+1][x] == ISLAND {
		pollute(grid, x, y+1)
	}
}

```

## 并查集

本质上也是 DFS, 只是统计计算结果有差别. 如果是 Island 就加入一个集合, 并把他相连的都加入同一个集合. 如果两个 island 有联通, 就合并他们. 最后 island 的数目即并查集的集合个数.

> Golang

```golang

type Point struct {
	X int
	Y int
}

type Union map[Point]bool   //set
type Unions map[*Union]bool // set

func (us Unions) Count() int {
	for union, _ := range us {
		if len(*union) == 0 {
			delete(us, union)
		}
	}
	return len(us)
}

func (us Unions) Merge(u1, u2 Union) Union {
	for k, v := range u2 {
		u1[k] = v
	}
	delete(us, &u2)
	return u1
}

func (us Unions) PushAlone(p Point) {
	newUnion := make(Union)
	newUnion[p] = true
	us[&newUnion] = true
	return
}

func (us Unions) Push(p1, p2 Point) {
	var p1Union, p2Union Union
	for union, _ := range us {
		if _, ok := (*union)[p1]; ok {
			p1Union = *union
		}
		if _, ok := (*union)[p2]; ok {
			p2Union = *union
		}
	}

	if p2Union != nil && p1Union != nil {
		us.Merge(p2Union, p1Union)
		return
	}

	if p2Union == nil && p1Union == nil {
		newUnion := make(Union)
		newUnion[p1] = true
		newUnion[p2] = true
		us[&newUnion] = true
		return
	}

	if p2Union != nil {
		p2Union[p1] = true
	}

	if p1Union != nil {
		p1Union[p2] = true
	}
}

const ISLAND = '1'
const SEA = '0'

func numIslands(grid [][]byte) int {
	if len(grid) == 0 {
		return 0
	}
	unions := make(Unions)

	xCount := len(grid[0])
	yCount := len(grid)

	for x := 0; x < xCount; x++ {
		for y := 0; y < yCount; y++ {
			if grid[y][x] == ISLAND {
				point := Point{X: x, Y: y}
				unions.PushAlone(point)
				unionIsland(&grid, unions, point)
			}
		}
	}

	return unions.Count()
}

func unionIsland(grid *[][]byte, unions Unions, p Point) {
	xCount := len((*grid)[0])
	yCount := len(*grid)

	x := p.X
	y := p.Y
	(*grid)[y][x] = SEA

	// diffuse
	if x-1 >= 0 && (*grid)[y][x-1] == ISLAND {
		nP := Point{X: x - 1, Y: y}
		unions.Push(p, nP)
		unionIsland(grid, unions, nP)
	}
	if x+1 < xCount && (*grid)[y][x+1] == ISLAND {
		nP := Point{X: x + 1, Y: y}
		unions.Push(p, nP)
		unionIsland(grid, unions, nP)
	}
	if y-1 >= 0 && (*grid)[y-1][x] == ISLAND {
		nP := Point{X: x, Y: y - 1}
		unions.Push(p, nP)
		unionIsland(grid, unions, nP)
	}
	if y+1 < yCount && (*grid)[y+1][x] == ISLAND {
		nP := Point{X: x, Y: y + 1}
		unions.Push(p, nP)
		unionIsland(grid, unions, nP)
	}
}

```
