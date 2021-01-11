---
layout: post
title:  LeetCode open-the-lock
date:   2021-01-11
Author: CBD
tags:   [LeetCode, BFS]
---

[https://leetcode-cn.com/problems/open-the-lock/](https://leetcode-cn.com/problems/open-the-lock/)

把当前队列的长度固定下来, `currentQueueLen := len(queue)`, 遍历完一轮后计数器加一.

> Golang

```golang

const FirstStr = "0000"

func openLock(deadends []string, target string) int {
	deadDict := make(map[string]bool)
	for i := 0; i < len(deadends); i++ {
		deadDict[deadends[i]] = true
	}

	queue := []string{FirstStr}
	visitedDict := map[string]bool{FirstStr: true}

	stepCount := 0

	for len(queue) > 0 {
		// 每经过一轮, stepCount 加一
		currentQueueLen := len(queue)

		for i := 0; i < currentQueueLen; i++ {
			// shift first item from queue
			current := queue[0]
			queue = queue[1:]


			if target == current {
				return stepCount
			}
			if _, ok := deadDict[current]; ok {
				continue
			}

			// 遍历 "0000" 的各种拨动情况, 一共 4 * 2 = 8 种
			for idx := 0; idx < len(target); idx++ {
				nextList := []string{
					turnUp(current, idx),
					turnDown(current, idx),
				}
				for _, next := range nextList {
					if _, ok := visitedDict[next]; !ok {
						queue = append(queue, next)
						visitedDict[next] = true
					}
				}
			}
		}

		stepCount++
	}
	return -1
}

func turnUp(nums string, idx int) string {
	runeList := []rune(nums)

	if runeList[idx] == '9' {
		runeList[idx] = '0'
	} else {
		i, _ := strconv.Atoi(string(runeList[idx]))
		i++
		runeList[idx] = []rune(strconv.Itoa(i))[0]
	}
	return string(runeList)
}

func turnDown(nums string, idx int) string {
	runeList := []rune(nums)

	if runeList[idx] == '0' {
		runeList[idx] = '9'
	} else {
		i, _ := strconv.Atoi(string(runeList[idx]))
		i--
		runeList[idx] = []rune(strconv.Itoa(i))[0]
	}
	return string(runeList)
}

```
