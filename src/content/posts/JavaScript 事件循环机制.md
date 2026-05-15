---
title: JavaScript 事件循环机制
published: 2022-07-11
description: '深入解析 JavaScript 事件循环、宏任务微任务队列与异步编程原理'
image: ''
tags: ["JS","事件循环"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 事件循环机制

## 概述

JavaScript 是单线程语言，但通过事件循环机制实现了异步编程，使其能够处理耗时操作而不阻塞主线程。理解事件循环对于编写高性能的 JavaScript 应用至关重要，特别是涉及网络请求、定时器、动画等异步操作的场景。

## 执行机制基础

### 单线程模型

```javascript
// JavaScript 是单线程执行
console.log('1');
console.log('2');
console.log('3');

// 输出：1, 2, 3（严格按顺序执行）

// 即使有异步操作，JavaScript 也只有一个执行线程
console.log('Start');
setTimeout(() => {
  console.log('Timeout');
}, 1000);
console.log('End');

// 输出：Start, End, Timeout
```

### 调用栈

```javascript
// 调用栈是 JavaScript 代码执行的工作区
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

// 执行顺序和栈的变化：
// 1. first() 入栈
// 2. second() 入栈
// 3. third() 入栈
// 4. third() 出栈，输出 "Third"
// 5. second() 出栈，输出 "Second End"
// 6. first() 出栈，输出 "First End"

// 输出：First -> Second -> Third -> Second End -> First End
```

### 同步和异步

```javascript
// 同步代码：立即执行
console.log('同步 1');
console.log('同步 2');

// 异步代码：稍后执行
setTimeout(() => {
  console.log('异步 1');
}, 1000);

console.log('同步 3');

// 输出顺序：同步 1 -> 同步 2 -> 同步 3 -> 异步 1
```

## 事件循环架构

### 基本组件

```javascript
// 事件循环的核心组件：
// 1. 调用栈（Call Stack）- 同步代码执行
// 2. Web APIs - 浏览器提供的异步API
// 3. 任务队列（Task Queue）- 宏任务队列
// 4. 微任务队列（Microtask Queue）
// 5. 事件循环（Event Loop）

// 完整的执行流程
console.log('1. 开始');

setTimeout(() => {
  console.log('4. setTimeout 回调');
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise.then 回调');
});

console.log('2. 结束');

// 执行顺序：
// 1. 调用栈执行：开始 -> 结束
// 2. 检查微任务队列：执行 Promise.then
// 3. 执行宏任务：setTimeout
// 4. 重复上述过程

// 输出：1 -> 2 -> 3 -> 4
```

### 事件循环流程

```javascript
// 事件循环的详细流程
function eventLoopSimulation() {
  const callStack = [];
  const microTaskQueue = [];
  const macroTaskQueue = [];

  function executeSyncCode() {
    console.log('执行同步代码');
    callStack.push('同步任务');
    callStack.pop();
  }

  function addMicroTask() {
    microTaskQueue.push('微任务');
  }

  function addMacroTask() {
    macroTaskQueue.push('宏任务');
  }

  function processMicroTasks() {
    while (microTaskQueue.length > 0) {
      const task = microTaskQueue.shift();
      console.log('执行微任务:', task);
    }
  }

  function processMacroTask() {
    if (macroTaskQueue.length > 0) {
      const task = macroTaskQueue.shift();
      console.log('执行宏任务:', task);
    }
  }

  return {
    run: function() {
      executeSyncCode();
      addMicroTask();
      addMacroTask();

      while (callStack.length === 0) {
        if (microTaskQueue.length > 0) {
          processMicroTasks();
        } else if (macroTaskQueue.length > 0) {
          processMacroTask();
        }
      }
    }
  };
}

eventLoopSimulation().run();
```

## 宏任务和微任务

### 宏任务（Macrotask）

```javascript
// 宏任务包括：
// 1. setTimeout
// 2. setInterval
// 3. setImmediate (Node.js)
// 4. I/O 操作
// 5. UI 渲染
// 6. MessageChannel

// 宏任务示例
console.log('1. 开始');

setTimeout(() => {
  console.log('3. setTimeout (宏任务)');
}, 0);

setInterval(() => {
  console.log('4. setInterval (宏任务)');
}, 1000);

setTimeout(() => {
  console.log('5. 嵌套 setTimeout');
  setTimeout(() => {
    console.log('6. 更深的嵌套');
  }, 0);
}, 0);

console.log('2. 结束');

// 执行顺序：1 -> 2 -> 3 -> 5 -> 4 -> 6
```

### 微任务（Microtask）

```javascript
// 微任务包括：
// 1. Promise.then/catch/finally
// 2. MutationObserver
// 3. process.nextTick (Node.js)
// 4. queueMicrotask

// 微任务示例
console.log('1. 开始');

Promise.resolve().then(() => {
  console.log('3. Promise.then (微任务)');
});

queueMicrotask(() => {
  console.log('4. queueMicrotask (微任务)');
});

Promise.resolve().then(() => {
  console.log('5. 第二个 Promise.then');
});

console.log('2. 结束');

// 执行顺序：1 -> 2 -> 3 -> 4 -> 5

// 微任务的优先级和执行顺序
console.log('1. 同步开始');

Promise.resolve().then(() => {
  console.log('3. 微任务 1');
  Promise.resolve().then(() => {
    console.log('5. 嵌套微任务');
  });
});

setTimeout(() => {
  console.log('6. 宏任务');
}, 0);

Promise.resolve().then(() => {
  console.log('4. 微任务 2');
});

console.log('2. 同步结束');

// 执行顺序：1 -> 2 -> 3 -> 4 -> 5 -> 6
```

### 执行优先级

```javascript
// 执行优先级：同步代码 > 微任务 > 宏任务
console.log('1. 同步开始');

// 微任务
Promise.resolve().then(() => {
  console.log('3. 微任务 1');
});

// 宏任务
setTimeout(() => {
  console.log('4. 宏任务 1');

  // 宏任务中的微任务
  Promise.resolve().then(() => {
    console.log('6. 宏任务中的微任务');
  });
}, 0);

// 微任务
Promise.resolve().then(() => {
  console.log('5. 微任务 2');
});

console.log('2. 同步结束');

// 执行顺序：1 -> 2 -> 3 -> 5 -> 4 -> 6

// 复杂示例
console.log('1. 开始');

setTimeout(() => {
  console.log('4. setTimeout 1');

  Promise.resolve().then(() => {
    console.log('7. setTimeout 1 中的微任务');
  });
}, 0);

Promise.resolve().then(() => {
  console.log('3. Promise 1');

  setTimeout(() => {
    console.log('8. Promise 1 中的 setTimeout');
  }, 0);
});

setTimeout(() => {
  console.log('5. setTimeout 2');
}, 0);

Promise.resolve().then(() => {
  console.log('6. Promise 2');
});

console.log('2. 结束');

// 执行顺序：1 -> 2 -> 3 -> 6 -> 4 -> 5 -> 7 -> 8
```

## async/await 和事件循环

### async/await 的本质

```javascript
// async/await 是 Promise 的语法糖
async function asyncFunction() {
  console.log('1. async 函数开始');

  const result = await Promise.resolve('2. await 结果');
  console.log(result);

  console.log('3. async 函数结束');
}

console.log('4. 同步代码');
asyncFunction();
console.log('5. 同步代码结束');

// 执行顺序：4 -> 1 -> 5 -> 2 -> 3
// await 会暂停函数执行，将函数剩余部分放入微任务队列
```

### async/await 的执行顺序

```javascript
console.log('1. 同步开始');

async function async1() {
  console.log('2. async1 开始');
  await async2();
  console.log('5. async1 结束');
}

async function async2() {
  console.log('3. async2');
}

async1();

console.log('4. 同步结束');

// 执行顺序：1 -> 2 -> 3 -> 4 -> 5
```

### 复杂的 async/await 场景

```javascript
console.log('1. 开始');

async function async1() {
  console.log('2. async1 开始');
  await async2();
  console.log('6. async1 结束');
  await async3();
  console.log('8. async1 最终结束');
}

async function async2() {
  console.log('3. async2');
}

async function async3() {
  console.log('7. async3');
}

async1();

new Promise(resolve => {
  console.log('4. Promise 构造函数');
  resolve();
}).then(() => {
  console.log('5. Promise then');
});

console.log('同步结束');

// 执行顺序：1 -> 2 -> 3 -> 4 -> 同步结束 -> 6 -> 7 -> 5 -> 8
```

## 实际应用场景

### 1. 网络请求处理

```javascript
// 使用事件循环优化网络请求
async function fetchMultipleUrls(urls) {
  const results = [];

  // 并行请求
  const promises = urls.map(async url => {
    try {
      const response = await fetch(url);
      const data = await response.json();
      return { url, data, success: true };
    } catch (error) {
      return { url, error, success: false };
    }
  });

  // 等待所有请求完成
  const allResults = await Promise.all(promises);
  results.push(...allResults);

  return results;
}

// 使用示例
const urls = [
  'https://api.example.com/users',
  'https://api.example.com/posts',
  'https://api.example.com/comments'
];

fetchMultipleUrls(urls).then(results => {
  console.log('所有请求完成:', results);
});
```

### 2. 批量数据处理

```javascript
// 使用微任务进行批量处理
class BatchProcessor {
  constructor(batchSize = 100) {
    this.batchSize = batchSize;
    this.queue = [];
    this.isProcessing = false;
  }

  add(item) {
    this.queue.push(item);

    if (!this.isProcessing) {
      this.process();
    }
  }

  async process() {
    this.isProcessing = true;

    while (this.queue.length > 0) {
      const batch = this.queue.splice(0, this.batchSize);
      await this.processBatch(batch);

      // 让出主线程，避免阻塞
      await new Promise(resolve => setTimeout(resolve, 0));
    }

    this.isProcessing = false;
  }

  async processBatch(batch) {
    console.log('处理批次:', batch.length);
    // 实际处理逻辑
    return batch;
  }
}

const processor = new BatchProcessor(50);

// 添加大量数据
for (let i = 0; i < 500; i++) {
  processor.add(i);
}
```

### 3. 动画和渲染优化

```javascript
// 使用 requestAnimationFrame 优化动画
class AnimationController {
  constructor() {
    this.animations = new Set();
    this.isRunning = false;
  }

  addAnimation(animation) {
    this.animations.add(animation);
    if (!this.isRunning) {
      this.start();
    }
  }

  removeAnimation(animation) {
    this.animations.delete(animation);
    if (this.animations.size === 0) {
      this.stop();
    }
  }

  start() {
    this.isRunning = true;
    this.loop();
  }

  stop() {
    this.isRunning = false;
  }

  loop() {
    if (!this.isRunning) return;

    // 清除微任务队列中的动画更新
    Promise.resolve().then(() => {
      this.animations.forEach(animation => animation.update());
    });

    // 请求下一帧
    requestAnimationFrame(() => this.loop());
  }
}

// 使用示例
const controller = new AnimationController();

const fadeIn = {
  element: document.createElement('div'),
  opacity: 0,
  update() {
    this.opacity += 0.01;
    this.element.style.opacity = this.opacity;
    if (this.opacity >= 1) {
      controller.removeAnimation(this);
    }
  }
};

document.body.appendChild(fadeIn.element);
controller.addAnimation(fadeIn);
```

### 4. 防抖和节流

```javascript
// 使用事件循环实现防抖
function debounce(fn, delay) {
  let timer = null;

  return function(...args) {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// 使用事件循环实现节流
function throttle(fn, delay) {
  let lastTime = 0;
  let timer = null;

  return function(...args) {
    const now = Date.now();

    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    } else if (!timer) {
      timer = setTimeout(() => {
        fn.apply(this, args);
        lastTime = Date.now();
        timer = null;
      }, delay - (now - lastTime));
    }
  };
}

// 使用示例
const searchInput = document.getElementById('search');

// 防抖：用户停止输入后才搜索
searchInput.addEventListener('input', debounce(function(e) {
  console.log('搜索:', e.target.value);
  performSearch(e.target.value);
}, 300));

// 节流：限制滚动事件处理频率
window.addEventListener('scroll', throttle(function() {
  console.log('滚动事件处理');
  handleScroll();
}, 200));
```

## 性能优化

### 1. 避免阻塞主线程

```javascript
// 不好的做法：同步大量计算
function heavyComputation(data) {
  const result = [];
  for (let i = 0; i < data.length; i++) {
    result.push(data[i] * 2);
  }
  return result;
}

// 更好的做法：分片处理
async function processInChunks(data, chunkSize = 1000) {
  const result = [];

  for (let i = 0; i < data.length; i += chunkSize) {
    const chunk = data.slice(i, i + chunkSize);
    const processed = processChunk(chunk);
    result.push(...processed);

    // 让出主线程
    await new Promise(resolve => setTimeout(resolve, 0));
  }

  return result;
}

function processChunk(chunk) {
  return chunk.map(item => item * 2);
}
```

### 2. 合理使用微任务

```javascript
// 微任务过多可能导致页面卡顿
function createMicrotasks() {
  for (let i = 0; i < 10000; i++) {
    Promise.resolve().then(() => {
      // 大量微任务
    });
  }
}

// 更好的做法：混合使用微任务和宏任务
function createBalancedTasks() {
  const taskCount = 10000;
  const batchSize = 100;

  for (let i = 0; i < taskCount; i += batchSize) {
    const end = Math.min(i + batchSize, taskCount);
    for (let j = i; j < end; j++) {
      Promise.resolve().then(() => {
        // 任务逻辑
      });
    }

    // 定期让出主线程
    if (i % 500 === 0) {
      setTimeout(() => {}, 0);
    }
  }
}
```

### 3. 优化 Promise 链

```javascript
// 不好的做法：过长的 Promise 链
function longPromiseChain(data) {
  return Promise.resolve(data)
    .then(step1)
    .then(step2)
    .then(step3)
    .then(step4)
    .then(step5)
    .then(step6)
    .then(step7)
    .then(step8)
    .then(step9)
    .then(step10);
}

// 更好的做法：使用 async/await
async function asyncProcess(data) {
  const result1 = await step1(data);
  const result2 = await step2(result1);
  const result3 = await step3(result2);
  const result4 = await step4(result3);
  const result5 = await step5(result4);

  return result5;
}

// 或者并行处理
async function parallelProcess(data) {
  const [result1, result2, result3] = await Promise.all([
    step1(data),
    step2(data),
    step3(data)
  ]);

  return processResults(result1, result2, result3);
}
```

## 调试技巧

### 1. 使用 Chrome DevTools

```javascript
// 在 DevTools 中查看任务队列
console.log('1. 同步代码');

setTimeout(() => {
  console.log('3. 宏任务');
  debugger; // 设置断点
}, 0);

Promise.resolve().then(() => {
  console.log('2. 微任务');
  debugger; // 设置断点
});

// 打开 DevTools -> Performance -> 开始录制
// 执行代码后停止录制，可以看到任务队列的情况
```

### 2. 性能分析

```javascript
// 使用 performance API
function measureAsyncPerformance() {
  const start = performance.now();

  setTimeout(() => {
    const end = performance.now();
    console.log(`setTimeout 延迟: ${end - start}ms`);
  }, 100);
}

measureAsyncPerformance();

// 使用 console.time 测量执行时间
console.time('async-operations');

Promise.resolve()
  .then(() => {
    console.timeLog('async-operations', 'after first then');
    return new Promise(resolve => setTimeout(resolve, 100));
  })
  .then(() => {
    console.timeLog('async-operations', 'after setTimeout');
    return Promise.resolve();
  })
  .then(() => {
    console.timeEnd('async-operations');
  });
```

### 3. 任务队列监控

```javascript
// 监控任务队列
function monitorTaskQueue() {
  let macroTaskCount = 0;
  let microTaskCount = 0;

  // 包装 setTimeout
  const originalSetTimeout = window.setTimeout;
  window.setTimeout = function(callback, delay, ...args) {
    macroTaskCount++;
    console.log(`宏任务队列: ${macroTaskCount}`);
    return originalSetTimeout.call(this, () => {
      macroTaskCount--;
      callback.apply(this, args);
    }, delay, ...args);
  };

  // 包装 Promise.then
  const originalThen = Promise.prototype.then;
  Promise.prototype.then = function(onFulfilled, onRejected) {
    microTaskCount++;
    console.log(`微任务队列: ${microTaskCount}`);
    return originalThen.call(this, function(value) {
      microTaskCount--;
      return onFulfilled ? onFulfilled(value) : value;
    }, onRejected);
  };
}

// 使用时谨慎，仅用于调试
// monitorTaskQueue();
```

## 常见面试题

### 1. 经典事件循环题目

```javascript
console.log('1');

setTimeout(() => {
  console.log('2');
  Promise.resolve().then(() => {
    console.log('3');
  });
}, 0);

new Promise((resolve) => {
  console.log('4');
  resolve();
}).then(() => {
  console.log('5');
});

console.log('6');

// 执行顺序：1 -> 4 -> 6 -> 5 -> 2 -> 3
```

### 2. async/await 混合题目

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

setTimeout(() => {
  console.log('setTimeout');
}, 0);

async1();

new Promise((resolve) => {
  console.log('promise1');
  resolve();
}).then(() => {
  console.log('promise2');
});

console.log('script end');

// 执行顺序：script start -> async1 start -> async2 -> promise1 -> script end -> async1 end -> promise2 -> setTimeout
```

### 3. 微任务和宏任务优先级

```javascript
console.log('1');

setTimeout(() => console.log('2'), 0);

Promise.resolve().then(() => console.log('3'));

Promise.resolve().then(() => {
  console.log('4');
  setTimeout(() => console.log('5'), 0);
});

console.log('6');

// 执行顺序：1 -> 6 -> 3 -> 4 -> 2 -> 5
```

### 4. 嵌套微任务

```javascript
console.log('1');

Promise.resolve().then(() => {
  console.log('2');
  Promise.resolve().then(() => {
    console.log('3');
  });
});

Promise.resolve().then(() => {
  console.log('4');
});

console.log('5');

// 执行顺序：1 -> 5 -> 2 -> 4 -> 3
```

## 总结

JavaScript 事件循环是理解异步编程的核心概念，掌握它对于编写高性能的前端应用至关重要。关键要点：

1. **单线程执行**：JavaScript 在主线程上同步执行代码，通过事件循环实现异步
2. **任务队列**：理解宏任务和微任务的区别和优先级
3. **执行顺序**：同步代码 > 微任务 > 宏任务的执行顺序
4. **async/await**：理解其 Promise 语法糖的本质和在事件循环中的表现
5. **实际应用**：在动画、网络请求、批量处理等场景中的合理使用
6. **性能优化**：避免阻塞主线程，合理使用微任务和宏任务
7. **调试技巧**：使用 DevTools 和性能分析工具优化异步代码

深入理解事件循环机制，能够帮助我们编写更高效、更可靠的 JavaScript 应用。在实际开发中，要根据具体需求选择合适的异步方式，避免因事件循环理解不当导致的性能问题。