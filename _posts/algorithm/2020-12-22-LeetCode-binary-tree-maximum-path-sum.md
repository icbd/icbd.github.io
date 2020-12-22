---
layout: post
title:  LeetCode binary-tree-maximum-path-sum
date:   2020-12-22
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum/)

题目的要求一条连线, 求连线各个节点的和最大. 连线可以不经过根节点. 没讲清楚的要求是: 连线是一根线, 不能有分叉.

在递归遍历二叉树的基础上, 判断左右节点加入的影响.

* 左则或者右侧, 选择加分最大的一边加入当前方案;
* 把左右和当前节点相加, 跟历史上最大的值比较, 如果更大就更新;
* DFS 返回选择当前节点后的最大值, 来通知上一层做选择;

`max` 需要手动赋值为最小值, `0.size` 得到数字所占的字节数, 乘以 8 得到 位数, 一般为 8*8=64.

* int64 最小值: `-(1<<63)` 即 `-9223372036854775808`
* int64 最大值: `(1<<63) -1` 即 `9223372036854775807`

>Ruby

```ruby
class TreeNode
  attr_accessor :val, :left, :right

  def initialize(val = 0, left = nil, right = nil)
    @val = val
    @left = left
    @right = right
  end
end

# @param {TreeNode} root
# @return {Integer}
def max_path_sum(root)
  @max = -(1 << (0.size * 8 - 1))
  dfs(root)
  @max
end

# @param {TreeNode} root
# @return {Integer}
def dfs(node)
  return 0 if node == nil

  left_sum = dfs(node.left)
  right_sum = dfs(node.right)

  current_sum = node.val + [left_sum, 0].max + [right_sum, 0].max
  @max = current_sum if current_sum > @max

  node.val + [left_sum, right_sum, 0].max
end


r1 = TreeNode.new(1, TreeNode.new(2), TreeNode.new(3))
p max_path_sum(r1) # 6

r2 = TreeNode.new(-10, TreeNode.new(9), TreeNode.new(20, TreeNode.new(15), TreeNode.new(7)))
p max_path_sum(r2) # 42
```

> Golang

```golang
package main

import (
	"fmt"
	"sort"
	"unsafe"
)

func main() {
	// [1,2,3] => 6
	r1 := TreeNode{Val: 1, Left: &TreeNode{Val: 2}, Right: &TreeNode{Val: 3}}
	fmt.Println(maxPathSum(&r1))

	// [-10,9,20,null,null,15,7] => 42
	r2 := TreeNode{Val: -10, Left: &TreeNode{Val: 9}, Right: &TreeNode{Val: 20, Left: &TreeNode{Val: 15}, Right: &TreeNode{Val: 7}}}
	fmt.Println(maxPathSum(&r2))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

var totalMax int

func maxPathSum(root *TreeNode) int {
	totalMax = -(1 << (int(unsafe.Sizeof(int(1)))*8 - 1)) // -(1<<63)
	dfs(root)
	return totalMax
}

func dfs(node *TreeNode) (currentMax int) {
	if node == nil {
		return 0
	}

	leftSum := dfs(node.Left)
	rightSum := dfs(node.Right)

	totalMaxCandidate := node.Val + max(leftSum, 0) + max(rightSum, 0)
	totalMax = max(totalMaxCandidate, totalMax)

	currentMax = node.Val + max(leftSum, rightSum, 0)
	return currentMax
}

func max(nums ...int) int {
	if len(nums) < 1 {
		panic("nums mustn't empty")
	}
	sort.Ints(nums)
	return nums[len(nums)-1]
}

```
