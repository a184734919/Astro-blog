---
title: JavaScript 高阶函数应用
published: 2022-09-26
description: '高阶函数的实际应用场景的详细介绍和学习笔记'
image: ''
tags: ["函数式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

高阶函数（Higher-Order Function）是函数式编程的核心概念之一。简单来说，高阶函数是指：
1. 接受一个或多个函数作为参数
2. 返回一个函数作为结果

JavaScript 的函数是一等公民，这使得高阶函数在 JavaScript 中应用非常广泛。常见的高阶函数包括 `map`、`filter`、`reduce`、`forEach` 等。

## 核心概念

### 什么是高阶函数

```javascript
// 接受函数作为参数的高阶函数
function operate(a, b, operation) {
  return operation(a, b);
}

const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

console.log(operate(5, 3, add));      // 8
console.log(operate(5, 3, multiply)); // 15
```

### 返回函数的高阶函数

```javascript
// 返回函数的高阶函数
function createMultiplier(multiplier) {
  return function(x) {
    return x * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

## 基本用法

### 数组高阶函数

#### map - 转换数组

```javascript
const numbers = [1, 2, 3, 4, 5];

// 基础用法
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 转换对象数组
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

const names = users.map(user => user.name);
console.log(names); // ['Alice', 'Bob']

// 高级用法：提取多个属性
const userDetails = users.map(({ id, name }) => ({
  userId: id,
  displayName: name
}));
```

#### filter - 过滤数组

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 基础过滤
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// 复杂过滤条件
const products = [
  { id: 1, name: 'Laptop', price: 1000, stock: 5 },
  { id: 2, name: 'Phone', price: 500, stock: 10 },
  { id: 3, name: 'Tablet', price: 300, stock: 0 }
];

const inStock = products.filter(item => item.stock > 0);
const affordable = products.filter(item => item.price < 600);
```

#### reduce - 归约数组

```javascript
// 数组求和
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

// 数组转对象
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

const userMap = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(userMap);
// { 1: { id: 1, name: 'Alice' }, 2: { id: 2, name: 'Bob' } }

// 统计词频
const words = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const wordCount = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});

console.log(wordCount); // { apple: 3, banana: 2, orange: 1 }
```

### 自定义高阶函数

#### 函数组合

```javascript
// 组合多个函数
function compose(...functions) {
  return function(x) {
    return functions.reduceRight((acc, fn) => fn(acc), x);
  };
}

const toUpper = str => str.toUpperCase();
const reverse = str => str.split('').reverse().join('');
const exclaim = str => str + '!';

const transform = compose(exclaim, reverse, toUpper);
console.log(transform('hello')); // 'OLLEH!'
```

#### 柯里化

```javascript
// 创建柯里化函数
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
```

#### 防抖与节流

```javascript
// 防抖函数
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// 节流函数
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = new Date().getTime();
    if (now - lastCall < delay) return;
    lastCall = now;
    func.apply(this, args);
  };
}

// 使用示例
const searchInput = document.getElementById('search');
const debouncedSearch = debounce(event => {
  console.log('Searching for:', event.target.value);
}, 300);

searchInput.addEventListener('input', debouncedSearch);
```

## 实际应用

### API 数据处理

```javascript
async function fetchUsers() {
  const response = await fetch('/api/users');
  const users = await response.json();

  // 使用高阶函数链式处理数据
  return users
    .filter(user => user.isActive)
    .map(user => ({
      id: user.id,
      name: user.name,
      email: user.email
    }))
    .sort((a, b) => a.name.localeCompare(b.name));
}
```

### 表单验证

```javascript
// 创建验证器
function createValidator(rules) {
  return function(value) {
    return rules.every(rule => rule.validator(value))
      ? { valid: true }
      : { valid: false, error: rules.find(rule => !rule.validator(value)).error };
  };
}

const emailValidator = createValidator([
  { validator: v => v.includes('@'), error: '必须包含 @ 符号' },
  { validator: v => v.includes('.'), error: '必须包含 . 符号' },
  { validator: v => v.length > 5, error: '长度必须大于 5' }
]);

console.log(emailValidator('test@email.com'));
console.log(emailValidator('invalid'));
```

### 事件处理

```javascript
// 创建事件处理器工厂
function createEventHandler(handler, options = {}) {
  return function(event) {
    event.preventDefault();
    if (options.validate && !options.validate()) {
      return;
    }
    handler(event);
  };
}

const formSubmitHandler = createEventHandler(
  (event) => {
    const formData = new FormData(event.target);
    console.log('Form submitted:', Object.fromEntries(formData));
  },
  { validate: () => form.checkValidity() }
);

document.getElementById('myForm').addEventListener('submit', formSubmitHandler);
```

## 注意事项

1. **性能考虑**：高阶函数会创建新的函数和闭包，在性能敏感的场景要谨慎使用
2. **可读性**：过度嵌套的高阶函数会降低代码可读性，适当拆分函数
3. **内存泄漏**：注意闭包中引用的变量，避免意外的内存占用
4. **this 绑定**：箭头函数不会绑定自己的 this，在类方法中使用时要注意

## 总结

高阶函数是 JavaScript 函数式编程的强大工具，能够帮助我们：
- 编写更简洁、声明式的代码
- 提高代码的可重用性和可维护性
- 实现复杂的数据转换和处理逻辑

掌握高阶函数的使用，将让你的 JavaScript 代码更加优雅和高效。