---
layout: post
title:  LeetCode subsets
date:   2020-12-21
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/subsets](https://leetcode-cn.com/problems/subsets)

## combination

`Array#combination` 内置的组合可以直接解出答案.
在 nums 中, 取 n 个数的组合. n 取 0 即空数组, n 最大取 `nums.size` 个.

> Ruby

```ruby
# @param {Integer[]} nums
# @return {Integer[][]}
def subsets(nums)
  results = []
  count = nums.size + 1
  count.times do |i|
    nums.combination(i).each { |item| results << item }
  end
  results
end

# res = subsets([1, 2, 3])
# p res, res.size

```

## 巧妙遍历

```text
[]
[], [1]
[], [1], [2], [1, 2]
[], [1], [2], [1, 2], [3], [1, 3], [2, 3], [1, 2, 3]
```

新的一次操作在上一次的基础上, 原样复制一份, 再在另外的一份加入n, 合并.

注意 Ruby `Array#clone` 是浅拷贝, 不能用在这里.

> Ruby

```ruby
# @param {Integer[]} nums
# @return {Integer[][]}
def subsets(nums)
  results = [[]]

  nums.each do |n|
    current_results_count = results.size
    current_results_count.times do |idx|
      results << results[idx] + [n]
    end
  end

  results
end

# res = subsets([1, 2, 3])
# p res, res.size
```

## 回溯法

特别注意 `path`, 他会在回溯的过程中改变, 赋值时应该使用他的拷贝值.

> Ruby

```ruby
# @param {Integer[]} nums
# @return {Integer[][]}
def subsets(nums)
  @results = []
  @nums = nums

  dfs([], 0)

  @results
end

def dfs(path, num_idx)
  @results << path + [] # generate new object, imitate deep array clone

  (num_idx...@nums.size).to_a.each do |i|
    path.push(@nums[i])
    dfs(path, i + 1)
    path.pop
  end
end

# res = subsets([1, 2, 3])
# p res, res.size

```

> Golang

```golang
package main

import "fmt"

func main() {
	fmt.Println(subsets([]int{1, 2, 3}))
}

func subsets(nums []int) [][]int {
	results := make([][]int, 0)
	path := make([]int, 0)
	dfs(path, 0, &results, &nums)
	return results
}

func dfs(path []int, idx int, results *[][]int, nums *[]int) {
	pathClone := make([]int, len(path))
	copy(pathClone, path)
	*results = append(*results, pathClone) // use cloned array

	for i := idx; i < len(*nums); i++ {
		path = append(path, (*nums)[i]) // push
		dfs(path, i+1, results, nums)
		path = path[0 : len(path)-1] // pop
	}
}

```
