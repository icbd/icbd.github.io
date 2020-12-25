---
layout: post
title:  LeetCode lowest-common-ancestor-of-a-binary-tree 
date:   2020-12-22
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

## 先从根节点找路径, 再比较最长相同祖先链

* 从根节点出发, 找到指定节点, 记录访问路径;
* 分别找到两条路径, 从左向右找到路径的最大公共部分.

```ruby
# Definition for a binary tree node.
class TreeNode
  attr_accessor :val, :left, :right

  def initialize(val)
    @val = val
    @left, @right = nil, nil
  end
end

# @param {TreeNode} root
# @param {TreeNode} p
# @param {TreeNode} q
# @return {TreeNode}
def lowest_common_ancestor(root, p, q)
  p1 = []
  p2 = []
  dfs(p, root, p1)
  dfs(q, root, p2)

  # p p1, p2

  pub = root.val
  [p1.size, p2.size].min.times do |i|
    pub = p1[i] if p1[i] == p2[i]
  end
  TreeNode.new(pub)
end

# @return bool found
def dfs(target, node, path)
  return false if node == nil

  path << node.val

  return true if node.val == target.val

  return true if dfs(target, node.left, path)

  return true if dfs(target, node.right, path)

  path.pop
  false
end


# [3,5,1,6,2,0,8,null,null,7,4]

n2 = TreeNode.new(2)
n2.left = TreeNode.new(7)
n2.right = TreeNode.new(4)

n5 = TreeNode.new(5)
n5.left = TreeNode.new(6)
n5.right = n2

n1 = TreeNode.new(1)
n1.left = TreeNode.new(0)
n1.right = TreeNode.new(8)

r = TreeNode.new(3)
r.left = n5
r.right = n1

p lowest_common_ancestor(r, n5, TreeNode.new(4))
```

## 划分为子问题递归求解

TIPS: Go方法模板的注释中, TreeNode 有误, 应该为,

```golang
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}
```

>Golang

```golang
package main

import (
	"fmt"
)

func main() {
	//[3,5,1,6,2,0,8,null,null,7,4], p = 5, q = 1

	n0 := &TreeNode{Val: 0}
	n1 := &TreeNode{Val: 1}
	n2 := &TreeNode{Val: 2}
	n3 := &TreeNode{Val: 3}
	n4 := &TreeNode{Val: 4}
	n5 := &TreeNode{Val: 5}
	n6 := &TreeNode{Val: 6}
	n7 := &TreeNode{Val: 7}
	n8 := &TreeNode{Val: 8}

	n3.Left = n5
	n3.Right = n1
	n5.Left = n6
	n5.Right = n2
	n2.Left = n7
	n2.Right = n4
	n1.Left = n0
	n1.Right = n8

	p := n5
	q := n4
	fmt.Println(lowestCommonAncestor(n3, p, q))
}

type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

var targetP *TreeNode
var targetQ *TreeNode

func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
	targetP = p
	targetQ = q
	return dfs(root)
}

func dfs(current *TreeNode) (lca *TreeNode) {
	// 到底也没找到节点
	if current == nil {
		return nil
	}

	// 找到了一个目标节点, 则不需要继续往下寻找;
	// LCA 为当前节点, 或者该节点的祖先节点;
	// 默认一定有解.
	if current == targetQ || current == targetP {
		return current
	}

	leftCA := dfs(current.Left)
	rightCA := dfs(current.Right)

	// p q 分布在两边, 那么当前节点就是 LCA
	if leftCA != nil && rightCA != nil {
		return current
	}

  // 哪边有结果, 就把哪边的结果往上传, **默认一定有解**.
  // 返回子问题的结果.
	if leftCA != nil {
		return leftCA
	}
	return rightCA
}

```
