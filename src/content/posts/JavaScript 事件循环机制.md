---
title: JavaScript 事件循环机制
published: 2022-07-11
description: '宏任务和微任务队列的详细介绍和学习笔记'
image: ''
tags: ["JS","事件循环"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 是单线程语言,但通过事件循环机制可以实现异步操作,避免阻塞主线程。理解事件循环对于编写高性能的 JavaScript 代码至关重要。

## JavaScript 执行机制

### 单线程模型

JavaScript 采用单线程模型,所有代码都在主线程上执行:

```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
console.log('3');

// 输出: 1, 3, 2
```

### 调用栈

JavaScript 使用调用栈来管理函数执行:

```javascript
function first() {
  console.log('First');
  second();
  console.log('First End');
}

function second() {
  console.log('Second');
  third();
}

function third() {
  console.log('Third');
}

first();

// 输出顺序: First -> Second -> Third -> First End
```

## 事件循环

### 基本结构

事件循环是 JavaScript 实现异步的核心机制,它不断检查调用栈和任务队列:

```
┌─────────────────────────────────────┐
│         调用栈 (Call Stack)          │
└─────────────────────────────────────┘
                ↓
┌─────────────────────────────────────┐
│          宏任务队列                   │
│  (setTimeout, setInterval, I/O)     │
└─────────────────────────────────────┘
                ↓
┌─────────────────────────────────────┐
│          微任务队列                   │
│    (Promise.then, MutationObserver) │
└─────────────────────────────────────┘
```

### 执行顺序

```javascript
console.log('Script Start');

setTimeout(() => {
  console.log('setTimeout 1');
}, 0);

Promise.resolve().then(() => {
  console.log('Promise 1');
});

console.log('Script End');

// 输出: Script Start -> Script End -> Promise 1 -> setTimeout 1
```

## 宏任务与微任务

### 宏任务 (Macrotask)

宏任务包括:
- `setTimeout`
- `setInterval`
- `setImmediate` (Node.js)
- `I/O 操作`
- `UI 渲染`

```javascript
console.log('Start');

setTimeout(() => console.log('setTimeout'), 0);

// 相当于一个宏任务
setTimeout(() => {
  console.log('Outer');
  setTimeout(() => console.log('Inner'), 0);
}, 0);

console.log('End');

// 输出: Start -> End -> setTimeout -> Outer -> Inner
```

### 微任务 (Microtask)

微任务包括:
- `Promise.then/catch/finally`
- `MutationObserver`
- `process.nextTick` (Node.js)
- `queueMicrotask`

```javascript
console.log('Start');

Promise.resolve().then(() => {
  console.log('Promise 1');
  Promise.resolve().then(() => {
    console.log('Promise 2');
  });
});

Promise.resolve().then(() => {
  console.log('Promise 3');
});

console.log('End');

// 输出: Start -> End -> Promise 1 -> Promise 2 -> Promise 3
```

### 执行规则

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

new Promise((resolve) => {
  console.log('3');
  resolve();
}).then(() => console.log('4'));

console.log('5');

// 输出: 1 -> 3 -> 5 -> 4 -> 2
```

## 实际应用场景

### 异步数据请求

```javascript
async function fetchData() {
  console.log('Fetching...');
  
  try {
    // 微任务队列
    const data = await fetch('/api/data');
    const json = await data.json();
    console.log('Data received:', json);
  } catch (error) {
    console.error('Error:', error);
  }
}

fetchData();
console.log('Continue...');
```

### DOM 操作与渲染

```javascript
function updateDOM() {
  document.body.style.backgroundColor = 'red';
  
  // 使用微任务确保在下一帧前执行
  Promise.resolve().then(() => {
    console.log('Background changed');
  });
  
  requestAnimationFrame(() => {
    console.log('Next frame');
  });
}

updateDOM();
```

### 批量更新

```javascript
let updates = [];

function scheduleUpdate(update) {
  updates.push(update);
  
  if (!updatePromise) {
    updatePromise = Promise.resolve().then(() => {
      console.log('Batch update:', updates);
      updates = [];
      updatePromise = null;
    });
  }
}

scheduleUpdate({ id: 1, value: 'A' });
scheduleUpdate({ id: 2, value: 'B' });
scheduleUpdate({ id: 3, value: 'C' });
```

## 注意事项

### 1. 避免任务饥饿

```javascript
// 错误示例: 可能导致微任务队列阻塞
while (true) {
  Promise.resolve().then(() => {
    // 大量微任务
  });
}

// 正确示例: 分批处理
async function processLargeArray(array) {
  const chunkSize = 100;
  
  for (let i = 0; i < array.length; i += chunkSize) {
    const chunk = array.slice(i, i + chunkSize);
    await processChunk(chunk);
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}
```

### 2. 正确使用 setTimeout

```javascript
// 延迟到宏任务队列
function defer(fn) {
  setTimeout(fn, 0);
}

// 确保在 DOM 更新后执行
function afterDOMUpdate(fn) {
  requestAnimationFrame(() => {
    setTimeout(fn, 0);
  });
}
```

### 3. 微任务陷阱

```javascript
// 陷阱: 微任务队列的深度嵌套
function runMicrotasks() {
  let count = 0;
  
  function recurse() {
    count++;
    if (count < 10000) {
      Promise.resolve().then(recurse);
    }
  }
  
  recurse();
  console.log('Microtasks scheduled');
}

// 更好的方案: 使用宏任务分割
function runTasksInChunks(tasks, chunkSize = 100) {
  let index = 0;
  
  function processChunk() {
    const end = Math.min(index + chunkSize, tasks.length);
    for (; index < end; index++) {
      tasks[index]();
    }
    
    if (index < tasks.length) {
      setTimeout(processChunk, 0);
    }
  }
  
  processChunk();
}
```

## 性能优化

### 减少主线程阻塞

```javascript
// 使用 Web Worker 处理繁重任务
const worker = new Worker('worker.js');

worker.onmessage = (e) => {
  console.log('Result:', e.data);
};

worker.postMessage({ data: heavyData });
```

### 优化 Promise 链

```javascript
// 避免过长的 Promise 链
// 不好
Promise.resolve()
  .then(() => step1())
  .then(() => step2())
  .then(() => step3())
  .then(() => step4())
  .then(() => step5());

// 更好: 使用 async/await
async function process() {
  await step1();
  await step2();
  await step3();
  await step4();
  await step5();
}
```

## 常见面试题

### 经典题 1

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

Promise.resolve().then(() => {
  console.log('4');
  setTimeout(() => console.log('5'), 0);
});

console.log('6');

// 输出: 1, 6, 3, 4, 2, 5
```

### 经典题 2

```javascript
async function async1() {
  console.log('async1 start');
  await async2();
  console.log('async1 end');
}

async function async2() {
  console.log('async2');
}

console.log('script start');

setTimeout(() => console.log('setTimeout'), 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then(() => console.log('promise2'));

console.log('script end');

// 输出: script start -> async1 start -> async2 -> promise1 -> script end -> async1 end -> promise2 -> setTimeout
```

## 总结

JavaScript 事件循环机制是理解异步编程的核心:

1. **单线程执行**: JavaScript 在主线程上同步执行代码
2. **任务队列**: 宏任务和微任务分别维护各自的队列
3. **执行顺序**: 微任务优先于宏任务执行
4. **实际应用**: 合理使用异步操作提升性能和用户体验
5. **注意事项**: 避免任务饥饿,正确处理异步边界

掌握事件循环机制有助于编写更高效、更可靠的 JavaScript 代码。