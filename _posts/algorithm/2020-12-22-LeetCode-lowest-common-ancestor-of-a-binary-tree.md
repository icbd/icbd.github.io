---
layout: post
title:  LeetCode lowest-common-ancestor-of-a-binary-tree 
date:   2020-12-22
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree/)

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
