---
title: JavaScript 内存管理
published: 2022-07-17
description: '深入理解 JavaScript 内存分配、垃圾回收机制与内存泄漏预防'
image: ''
tags: ["JS","内存"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 内存管理

## 概述

JavaScript 具有自动内存管理机制，开发者不需要手动分配和释放内存，但这并不意味着可以忽视内存问题。理解内存管理原理对于编写高性能代码、避免内存泄漏至关重要，特别是在构建大型应用或处理大量数据时。

## 内存基础

### 内存生命周期

```javascript
// 内存管理的三个阶段：
// 1. 分配：声明变量、对象、函数时自动分配内存
// 2. 使用：读写已分配的内存
// 3. 释放：不再使用时由垃圾回收器自动回收

// 1. 内存分配
let number = 42;                    // 分配数字内存
let string = "Hello, World";       // 分配字符串内存
let object = { name: '张三' };      // 分配对象内存
let array = [1, 2, 3];             // 分配数组内存

// 2. 内存使用
number = number + 8;                // 使用和修改数字
string = string.toUpperCase();      // 使用和修改字符串
object.name = '李四';               // 使用和修改对象
array.push(4);                      // 使用和修改数组

// 3. 内存释放（手动解除引用，等待垃圾回收）
number = null;
string = null;
object = null;
array = null;
```

### 栈内存和堆内存

```javascript
// 栈内存：存储基本类型和引用类型的引用
function stackAndHeapExample() {
  // 基本类型存储在栈中
  let number = 42;
  let string = "Hello";
  let boolean = true;

  // 引用类型的引用存储在栈中，实际对象存储在堆中
  let object = { name: '张三' };     // 栈中存储对象引用，堆中存储对象数据
  let array = [1, 2, 3];           // 栈中存储数组引用，堆中存储数组数据

  // 栈内存的特点：
  // - 自动分配和释放
  // - 存储速度快
  // - 内存空间有限
  // - 按顺序存取（LIFO）

  // 堆内存的特点：
  // - 动态分配
  // - 存储空间大
  // - 需要手动管理引用
  // - 访问速度相对较慢
}

stackAndHeapExample();
```

### 垃圾回收机制

```javascript
// JavaScript 主要使用两种垃圾回收算法：

// 1. 引用计数算法（已废弃，现代引擎不再使用）
function referenceCountingExample() {
  let obj1 = { name: 'Object 1' }; // 引用计数: 1
  let obj2 = obj1;                 // 引用计数: 2
  obj1 = null;                     // 引用计数: 1
  obj2 = null;                     // 引用计数: 0 -> 可回收

  // 循环引用问题（引用计数算法的致命缺陷）
  function createCycleReference() {
    let objA = { name: 'A' };
    let objB = { name: 'B' };

    objA.ref = objB;
    objB.ref = objA;

    // 函数执行后，objA 和 objB 相互引用
    // 引用计数无法回收它们，导致内存泄漏
  }

  createCycleReference();
}

// 2. 标记-清除算法（现代引擎使用）
function markAndSweepExample() {
  let root = { name: 'Root' };     // 可达对象（从全局可访问）
  let child = { name: 'Child' };   // 不可达对象（被局部变量引用）

  root.child = child;              // child 变为可达对象

  // 垃圾回收过程：
  // 1. 标记阶段：从根对象开始，递归标记所有可达对象
  // 2. 清除阶段：回收所有未被标记的对象
}

markAndSweepExample();

// 函数执行后，root 对象不可达，将被回收
```

## V8 垃圾回收机制

### 分代回收

```javascript
// V8 引擎采用分代回收策略，将内存分为两代：

// 1. 新生代（New Space）
//   - 存储生命周期短的对象
//   - 空间小（通常 1-8MB）
//   - 使用 Scavenge 算法

function createShortLivedObjects() {
  for (let i = 0; i < 1000; i++) {
    let temp = { id: i, data: 'temp data' };
    // 这些对象很可能在下次 GC 中被回收
  }
}

createShortLivedObjects();

// 2. 老生代（Old Space）
//   - 存储生命周期长的对象
//   - 空间大
//   - 使用标记-清除 + 标记-整理算法

const longLivedConfig = {
  apiUrl: 'https://api.example.com',
  version: '1.0.0',
  timeout: 5000
};
// 这个对象会长期存在，最终移到老生代
```

### Scavenge 算法

```javascript
// Scavenge 算法将新生代内存分为两半：From 空间和 To 空间

function scavengeAlgorithmExample() {
  // 模拟 Scavenge 算法的执行过程
  const fromSpace = [
    { id: 1, data: 'active', alive: true },
    { id: 2, data: 'garbage', alive: false },
    { id: 3, data: 'active', alive: true },
    { id: 4, data: 'garbage', alive: false }
  ];

  console.log('GC 开始前 - From 空间:', fromSpace.length);

  // Scavenge 算法执行：
  // 1. 从 From 空间复制存活对象到 To 空间
  // 2. 存活对象年龄加 1
  // 3. 如果对象年龄超过阈值，晋升到老生代
  // 4. 交换 From 和 To 空间
  // 5. 清空新的 From 空间

  const toSpace = fromSpace.filter(obj => obj.alive);

  // 存活对象年龄加 1
  toSpace.forEach(obj => {
    obj.age = (obj.age || 0) + 1;
  });

  // 晋升条件（对象年龄 >= 2）
  const promoted = toSpace.filter(obj => obj.age >= 2);
  const retained = toSpace.filter(obj => obj.age < 2);

  console.log('GC 执行后 - To 空间:', retained.length);
  console.log('晋升到老生代:', promoted.length);
  console.log('回收对象:', fromSpace.length - toSpace.length);
}

scavengeAlgorithmExample();
```

### 标记-清除与标记-整理

```javascript
// 老生代使用标记-清除和标记-整理算法

function markSweepCompactExample() {
  // 模拟内存碎片情况
  const memory = [
    null,              // 碎片
    { id: 1, alive: true },
    null,              // 碎片
    null,              // 碎片
    { id: 2, alive: true },
    null,              // 碎片
    { id: 3, alive: true },
    null,              // 碎片
    null               // 碎片
  ];

  console.log('整理前:', memory.filter(item => item !== null).length, '个对象');

  // 标记-整理算法执行：
  // 1. 标记阶段：标记所有可达对象
  // 2. 整理阶段：将存活对象向一端移动，消除碎片
  // 3. 清除阶段：回收剩余空间

  const compacted = memory.filter(item => item !== null);

  console.log('整理后:', compacted.length, '个对象');
  console.log('碎片消除:', memory.length - compacted.length, '个空位');

  return compacted;
}

const cleanMemory = markSweepCompactExample();
```

## 垃圾回收优化

### 对象池模式

```javascript
// 使用对象池减少内存分配和垃圾回收压力
class ObjectPool {
  constructor(createFn, resetFn, initialSize = 10) {
    this.createFn = createFn;
    this.resetFn = resetFn;
    this.pool = [];
    this.activeObjects = new Set();

    // 预分配对象
    for (let i = 0; i < initialSize; i++) {
      this.pool.push(this.createFn());
    }
  }

  acquire() {
    // 从池中获取对象
    if (this.pool.length > 0) {
      const obj = this.pool.pop();
      this.activeObjects.add(obj);
      return obj;
    }

    // 池为空时创建新对象
    const newObj = this.createFn();
    this.activeObjects.add(newObj);
    return newObj;
  }

  release(obj) {
    if (this.activeObjects.has(obj)) {
      this.activeObjects.delete(obj);
      this.resetFn(obj);
      this.pool.push(obj);
    }
  }

  getStats() {
    return {
      poolSize: this.pool.length,
      activeObjects: this.activeObjects.size,
      totalCreated: this.pool.length + this.activeObjects.size
    };
  }
}

// 使用示例：创建粒子系统的对象池
const particlePool = new ObjectPool(
  () => ({
    x: 0,
    y: 0,
    vx: 0,
    vy: 0,
    life: 0,
    color: 'white'
  }),
  (particle) => {
    particle.x = 0;
    particle.y = 0;
    particle.vx = 0;
    particle.vy = 0;
    particle.life = 0;
    particle.color = 'white';
  },
  1000
);

// 获取粒子
const particle1 = particlePool.acquire();
particle1.x = 100;
particle1.y = 200;
particle1.life = 1.0;

// 使用完后归还
particlePool.release(particle1);

console.log('池状态:', particlePool.getStats());
```

### 内存监控

```javascript
// 使用 Chrome DevTools 内存面板
function takeHeapSnapshot() {
  // 在浏览器中：
  // 1. 打开 Chrome DevTools (F12)
  // 2. 切换到 Memory 面板
  // 3. 选择 "Heap snapshot"
  // 4. 点击 "Take snapshot"
  console.log('请在 Chrome DevTools 中手动拍摄堆快照');
}

// 使用 Performance API 监控内存
function monitorMemory() {
  if (performance.memory) {
    const memory = performance.memory;
    const usedMB = memory.usedJSHeapSize / 1024 / 1024;
    const totalMB = memory.totalJSHeapSize / 1024 / 1024;
    const limitMB = memory.jsHeapSizeLimit / 1024 / 1024;

    console.log('内存使用情况:');
    console.log(`已使用: ${usedMB.toFixed(2)} MB`);
    console.log(`总共分配: ${totalMB.toFixed(2)} MB`);
    console.log(`内存限制: ${limitMB.toFixed(2)} MB`);
    console.log(`使用率: ${(usedMB / limitMB * 100).toFixed(2)}%`);

    return {
      used: usedMB,
      total: totalMB,
      limit: limitMB,
      usage: usedMB / limitMB
    };
  } else {
    console.log('当前浏览器不支持 performance.memory API');
    return null;
  }
}

// 定期监控内存
setInterval(() => {
  const stats = monitorMemory();
  if (stats && stats.usage > 0.9) {
    console.warn('内存使用率过高，请检查内存泄漏！');
  }
}, 5000);
```

### 手动触发垃圾回收

```javascript
// 在 Chrome 中手动触发垃圾回收（需要启动参数）
function manualGarbageCollection() {
  // 需要以 --js-flags="--expose-gc" 参数启动 Chrome
  if (typeof gc !== 'undefined') {
    console.log('触发手动垃圾回收');
    gc();
  } else {
    console.log('无法手动触发垃圾回收');
    console.log('请使用以下参数启动 Chrome：');
    console.log('--js-flags="--expose-gc"');
  }
}

// 或者使用 Chrome DevTools
function forceGCInDevTools() {
  // 在 DevTools Console 中执行：
  // window.gc(); // 手动触发 GC
  console.log('在 DevTools Console 中输入 window.gc() 手动触发 GC');
}
```

## 内存泄漏场景与预防

### 1. 全局变量

```javascript
// 错误示例：意外创建全局变量
function processData() {
  globalData = []; // 隐式全局变量
  window.importantData = {}; // 显式全局变量
}

function processDataCorrect() {
  const data = []; // 局部变量
  // 使用完后自动释放
}

// 使用模块化避免全局变量
const DataProcessor = (() => {
  let privateData = [];

  return {
    addData(item) {
      privateData.push(item);
    },
    getData() {
      return [...privateData]; // 返回副本
    },
    clear() {
      privateData = [];
    }
  };
})();

// 使用
DataProcessor.addData({ id: 1, name: 'Item 1' });
const data = DataProcessor.getData();
DataProcessor.clear();
```

### 2. 闭包内存泄漏

```javascript
// 错误示例：闭包引用大对象
function createClosureLeak() {
  const largeData = new Array(1000000).fill('data');

  return function() {
    console.log('Function called');
    // largeData 一直被引用，无法释放
  };
}

const leak = createClosureLeak();

// 解决方案 1：只引用必要的数据
function createClosureFixed() {
  const largeData = new Array(1000000).fill('data');
  const summary = { length: largeData.length };
  largeData = null; // 解除引用

  return function() {
    console.log('Data length:', summary.length);
  };
}

// 解决方案 2：手动清理
function createClosureWithCleanup() {
  let largeData = new Array(1000000).fill('data');

  const closure = function() {
    if (largeData) {
      console.log('Processing data...');
      // 使用数据
    } else {
      console.log('Data already cleaned');
    }
  };

  // 提供清理方法
  closure.cleanup = function() {
    largeData = null;
    console.log('Closure cleaned up');
  };

  return closure;
}

const cleanableClosure = createClosureWithCleanup();
cleanableClosure();
cleanableClosure.cleanup();
```

### 3. 定时器内存泄漏

```javascript
// 错误示例：未清理的定时器
function startPolling() {
  setInterval(() => {
    fetch('/api/status')
      .then(response => response.json())
      .then(data => console.log('Status:', data));
  }, 1000);
}

startPolling();
// 定时器一直在运行，回调函数一直被引用

// 正确做法：保存定时器 ID 并提供清理方法
class PollingManager {
  constructor() {
    this.timerIds = [];
  }

  start(url, interval = 1000) {
    const timerId = setInterval(() => {
      fetch(url)
        .then(response => response.json())
        .then(data => console.log('Data:', data));
    }, interval);

    this.timerIds.push(timerId);
    return timerId;
  }

  stop(timerId) {
    clearInterval(timerId);
    this.timerIds = this.timerIds.filter(id => id !== timerId);
  }

  stopAll() {
    this.timerIds.forEach(id => clearInterval(id));
    this.timerIds = [];
  }
}

const polling = new PollingManager();
const timerId = polling.start('/api/status', 1000);

// 停止定时器
polling.stop(timerId);
// 或停止所有定时器
polling.stopAll();
```

### 4. DOM 引用内存泄漏

```javascript
// 错误示例：DOM 元素被移除但 JavaScript 仍引用
let cachedElements = [];

function cacheDOMElements() {
  cachedElements = document.querySelectorAll('.item');
  // 即使 DOM 被删除，cachedElements 仍然保持引用
}

document.addEventListener('DOMContentLoaded', cacheDOMElements);

// 解决方案 1：手动清理
function clearDOMCache() {
  cachedElements = [];
}

// 解决方案 2：使用 WeakMap
const elementCache = new WeakMap();

function cacheElementWithWeakMap(element, data) {
  elementCache.set(element, data);
  // WeakMap 不会阻止垃圾回收
}

// DOM 元素被移除后，相关数据会自动回收
```

### 5. 事件监听器内存泄漏

```javascript
// 错误示例：未移除的事件监听器
function setupEventListeners() {
  const buttons = document.querySelectorAll('.button');
  buttons.forEach(button => {
    button.addEventListener('click', function() {
      console.log('Button clicked');
    });
  });
}

setupEventListeners();
// 即使按钮被移除，监听器仍然存在

// 正确做法：保存监听器并移除
class EventManager {
  constructor() {
    this.listeners = new Map();
  }

  addListener(element, eventType, handler) {
    element.addEventListener(eventType, handler);

    // 保存监听器信息
    const elementListeners = this.listeners.get(element) || [];
    elementListeners.push({ eventType, handler });
    this.listeners.set(element, elementListeners);
  }

  removeListener(element, eventType, handler) {
    element.removeEventListener(eventType, handler);

    const elementListeners = this.listeners.get(element) || [];
    const updated = elementListeners.filter(
      listener => !(listener.eventType === eventType && listener.handler === handler)
    );

    if (updated.length > 0) {
      this.listeners.set(element, updated);
    } else {
      this.listeners.delete(element);
    }
  }

  removeAllListeners(element) {
    const elementListeners = this.listeners.get(element) || [];
    elementListeners.forEach(({ eventType, handler }) => {
      element.removeEventListener(eventType, handler);
    });
    this.listeners.delete(element);
  }

  clearAll() {
    this.listeners.forEach((listeners, element) => {
      listeners.forEach(({ eventType, handler }) => {
        element.removeEventListener(eventType, handler);
      });
    });
    this.listeners.clear();
  }
}

// 使用示例
const eventManager = new EventManager();

const button = document.createElement('button');
button.textContent = 'Click me';

function handleClick() {
  console.log('Button clicked');
}

eventManager.addListener(button, 'click', handleClick);

// 移除单个监听器
eventManager.removeListener(button, 'click', handleClick);

// 移除元素的所有监听器
eventManager.removeAllListeners(button);

// 清除所有监听器
eventManager.clearAll();
```

## 内存优化技巧

### 1. 数据结构优化

```javascript
// 使用适合的数据结构减少内存占用

// 使用 TypedArray 处理数值数据
const regularArray = new Array(1000000).fill(0);
const typedArray = new Int32Array(1000000); // 更节省内存

console.log('常规数组内存:', regularArray.length * 8); // 约 8MB
console.log('TypedArray 内存:', typedArray.byteLength); // 约 4MB

// 使用 Set 和 Map 替代对象
const regularObject = {};
const map = new Map();

// 对于大量键值对，Map 通常更高效
for (let i = 0; i < 10000; i++) {
  regularObject[`key${i}`] = `value${i}`;
  map.set(`key${i}`, `value${i}`);
}

// 使用 Set 替代数组进行成员检查
const array = [1, 2, 3, 4, 5];
const set = new Set(array);

console.log(array.includes(3)); // O(n) 复杂度
console.log(set.has(3));        // O(1) 复杂度
```

### 2. 字符串优化

```javascript
// 字符串拼接优化

// 不好的做法：频繁创建临时字符串
let result = '';
for (let i = 0; i < 1000; i++) {
  result += 'item' + i;
}

// 更好的做法：使用数组 join
const parts = [];
for (let i = 0; i < 1000; i++) {
  parts.push('item' + i);
}
const result = parts.join('');

// 使用模板字符串
const result = Array(1000).fill(0).map((_, i) => `item${i}`).join('');

// 字符串缓存
const stringCache = new Map();

function getCachedString(template, values) {
  const cacheKey = template + JSON.stringify(values);

  if (stringCache.has(cacheKey)) {
    return stringCache.get(cacheKey);
  }

  const result = template.replace(/\{(\d+)\}/g, (_, index) => values[index]);
  stringCache.set(cacheKey, result);

  return result;
}

// 使用缓存
const str1 = getCachedString('Hello, {0}!', ['World']);
const str2 = getCachedString('Hello, {0}!', ['World']); // 从缓存获取
```

### 3. 对象优化

```javascript
// 避免不必要的对象创建

// 不好的做法：每次调用都创建新对象
function createPointBad(x, y) {
  return { x, y };
}

for (let i = 0; i < 1000; i++) {
  const point = createPointBad(i, i * 2); // 创建 1000 个对象
}

// 更好的做法：重用对象
const reusablePoint = { x: 0, y: 0 };

function updatePoint(x, y) {
  reusablePoint.x = x;
  reusablePoint.y = y;
  return reusablePoint;
}

for (let i = 0; i < 1000; i++) {
  const point = updatePoint(i, i * 2); // 只创建 1 个对象
}

// 使用对象池（前面已展示）
// 使用 flyweight 模式共享相同数据
const flyweightFactory = {
  data: new Map(),

  get(key) {
    if (this.data.has(key)) {
      return this.data.get(key);
    }

    const value = this.createValue(key);
    this.data.set(key, value);
    return value;
  },

  createValue(key) {
    return { key, data: `Data for ${key}` };
  }
};

// 共享相同数据
const value1 = flyweightFactory.get('shared');
const value2 = flyweightFactory.get('shared');
console.log(value1 === value2); // true - 相同的引用
```

### 4. 数组优化

```javascript
// 数组操作优化

// 预分配数组大小
const array1 = [];
for (let i = 0; i < 10000; i++) {
  array1.push(i); // 可能需要多次扩容
}

const array2 = new Array(10000);
for (let i = 0; i < 10000; i++) {
  array2[i] = i; // 预分配，性能更好
}

// 使用适当的方法处理数组
// 不好的做法
function filterMapBad(arr) {
  return arr
    .filter(item => item > 0)
    .map(item => item * 2);
}

// 更好的做法：一次遍历
function filterMapBetter(arr) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] > 0) {
      result.push(arr[i] * 2);
    }
  }
  return result;
}

// 使用类型化数组处理大量数值数据
const float32Array = new Float32Array(1000000);
const int32Array = new Int32Array(1000000);
const uint8Array = new Uint8Array(1000000); // 最节省内存
```

## 内存分析工具

### Chrome DevTools Memory 面板

```javascript
// 使用 Chrome DevTools 进行内存分析

function memoryAnalysisExample() {
  console.log('=== 内存分析指南 ===');

  console.log('1. Heap Snapshots（堆快照）');
  console.log('   - 拍摄当前内存状态快照');
  console.log('   - 对比两个快照，发现内存泄漏');
  console.log('   - 分析对象引用关系');

  console.log('2. Allocation Timeline（分配时间线）');
  console.log('   - 记录内存分配随时间的变化');
  console.log('   - 识别内存分配热点');
  console.log('   - 发现频繁的内存分配');

  console.log('3. Allocation Sampling（分配采样）');
  console.log('   - 轻量级的内存分配分析');
  console.log('   - 适合长时间运行的应用');
  console.log('   - 对性能影响较小');

  console.log('使用步骤：');
  console.log('1. 打开 Chrome DevTools (F12)');
  console.log('2. 切换到 Memory 面板');
  console.log('3. 选择分析类型');
  console.log('4. 开始录制');
  console.log('5. 执行要分析的代码');
  console.log('6. 停止录制并分析结果');
}

memoryAnalysisExample();
```

### 内存泄漏检测

```javascript
// 简单的内存泄漏检测器
class MemoryLeakDetector {
  constructor(threshold = 100) {
    this.snapshots = [];
    this.threshold = threshold;
  }

  takeSnapshot(label) {
    if (!performance.memory) {
      console.warn('当前浏览器不支持 performance.memory');
      return;
    }

    const snapshot = {
      label,
      timestamp: Date.now(),
      memory: {
        used: performance.memory.usedJSHeapSize,
        total: performance.memory.totalJSHeapSize,
        limit: performance.memory.jsHeapSizeLimit
      }
    };

    this.snapshots.push(snapshot);
    console.log(`快照 "${label}":`, this.formatMemory(snapshot.memory));

    this.detectLeaks();
  }

  detectLeaks() {
    if (this.snapshots.length < 2) return;

    const latest = this.snapshots[this.snapshots.length - 1];
    const previous = this.snapshots[this.snapshots.length - 2];
    const diff = latest.memory.used - previous.memory.used;
    const diffMB = diff / 1024 / 1024;

    console.log(`内存变化: ${diffMB >= 0 ? '+' : ''}${diffMB.toFixed(2)} MB`);

    if (diffMB > this.threshold) {
      console.warn(`可能存在内存泄漏！内存增长了 ${diffMB.toFixed(2)} MB`);
    }
  }

  formatMemory(memory) {
    return {
      used: `${(memory.used / 1024 / 1024).toFixed(2)} MB`,
      total: `${(memory.total / 1024 / 1024).toFixed(2)} MB`,
      usage: `${(memory.used / memory.limit * 100).toFixed(2)}%`
    };
  }

  compareSnapshots(index1, index2) {
    if (index1 >= this.snapshots.length || index2 >= this.snapshots.length) {
      console.error('无效的快照索引');
      return;
    }

    const snapshot1 = this.snapshots[index1];
    const snapshot2 = this.snapshots[index2];
    const diff = snapshot2.memory.used - snapshot1.memory.used;

    console.log(`比较快照 "${snapshot1.label}" 和 "${snapshot2.label}":`);
    console.log(`内存变化: ${(diff / 1024 / 1024).toFixed(2)} MB`);
    console.log(`时间差: ${((snapshot2.timestamp - snapshot1.timestamp) / 1000).toFixed(2)} 秒`);
  }
}

// 使用示例
const leakDetector = new MemoryLeakDetector(10);

// 初始快照
leakDetector.takeSnapshot('初始状态');

// 执行可能产生内存泄漏的代码
function simulateLeak() {
  const data = [];
  for (let i = 0; i < 10000; i++) {
    data.push(new Array(100).fill('data'));
  }
  return data;
}

const leakedData = simulateLeak();
leakDetector.takeSnapshot('操作后');

// 对比快照
leakDetector.compareSnapshots(0, 1);
```

## 最佳实践

### 1. 及时释放引用

```javascript
// 及时释放不再需要的引用
function processData() {
  // 获取大数据
  const largeData = fetchLargeData();

  // 处理数据
  const processed = processLargeData(largeData);

  // 立即释放大数据引用
  largeData = null;

  return processed;
}

// 使用 finally 确保释放
function safeProcess() {
  let resource = null;

  try {
    resource = acquireResource();
    return processResource(resource);
  } catch (error) {
    handleError(error);
  } finally {
    if (resource) {
      releaseResource(resource);
      resource = null;
    }
  }
}
```

### 2. 使用 WeakMap 和 WeakSet

```javascript
// WeakMap 不会阻止垃圾回收
const cache = new WeakMap();

function getCachedData(obj) {
  if (cache.has(obj)) {
    return cache.get(obj);
  }

  const data = computeData(obj);
  cache.set(obj, data);
  return data;
}

// WeakSet 用于跟踪对象
const processedObjects = new WeakSet();

function processObject(obj) {
  if (processedObjects.has(obj)) {
    console.log('对象已处理');
    return;
  }

  // 处理对象
  doProcessing(obj);

  // 标记为已处理
  processedObjects.add(obj);
}

// 当 obj 不再被其他地方引用时，相关缓存会被自动回收
```

### 3. 分批处理大数据

```javascript
// 分批处理大数据，避免内存峰值
async function processLargeArray(array, batchSize = 1000) {
  const results = [];

  for (let i = 0; i < array.length; i += batchSize) {
    const batch = array.slice(i, i + batchSize);
    const batchResults = await processBatch(batch);
    results.push(...batchResults);

    // 让出主线程，允许垃圾回收
    await new Promise(resolve => setTimeout(resolve, 0));
  }

  return results;
}

async function processBatch(batch) {
  return batch.map(item => {
    // 处理单个项目
    return item * 2;
  });
}

// 使用示例
const largeArray = Array(10000).fill(0).map((_, i) => i);
processLargeArray(largeArray).then(results => {
  console.log('处理完成，结果数量:', results.length);
});
```

### 4. 定期清理

```javascript
// 定期清理缓存和无用数据
class CacheManager {
  constructor(maxSize = 1000, maxAge = 60000) {
    this.cache = new Map();
    this.timestamps = new Map();
    this.maxSize = maxSize;
    this.maxAge = maxAge;
  }

  set(key, value) {
    // 清理过期数据
    this.cleanup();

    // 检查大小限制
    if (this.cache.size >= this.maxSize) {
      // 移除最旧的数据
      const oldestKey = this.findOldestKey();
      this.delete(oldestKey);
    }

    this.cache.set(key, value);
    this.timestamps.set(key, Date.now());
  }

  get(key) {
    const timestamp = this.timestamps.get(key);
    if (!timestamp) return null;

    // 检查是否过期
    if (Date.now() - timestamp > this.maxAge) {
      this.delete(key);
      return null;
    }

    return this.cache.get(key);
  }

  delete(key) {
    this.cache.delete(key);
    this.timestamps.delete(key);
  }

  cleanup() {
    const now = Date.now();
    const expiredKeys = [];

    for (const [key, timestamp] of this.timestamps) {
      if (now - timestamp > this.maxAge) {
        expiredKeys.push(key);
      }
    }

    expiredKeys.forEach(key => this.delete(key));
  }

  findOldestKey() {
    let oldestKey = null;
    let oldestTimestamp = Infinity;

    for (const [key, timestamp] of this.timestamps) {
      if (timestamp < oldestTimestamp) {
        oldestTimestamp = timestamp;
        oldestKey = key;
      }
    }

    return oldestKey;
  }

  startAutoCleanup(interval = 10000) {
    this.cleanupInterval = setInterval(() => this.cleanup(), interval);
  }

  stopAutoCleanup() {
    if (this.cleanupInterval) {
      clearInterval(this.cleanupInterval);
      this.cleanupInterval = null;
    }
  }
}

// 使用示例
const cache = new CacheManager(1000, 60000);
cache.startAutoCleanup();

cache.set('key1', 'value1');
cache.set('key2', 'value2');

const value = cache.get('key1');
console.log('缓存值:', value);

// 停止自动清理
cache.stopAutoCleanup();
```

## 总结

JavaScript 内存管理是一个复杂但重要的主题。虽然 JavaScript 有自动垃圾回收机制，但理解内存原理和最佳实践对于编写高性能应用至关重要。

关键要点：

1. **内存生命周期**：理解分配、使用、释放三个阶段
2. **垃圾回收机制**：掌握标记-清除算法和 V8 的分代回收策略
3. **内存泄漏场景**：识别全局变量、闭包、定时器、DOM 引用等常见泄漏
4. **预防措施**：使用 WeakMap、及时清理引用、避免不必要的全局变量
5. **优化技巧**：对象池、数据结构优化、分批处理等
6. **分析工具**：熟练使用 Chrome DevTools 进行内存分析
7. **最佳实践**：及时释放引用、使用缓存管理、定期清理等

良好的内存管理习惯能够显著提升应用的性能和稳定性。在实际开发中，要特别注意长期运行的应用和大数据处理场景，定期进行内存分析和优化。