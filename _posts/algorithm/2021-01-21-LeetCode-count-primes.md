---
layout: post
title:  LeetCode count-primes
date:   2021-01-21
Author: CBD
tags:   [LeetCode]
---

[https://leetcode-cn.com/problems/count-primes/](https://leetcode-cn.com/problems/count-primes/)

> JavaScript

```javascript
var countPrimes = function (n) {
        const table = Array(n + 1).fill(true);
        for (let i = 2; i <= n; i++) {
            if (table[i]) {
                const step = i;
                for (let num = step + step; num <= n; num += step) {
                    table[num] = false;
                }
            }
        }
        let count = 0;
        for (let i = 2; i < n; i++) {
            if (table[i]) {
                count++;
            }
        }
        return count;
    };
```
