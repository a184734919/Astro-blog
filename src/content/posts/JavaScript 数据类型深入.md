---
title: JavaScript 数据类型深入
published: 2022-01-07
description: '深入理解 JavaScript 中的基本类型、引用类型以及它们之间的区别与应用场景'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 数据类型深入

## 概述

在 JavaScript 中，数据类型是最基础也是最重要的知识点之一。  
无论是变量声明、函数参数传递，还是对象操作、异步编程，都离不开对数据类型的理解。

JavaScript 的数据类型主要分为：

- 基本类型（Primitive Types）
- 引用类型（Reference Types）

理解它们之间的区别，对于写出高质量、可维护的代码非常重要。

---

## JavaScript 数据类型分类

JavaScript 中常见的数据类型如下：

### 基本类型（Primitive）

```javascript
String
Number
Boolean
Undefined
Null
Symbol
BigInt
```

### 引用类型（Reference）

```javascript
Object
Array
Function
Date
RegExp
Map
Set
```

其中，数组、函数本质上也都是对象。

---

## 核心概念

### 基本类型

基本类型的数据存储在栈内存中，特点如下：

- 占用空间小
- 存储的是实际值
- 值之间互不影响

示例：

```javascript
let a = 10;
let b = a;

b = 20;

console.log(a); // 10
console.log(b); // 20
```

这里 `b` 修改后不会影响 `a`。

---

### 引用类型

引用类型的数据存储在堆内存中，变量中保存的是引用地址。

示例：

```javascript
let obj1 = {
  name: 'Tom'
};

let obj2 = obj1;

obj2.name = 'Jerry';

console.log(obj1.name); // Jerry
```

因为 `obj1` 与 `obj2` 指向同一块内存，所以修改其中一个会影响另一个。

---

## typeof 与 instanceof

### typeof

用于检测基本数据类型。

```javascript
typeof 'hello'; // string
typeof 123; // number
typeof true; // boolean
typeof undefined; // undefined
typeof Symbol(); // symbol
```

特殊情况：

```javascript
typeof null; // object
```

这是 JavaScript 历史遗留问题。

---

### instanceof

用于检测对象的原型链。

```javascript
[] instanceof Array; // true

{} instanceof Object; // true

function(){} instanceof Function; // true
```

---

## 基本用法

### 数据类型转换

JavaScript 中经常会发生隐式转换。

#### 转字符串

```javascript
String(123); // "123"
123 + ''; // "123"
```

#### 转数字

```javascript
Number('100'); // 100
parseInt('123px'); // 123
```

#### 转布尔值

```javascript
Boolean(1); // true
Boolean(0); // false
Boolean(''); // false
```

---

## 深拷贝与浅拷贝

引用类型在开发中经常涉及对象复制问题。

### 浅拷贝

```javascript
const obj = {
  name: 'Tom',
  info: {
    age: 18
  }
};

const copy = { ...obj };

copy.info.age = 20;

console.log(obj.info.age); // 20
```

浅拷贝只复制第一层。

---

### 深拷贝

#### JSON 方法

```javascript
const newObj = JSON.parse(JSON.stringify(obj));
```

缺点：

- 无法复制函数
- 无法处理 `undefined`
- 无法处理循环引用

---

## 实际应用

在实际项目开发中：

### 1. 表单数据处理

```javascript
const age = Number(inputValue);
```

避免字符串导致计算错误。

---

### 2. 接口数据校验

```javascript
if (typeof data === 'object' && data !== null) {
  console.log('合法对象');
}
```

---

### 3. 深拷贝状态管理

在 Vue 或 React 中，经常需要避免直接修改原对象。

```javascript
const newState = JSON.parse(JSON.stringify(state));
```

---

## 常见面试题

### null 和 undefined 的区别？

- `undefined` 表示变量已声明但未赋值
- `null` 表示“空对象指针”

```javascript
let a;
console.log(a); // undefined

let b = null;
console.log(b); // null
```

---

### 为什么 typeof null 是 object？

这是 JavaScript 底层实现的历史遗留问题。

在早期版本中，类型使用二进制表示，而 `null` 的二进制表示被错误识别为了对象。

---

### == 和 === 的区别？

#### ==

会进行隐式类型转换。

```javascript
1 == '1'; // true
```

#### ===

严格比较，不进行类型转换。

```javascript
1 === '1'; // false
```

开发中推荐使用 `===`。

---

## 注意事项

### 1. 注意隐式类型转换

```javascript
[] + []; // ""
[] + {}; // "[object Object]"
```

这类问题容易导致 Bug。

---

### 2. 谨慎使用浅拷贝

复杂对象嵌套时容易产生数据污染。

---

### 3. 避免使用 ==

推荐统一使用：

```javascript
===
```

提高代码可读性与稳定性。

---

## 最佳实践

### 推荐使用 const

```javascript
const user = {};
```

避免变量被意外修改。

---

### 合理判断空值

```javascript
if (value == null) {
  // 同时判断 null 和 undefined
}
```

---

### 使用 Array.isArray()

```javascript
Array.isArray([]);
```

比 `typeof` 更准确。

---

## 总结

JavaScript 数据类型是前端开发中的核心基础。

本文主要介绍了：

- 基本类型与引用类型
- typeof 与 instanceof
- 数据类型转换
- 深拷贝与浅拷贝
- 常见面试题
- 开发中的最佳实践

掌握这些内容后，你将能够更深入地理解 JavaScript 的运行机制，并在实际开发中写出更加健壮、可维护的代码。

后续还可以继续深入学习：

- 原型与原型链
- 闭包
- 执行上下文
- 垃圾回收机制
- ES6+ 新特性

这些知识点与数据类型密切相关。
