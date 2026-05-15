---
title: Node.js 核心模块 events
published: 2023-01-19
description: '事件驱动机制详解的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `events` 模块是 Node.js 事件驱动架构的核心。它提供了一个简单的观察者模式实现，使得对象可以发布和订阅事件。这种机制在 Node.js 中无处不在，从文件系统操作到网络通信，几乎所有异步操作都基于事件。

## 核心概念

### 事件触发器（EventEmitter）
`EventEmitter` 是 `events` 模块的核心类，它提供了一种处理事件的方式。

### 事件监听器
监听器是当特定事件被触发时执行的回调函数。

### 事件循环
Node.js 使用事件循环来处理异步操作，`EventEmitter` 与事件循环紧密协作。

### 内存泄漏风险
不当使用事件监听器可能导致内存泄漏，需要及时移除不需要的监听器。

## 基本用法

```javascript
const EventEmitter = require('events');

// 1. 创建 EventEmitter 实例
const myEmitter = new EventEmitter();

// 2. 注册事件监听器
myEmitter.on('event', () => {
  console.log('事件发生了！');
});

// 3. 触发事件
myEmitter.emit('event');
// 输出：事件发生了！

// 4. 带参数的事件
myEmitter.on('greet', (name) => {
  console.log(`你好，${name}！`);
});

myEmitter.emit('greet', 'Alice');
// 输出：你好，Alice！

// 5. 只触发一次的事件
myEmitter.once('connect', () => {
  console.log('首次连接');
});

myEmitter.emit('connect'); // 输出：首次连接
myEmitter.emit('connect'); // 无输出

// 6. 移除事件监听器
const handler = () => console.log('监听器被调用');
myEmitter.on('test', handler);
myEmitter.off('test', handler); // 移除特定监听器
myEmitter.removeAllListeners('test'); // 移除所有监听器
```

### 创建自定义事件类
```javascript
const EventEmitter = require('events');

class MyServer extends EventEmitter {
  constructor(port) {
    super();
    this.port = port;
  }

  start() {
    console.log(`服务器启动在端口 ${this.port}`);
    this.emit('start', { port: this.port });
  }

  stop() {
    console.log('服务器停止');
    this.emit('stop');
  }
}

const server = new MyServer(3000);

server.on('start', (info) => {
  console.log(`启动事件触发: ${JSON.stringify(info)}`);
});

server.on('stop', () => {
  console.log('停止事件触发');
});

server.start(); // 启动服务器
```

### 错误处理
```javascript
const myEmitter = new EventEmitter();

// 重要：始终监听 'error' 事件
myEmitter.on('error', (err) => {
  console.error('捕获到错误:', err);
});

// 如果没有错误监听器，未捕获的 error 会导致进程退出
myEmitter.emit('error', new Error('发生了错误'));
```

### 异步事件处理
```javascript
const { EventEmitter } = require('events');

class AsyncEmitter extends EventEmitter {
  async emitAsync(eventName, ...args) {
    const listeners = this.listeners(eventName);
    for (const listener of listeners) {
      await listener.apply(this, args);
    }
  }
}

const emitter = new AsyncEmitter();

emitter.on('async-task', async (data) => {
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log(`异步任务完成: ${data}`);
});

(async () => {
  await emitter.emitAsync('async-task', 'Hello');
})();
```

## 实际应用

### HTTP 服务器事件处理
```javascript
const http = require('http');
const server = http.createServer();

server.on('request', (req, res) => {
  console.log(`收到请求: ${req.method} ${req.url}`);
  res.end('Hello World');
});

server.on('listening', () => {
  console.log('服务器开始监听');
});

server.on('error', (err) => {
  console.error('服务器错误:', err);
});

server.listen(3000);
```

### 文件处理进度
```javascript
const fs = require('fs');
const EventEmitter = require('events');

class FileReader extends EventEmitter {
  constructor(filePath) {
    super();
    this.filePath = filePath;
  }

  read() {
    const stream = fs.createReadStream(this.filePath);
    let totalBytes = 0;

    stream.on('data', (chunk) => {
      totalBytes += chunk.length;
      this.emit('progress', {
        bytesRead: totalBytes,
        chunkSize: chunk.length
      });
    });

    stream.on('end', () => {
      this.emit('complete', { totalBytes });
    });

    stream.on('error', (err) => {
      this.emit('error', err);
    });
  }
}

const reader = new FileReader('large-file.txt');

reader.on('progress', (info) => {
  console.log(`已读取 ${info.bytesRead} 字节`);
});

reader.on('complete', (info) => {
  console.log(`文件读取完成，共 ${info.totalBytes} 字节`);
});

reader.on('error', (err) => {
  console.error('读取错误:', err);
});

reader.read();
```

### 简单的发布订阅系统
```javascript
class EventBus extends EventEmitter {
  subscribe(event, callback) {
    this.on(event, callback);
    return () => this.off(event, callback);
  }

  publish(event, data) {
    this.emit(event, data);
  }
}

const eventBus = new EventBus();

// 订阅事件
const unsubscribe = eventBus.subscribe('user:login', (user) => {
  console.log(`用户 ${user.name} 登录了`);
});

// 发布事件
eventBus.publish('user:login', { name: 'Alice', id: 1 });

// 取消订阅
unsubscribe();
```

## 注意事项

1. **错误处理**：始终为 `EventEmitter` 添加 `'error'` 事件监听器，否则未捕获的错误会导致进程崩溃
2. **内存泄漏**：及时移除不再需要的监听器，使用 `once()` 代替 `on()` 处理只需执行一次的事件
3. **事件名称规范**：使用冒号分隔的命名空间（如 `user:login`）来组织事件
4. **最大监听器数量**：单个事件的监听器数量有默认限制（10），可通过 `emitter.setMaxListeners(n)` 调整
5. **异步处理**：事件监听器可能是异步的，需要谨慎处理异步操作
6. **this 绑定**：使用箭头函数或 `bind()` 确保 `this` 的正确指向

## 总结

通过本文的学习，相信你已经对 Node.js 核心模块 events 有了更深入的理解。`events` 模块是 Node.js 事件驱动架构的基础，掌握它能够帮助你更好地理解 Node.js 的工作原理，并构建出更加模块化、可维护的应用程序。在实际开发中，合理使用事件机制可以大大提高代码的灵活性和可扩展性。