---
layout: post
title:  JavaScript Deal With Emoji String 
date:   2021-02-02
Author: CBD
tags:   [JavaScript]
---

JavaScript 使用 utf-16 的变长编码.
基本平面内的常用字符, 每个占用 2 个字节;
分布在扩展平面的特殊字符, 每个占用 4 个字节;

```js
"EN".length // 2

"中文".length // 2

"中文😀".length // 4
```

`length` 发明时, JavaScript 还用的 UCS-2 编码, 可以简单理解 UCS-2 是只包含基本平面的 UTF-16 ,是固定2 字节宽度的, 没有变长的问题. 兼容性考虑, length 的表现一直没有发生变化.

逻辑上来说, 一个 Emoji 应该算作一个字符, length 把他们算作了两个. 这时我们应该使用更现代的处理方式, 用码点 CodePoint 来计算字符个数.

string 的 `for ... of` 迭代就可以正确处理好码点:

```js
let str = "中文😀";
let len = 0;
for (ch of str) {
    len++
}
console.log(len); // 3
```

更推荐的方法是借助 `Array.from(arr)`, 他会把可迭代对象(实现了`Symbol.iterator`方法)或者类数组对象(含索引和 length 属性)包装为一个真正的数组对象.

```js
let len = Array.from(str).length; // 3
```

同样的, `slice` Emoji 字符串也应该通过码点来截断:

```js
function slice(str, start, end) {
  return Array.from(str).slice(start, end).join('');
}
```

## 参考

[https://www.ruanyifeng.com/blog/2014/12/unicode.html](https://www.ruanyifeng.com/blog/2014/12/unicode.html)
