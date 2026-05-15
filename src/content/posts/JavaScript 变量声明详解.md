---
title: JavaScript 变量声明详解
published: 2022-01-01
description: '深入理解 let、const、var 的区别、作用域以及最佳实践'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 变量声明详解

## 概述

变量声明是 JavaScript 中最基础的语法之一。

在 ES6 之前，JavaScript 只有：

```javascript
var
```

而在 ES6 推出后，又新增了：

```javascript
let
const
```

这三者在：

- 作用域
- 变量提升
- 重复声明
- 修改能力

等方面存在明显区别。

掌握它们之间的差异，是写出现代 JavaScript 高质量代码的重要基础。

---

# var 详解

## 基本用法

```javascript
var name = 'Tom';

console.log(name);
```

---

## 函数作用域

`var` 只有函数作用域，没有块级作用域。

```javascript
if (true) {
  var age = 18;
}

console.log(age); // 18
```

即使在 `if` 代码块中声明，外部仍然可以访问。

---

## 变量提升

```javascript
console.log(a); // undefined

var a = 10;
```

实际执行过程：

```javascript
var a;

console.log(a);

a = 10;
```

这就是所谓的：

- 变量提升（Hoisting）

---

## 允许重复声明

```javascript
var num = 1;

var num = 2;

console.log(num); // 2
```

这容易导致代码覆盖问题。

---

# let 详解

## 基本用法

```javascript
let count = 100;

console.log(count);
```

---

## 块级作用域

```javascript
if (true) {
  let message = 'Hello';
}

console.log(message);
```

会报错：

```javascript
ReferenceError
```

因为 `let` 具有块级作用域。

---

## 不存在变量提升

```javascript
console.log(a);

let a = 1;
```

会直接报错。

这是因为：

- 存在暂时性死区（TDZ）

---

## 不允许重复声明

```javascript
let num = 1;

let num = 2;
```

会报错。

这样可以避免变量污染。

---

# const 详解

## 基本用法

```javascript
const PI = 3.14;
```

---

## 必须初始化

```javascript
const name;
```

会报错：

```javascript
Missing initializer in const declaration
```

---

## 不允许重新赋值

```javascript
const age = 18;

age = 20;
```

会报错。

---

## 对象可以修改内容

```javascript
const user = {
  name: 'Tom'
};

user.name = 'Jerry';

console.log(user.name);
```

这里不会报错。

因为：

- const 保证的是引用地址不变
- 不保证对象内部不可变

---

# let、const、var 区别总结

| 特性 | var | let | const |
|---|---|---|---|
| 作用域 | 函数作用域 | 块级作用域 | 块级作用域 |
| 变量提升 | 有 | 有但存在 TDZ | 有但存在 TDZ |
| 可重复声明 | 可以 | 不可以 | 不可以 |
| 可修改值 | 可以 | 可以 | 不可以 |
| 必须初始化 | 否 | 否 | 是 |

---

# 暂时性死区（TDZ）

## 什么是 TDZ？

在变量声明之前无法访问变量。

```javascript
{
  console.log(a);

  let a = 10;
}
```

会报错。

---

## 为什么需要 TDZ？

目的是：

- 减少变量使用错误
- 提高代码安全性
- 增强代码可读性

---

# 实际应用

## 1. 使用 const 定义常量

```javascript
const BASE_URL = 'https://api.test.com';
```

---

## 2. 使用 let 定义可变变量

```javascript
let currentPage = 1;
```

---

## 3. 避免使用 var

```javascript
// 不推荐
var data = [];
```

现代开发中已经很少使用 `var`。

---

# 循环中的区别

## var 的问题

```javascript
for (var i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

输出：

```javascript
3
3
3
```

因为：

- var 没有块级作用域
- 所有回调共享同一个 i

---

## let 的解决方案

```javascript
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i);
  }, 100);
}
```

输出：

```javascript
0
1
2
```

---

# 常见面试题

## let 和 const 有什么区别？

共同点：

- 都是块级作用域
- 都存在 TDZ
- 都不能重复声明

区别：

- let 可以重新赋值
- const 不可以重新赋值

---

## const 定义的对象为什么还能修改？

```javascript
const obj = {
  name: 'Tom'
};

obj.name = 'Jerry';
```

因为 const 只保证：

- 引用地址不变

不保证：

- 对象内部数据不可变

---

## 为什么不推荐使用 var？

主要原因：

- 容易变量污染
- 没有块级作用域
- 容易出现闭包问题
- 可读性较差

---

# 最佳实践

## 优先使用 const

```javascript
const user = {};
```

默认所有变量先使用 const。

---

## 需要修改时再使用 let

```javascript
let total = 0;
```

---

## 避免使用 var

现代项目中建议：

```javascript
禁止使用 var
```

---

## 保持变量作用域最小化

```javascript
if (success) {
  const result = data;
}
```

减少变量污染。

---

# 注意事项

## const 不等于不可变对象

如果需要真正不可变：

```javascript
Object.freeze(obj);
```

---

## 不要滥用全局变量

全局变量容易：

- 命名冲突
- 难以维护
- 增加耦合

---

## 注意 TDZ

```javascript
console.log(a);

let a = 1;
```

这类错误在开发中非常常见。

---

# 总结

变量声明是 JavaScript 最核心的基础知识之一。

本文重点介绍了：

- var 的特点与问题
- let 的块级作用域
- const 的不可重新赋值
- 暂时性死区
- 三者之间的区别
- 实际开发最佳实践

现代 JavaScript 开发推荐：

- 默认使用 const
- 需要修改时使用 let
- 避免使用 var

掌握这些内容后，你将能够写出：

- 更安全
- 更清晰
- 更现代化

的 JavaScript 代码。

后续建议继续学习：

- 作用域链
- 闭包
- 执行上下文
- ES6 新特性
- JavaScript 内存机制
