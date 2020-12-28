---
layout: post
title:  LeetCode sort-list
date:   2020-12-27
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/sort-list/](https://leetcode-cn.com/problems/sort-list/)

把链表转化为数组, 排好顺序再恢复为链表.

> Golang

```golang
func sortList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}

	list := make([]int, 0)
	for head != nil {
		list = append(list, head.Val)
		head = head.Next
	}
	sort.Ints(list)
	result := &ListNode{Val: list[0]}
	tail := result
	for i := 1; i < len(list); i++ {
		tail.Next = &ListNode{Val: list[i]}
		tail = tail.Next
	}
	return result
}
```

## 快速排序

> Ruby

```ruby
  def quick_sort
    return [] if self.empty?

    center, *rest = self
    left, right = rest.partition { |ele| ele < center }
    left.quick_sort + [center] + right.quick_sort
  end
```

> Golang

```golang

func sortArray(nums []int) []int {
	quickSort(nums, 0, len(nums)-1)
	return nums
}

func quickSort(nums []int, leftIdx int, rightIdx int) {
	if len(nums) <= 1 {
		return
	}
	middelIdx := partition(nums, leftIdx, rightIdx)
	quickSort(nums, middelIdx+1, rightIdx)
	quickSort(nums, leftIdx, middelIdx-1)
}
```

原地 partition 也是一个重点, 参数要求 `[left, right]`, 返回迭代完的中间值的位置, 方便下次迭代

### partition one

以最左边元素为基准.

从左到右, 使用慢索引记录已经被交换的 small 值位置, 以后每遇到一个 small, 就跟慢索引的位置交换. 最后把基准和慢索引交换.

> Golang

```golang
func partition(nums []int, left, right int) (middleIndex int) {
	target := nums[left]
	smallPtr := left
	for i := left + 1; i <= right; i++ {
		if nums[i] < target {
			smallPtr += 1
			nums[smallPtr], nums[i] = nums[i], nums[smallPtr]
		}
	}
	nums[smallPtr], nums[left] = nums[left], nums[smallPtr]
	return smallPtr
}
```

### partition two

> Golang

```golang
func partition(nums []int, left, right int) (middleIndex int) {
	target := nums[left]
	for left < right {
		for left < right && nums[right] > target {
			right -= 1
		}
		nums[left] = nums[right] // 从右边找第一个小于 target 的值, 把小值放到最左边
		for left < right && nums[left] < target {
			left += 1
		}
		nums[right] = nums[left] // 从左边找第一个大于 target 的值, 把大值放到刚才小值空缺的位置
	}
	
	nums[left] = target // left 从来没被赋值, 他正好也是中间值的位置
	return left
}
```

## 归并排序

> Ruby

```ruby
# 归并排序 O(nlogn)
  def merge_sort
    return self if size == 1
    left = self[0, size/2]
    right = self[size/2, size - size/2]
    Array.merge(left.merge_sort, right.merge_sort)
  end

  def self.merge(left, right)
    ans = []
    until left.empty? or right.empty?
      smaller = left.first < right.first ? left.shift : right.shift
      ans.push(smaller)
    end

    ans + left + right
  end
```

`mergeTwoLists` 可以利用 [https://leetcode-cn.com/problems/merge-two-sorted-lists/](https://leetcode-cn.com/problems/merge-two-sorted-lists/) 已经写好的方法.

> Golang

```golang
func sortList(head *ListNode) *ListNode {
	if head == nil || head.Next == nil {
		return head
	}
	// 以下至少两个节点
	one := head // not nil
	oneTail := one
	head = head.Next
	two := head // not nil
	twoTail := two
	head = head.Next
	for head != nil {
		oneTail.Next = head
		oneTail = oneTail.Next
		head = head.Next

		if head != nil {
			twoTail.Next = head
			head = head.Next
			twoTail = twoTail.Next
		}
	}
	oneTail.Next = nil
	twoTail.Next = nil
	return mergeTwoLists(sortList(one), sortList(two))
}

func mergeTwoLists(l1 *ListNode, l2 *ListNode) *ListNode {
	result := &ListNode{Val: -1 << 63}
	tail := result
	for l1 != nil && l2 != nil {
		if l1.Val < l2.Val {
			tail.Next = l1
			l1 = l1.Next
		} else {
			tail.Next = l2
			l2 = l2.Next
		}
		tail = tail.Next
	}
	if l1 != nil {
		tail.Next = l1
	}

	if l2 != nil {
		tail.Next = l2
	}
	return result.Next
}

```
