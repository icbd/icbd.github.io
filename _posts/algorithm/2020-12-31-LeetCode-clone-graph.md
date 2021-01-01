---
layout: post
title:  LeetCode clone-graph
date:   2020-12-31
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/clone-graph/](https://leetcode-cn.com/problems/clone-graph/)

`fetch` 只负责克隆 `Val`, 由主函数递归的串联节点之间的关系.

> Golang

```golang
package main

import "fmt"

func main() {
	n1 := &Node{Val: 1}
	n2 := &Node{Val: 2}
	n3 := &Node{Val: 3}
	n4 := &Node{Val: 4}
	n1.Neighbors = []*Node{n4, n2}
	n2.Neighbors = []*Node{n1, n3}
	n3.Neighbors = []*Node{n2, n4}
	n4.Neighbors = []*Node{n3, n1}

	res := cloneGraph(n1)
	fmt.Println(res)
}

type Node struct {
	Val       int
	Neighbors []*Node
}

func cloneGraph(node *Node) *Node {
	if node == nil {
		return nil
	}

	copiedNode := fetch(node)
	if copiedNode.Neighbors == nil {
		copiedNode.Neighbors = make([]*Node, len(node.Neighbors))
		for i := 0; i < len(node.Neighbors); i++ {
			copiedNeighbor := cloneGraph(node.Neighbors[i])
			copiedNode.Neighbors[i] = copiedNeighbor
		}
	}
	return copiedNode
}

var mapping map[*Node]*Node

func fetch(node *Node) *Node {
	if mapping == nil {
		mapping = make(map[*Node]*Node)
	}

	if copyNode, ok := mapping[node]; ok {
		return copyNode
	} else {
		copyNode = &Node{Val: node.Val}
		mapping[node] = copyNode
		return copyNode
	}
}

```
