---
layout: post
title:  LeetCode implement strstr
date:   2020-12-21
Author: CBD
tags: [LeetCode]
---

[https://leetcode-cn.com/problems/implement-strstr/](https://leetcode-cn.com/problems/implement-strstr/)

> Ruby

```ruby
# @param {String} haystack
# @param {String} needle
# @return {Integer}
def str_str(haystack, needle)
    return 0 if needle == ""

    idx = haystack.index(needle)
    idx.nil? ? -1 : idx
end
```

> Ruby

```ruby
# @param {String} haystack
# @param {String} needle
# @return {Integer}
def str_str(haystack, needle)
  return 0 if needle.empty?

  return -1 if needle.length > haystack.length

  (haystack.length - needle.length + 1).times do |idx|
    target_str = haystack[idx...(idx + needle.length)]
    return idx if needle == target_str
  end

  -1
end
```

> Golang

```golang

func strStr(haystack string, needle string) int {
	if len(needle) == 0 {
		return 0
	}

	for idx := 0; idx <= (len(haystack) - len(needle) + 1); idx += 1 {
		strEndIdx := idx + len(needle)
		if strEndIdx > len(haystack) {
			return -1
		}

		if needle == haystack[idx:strEndIdx] {
			return idx
		}
	}

	return -1
}

```
