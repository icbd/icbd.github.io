---
layout: post
title:  Implement an LRU Cache with Ruby
date:   2021-01-17
Author: CBD
tags:   [Ruby, LRU]
---

LRU: `Least Recently Used` 最近最少使用. 剔除未使用时间最长的元素.

**假设最近使用的元素接下来还会被使用.**

用 Hash 维护 `key => Node Pointer`.

`Node` 用双向链表串联. 读取和插入都能在 `O(1)` 完成.

> Ruby

```ruby
require 'byebug'

module LRU
  class Node
    attr_accessor :key, :value, :last_node, :next_node

    def initialize(key, value = -1, last_node = nil, next_node = nil)
      @key = key
      @value = value
      @last_node = last_node
      @next_node = next_node
    end
  end

  class DoubleLinked
    attr_reader :head, :tail, :size

    def initialize
      @head = Node.new("head")
      @tail = Node.new("tail")
      @head.next_node = @tail
      @tail.last_node = @head

      @size = 0
    end

    # shift node to head
    # @return size
    def shift(node)
      second = head.next_node
      second.last_node = node

      node.next_node = second
      node.last_node = head

      head.next_node = node

      @size += 1
    end

    # pop from tail
    # @return node / nil
    def pop
      return if size == 0

      deviate(@tail.last_node)
    end

    # deviate from DoubleLinked
    # @return node
    def deviate(node)
      node.last_node.next_node = node.next_node
      node.next_node.last_node = node.last_node
      @size -= 1
      node
    end
  end
end

class LRUCache
  include LRU

  attr_reader :capacity, :table

  def initialize(capacity)
    @capacity = capacity
    @table = {} # key => Node
    @linked = LRU::DoubleLinked.new
  end

  # @return int value
  def get(key)
    node = touch(key)
    node&.value
  end

  # @return node
  def put(key, value)
    node = touch(key)
    if node != nil
      node.value = value
      return node
    end

    node = Node.new(key, value)
    cut_lru
    @linked.shift(node)
    @table[key] = node
    node
  end

  private

  def touch(key)
    node = table[key]
    return if node.nil?

    @linked.deviate(node)
    @linked.shift(node)

    node
  end

  def cut_lru
    while @linked.size >= @capacity
      node = @linked.pop
      @table.delete(node.key)
    end
  end
end


require 'rspec'
RSpec.describe LRU do
  context 'DoubleLinked' do
    before(:each) do
      @linked = LRU::DoubleLinked.new
      @n1 = LRU::Node.new("n1", 1)
      @n2 = LRU::Node.new("n2", 2)
      @n3 = LRU::Node.new("n3", 3)
      @linked.shift(@n1)
      @linked.shift(@n2)
      @linked.shift(@n3)
    end

    it '#size' do
      expect(@linked.size).to eq 3
    end

    it 'ordered' do
      node = @linked.head.next_node
      expect(node.value).to eq 3
      node = node.next_node
      expect(node.value).to eq 2
      node = node.next_node
      expect(node.value).to eq 1
      node = node.next_node
      expect(node).to eq @linked.tail
    end

    it 'pop last item form tail' do
      expect(@linked.pop.value).to eq 1
      expect(@linked.size).to eq 2
    end

    it 'deviate item from linked' do
      @linked.deviate(@n2)
      # head -> 3 -> 2(deviate) -> 1
      expect(@linked.size).to eq 2
      expect(@linked.head.next_node.next_node.value).to eq 1
    end
  end

  context 'LRUCache' do
    before do
      @cache = LRUCache.new(2)
    end

    it 'cut lru (private)' do
      @cache.put("k1", 1)
      @cache.put("k2", 2)
      expect(@cache.table.size).to eq @cache.capacity
      @cache.send(:cut_lru)
      expect(@cache.table.size).to eq @cache.capacity - 1
    end

    it 'put' do
      @cache.put("k1", 1)
      @cache.put("k2", 2)
      expect(@cache.table.size).to eq @cache.capacity

      @cache.put("k2", 22)
      expect(@cache.table.size).to eq @cache.capacity
      expect(@cache.get("k2")).to eq 22

      @cache.put("k3", 3)
      expect(@cache.table.size).to eq @cache.capacity
      expect(@cache.get("k1")).to be_nil
    end
  end
end

```
