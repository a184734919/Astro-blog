---
title: JavaScript Web Workers
published: 2022-12-11
description: '多线程处理方案的详细介绍和学习笔记'
image: ''
tags: ["性能"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript Web Workers 是 HTML5 提供的多线程解决方案，允许在后台线程中运行 JavaScript 脚本，而不会阻塞主线程（UI 线程）。这对于执行计算密集型任务、处理大量数据或执行耗时操作非常有用。

## 核心概念

### 主线程与 Worker 线程

Web Workers 提供了两个线程之间的通信机制：

- **主线程**：负责 UI 渲染和用户交互
- **Worker 线程**：在后台独立运行，执行耗时任务

### 消息传递

主线程和 Worker 线程之间通过消息传递进行通信，使用 `postMessage()` 方法发送数据，通过 `onmessage` 事件接收数据。

### 同源策略

Worker 脚本必须遵守同源策略，只能加载同源下的脚本文件。

## 基本用法

### 创建 Worker

```javascript
// 主线程中创建 Worker
const worker = new Worker('worker.js');
```

### Worker 脚本示例

```javascript
// worker.js
self.onmessage = function(e) {
  console.log('主线程发送的数据:', e.data);

  // 执行耗时计算
  const result = fibonacci(40);

  // 将结果发送回主线程
  self.postMessage(result);
};

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}
```

### 主线程通信

```javascript
const worker = new Worker('worker.js');

// 发送数据给 Worker
worker.postMessage({
  type: 'calculate',
  data: 40
});

// 接收 Worker 返回的数据
worker.onmessage = function(e) {
  console.log('Worker 计算结果:', e.data);
};

// 监听错误
worker.onerror = function(e) {
  console.error('Worker 错误:', e.message);
};

// 终止 Worker
// worker.terminate();
```

### 在 Worker 中发送消息

```javascript
// 在 worker.js 中
self.postMessage({
  type: 'result',
  data: '计算完成'
});
```

## 实际应用

### 大数据计算

```javascript
// 主线程
const worker = new Worker('data-worker.js');
const largeArray = Array.from({ length: 1000000 }, (_, i) => i);

worker.postMessage(largeArray);

worker.onmessage = function(e) {
  console.log('处理后的数据:', e.data);
  worker.terminate();
};
```

```javascript
// data-worker.js
self.onmessage = function(e) {
  const data = e.data;

  // 执行耗时操作：对大数据进行排序、筛选等
  const sorted = data.sort((a, b) => b - a);
  const filtered = sorted.filter(n => n % 2 === 0);

  self.postMessage(filtered);
};
```

### 图片处理

```javascript
// 主线程
const imageWorker = new Worker('image-worker.js');

function processImage(imageData) {
  imageWorker.postMessage(imageData);
}

imageWorker.onmessage = function(e) {
  const processedData = e.data;
  // 在 Canvas 上渲染处理后的图像
  ctx.putImageData(processedData, 0, 0);
};
```

```javascript
// image-worker.js
self.onmessage = function(e) {
  const imageData = e.data;
  const pixels = imageData.data;

  // 应用图像滤镜（如灰度化）
  for (let i = 0; i < pixels.length; i += 4) {
    const avg = (pixels[i] + pixels[i + 1] + pixels[i + 2]) / 3;
    pixels[i] = avg;     // R
    pixels[i + 1] = avg; // G
    pixels[i + 2] = avg; // B
  }

  self.postMessage(imageData);
};
```

### 使用 Blob 创建内联 Worker

```javascript
const workerCode = `
  self.onmessage = function(e) {
    const result = e.data.map(n => n * 2);
    self.postMessage(result);
  };
`;

const blob = new Blob([workerCode], { type: 'application/javascript' });
const workerUrl = URL.createObjectURL(blob);
const worker = new Worker(workerUrl);

worker.postMessage([1, 2, 3, 4, 5]);

worker.onmessage = function(e) {
  console.log('处理结果:', e.data);
  URL.revokeObjectURL(workerUrl);
  worker.terminate();
};
```

## Worker 类型

### Dedicated Workers（专用 Worker）

最常用的 Worker 类型，只能被创建它的脚本使用。

```javascript
const worker = new Worker('worker.js');
```

### Shared Workers（共享 Worker）

可以被多个浏览上下文（多个窗口或标签页）共享访问。

```javascript
// 创建 Shared Worker
const sharedWorker = new SharedWorker('shared-worker.js');

// 发送消息
sharedWorker.port.postMessage('hello');

// 接收消息
sharedWorker.port.onmessage = function(e) {
  console.log(e.data);
};
```

```javascript
// shared-worker.js
const connections = [];

self.onconnect = function(e) {
  const port = e.ports[0];
  connections.push(port);

  port.onmessage = function(e) {
    // 向所有连接的客户端广播消息
    connections.forEach(conn => {
      conn.postMessage(e.data);
    });
  };

  port.start();
};
```

## 高级用法

### 动态导入 Worker 脚本

```javascript
async function createWorker() {
  const response = await fetch('dynamic-worker.js');
  const workerCode = await response.text();
  const blob = new Blob([workerCode], { type: 'application/javascript' });
  const workerUrl = URL.createObjectURL(blob);

  return new Worker(workerUrl);
}

const worker = await createWorker();
```

### 使用 Worker 处理 WebSocket

```javascript
// websocket-worker.js
const ws = new WebSocket('wss://example.com/socket');

self.onmessage = function(e) {
  // 处理来自主线程的消息
  ws.send(JSON.stringify(e.data));
};

ws.onmessage = function(e) {
  // 转发 WebSocket 消息到主线程
  self.postMessage(JSON.parse(e.data));
};

ws.onerror = function(e) {
  self.postMessage({ type: 'error', message: e.message });
};
```

### 错误处理和重试机制

```javascript
class WorkerManager {
  constructor(scriptPath) {
    this.scriptPath = scriptPath;
    this.maxRetries = 3;
    this.retryCount = 0;
    this.worker = null;
  }

  create() {
    this.worker = new Worker(this.scriptPath);
    this.worker.onerror = this.handleError.bind(this);
    return this.worker;
  }

  handleError(error) {
    console.error('Worker 错误:', error.message);

    if (this.retryCount < this.maxRetries) {
      this.retryCount++;
      console.log(`尝试重启 Worker (${this.retryCount}/${this.maxRetries})`);

      setTimeout(() => {
        this.worker = this.create();
      }, 1000 * this.retryCount);
    }
  }
}

const manager = new WorkerManager('worker.js');
const worker = manager.create();
```

## 注意事项

### Worker 的限制

1. **无法访问 DOM**
   - Worker 不能访问 `document`、`window`、`parent` 对象
   - 可以访问 `navigator`、`location`、`XMLHttpRequest` 等 API

2. **不能使用同步 XHR**
   - Worker 中的 XHR 必须是异步的

3. **同源限制**
   - Worker 脚本必须与主线程同源

4. **内存消耗**
   - 每个 Worker 都会占用额外的内存
   - 及时终止不再使用的 Worker

### 性能考虑

1. **启动开销**
   - 创建 Worker 有一定的启动成本
   - 对于非常短的任务，可能不值得使用 Worker

2. **数据传输开销**
   - 数据在主线程和 Worker 之间传递会被复制
   - 大数据传输建议使用 Transferable Objects

```javascript
// 使用 Transferable Objects 避免数据复制
const largeBuffer = new ArrayBuffer(1024 * 1024);
worker.postMessage(largeBuffer, [largeBuffer]);
```

3. **Worker 数量限制**
   - 浏览器对同时运行的 Worker 数量有限制
   - 建议根据硬件配置合理控制数量

### 调试技巧

1. **使用 Chrome DevTools**
   - 在 Sources 面板中可以看到 Worker 线程
   - 支持断点调试

2. **日志记录**
```javascript
// 在 Worker 中
console.log('Worker 日志');

// 主线程
worker.onmessage = function(e) {
  console.log('收到 Worker 消息:', e.data);
};
```

3. **错误监控**
```javascript
worker.onerror = function(e) {
  console.error('Worker 错误详情:', {
    message: e.message,
    filename: e.filename,
    lineno: e.lineno,
    colno: e.colno
  });
};
```

## 总结

JavaScript Web Workers 为前端提供了真正的多线程能力，能够有效提升应用的性能和用户体验。通过合理使用 Web Workers，我们可以：

- 在后台执行耗时计算，避免阻塞 UI
- 处理大量数据，保持界面响应
- 实现复杂的业务逻辑，提升代码组织性

关键要点：
- 理解主线程与 Worker 线程的通信机制
- 注意 Worker 的限制和适用场景
- 合理管理 Worker 的生命周期
- 优化数据传输，减少性能开销
- 做好错误处理和调试

在实际项目中，根据具体需求选择合适的 Worker 类型，并结合其他 Web API 构建高性能的前端应用。