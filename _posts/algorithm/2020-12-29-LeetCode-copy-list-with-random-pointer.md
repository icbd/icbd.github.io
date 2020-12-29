---
layout: post
title:  LeetCode copy-list-with-random-pointer
date:   2020-12-29
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/copy-list-with-random-pointer/](https://leetcode-cn.com/problems/copy-list-with-random-pointer/)

提取 `fetch` 方法只负责节点的产生, 在主方法中构建关联关系.

> Golang

```golang
// [originalNodePtr](copiedNodePtr)
var mapping map[*Node]*Node

func copyRandomList(head *Node) *Node {
	if head == nil {
		return head
	}

	mapping = make(map[*Node]*Node, 0)

	resTail := &Node{Val: -1}
	resHead := resTail
	for tail := head; tail != nil; tail = tail.Next {
		node := fetch(tail)
		node.Random = fetch(tail.Random)
		resTail.Next = node
		resTail = resTail.Next
	}
	return resHead.Next
}

// fetch or set
func fetch(originNode *Node) (copyNode *Node) {
	if originNode == nil {
		return nil
	}
	node, ok := mapping[originNode]
	if ok {
		return node
	}
	copyNode = &Node{Val: originNode.Val}
	mapping[originNode] = copyNode
	return copyNode
}

```
