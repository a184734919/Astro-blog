---
title: Node.js 入门介绍
published: 2023-01-01
description: 'Node.js 的特点和安装的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时环境，使 JavaScript 能够在服务器端运行。它由 Ryan Dahl 于 2009 年创建，采用事件驱动、非阻塞 I/O 模型，特别适合构建高并发、可扩展的网络应用。

## 核心概念

理解核心概念是掌握任何技术的基础。

### 事件驱动

Node.js 采用事件驱动架构，所有 I/O 操作都是异步的，不会阻塞主线程。当 I/O 操作完成时，会触发相应的事件回调。

### 单线程

Node.js 使用单线程处理请求，通过事件循环机制实现高并发。虽然主线程是单线程的，但底层通过 libuv 实现了线程池来处理 CPU 密集型任务。

### 非阻塞 I/O

所有 I/O 操作（文件读写、网络请求等）都是非阻塞的，使用回调函数或 Promise 处理结果。

### NPM 生态系统

Node.js 自带 npm (Node Package Manager)，是世界上最大的开源库生态系统，拥有超过 150 万个包。

## 基本用法

### 安装 Node.js

```bash
# 官网下载安装
# 访问 https://nodejs.org 下载安装包

# 使用 nvm 安装（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20

# 验证安装
node -v
npm -v
```

### Hello World

```javascript
// 创建 httpServer.js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});

// 运行
// node httpServer.js
```

### 使用 ES Modules

```javascript
// 使用 import/export 语法（需要 package.json 中设置 "type": "module"）
import http from 'http';

const server = http.createServer((req, res) => {
  res.end('Hello from ES Modules');
});

server.listen(3000);
```

### REPL 交互式环境

```bash
# 启动 REPL
node

# 在 REPL 中
> 1 + 1
2
> const greet = (name) => `Hello, ${name}!`
undefined
> greet('World')
'Hello, World!'
> .exit  # 退出
```

## 实际应用

在实际项目中，我们需要根据具体场景灵活运用。

### 适合使用 Node.js 的场景

- **实时应用**：聊天应用、在线游戏
- **RESTful API 服务**：后端 API 开发
- **单页面应用**：React/Vue 应用的 SSR
- **工具链**：构建工具、代码生成器
- **微服务架构**：轻量级服务组件

### 典型应用案例

- Netflix（流媒体服务）
- PayPal（支付平台）
- LinkedIn（社交网络）
- Uber（出行服务）

## 注意事项

1. 注意边界情况
   - 避免在单线程中执行 CPU 密集型任务
   - 合理使用 Worker Threads 处理计算密集型操作

2. 考虑性能影响
   - 避免内存泄漏（及时清理事件监听器）
   - 使用缓存优化重复计算
   - 合理设置集群模式利用多核 CPU

3. 遵循最佳实践
   - 使用异步编程避免阻塞
   - 做好错误处理和日志记录
   - 使用 process.env 管理环境变量
   - 遵守 Semantic Versioning 版本管理

## 总结

通过本文的学习，相信你已经对 Node.js 有了更深入的理解。Node.js 是一个强大的 JavaScript 运行时环境，特别适合构建高性能、可扩展的后端应用。掌握事件驱动、非阻塞 I/O 等核心概念，结合丰富的 npm 生态系统，能够极大地提升开发效率。