---
title: Node.js 核心模块 cluster
published: 2023-02-14
description: '多进程集群模式的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 是单线程的，这意味着它只能利用单个 CPU 核心。为了充分利用多核 CPU 的性能，Node.js 提供了 `cluster` 模块，可以轻松创建多个子进程（Worker）来共享同一个服务器端口，从而实现负载均衡和更高的并发处理能力。

## 核心概念

### Master 进程

主进程负责管理所有 Worker 进程，包括创建、监控、重启等工作。

### Worker 进程

工作进程是实际处理请求的进程，它们共享同一个服务器端口，Node.js 通过操作系统的进程调度来实现负载均衡。

### IPC 通信

主进程和 Worker 进程之间可以通过 IPC（进程间通信）机制进行消息传递。

### 负载均衡策略

Node.js 提供两种负载均衡策略：
- **Round-robin（轮询）**：默认策略，除了 Windows 平台外
- **操作系统调度**：Windows 平台的默认策略

## 基本用法

### 创建简单的 HTTP 集群

```javascript
const cluster = require('cluster');
const http = require('http');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 正在运行`);

  // 衍生工作进程
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  cluster.on('exit', (worker, code, signal) => {
    console.log(`工作进程 ${worker.process.pid} 已退出`);
    // 自动重启工作进程
    cluster.fork();
  });
} else {
  // 工作进程可以共享同一个 TCP 端口
  http.createServer((req, res) => {
    res.writeHead(200);
    res.end(`你好，来自进程 ${process.pid} 的响应\n`);
  }).listen(8000);

  console.log(`工作进程 ${process.pid} 已启动`);
}
```

### 使用配置对象创建 Worker

```javascript
const cluster = require('cluster');

if (cluster.isMaster) {
  // 使用配置对象创建 Worker
  const worker = cluster.fork({
    WORKER_TYPE: 'api'
  });

  worker.on('listening', (address) => {
    console.log(`Worker ${worker.id} 正在监听 ${address.port}`);
  });
}
```

### 主进程与 Worker 进程通信

```javascript
const cluster = require('cluster');

if (cluster.isMaster) {
  const worker = cluster.fork();

  // 发送消息给 Worker
  worker.send({ cmd: 'start', data: { count: 10 } });

  // 接收 Worker 的消息
  worker.on('message', (msg) => {
    console.log('主进程收到消息:', msg);
  });
} else {
  // Worker 接收主进程消息
  process.on('message', (msg) => {
    if (msg.cmd === 'start') {
      console.log('Worker 收到开始指令:', msg.data);
      // 发送响应给主进程
      process.send({ status: 'completed', result: msg.data.count * 2 });
    }
  });
}
```

### 管理 Worker 进程

```javascript
const cluster = require('cluster');

if (cluster.isMaster) {
  const workers = [];

  // 创建多个 Worker
  for (let i = 0; i < 3; i++) {
    const worker = cluster.fork();
    workers.push(worker);
  }

  // 断开 Worker 连接
  workers[0].disconnect();

  // 杀死 Worker
  workers[1].kill('SIGTERM');

  // 监听 Worker 事件
  cluster.on('online', (worker) => {
    console.log(`Worker ${worker.id} 已上线`);
  });

  cluster.on('listening', (worker, address) => {
    console.log(`Worker ${worker.id} 正在监听 ${address.port}`);
  });

  cluster.on('disconnect', (worker) => {
    console.log(`Worker ${worker.id} 已断开连接`);
  });
}
```

## 实际应用

### 构建高可用的 Express 应用

```javascript
const cluster = require('cluster');
const express = require('express');
const numCPUs = require('os').cpus().length;

