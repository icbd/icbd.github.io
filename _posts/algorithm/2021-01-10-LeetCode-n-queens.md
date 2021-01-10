---
layout: post
title:  LeetCode n-queens N皇后问题
date:   2021-01-10
Author: CBD
tags:   [LeetCode, Backtrace]
---

[https://leetcode-cn.com/problems/n-queens/](https://leetcode-cn.com/problems/n-queens/)

这是一个典型的回溯问题, 很容易写出伪代码:

```text
def solution(path)
  if isCompleted(path)
    result << path
    return
  end

  selections.each do |selection|
    path << selection
    solution(path)
    path.pop
  end
end
```

很明显, 处理几个条件都涉及到 path 的遍历, 复杂度很高, 再继续观察简化问题的思路.

`Q` 代表 Queen, `B` 代表空白.

|y\x|0|1|2|3|
|---|---|---|---|---|
|0|B|Q|B|B|
|1|B|B|B|Q|
|2|Q|B|B|B|
|3|B|B|Q|B|

N 个 Q 放在 N x N 的棋盘上:

* 令每个 Q 在横行上不相遇, 那么可知每行有且仅有一个 Q (y 坐标相同);
* 令每个 Q 在竖列上不相遇, 那么可知每列有且仅有一个 Q (x 坐标相同);
* 令每个 Q 在斜线上不相遇, 由于斜线数多于行列数, 不需要保证一定存在, 但如果存在就是唯一的;
* 从左上指向右下方的斜线上, x - y 的差值相同;
* 从右下指向左上方的斜线上, x + y 的和相同;

经过以上的规律, 我们可以把 path 从二位数组简化到一维数组.

以上面 4x4 的棋盘为例, 转为一维数组就是 `[2,0,3,1]`, 表示在第 x 列的 Queen 的 y 值.

按照从左到右的顺序安排, 第一列的 Queen 可以随便放, 有 4 种方式, 那么第二列的 Queen 只能从剩下的三个行中选一个, 也就是 3 种, 以此类推, 所有的可能就是 `4 * 3 * 2 * 1 = 4!` , 时间复杂度即 `n!` . 所以题目限制了 `1 <= n <= 9` .

> Golang

```golang
package main

import (
	"fmt"
)

func main() {
	res := solveNQueens(4)
	fmt.Println(res)
}

const UNVISITED = 0

var results [][]int

var yVisited map[int]bool
var toTopRightVisited map[int]bool
var toBottomRightVisited map[int]bool

// 1<=n<=9
func solveNQueens(n int) [][]string {
	results = make([][]int, 0)

	yVisited = make(map[int]bool)
	toTopRightVisited = make(map[int]bool)
	toBottomRightVisited = make(map[int]bool)

	path := make([]int, n)
	helper(path, 0)

	// format
	solutions := make([][]string, len(results))
	for i := 0; i < len(results); i++ {
		//fmt.Println(results[i])
		solutions[i] = formatSolution(results[i])
	}
	return solutions
}

func helper(path []int, queenIdx int) {
	n := len(path)
	if queenIdx == n {
		temp := make([]int, n)
		copy(temp, path)
		results = append(results, temp)
		return
	}

	for y := 0; y < n; y++ {
		topRight := queenIdx + y
		topBottom := queenIdx - y
		if yVisited[y] || toTopRightVisited[topRight] || toBottomRightVisited[topBottom] {
			continue
		}

		// push to path
		path[queenIdx] = y
		yVisited[y] = true
		toTopRightVisited[topRight] = true
		toBottomRightVisited[topBottom] = true
		queenIdx++

		helper(path, queenIdx)

		// pop from path
		queenIdx--
		path[queenIdx] = UNVISITED
		yVisited[y] = false
		toTopRightVisited[topRight] = false
		toBottomRightVisited[topBottom] = false
	}
}

// 由于 NxN 的棋盘是正方形的, 旋转 90 度不影响结果的正确性.
// 为了方便在输出结果的时候调换一下 xy 的顺序
func formatSolution(result []int) []string {
	n := len(result)
	formatted := make([]string, n)

	for y := 0; y < n; y++ {
		line := make([]byte, n)
		for x := 0; x < n; x++ {
			if x == result[y] {
				line[x] = 'Q'
			} else {
				line[x] = '.'
			}
		}
		formatted[y] = string(line)
	}

	return formatted
}

```
