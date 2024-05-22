---
layout: post
title: JavaScript PropertyDescriptor
date: 2024-05-22
Author: CBD
tags: [ JavaScript ]
---

## 数据属性 & 访问器属性

```javascript
'use strict'

const obj = {
    // 数据属性
    hi: 'Hi',
    age: 18,

    // 访问器属性
    get name() {
        this._name
    },
    set name(value) {
        this._name = value
    },
}

console.log(Object.getOwnPropertyDescriptors(obj))
```

```log
age
: 
{value: 18, writable: true, enumerable: true, configurable: true}
hi
: 
{value: 'Hi', writable: true, enumerable: true, configurable: true}
name
: 
{enumerable: true, configurable: true, get: ƒ, set: ƒ}
```

对象的属性上不仅保存了值本身, 还保存了一些属性描述符, 用来控制该属性的行为.

默认情况下, `configurable` `enumerable` `writable` 都是 true.

```ts
declare type PropertyKey = string | number | symbol;

interface PropertyDescriptor {
    configurable?: boolean;
    enumerable?: boolean;
    value?: any;
    writable?: boolean;

    get?(): any;

    set?(v: any): void;
}

interface PropertyDescriptorMap {
    [key: PropertyKey]: PropertyDescriptor;
}
```

## enumerable

把 `enumerable` 设置为 false 后,
属性将不在 `for...in` / `Object.keys()` / `JSON.stringify()` /  `Object.assign()` 中被枚举出来,
但 `Object.getOwnPropertyNames()` 仍然可以.

## writable

`writable` 为 false 后, 属性变为只读的, 即不能修改属性的值(尝试修改会抛异常), 但可以删掉该属性.

```javascript
'use strict'

const obj = {name: 'Bob'}
obj.name = 'Bob Dylan'
console.log(obj.name) // 'Bob Dylan'

Object.defineProperty(obj, 'name', {writable: false})
try {
    obj.name = 'John'
} catch (e) {
    console.log(e.toString()) // TypeError: Cannot assign to read only property 'name' of object '#<Object>'
}

delete obj['name']

```

## configurable

如果属性是 `configurable`, 那么意味着冻结该属性, 只读, 不能修改, 不能删除.

`configurable` 一旦被设置为 false 后, 就再也不能被设置为 true 了.

## defineProperty

如果使用 `defineProperty` 定义新属性, 一定要记得设置 `writable` `configurable` 三个属性, 否则属性值为 false.

```javascript
'use strict'

const obj = {}

Object.defineProperty(obj, 'newKey', { enumerable: true })
console.log(Object.getOwnPropertyDescriptors(obj))

```

```log
newKey
: 
{value: undefined, writable: false, enumerable: true, configurable: false}
```
