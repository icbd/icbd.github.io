---
layout: post
title:  JavaScript 常用装饰器 
date:   2021-02-04
Author: CBD
tags:   [JavaScript]
---

先写若干业务逻辑方法, 装饰器用来装饰业务逻辑方法, 在方法原有功能的基础上, 包装一些通用逻辑.

* 以非侵入的方式扩展原有方法;
* 复用通用的装饰逻辑;

## 间谍装饰器

用来收集方法的入参和结果.

> JavaScript

```javascript
function spy(func) {
    return function f(...args) {
        f.calls ||= [];

        const result = func.call(this, ...args);
        f.calls.push([args, result]);
        return result;
    }
}

function sum(a, b) {
    return a + b;
}

sum = spy(sum);
console.log(sum(1, 2)); // 3
console.log(sum(3, 4)); // 7
console.log(sum.calls); // [ [ [ 1, 2 ], 3 ], [ [ 3, 4 ], 7 ] ]

```

要点:

* js 的方法也是对象, `calls` 是方法上的属性;
* 为了兼容对象方法, 使用 `call` 把 `this` 传递给 `func`;
* `return function f(...args)` 实际上是用表达式的方式来声明函数, `f` 是函数的标签, 仅函数内部可用, 用来调用函数自身.

## 缓存装饰器

用来缓存耗时函数的结果.

> JavaScript

```javascript
function cache(func) {
    return function f() {
        f.cacheMap ||= new Map();

        const hashKey = [].join.call(arguments);
        if (f.cacheMap.has(hashKey)) {
            console.log("from cache:");
            return f.cacheMap.get(hashKey);
        }

        const result = func.call(this, ...arguments);
        f.cacheMap.set(hashKey, result);
        return result;
    }
}

function sum(a, b) {
    return a + b;
}

sum = cache(sum);
console.log(sum(1, 2)); // 3
console.log(sum(1, 2)); // from cache: 3

```

`[].join.call(arguments)` 叫"方法借用".

`arguments`是个类数组, 不能直接使用 `Array#join`, `[].join` 提取了 `join` 方法, 通过 `call` 让 `join` 运行在 `arguments` 的上下文, 就好像借用一样, 故名"方法借用", 思路跟接口或者鸭子类型一样.

## 延时装饰器

> JavaScript

```javascript
function delay(func, ms) {
    return function () {
        return setTimeout(() => {
            const result = func.call(this, ...arguments);
            console.log(result);
        }, ms);
    }
}

function sum(a, b) {
    return a + b;
}

sum = delay(sum, 2000);
console.log(sum(1, 2)); // TimeoutID or TimeoutObject

```

## 防抖装饰器

1. 给高频调用添加冷静期, 在冷静期结束时触发该调用;
2. 如果在冷静期内又收到调用, 取消之前的调用并重复上一步;

> JavaScript

```javascript
function debounce(func, ms) {
    return function f() {
        if (f.timeout) {
            clearTimeout(f.timeout);
        }

        f.timeout = setTimeout(() => {
                func.call(this, ...arguments);
            },
            ms
        );
        return f.timeout;
    }
}

function sum(a, b) {
    console.log(">")
    return a + b;
}

sum = debounce(sum, 300);

// 在防抖间隔内, 只执行一次
sum(1, 2);
sum(1, 2);
sum(1, 2);

// 超过防抖间隔, 又执行一次
setTimeout(() => sum(1, 2), 500);

```

防抖的副作用是增加方法执行的延迟, 所以防抖间隔不宜设置过大.

如果事件规律得在时间间隔内触发, 会造成方法的执行一直被推迟:

```javascript
// 间隔 600ms, 事件每隔 500ms 触发一次, 导致事件一致推迟执行.
sum = debounce(sum, 600);
setInterval(() => {
    sum(1, 2);
}, 500);
```

## 节流装饰器

在时间间隔内, 合并多个调用为一个调用.

防抖是先等待再执行, 节流是先执行再等待. 有助于提高响应体验.

> JavaScript

```javascript
function throttle(func, ms) {
    return function f() {
        f.jump ||= false;

        if (f.jump) {
            return;
        }
        f.jump = true;
        func.call(this, ...arguments);

        setTimeout(function () {
            f.jump = false;
        }, ms);
    }
}

function sum(a, b) {
    console.log(">")
    return a + b;
}

sum = throttle(sum, 500);

sum(1, 2);
sum(1, 2);
sum(1, 2);

setTimeout(() => {
    sum(1, 2);
}, 600);

```

节流能解决防抖事件被推迟的问题: 间隔内多个调用时, 能保证间隔内至少有一个调用执行:

```js
sum = throttle(sum, 500);
setInterval(() => {
    sum(1, 2)
}, 300);
```