if (cluster.isMaster) {
  console.log(`主进程 ${process.pid} 启动`);

  // 创建工作进程
  for (let i = 0; i < numCPUs; i++) {
    cluster.fork();
  }

  // 优雅重启
  process.on('SIGUSR2', () => {
    console.log('收到重启信号，开始优雅重启...');

    const workers = Object.values(cluster.workers);

    // 逐个重启 Worker
    workers.forEach((worker, index) => {
      setTimeout(() => {
        console.log(`重启 Worker ${worker.id}`);
        worker.send('shutdown');

        worker.on('disconnect', () => {
          cluster.fork();
        });
      }, index * 1000);
    });
  });

  cluster.on('exit', (worker) => {
    console.log(`Worker ${worker.process.pid} 意外退出，重新启动`);
    cluster.fork();
  });
} else {
  const app = express();

  app.get('/', (req, res) => {
    res.json({
      process: process.pid,
      message: 'Hello from cluster worker'
    });
  });

  const server = app.listen(3000, () => {
    console.log(`Worker ${process.pid} 监听端口 3000`);
  });

  // 处理优雅关闭
  process.on('message', (msg) => {
    if (msg === 'shutdown') {
      server.close(() => {
        console.log(`Worker ${process.pid} 正在关闭`);
        process.exit(0);
      });

      // 强制关闭超时
      setTimeout(() => {
        console.log(`Worker ${process.pid} 强制关闭`);
        process.exit(1);
      }, 5000);
    }
  });
}
```

### 性能监控的集群应用

```javascript
const cluster = require('cluster');
const http = require('http');

if (cluster.isMaster) {
  const workers = [];
  const workerStats = {};

  // 创建 Worker 并收集统计信息
  for (let i = 0; i < 4; i++) {
    const worker = cluster.fork();
    workers.push(worker);
    workerStats[worker.id] = { requests: 0, startTime: Date.now() };
  }

  workers.forEach(worker => {
    worker.on('message', (msg) => {
      if (msg.type === 'request') {
        workerStats[worker.id].requests++;
      }
    });
  });

  // 定期输出统计信息
  setInterval(() => {
    console.log('--- Worker 统计 ---');
    Object.keys(workerStats).forEach(id => {
      const stats = workerStats[id];
      const uptime = (Date.now() - stats.startTime) / 1000;
      const rps = (stats.requests / uptime).toFixed(2);
      console.log(`Worker ${id}: ${stats.requests} 请求, ${rps} RPS`);
    });
  }, 5000);
} else {
  http.createServer((req, res) => {
    // 通知主进程有新请求
    process.send({ type: 'request' });

    res.writeHead(200);
    res.end(`响应来自 Worker ${process.pid}\n`);
  }).listen(3000);
}
```

## 注意事项

### 1. 状态共享问题

每个 Worker 进程都有独立的内存空间，不要在 Worker 之间共享可变状态。

```javascript
// ❌ 错误：每个 Worker 都有独立的计数器
let counter = 0;

// ✅ 正确：使用共享存储（Redis、数据库等）
const redis = require('redis');
const client = redis.createClient();
```

### 2. 文件描述符限制

创建多个 Worker 进程会增加文件描述符的使用，注意系统限制。

```javascript
// 检查当前限制
console.log('ulimit -n:', require('fs').readFileSync('/proc/self/limits', 'utf8'));
```

### 3. 优雅关闭

实现优雅关闭以确保正在处理的请求能够完成。

```javascript
process.on('SIGTERM', () => {
  server.close(() => {
    process.exit(0);
  });
});
```

### 4. 内存泄漏检测

定期监控 Worker 进程的内存使用情况，及时发现内存泄漏。

```javascript
setInterval(() => {
  const usage = process.memoryUsage();
  console.log(`内存使用: ${Math.round(usage.rss / 1024 / 1024)}MB`);
}, 60000);
```

### 5. 适当的 Worker 数量

不要创建过多的 Worker 进程，通常设置为 CPU 核心数即可。

```javascript
const numCPUs = require('os').cpus().length;
const workerCount = Math.min(numCPUs, 4); // 限制最大数量
```

## 总结

通过 `cluster` 模块，我们可以充分利用多核 CPU 的性能，构建高并发、高可用的 Node.js 应用。关键要点包括：

- 使用主进程管理多个 Worker 进程
- 实现 Worker 之间的负载均衡
- 注意 Worker 之间的状态隔离
- 实现优雅关闭和自动重启
- 监控 Worker 进程的性能和状态

合理使用 `cluster` 模块可以显著提升应用的性能和可靠性。