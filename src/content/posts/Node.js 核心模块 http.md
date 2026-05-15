---
title: Node.js 核心模块 http
published: 2023-01-15
description: 'HTTP 服务器和客户端的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `http` 模块提供了创建 HTTP 服务器和客户端的能力。它是 Node.js 最核心的模块之一，使得使用 JavaScript 构建网络应用变得简单高效。无论是简单的 API 服务还是复杂的 Web 应用，`http` 模块都能胜任。

## 核心概念

### HTTP 请求/响应模型
- 请求（Request）：客户端发送给服务器的数据，包含方法、URL、头部和主体
- 响应（Response）：服务器返回给客户端的数据，包含状态码、头部和主体

### 事件驱动
`http` 模块基于事件驱动架构，通过监听事件来处理请求和响应。

### 流式处理
请求和响应体是可读可写的流，支持分块传输，适合处理大文件。

## 基本用法

### 创建简单的 HTTP 服务器
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

### 路由处理
```javascript
const server = http.createServer((req, res) => {
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } else if (req.url === '/api' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'API Response' }));
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});
```

### 发送 HTTP 请求（客户端）
```javascript
const http = require('http');

const options = {
  hostname: 'example.com',
  port: 80,
  path: '/api/data',
  method: 'GET',
};

const req = http.request(options, (res) => {
  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('end', () => {
    console.log(JSON.parse(data));
  });
});

req.on('error', (error) => {
  console.error(`Request error: ${error.message}`);
});

req.end();
```

### 处理 POST 请求
```javascript
const server = http.createServer((req, res) => {
  if (req.url === '/submit' && req.method === 'POST') {
    let body = '';

    req.on('data', (chunk) => {
      body += chunk.toString();
    });

    req.on('end', () => {
      try {
        const data = JSON.parse(body);
        console.log('Received data:', data);

        res.writeHead(200, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ success: true }));
      } catch (error) {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Invalid JSON' }));
      }
    });
  }
});
```

## 实际应用

### 静态文件服务器
```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer((req, res) => {
  const filePath = path.join(__dirname, 'public', req.url === '/' ? 'index.html' : req.url);

  fs.readFile(filePath, (err, data) => {
    if (err) {
      res.writeHead(404, { 'Content-Type': 'text/plain' });
      res.end('404 Not Found');
    } else {
      const ext = path.extname(filePath);
      const contentType = {
        '.html': 'text/html',
        '.css': 'text/css',
        '.js': 'application/javascript',
        '.json': 'application/json',
        '.png': 'image/png',
      }[ext] || 'application/octet-stream';

      res.writeHead(200, { 'Content-Type': contentType });
      res.end(data);
    }
  });
});

server.listen(8080);
```

### RESTful API 基础
```javascript
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' },
];

const server = http.createServer((req, res) => {
  res.setHeader('Content-Type', 'application/json');

  if (req.url === '/users' && req.method === 'GET') {
    res.end(JSON.stringify(users));
  } else if (req.url.match(/\/users\/\d+/) && req.method === 'GET') {
    const id = parseInt(req.url.split('/')[2]);
    const user = users.find(u => u.id === id);
    user ? res.end(JSON.stringify(user)) : res.writeHead(404).end(JSON.stringify({ error: 'User not found' }));
  } else {
    res.writeHead(404).end(JSON.stringify({ error: 'Not Found' }));
  }
});
```

## 注意事项

1. **安全性**：始终对用户输入进行验证和清理，防止 XSS、注入攻击
2. **错误处理**：妥善处理各种错误情况，避免服务器崩溃
3. **性能优化**：合理使用流式处理，避免将大文件全部读入内存
4. **HTTP 状态码**：正确使用状态码，如 200、404、500 等
5. **头部设置**：正确设置 Content-Type、Content-Length 等响应头
6. **HTTPS**：生产环境建议使用 `https` 模块确保数据传输安全

## 总结

通过本文的学习，相信你已经对 Node.js 核心模块 http 有了更深入的理解。`http` 模块是构建网络应用的基础，掌握它的使用对于 Node.js 开发至关重要。在实际项目中，许多流行的框架如 Express.js 都是基于 `http` 模块构建的，理解底层原理有助于更好地使用这些框架。