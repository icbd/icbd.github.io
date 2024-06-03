---
layout: post
title: JavaScript prototype
date: 2024-06-03
Author: CBD
tags: [ JavaScript ]
---

```javascript
'use strict'

function P() {}
const p = new P()

function Q() {}

// 用 Object.getPrototypeOf() 获取对象的原型对象
p.__proto__ === Object.getPrototypeOf(p) // true
// 用 .prototype 从构造函数中获取原型对象
Object.getPrototypeOf(p) === P.prototype // true

// 每个函数都对应各自不同的原型对象
P.prototype != Q.prototype // true

const prototypeObjectOfP = Object.getPrototypeOf(p)
// 函数(包括构造函数)自动会创建一个原型对象,
// 该原型对象自动含有 `.constructor` 属性, 并将该属性指向函数本身.
prototypeObjectOfP.constructor === P // true

// 原型对象就是一个普通的对象
typeof prototypeObjectOfP // 'object'
// 原型对象的的原型对象为 `Object.prototype`
Object.getPrototypeOf(prototypeObjectOfP) === Object.prototype // true

// Object 是一个构造方法
typeof Object // 'function'
// Object.prototype 是原型对象继承链的祖先
typeof Object.prototype // 'object'

// Object.prototype 的原型对象不存在, 为 null
Object.getPrototypeOf(Object.prototype) === null // true

```