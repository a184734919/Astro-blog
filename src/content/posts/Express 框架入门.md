---
title: Express 框架入门
published: 2023-03-08
description: 'Express 基础路由和中间件的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## Express 框架概述

Express 是 Node.js 平台上最流行、最灵活的 Web 应用框架，它提供了丰富的 HTTP 工具和方法，让开发 Web 应用变得更加简单和高效。Express 被广泛应用于各种场景，从简单的静态网站到复杂的 RESTful API 服务。

### 为什么选择 Express？

- **简洁轻量**：最小化核心，按需扩展
- **灵活性强**：开发者可以自由选择使用什么库和中间件
- **丰富的中间件生态系统**：数以千计的中间件可供选择
- **优秀的文档**：详细的官方文档和活跃的社区
- **广泛的使用**：被许多知名公司采用

### Express 核心特性

- 路由系统（支持多种路由方式）
- 中间件机制（强大的请求处理链）
- 模板引擎支持（EJS、Pug、Handlebars 等）
- 静态文件服务
- 会话和Cookie管理
- 错误处理机制

## 快速开始

### 安装 Express

```bash
# 创建项目目录
mkdir my-express-app
cd my-express-app

# 初始化项目
npm init -y

# 安装 Express
npm install express
```

### 创建基础服务器

```javascript
const express = require('express');
const app = express();
const PORT = process.env.PORT || 3000;

// 基础路由
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// 启动服务器
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

### 完整的 Express 应用结构

```javascript
const express = require('express');
const app = express();

// 内置中间件
app.use(express.json());              // 解析 JSON 请求体
app.use(express.urlencoded({ extended: true })); // 解析 URL 编码的请求体
app.use(express.static('public'));   // 静态文件服务

// 基础路由
app.get('/', (req, res) => {
  res.send('Hello World!');
});

// JSON 响应
app.get('/api/data', (req, res) => {
  res.json({
    message: 'Success',
    data: { id: 1, name: 'Test' }
  });
});

// POST 请求处理
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  res.status(201).json({
    success: true,
    user: { name, email }
  });
});

// 404 处理
app.use((req, res) => {
  res.status(404).send('Not Found');
});

// 错误处理
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});

// 启动服务器
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server is running on port ${PORT}`);
});
```

## 请求与响应对象

### Request 对象 (req)

Request 对象包含请求的所有信息：

```javascript
app.get('/api/req-info', (req, res) => {
  const reqInfo = {
    method: req.method,           // HTTP 方法
    url: req.url,                 // 请求 URL
    path: req.path,               // 请求路径（不含查询字符串）
    query: req.query,             // 查询参数对象
    params: req.params,           // 路由参数
    headers: req.headers,         // 请求头
    body: req.body,               // 请求体
    ip: req.ip,                   // 客户端 IP
    protocol: req.protocol,       // 协议
    secure: req.secure,           // 是否 HTTPS
    cookies: req.cookies,         // Cookie（需使用 cookie-parser）
    originalUrl: req.originalUrl  // 原始请求 URL
  };
  res.json(reqInfo);
});
```

### Response 对象 (res)

Response 对象用于发送响应：

```javascript
app.get('/api/res-demo', (req, res) => {
  // 设置状态码
  res.status(200);

  // 设置响应头
  res.set('Content-Type', 'text/html; charset=utf-8');

  // 发送文本
  res.send('Hello Express!');

  // 或者发送 JSON
  // res.json({ message: 'Success' });

  // 或者发送文件
  // res.sendFile('/path/to/file.html');

  // 或者重定向
  // res.redirect('/login');
});
```

### 常用响应方法

```javascript
app.get('/api/res-methods', (req, res) => {
  // 状态码 + JSON
  res.status(201).json({ created: true });

  // 发送 HTML
  res.send('<h1>HTML Content</h1>');

  // 发送文件
  res.download('/path/to/file.pdf');

  // 设置 Cookie
  res.cookie('user', 'john', { maxAge: 900000 });

  // 清除 Cookie
  res.clearCookie('user');

  // 重定向
  res.redirect('/new-location');

  // 结束响应
  res.end();
});
```

## 基础路由

### HTTP 方法路由

```javascript
// GET 请求
app.get('/users', (req, res) => {
  res.json(['Alice', 'Bob', 'Charlie']);
});

// POST 请求
app.post('/users', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

// PUT 请求
app.put('/users/:id', (req, res) => {
  res.json({ message: 'User updated', id: req.params.id });
});

// DELETE 请求
app.delete('/users/:id', (req, res) => {
  res.json({ message: 'User deleted', id: req.params.id });
});

// PATCH 请求
app.patch('/users/:id', (req, res) => {
  res.json({ message: 'User partially updated', id: req.params.id });
});

// 处理所有 HTTP 方法
app.all('/api/*', (req, res) => {
  res.send('API endpoint');
});
```

### 路由参数

```javascript
// 路径参数
app.get('/users/:userId', (req, res) => {
  const userId = req.params.userId;
  res.json({ userId });
});

// 多个路径参数
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// 可选参数
app.get('/books/:title?', (req, res) => {
  const title = req.params.title || 'default';
  res.json({ title });
});

// 查询参数
app.get('/search', (req, res) => {
  const { keyword, page = 1, limit = 10 } = req.query;
  res.json({ keyword, page, limit });
});
```

