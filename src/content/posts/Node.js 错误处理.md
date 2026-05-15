---
title: Node.js 错误处理
published: 2023-02-25
description: '错误类型和异常捕获的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 中的错误处理是构建健壮应用的关键。错误处理不仅仅是 try-catch，还包括错误类型、错误传播、异步错误处理等多个方面。本文将详细介绍 Node.js 错误处理的各种方式。

## 错误类型

### 1. 系统错误（System Errors）

由底层操作系统触发的错误，通常来自系统调用。

```javascript
const fs = require('fs');

// 文件不存在错误
fs.readFile('/nonexistent/file.txt', (err, data) => {
  if (err) {
    console.error('错误代码:', err.code);      // 'ENOENT'
    console.error('错误信息:', err.message);    // 'ENOENT: no such file or directory'
    console.error('系统调用:', err.syscall);    // 'open'
    console.error('路径:', err.path);           // '/nonexistent/file.txt'
  }
});
```

常见的系统错误代码：
- `ENOENT`: 文件或目录不存在
- `EACCES`: 权限拒绝
- `EEXIST`: 文件已存在
- `ENOSPC`: 磁盘空间不足

### 2. 用户自定义错误

继承 `Error` 类创建自定义错误。

```javascript
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class DatabaseError extends Error {
  constructor(message, query) {
    super(message);
    this.name = 'DatabaseError';
    this.query = query;
    this.date = new Date();
  }
}

function validateEmail(email) {
  if (!email || !email.includes('@')) {
    throw new ValidationError('无效的邮箱格式', 'email');
  }
  return email;
}

try {
  validateEmail('invalid-email');
} catch (err) {
  if (err instanceof ValidationError) {
    console.error(`字段 ${err.field} 验证失败: ${err.message}`);
  }
}
```

### 3. 断言错误（Assertion Error）

使用 `assert` 模块进行测试和验证。

```javascript
const assert = require('assert');

// 简单断言
assert.strictEqual(1 + 1, 2, '数学运算错误');

// 深度比较
assert.deepStrictEqual(
  { name: 'Alice' },
  { name: 'Alice' },
  '对象不相等'
);

// 条件断言
assert.ok(true, '条件必须为真');
```

## 同步错误处理

### try-catch-finally

```javascript
function parseJSON(str) {
  try {
    const obj = JSON.parse(str);
    console.log('解析成功:', obj);
    return obj;
  } catch (err) {
    console.error('JSON 解析失败:', err.message);
    return null;
  } finally {
    console.log('无论成功或失败都会执行');
  }
}

parseJSON('{"name": "Alice"}');    // 解析成功
parseJSON('{invalid json}');       // 解析失败
```

### 错误传播

```javascript
function readFile(path) {
  const fs = require('fs');
  try {
    return fs.readFileSync(path, 'utf8');
  } catch (err) {
    // 可以选择处理错误或重新抛出
    throw new Error(`读取文件失败: ${err.message}`);
  }
}

function processFile(path) {
  try {
    const content = readFile(path);
    return content.toUpperCase();
  } catch (err) {
    console.error('处理文件时出错:', err.message);
    // 返回默认值而不是重新抛出
    return '';
  }
}
```

## 异步错误处理

### 回调模式（Callback Style）

Node.js 风格的回调函数第一个参数是错误对象。

```javascript
const fs = require('fs');

// 回调风格的错误处理
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    // 处理错误
    console.error('读取文件失败:', err.message);
    return;
  }
  // 处理数据
  console.log('文件内容:', data);
});

// 错误优先的函数
function fetchData(callback) {
  const success = Math.random() > 0.5;

  setTimeout(() => {
    if (success) {
      callback(null, { data: 'success' });
    } else {
      callback(new Error('获取数据失败'));
    }
  }, 1000);
}

fetchData((err, result) => {
  if (err) {
    console.error('错误:', err.message);
    return;
  }
  console.log('结果:', result);
});
```

### Promise 错误处理

```javascript
// 使用 Promise 封装异步操作
function readFilePromise(path) {
  const fs = require('fs').promises;
  return fs.readFile(path, 'utf8');
}

// 链式调用
readFilePromise('data.txt')
  .then(data => {
    console.log('文件内容:', data);
    return data.toUpperCase();
  })
  .then(upperData => {
    console.log('大写内容:', upperData);
  })
  .catch(err => {
    console.error('错误:', err.message);
  });

// async/await 写法
async function processFile() {
  try {
    const data = await readFilePromise('data.txt');
    const upperData = data.toUpperCase();
    console.log('处理结果:', upperData);
    return upperData;
  } catch (err) {
    console.error('处理失败:', err.message);
    throw err; // 可以选择重新抛出
  }
}
```

### EventEmitter 错误处理

```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}

const myEmitter = new MyEmitter();

// 必须监听 'error' 事件，否则会导致进程退出
myEmitter.on('error', (err) => {
  console.error('捕获到错误:', err.message);
});

myEmitter.on('data', (data) => {
  if (!data) {
    myEmitter.emit('error', new Error('数据为空'));
    return;
  }
  console.log('处理数据:', data);
});

// 触发错误
myEmitter.emit('error', new Error('模拟错误'));
```

