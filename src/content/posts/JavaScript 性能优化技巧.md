---
title: JavaScript 性能优化技巧
published: 2022-10-08
description: '常见性能优化方法总结的详细介绍和学习笔记'
image: ''
tags: ["性能"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 性能优化是前端开发中的关键环节，直接影响用户体验和应用的整体性能。本文将系统性地介绍 JavaScript 常见的性能优化方法，涵盖代码层面、数据结构、算法优化等多个维度。

## 核心概念

### 性能优化原则

1. **测量优先**：优化前先测量，找到真正的瓶颈
2. **权衡取舍**：优化通常会在代码可读性、可维护性上做出妥协
3. **渐进优化**：先做影响最大的优化，再处理细节
4. **避免过早优化**：先保证正确性，再考虑性能

### 性能分析工具

```javascript
// 使用 console.time 测量执行时间
console.time('operation');
// 执行代码
for (let i = 0; i < 1000000; i++) {
  // 一些操作
}
console.timeEnd('operation');

// 使用 performance.now() 获取更精确的时间
const start = performance.now();
// 执行代码
const end = performance.now();
console.log(`执行时间: ${end - start}ms`);
```

## 基本用法

### 变量声明优化

```javascript
// ❌ 不推荐：重复声明
function bad() {
  var x = 1;
  var y = 2;
  var z = 3;
}

// ✅ 推荐：集中声明
function good() {
  const x = 1, y = 2, z = 3;
}

// ✅ 推荐：使用 let/const 代替 var
function modern() {
  let x = 1;
  const y = 2;
  const z = 3;
}

// ✅ 推荐：块级作用域减少变量提升
function scoped() {
  if (true) {
    const temp = 'local';
    console.log(temp);
  }
  // temp 在此处不可访问
}
```

### 循环优化

```javascript
// ❌ 不推荐：在循环中计算数组长度
for (let i = 0; i < array.length; i++) {
  // 循环体
}

// ✅ 推荐：缓存数组长度
for (let i = 0, len = array.length; i < len; i++) {
  // 循环体
}

// ✅ 推荐：使用倒序循环（有时更快）
for (let i = array.length - 1; i >= 0; i--) {
  // 循环体
}

// ✅ 推荐：使用 for...of（可读性好，性能接近）
for (const item of array) {
  // 处理 item
}

// ✅ 推荐：使用 while 循环（简单场景）
let i = 0;
while (i < array.length) {
  // 处理 array[i]
  i++;
}
```

### DOM 操作优化

```javascript
// ❌ 不推荐：频繁操作 DOM
function badDOM() {
  const container = document.getElementById('container');
  for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    container.appendChild(div); // 每次都触发重排
  }
}

// ✅ 推荐：使用文档片段
function goodDOM() {
  const container = document.getElementById('container');
  const fragment = document.createDocumentFragment();

  for (let i = 0; i < 1000; i++) {
    const div = document.createElement('div');
    div.textContent = `Item ${i}`;
    fragment.appendChild(div);
  }

  container.appendChild(fragment); // 只触发一次重排
}

// ✅ 推荐：批量更新样式
function batchStyleUpdates() {
  const element = document.getElementById('element');
  // 先添加 class，再修改样式
  element.classList.add('updating');
  element.style.width = '200px';
  element.style.height = '100px';
  element.style.background = 'blue';
  // 移除 class
  element.classList.remove('updating');
}
```

### 事件处理优化

```javascript
// ✅ 推荐：使用事件委托
// ❌ 不推荐：为每个元素绑定事件
function badEventBinding() {
  const items = document.querySelectorAll('.item');
  items.forEach(item => {
    item.addEventListener('click', function() {
      console.log('Item clicked');
    });
  });
}

// ✅ 推荐：使用事件委托
function goodEventDelegation() {
  const container = document.getElementById('container');
  container.addEventListener('click', function(event) {
    if (event.target.classList.contains('item')) {
      console.log('Item clicked');
    }
  });
}

// ✅ 推荐：使用防抖和节流
// 防抖：只在最后一次触发后执行
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => func.apply(this, args), delay);
  };
}

// 节流：固定时间间隔执行一次
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      func.apply(this, args);
    }
  };
}
```

## 实际应用

### 数据结构选择优化

```javascript
// ✅ 推荐：根据场景选择合适的数据结构

// 快速查找：使用 Map 或 Object
const lookupMap = new Map();
lookupMap.set('key1', 'value1');
lookupMap.get('key1'); // O(1)

// 有序数据：使用数组 + 二分查找
const sortedArray = [1, 3, 5, 7, 9];
function binarySearch(arr, target) {
  let left = 0, right = arr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) return mid;
    if (arr[mid] < target) left = mid + 1;
    else right = mid - 1;
  }
  return -1;
}

// 去重：使用 Set
const uniqueArray = [...new Set([1, 2, 2, 3, 3, 4])]; // [1, 2, 3, 4]

// 频繁增删：使用链表（实现略）
```

### 字符串操作优化

```javascript
// ❌ 不推荐：在循环中拼接字符串
function badStringConcat() {
  let result = '';
  for (let i = 0; i < 10000; i++) {
    result += 'text'; // 每次都创建新字符串
  }
  return result;
}

// ✅ 推荐：使用数组 join
function goodStringConcat() {
  const parts = [];
  for (let i = 0; i < 10000; i++) {
    parts.push('text');
  }
  return parts.join('');
}

// ✅ 推荐：使用模板字符串（ES6）
function templateConcat() {
  const name = 'Alice';
  const age = 25;
  return `Name: ${name}, Age: ${age}`;
}
```

### 数组操作优化

```javascript
// ✅ 推荐：使用原生数组方法
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 使用 filter 和 map 链式调用
const result = numbers
  .filter(num => num % 2 === 0)
  .map(num => num * 2)
  .reduce((sum, num) => sum + num, 0);

// ✅ 推荐：使用 TypedArray 处理大量数值数据
const typedArray = new Float32Array(1000000);
for (let i = 0; i < typedArray.length; i++) {
  typedArray[i] = Math.random();
}

// ✅ 推荐：避免在循环中修改数组长度
function safeArrayIteration() {
  const array = [1, 2, 3, 4, 5];
  const toRemove = [2, 4];

  // 创建新数组而不是修改原数组
  const filtered = array.filter(item => !toRemove.includes(item));
  return filtered;
}
```

### 异步操作优化

```javascript
// ✅ 推荐：使用 Promise.all 并行执行
async function parallelFetch() {
  const urls = ['/api/user', '/api/posts', '/api/comments'];
  const promises = urls.map(url => fetch(url).then(res => res.json()));
  const results = await Promise.all(promises);
  return results;
}

// ✅ 推荐：使用 Promise.allSettled 处理部分失败
async function safeParallelFetch() {
  const urls = ['/api/user', '/api/posts', '/api/comments'];
  const promises = urls.map(url => fetch(url).then(res => res.json()));
  const results = await Promise.allSettled(promises);

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return { successful, failed };
}

// ✅ 推荐：使用 async/await 避免回调地狱
async function fetchData() {
  try {
    const user = await fetch('/api/user').then(res => res.json());
    const posts = await fetch(`/api/posts?userId=${user.id}`).then(res => res.json());
    return posts;
  } catch (error) {
    console.error('Error:', error);
  }
}
```

### 内存管理优化

```javascript
// ✅ 推荐：及时清理大对象
function processData() {
  const largeData = new Array(1000000).fill(null);

  // 处理数据
  const result = largeData.filter(item => item !== null);

  // 及时清理
  largeData.length = 0; // 清空数组
  return result;
}

// ✅ 推荐：避免闭包中的内存泄漏
function createEventHandlers() {
  const elements = document.querySelectorAll('.item');
  const handlers = [];

  elements.forEach((element, index) => {
    const handler = () => {
      console.log(`Item ${index} clicked`);
    };
    element.addEventListener('click', handler);
    handlers.push({ element, handler });
  });

  // 返回清理函数
  return function cleanup() {
    handlers.forEach(({ element, handler }) => {
      element.removeEventListener('click', handler);
    });
  };
}

// ✅ 推荐：使用 WeakMap 避免内存泄漏
const weakMap = new WeakMap();
function cacheData(obj) {
  weakMap.set(obj, { processed: true });
}

function getCachedData(obj) {
  return weakMap.get(obj);
}
```

### 算法优化

```javascript
// ✅ 推荐：选择合适的算法
// 记忆化递归（避免重复计算）
const memo = new Map();
function fibonacci(n) {
  if (n <= 1) return n;
  if (memo.has(n)) return memo.get(n);

  const result = fibonacci(n - 1) + fibonacci(n - 2);
  memo.set(n, result);
  return result;
}

// ✅ 推荐：使用 Set 去重（O(n)）
function uniqueArray(array) {
  return [...new Set(array)];
}

// ✅ 推荐：批量处理
function batchProcess(items, batchSize, processFn) {
  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    processFn(batch);
  }
}
```

## 注意事项

1. **不要过早优化**：先确保代码正确，再考虑性能
2. **权衡可读性**：性能优化不应牺牲代码可读性
3. **避免过度优化**：微小的性能提升不值得付出复杂性
4. **测量实际效果**：使用性能分析工具验证优化效果
5. **考虑用户体验**：某些优化可能影响用户体验

## 总结

JavaScript 性能优化是一个持续的过程，需要结合实际场景和需求进行。关键要点：

- **代码层面**：使用现代语法，避免不必要的计算和操作
- **数据结构**：选择合适的数据结构提升访问效率
- **DOM 操作**：减少重排重绘，使用事件委托
- **异步处理**：合理使用 Promise 和 async/await
- **内存管理**：及时清理大对象，避免内存泄漏
- **算法优化**：选择合适算法，使用缓存和记忆化

记住，最好的优化是让代码简单、清晰、易于理解。在性能成为问题之前，专注于编写高质量的代码。