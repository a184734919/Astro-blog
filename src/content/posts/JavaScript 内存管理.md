---
title: JavaScript 内存管理
published: 2022-07-17
description: '垃圾回收机制与内存泄漏的详细介绍和学习笔记'
image: ''
tags: ["JS","内存"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 具有自动垃圾回收机制,开发者不需要手动分配和释放内存。但了解内存管理原理对于编写高性能代码和避免内存泄漏至关重要。

## 内存生命周期

### 三个阶段

1. **分配**: 声明变量、对象等时自动分配内存
2. **使用**: 读写已分配的内存
3. **释放**: 不再使用时自动回收

```javascript
// 1. 分配
let person = {
  name: 'Alice',
  age: 25
};

// 2. 使用
person.age = 26;

// 3. 释放 (自动)
person = null; // 引用解除,等待 GC 回收
```

## 垃圾回收机制

### 引用计数算法

```javascript
let obj1 = { name: 'Object 1' }; // 引用计数: 1
let obj2 = obj1;                 // 引用计数: 2
obj1 = null;                     // 引用计数: 1
obj2 = null;                     // 引用计数: 0, 可回收

// 循环引用问题
function createCycle() {
  let objA = {};
  let objB = {};
  objA.ref = objB;
  objB.ref = objA;
  
  // 函数执行后,objA 和 objB 仍然相互引用
  // 引用计数算法无法回收它们
}
```

### 标记-清除算法

现代 JavaScript 引擎主要使用标记-清除算法:

```javascript
// 1. 标记阶段: 从根对象开始遍历,标记所有可达对象
// 2. 清除阶段: 回收所有未标记的对象

function example() {
  let root = { name: 'Root' };
  let child1 = { name: 'Child1', parent: root };
  let child2 = { name: 'Child2', parent: root };
  
  // root, child1, child2 都会被标记为可达
}

example();
// 函数执行后,所有对象都不可达,会被回收
```

## V8 垃圾回收优化

### 分代回收

V8 将内存分为新生代和老生代:

```javascript
// 新生代: 存放生命周期短的对象
function createShortLivedObjects() {
  for (let i = 0; i < 1000; i++) {
    let temp = { id: i };
    // 这些对象很可能很快被回收
  }
}

// 老生代: 存放生命周期长的对象
const config = {
  apiUrl: 'https://api.example.com',
  version: '1.0.0'
};
// config 会长期存在
```

### Scavenge 算法 (新生代)

```javascript
// 将内存分为 From 空间和 To 空间
// 存活对象从 From 复制到 To
// 交换 From 和 To

function simulateScavenge() {
  const fromSpace = [
    { id: 1, alive: true },
    { id: 2, alive: false },
    { id: 3, alive: true }
  ];
  
  const toSpace = fromSpace.filter(obj => obj.alive);
  // 只有存活的对象被复制到 toSpace
}
```

### 标记-整理 (老生代)

```javascript
// 标记-清除 + 内存整理,减少碎片
function simulateMarkCompact() {
  const memory = [
    null, { id: 1 }, null, null, { id: 2 }, null, { id: 3 }
  ];
  
  // 整理后
  const compacted = memory.filter(obj => obj !== null);
  // [{ id: 1 }, { id: 2 }, { id: 3 }]
}
```

## 内存泄漏常见场景

### 1. 全局变量

```javascript
// 错误示例
function process() {
  globalData = []; // 隐式全局变量
}

// 正确做法
function process() {
  const data = [];
  // 使用完后自动释放
}

// 或者使用模块作用域
const processData = (() => {
  let data = [];
  return {
    add(item) {
      data.push(item);
    },
    clear() {
      data = [];
    }
  };
})();
```

### 2. 闭包

```javascript
// 错误示例
function createHandler() {
  let largeArray = new Array(1000000).fill('data');
  
  return function() {
    console.log('Handler called');
    // largeArray 一直被引用,无法释放
  };
}

const handler = createHandler();

// 正确做法
function createHandler() {
  let largeArray = new Array(1000000).fill('data');
  
  let processed = processData(largeArray);
  
  return function() {
    console.log('Handler called');
    console.log(processed);
  };
}

// 或者在不需要时手动清理
function createHandler() {
  let largeArray = new Array(1000000).fill('data');
  
  return function() {
    console.log('Handler called');
  };
}

const handler = createHandler();
handler = null; // 解除引用
```

### 3. 定时器

```javascript
// 错误示例
function startPolling() {
  setInterval(() => {
    fetch('/api/status')
      .then(data => console.log(data));
  }, 1000);
}

// 正确做法: 保存定时器 ID
let timerId;

function startPolling() {
  timerId = setInterval(() => {
    fetch('/api/status')
      .then(data => console.log(data));
  }, 1000);
}

function stopPolling() {
  clearInterval(timerId);
}
```

### 4. DOM 引用

```javascript
// 错误示例
let elements = [];

function cacheElements() {
  elements = document.querySelectorAll('.item');
  // 即使 DOM 被删除,elements 仍然保持引用
}

// 正确做法: 使用弱引用或手动清理
let elementMap = new WeakMap();

function cacheElements() {
  const items = document.querySelectorAll('.item');
  items.forEach(item => {
    elementMap.set(item, { cached: true });
  });
  // WeakMap 不会阻止垃圾回收
}
```

### 5. 事件监听器

```javascript
// 错误示例
function setupListeners() {
  const button = document.getElementById('button');
  button.addEventListener('click', () => {
    console.log('Clicked');
    // 如果 button 被移除,监听器仍然存在
  });
}

// 正确做法: 移除监听器
let clickHandler;

function setupListeners() {
  const button = document.getElementById('button');
  clickHandler = () => console.log('Clicked');
  button.addEventListener('click', clickHandler);
}

function cleanupListeners() {
  const button = document.getElementById('button');
  if (button && clickHandler) {
    button.removeEventListener('click', clickHandler);
  }
}
```

## 内存管理最佳实践

### 1. 及时解除引用

```javascript
function processData() {
  const data = fetchLargeData();
  const result = transform(data);
  
  // 及时释放大对象
  data = null;
  
  return result;
}
```

### 2. 避免不必要的对象创建

```javascript
// 不好的做法
function processArray(arr) {
  return arr.map(item => {
    return {
      ...item,
      processed: true
    };
  });
}

// 更好的做法
function processArray(arr) {
  return arr.map(item => ({
    ...item,
    processed: true
  }));
}
```

### 3. 使用对象池

```javascript
class ObjectPool {
  constructor(createFn, resetFn) {
    this.pool = [];
    this.createFn = createFn;
    this.resetFn = resetFn;
  }
  
  acquire() {
    return this.pool.length > 0 
      ? this.pool.pop()
      : this.createFn();
  }
  
  release(obj) {
    this.resetFn(obj);
    this.pool.push(obj);
  }
}

// 使用对象池
const pool = new ObjectPool(
  () => ({ data: [], index: 0 }),
  (obj) => { obj.data = []; obj.index = 0; }
);

function processWithPool() {
  const obj = pool.acquire();
  // 使用 obj
  obj.data.push('item');
  pool.release(obj);
}
```

### 4. 使用 WeakMap 和 WeakSet

```javascript
// WeakMap 不会阻止垃圾回收
const metadata = new WeakMap();

function addMetadata(obj, meta) {
  metadata.set(obj, meta);
}

// 当 obj 不再被引用时,meta 也会被自动回收
```

## 内存分析工具

### Chrome DevTools

```javascript
// 1. 打开 Chrome DevTools (F12)
// 2. 切换到 Memory 面板
// 3. 使用 Heap Snapshot 分析堆内存

// 示例: 触发内存快照
function takeSnapshot() {
  // 在代码中设置断点
  // 在 DevTools 中点击 "Take snapshot"
}

function forceGC() {
  // 在 DevTools Console 中执行
  if (window.gc) {
    window.gc(); // 手动触发垃圾回收
  }
}
```

### 性能监控

```javascript
// 使用 Performance API
const performanceObserver = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log('Memory:', entry);
  }
});

performanceObserver.observe({ entryTypes: ['measure'] });

// 测量内存使用
function measureMemory() {
  if (performance.memory) {
    console.log('Used:', performance.memory.usedJSHeapSize / 1024 / 1024, 'MB');
    console.log('Total:', performance.memory.totalJSHeapSize / 1024 / 1024, 'MB');
  }
}
```

## 实际应用案例

### 大数据列表优化

```javascript
// 虚拟滚动,只渲染可见区域
class VirtualScroll {
  constructor(container, itemHeight, data) {
    this.container = container;
    this.itemHeight = itemHeight;
    this.data = data;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight) + 2;
  }
  
  render(startIndex) {
    const visibleData = this.data.slice(
      startIndex,
      startIndex + this.visibleCount
    );
    
    this.container.innerHTML = visibleData.map(item => `
      <div style="height:${this.itemHeight}px">${item}</div>
    `).join('');
  }
}
```

### Canvas 内存管理

```javascript
class CanvasManager {
  constructor(canvas) {
    this.canvas = canvas;
    this.context = canvas.getContext('2d');
    this.imageCache = new Map();
  }
  
  loadImage(src) {
    if (this.imageCache.has(src)) {
      return Promise.resolve(this.imageCache.get(src));
    }
    
    return new Promise((resolve) => {
      const img = new Image();
      img.onload = () => {
        this.imageCache.set(src, img);
        resolve(img);
      };
      img.src = src;
    });
  }
  
  clearCache() {
    this.imageCache.clear();
  }
}
```

## 性能优化技巧

### 1. 避免频繁的内存分配

```javascript
// 不好的做法
function animate() {
  const tempArray = new Array(1000);
  // 使用 tempArray
  requestAnimationFrame(animate);
}

// 更好的做法
function animate() {
  // 重用数组
  if (!this.tempArray) {
    this.tempArray = new Array(1000);
  }
  // 使用 this.tempArray
  requestAnimationFrame(animate);
}
```

### 2. 使用 TypedArray

```javascript
// 处理大量数值数据时使用 TypedArray
const data = new Float64Array(1000000);

for (let i = 0; i < data.length; i++) {
  data[i] = Math.random();
}

// 比普通数组更节省内存
```

### 3. 字符串优化

```javascript
// 避免字符串拼接创建临时对象
// 不好的做法
let result = '';
for (let i = 0; i < 1000; i++) {
  result += 'item';
}

// 更好的做法
const parts = [];
for (let i = 0; i < 1000; i++) {
  parts.push('item');
}
const result = parts.join('');

// 使用模板字符串
const result = Array(1000).fill('item').join('');
```

## 总结

JavaScript 内存管理的关键点:

1. **自动管理**: JavaScript 有自动垃圾回收,但仍需注意内存使用
2. **回收机制**: 理解引用计数和标记-清除算法
3. **V8 优化**: 新生代和老生代的分代回收策略
4. **避免泄漏**: 注意全局变量、闭包、定时器、DOM 引用等场景
5. **最佳实践**: 及时解除引用、使用对象池、WeakMap 等
6. **工具支持**: 使用 Chrome DevTools 进行内存分析
7. **性能优化**: 减少不必要的内存分配,优化数据结构

良好的内存管理习惯能够显著提升应用的性能和稳定性。