## 全局错误处理

### uncaughtException

捕获未被捕获的同步异常（不建议用于恢复）。

```javascript
process.on('uncaughtException', (err) => {
  console.error('未捕获的异常:', err);
  // 执行清理工作
  // 不要继续运行，应该优雅退出
  process.exit(1);
});

// 不要在 uncaughtException 中恢复运行
throw new Error('未捕获的异常');
```

### unhandledRejection

捕获未处理的 Promise rejection。

```javascript
process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的 Promise 拒绝:', reason);
  console.error('Promise:', promise);
  // 记录错误日志
});

// 未处理的 rejection
Promise.reject(new Error('未处理的 rejection'));
```

## 最佳实践

### 1. 始终处理异步错误

```javascript
// ❌ 错误：未处理 Promise 错误
async function fetchData() {
  const data = await fetch('https://api.example.com/data');
  return data.json();
}
fetchData();

// ✅ 正确：使用 try-catch
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    return await response.json();
  } catch (err) {
    console.error('获取数据失败:', err);
    return null;
  }
}
fetchData();
```

### 2. 错误信息要详细但安全

```javascript
// ❌ 错误：暴露敏感信息
function authenticate(username, password) {
  if (password === 'wrong') {
    throw new Error(`用户 ${username} 的密码 ${password} 错误`);
  }
}

// ✅ 正确：不暴露敏感信息
function authenticate(username, password) {
  if (!isValidPassword(password)) {
    throw new Error('认证失败');
  }
}
```

### 3. 使用错误分类

```javascript
class AppError extends Error {
  constructor(message, statusCode = 500) {
    super(message);
    this.name = 'AppError';
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

class NotFoundError extends AppError {
  constructor(message = '资源未找到') {
    super(message, 404);
    this.name = 'NotFoundError';
  }
}

class UnauthorizedError extends AppError {
  constructor(message = '未授权') {
    super(message, 401);
    this.name = 'UnauthorizedError';
  }
}

function findUser(id) {
  const user = users[id];
  if (!user) {
    throw new NotFoundError(`用户 ${id} 不存在`);
  }
  return user;
}
```

### 4. 统一错误处理中间件（Express 示例）

```javascript
const express = require('express');
const app = express();

// 404 处理
app.use((req, res, next) => {
  const err = new NotFoundError(`路由 ${req.originalUrl} 不存在`);
  next(err);
});

// 统一错误处理
app.use((err, req, res, next) => {
  // 设置状态码
  const statusCode = err.statusCode || 500;

  // 响应错误
  res.status(statusCode).json({
    success: false,
    error: {
      message: err.message,
      code: err.code,
      // 仅在开发环境返回堆栈
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack
      })
    }
  });
});
```

### 5. 日志记录

```javascript
const fs = require('fs');
const path = require('path');

function logError(err) {
  const timestamp = new Date().toISOString();
  const logMessage = `[${timestamp}] ERROR: ${err.message}\n${err.stack}\n\n`;

  const logFile = path.join(__dirname, 'errors.log');
  fs.appendFileSync(logFile, logMessage);

  // 也可以使用专业日志库
  // const winston = require('winston');
  // winston.error(err.message, { stack: err.stack });
}

// 使用
process.on('uncaughtException', (err) => {
  logError(err);
  process.exit(1);
});
```

## 实际应用场景

### HTTP 服务器错误处理

```javascript
const http = require('http');

const server = http.createServer(async (req, res) => {
  try {
    // 路由处理
    if (req.url === '/api/data') {
      const data = await fetchData();
      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify(data));
    } else {
      res.writeHead(404, { 'Content-Type': 'text/plain' });
      res.end('Not Found');
    }
  } catch (err) {
    console.error('请求处理错误:', err);
    res.writeHead(500, { 'Content-Type': 'text/plain' });
    res.end('Internal Server Error');
  }
});

server.listen(3000);
```

### 数据库操作错误处理

```javascript
async function createUser(userData) {
  const client = await db.connect();

  try {
    await client.query('BEGIN');

    const result = await client.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      [userData.name, userData.email]
    );

    await client.query('COMMIT');
    return result.rows[0];
  } catch (err) {
    await client.query('ROLLBACK');

    // 判断错误类型
    if (err.code === '23505') { // 唯一约束违反
      throw new ConflictError('邮箱已被注册');
    } else {
      throw err;
    }
  } finally {
    client.release();
  }
}
```

## 总结

Node.js 错误处理涉及多个层面：

1. **理解错误类型**：系统错误、自定义错误、断言错误
2. **同步错误处理**：使用 try-catch-finally
3. **异步错误处理**：回调、Promise、async/await、EventEmitter
4. **全局错误处理**：uncaughtException、unhandledRejection
5. **最佳实践**：分类错误、详细记录、安全处理、统一处理

良好的错误处理是构建可靠 Node.js 应用的重要组成部分，能够在出现问题时提供清晰的错误信息，帮助快速定位和解决问题。