## 中间件机制

Express 的核心是中间件机制，它允许你在请求处理流程中插入自定义的处理逻辑。

### 中间件的基本结构

```javascript
// 中间件函数有 3 个参数：req, res, next
const myMiddleware = (req, res, next) => {
  console.log('中间件处理中...');
  // 调用 next() 将控制权传递给下一个中间件
  next();
};

// 使用中间件
app.use(myMiddleware);
```

### 应用级中间件

```javascript
// 应用到所有路由
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url} - ${new Date().toISOString()}`);
  next();
});

// 应用到特定路径
app.use('/api', (req, res, next) => {
  console.log('API Request');
  next();
});

// 条件中间件
if (process.env.NODE_ENV === 'development') {
  app.use((req, res, next) => {
    console.log('Development mode');
    next();
  });
}
```

### 路由级中间件

```javascript
// 为特定路由添加中间件
app.get('/protected', authenticateUser, (req, res) => {
  res.json({ message: 'Protected resource' });
});

function authenticateUser(req, res, next) {
  // 模拟认证逻辑
  if (req.headers.authorization) {
    next(); // 认证通过
  } else {
    res.status(401).json({ error: 'Unauthorized' });
  }
}

// 多个中间件
app.get('/data',
  validateRequest,
  checkPermission,
  processData,
  (req, res) => {
    res.json({ success: true });
  }
);
```

## 实用中间件示例

### 日志记录中间件

```javascript
const morgan = require('morgan');

// 使用 morgan 进行日志记录
app.use(morgan('dev'));

// 自定义日志中间件
app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
  });
  
  next();
});
```

### 验证中间件

```javascript
const validateUser = (req, res, next) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'Name and email are required' });
  }
  
  if (!email.includes('@')) {
    return res.status(400).json({ error: 'Invalid email format' });
  }
  
  next();
};

app.post('/users', validateUser, (req, res) => {
  res.status(201).json({ message: 'User created successfully' });
});
```

## 最佳实践

### 1. 模块化代码

```javascript
// routes/users.js
const express = require('express');
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.post('/', (req, res) => {
  res.status(201).json(req.body);
});

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/api/users', usersRouter);
```

### 2. 环境变量

```javascript
// 安装 dotenv
// npm install dotenv

require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;
const ENV = process.env.NODE_ENV || 'development';

console.log(`Running in ${ENV} mode on port ${PORT}`);
```

### 3. 安全性

```javascript
const helmet = require('helmet');
const cors = require('cors');

// 安全相关中间件
app.use(helmet()); // 安全头部
app.use(cors());   // CORS 支持

// 限制请求体大小
app.use(express.json({ limit: '10mb' }));
```

### 4. 启动配置

```javascript
const server = app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

// 优雅关闭
process.on('SIGTERM', () => {
  console.log('SIGTERM signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
  });
});

process.on('SIGINT', () => {
  console.log('SIGINT signal received: closing HTTP server');
  server.close(() => {
    console.log('HTTP server closed');
  });
});
```

## 项目结构建议

```
my-express-app/
├── src/
│   ├── config/
│   │   ├── database.js
│   │   └── config.js
│   ├── controllers/
│   │   ├── userController.js
│   │   └── postController.js
│   ├── middlewares/
│   │   ├── auth.js
│   │   └── errorHandler.js
│   ├── models/
│   │   ├── User.js
│   │   └── Post.js
│   ├── routes/
│   │   ├── userRoutes.js
│   │   └── postRoutes.js
│   ├── services/
│   │   └── userService.js
│   └── utils/
│       └── logger.js
├── public/
│   ├── images/
│   ├── css/
│   └── js/
├── tests/
├── .env
├── .gitignore
├── package.json
└── server.js
```

## 常见问题和解决方案

### 问题 1：CORS 错误

```javascript
const cors = require('cors');

// 基础配置
app.use(cors());

// 或者自定义配置
app.use(cors({
  origin: ['https://example.com', 'https://app.example.com'],
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
```

### 问题 2：请求体解析失败

```javascript
// 确保 JSON 解析中间件在路由之前
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
```

### 问题 3：路由顺序问题

```javascript
// 具体路由应该在通配符路由之前
app.get('/users/profile', (req, res) => {
  res.json({ message: 'Profile' });
});

// 通配符路由放在最后
app.get('/users/*', (req, res) => {
  res.json({ message: 'Other user routes' });
});
```

## 总结

Express 是一个功能强大且灵活的 Node.js Web 框架，通过本篇入门指南，你已经了解了：

- Express 的核心概念和特性
- 如何创建和配置 Express 应用
- 请求和响应对象的使用
- 基础路由和参数处理
- 中间件机制和工作原理
- 实用中间件示例和最佳实践
- 项目结构建议和常见问题解决

Express 的真正威力在于其简洁的 API 和丰富的生态系统。接下来可以深入学习路由系统、中间件开发、错误处理等高级主题，以构建更加健壮和可维护的应用程序。

要成为 Express 专家，建议继续学习：路由系统设计、中间件开发、安全最佳实践、性能优化、测试策略等内容。