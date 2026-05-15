---
title: Express 路由系统
published: 2023-03-15
description: '路由的设计和组织的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 路由系统概述

Express 的路由系统是其核心功能之一，它决定了应用如何响应客户端对特定端点（URL路径）的 HTTP 请求。良好的路由设计对于构建可维护、可扩展的应用至关重要。

### 路由的核心概念

- **路由定义**：将 HTTP 方法和 URL 路径映射到处理函数
- **路由参数**：从 URL 中提取动态数据
- **路由组织**：合理组织和管理路由结构
- **路由优先级**：理解路由匹配顺序

### 路由的基本结构

```javascript
app.METHOD(PATH, HANDLER)

// METHOD: HTTP 请求方法（get, post, put, delete 等）
// PATH: 服务器上的路径
// HANDLER: 当路由匹配时执行的函数
```

## 基础路由定义

### 简单路由

```javascript
const express = require('express');
const app = express();

// GET 请求
app.get('/', (req, res) => {
  res.send('Home Page');
});

app.get('/about', (req, res) => {
  res.send('About Page');
});

// POST 请求
app.post('/api/users', (req, res) => {
  res.status(201).json({ message: 'User created' });
});

// PUT 请求
app.put('/api/users/:id', (req, res) => {
  res.json({ message: 'User updated' });
});

// DELETE 请求
app.delete('/api/users/:id', (req, res) => {
  res.json({ message: 'User deleted' });
});

// PATCH 请求
app.patch('/api/users/:id', (req, res) => {
  res.json({ message: 'User partially updated' });
});
```

### 路由方法详解

```javascript
// 支持所有 HTTP 方法
app.get('/books', getBooksHandler);
app.post('/books', createBookHandler);
app.put('/books/:id', updateBookHandler);
app.delete('/books/:id', deleteBookHandler);
app.all('/api/*', apiHandler);        // 匹配所有 HTTP 方法
app.use('/static', staticHandler);     // 匹配所有方法

// 方法别名
app.get('/route1', handler);
app.head('/route1', handler);
app.options('/route1', handler);
```

## 路由参数

### 路径参数

路径参数允许你从 URL 中提取动态数据：

```javascript
// 基础参数
app.get('/users/:userId', (req, res) => {
  const userId = req.params.userId;
  res.json({ userId });
});
// /users/123 -> { userId: '123' }

// 多个参数
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});
// /users/1/posts/45 -> { userId: '1', postId: '45' }

// 可选参数（带 ?）
app.get('/books/:title?', (req, res) => {
  const title = req.params.title || 'All Books';
  res.json({ title });
});
// /books -> { title: 'All Books' }
// /books/nodejs -> { title: 'nodejs' }
```

### 正则表达式路由

```javascript
// 匹配特定模式
app.get(/.*fly$/, (req, res) => {
  res.send('Matches any path ending with "fly"');
});
// /butterfly, /dragonfly 等都会匹配

// 数字参数
app.get('/users/:userId(\\d+)', (req, res) => {
  const userId = req.params.userId;
  res.json({ userId: parseInt(userId) });
});
// /users/123 -> 匹配
// /users/abc -> 不匹配

// 复杂正则
app.get(/^\/users\/(\d{4})-(\d{2})-(\d{2})$/, (req, res) => {
  const [_, year, month, day] = req.path.match(/^\/users\/(\d{4})-(\d{2})-(\d{2})$/);
  res.json({ date: `${year}-${month}-${day}` });
});
```

### 查询参数

```javascript
// 获取查询参数
app.get('/search', (req, res) => {
  const { keyword, page = 1, limit = 10, sort } = req.query;
  
  res.json({
    keyword,
    page: parseInt(page),
    limit: parseInt(limit),
    sort
  });
});
// /search?keyword=nodejs&page=2&limit=20&sort=date

// 数组查询参数
app.get('/products', (req, res) => {
  const { categories, colors } = req.query;
  
  // 处理数组参数
  const categoryList = Array.isArray(categories) ? categories : [categories];
  const colorList = Array.isArray(colors) ? colors : [colors];
  
  res.json({ categories: categoryList, colors: colorList });
});
// /products?categories=electronics,books&colors=red,blue
```

## 路由组织与模块化

### 路由模块化

将路由按功能模块化，提高代码的可维护性：

```javascript
// routes/userRoutes.js
const express = require('express');
const router = express.Router();
const userController = require('../controllers/userController');

// 用户路由
router.get('/', userController.getAllUsers);
router.get('/:id', userController.getUserById);
router.post('/', userController.createUser);
router.put('/:id', userController.updateUser);
router.delete('/:id', userController.deleteUser);

// 用户资料路由
router.get('/:id/profile', userController.getUserProfile);
router.put('/:id/profile', userController.updateProfile);

module.exports = router;

// routes/productRoutes.js
const express = require('express');
const router = express.Router();
const productController = require('../controllers/productController');

// 产品路由
router.get('/', productController.getProducts);
router.get('/:id', productController.getProduct);
router.post('/', productController.createProduct);
router.put('/:id', productController.updateProduct);
router.delete('/:id', productController.deleteProduct);

module.exports = router;

// app.js
const express = require('express');
const app = express();

const userRoutes = require('./routes/userRoutes');
const productRoutes = require('./routes/productRoutes');

// 使用路由模块
app.use('/api/users', userRoutes);
app.use('/api/products', productRoutes);

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### 高级路由组织

```javascript
// routes/index.js
const express = require('express');
const router = express.Router();

// 导入各个模块路由
const authRoutes = require('./authRoutes');
const userRoutes = require('./userRoutes');
const productRoutes = require('./productRoutes');
const orderRoutes = require('./orderRoutes');

// 健康检查
router.get('/health', (req, res) => {
  res.json({ status: 'OK', timestamp: new Date().toISOString() });
});

// API 版本控制
router.use('/api/v1/auth', authRoutes);
router.use('/api/v1/users', userRoutes);
router.use('/api/v1/products', productRoutes);
router.use('/api/v1/orders', orderRoutes);

// 文档路由
router.use('/api/docs', express.static('docs'));

module.exports = router;

// app.js
const express = require('express');
const app = express();
const routes = require('./routes');

// 全局中间件
app.use(express.json());

// 使用主路由
app.use('/', routes);

// 404 处理
app.use((req, res) => {
  res.status(404).json({ error: 'Not Found' });
});

module.exports = app;
```

## 路由中间件

### 路由特定中间件

```javascript
// 认证中间件
const authenticate = (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
};

// 权限检查中间件
const authorize = (roles) => {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    next();
  };
};

// 验证中间件
const validateRequest = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body);
    
    if (error) {
      return res.status(400).json({ 
        error: 'Validation failed', 
        details: error.details 
      });
    }
    
    next();
  };
};

// 使用中间件
app.post('/api/admin/users', 
  authenticate,
  authorize(['admin']),
  validateRequest(userValidationSchema),
  userController.createUser
);
```

### 路由链式中间件

```javascript
// 多个中间件按顺序执行
app.get('/api/products',
  authenticate,
  validateQueryParams,
  cacheMiddleware,
  productController.getProducts
);

// 带条件的中间件
app.get('/api/users/:id',
  (req, res, next) => {
    // 检查是否有权限查看其他用户的信息
    if (req.params.id !== req.user.id && req.user.role !== 'admin') {
      return res.status(403).json({ error: 'Access denied' });
    }
    next();
  },
  userController.getUserById
);
```

## 路由匹配规则

### 匹配优先级

```javascript
// 路由按定义顺序匹配，先定义的优先级更高
app.get('/users', getAllUsers);           // 第一优先级
app.get('/users/profile', getProfile);     // 第二优先级（会被第一个匹配）
app.get('/users/:id', getUserById);        // 第三优先级

// 正确的顺序
app.get('/users/profile', getProfile);     // 最具体的在前
app.get('/users/:id', getUserById);        // 通用的在后
```

### next() 函数的用法

```javascript
// 跳过剩余的路由处理
app.get('/user/:id', (req, res, next) => {
  if (req.params.id === '0') {
    next('route');  // 跳过当前路由中的剩余中间件
  } else {
    next();
  }
}, (req, res) => {
  res.send('regular');
});

// 传递错误
app.get('/user/:id', (req, res, next) => {
  if (!isValidUserId(req.params.id)) {
    next(new Error('Invalid user ID'));  // 传递错误到错误处理中间件
  }
  next();
});

// 控制权传递到下一个路由
app.get('/api/data', (req, res, next) => {
  // 前置处理
  console.log('Processing data request');
  next();
});
```

## RESTful API 路由设计

### 标准 RESTful 路由

```javascript
// 用户资源路由
app.get('/api/users',           // 获取所有用户
  userController.index);

app.get('/api/users/new',       // 显示创建用户表单（Web 应用）
  userController.new);

app.post('/api/users',          // 创建新用户
  userController.create);

app.get('/api/users/:id',       // 获取特定用户
  userController.show);

app.get('/api/users/:id/edit',  // 显示编辑用户表单（Web 应用）
  userController.edit);

app.put('/api/users/:id',       // 更新用户
  userController.update);

app.patch('/api/users/:id',     // 部分更新用户
  userController.patch);

app.delete('/api/users/:id',    // 删除用户
  userController.delete);

// 嵌套资源路由
app.get('/api/users/:userId/posts',           // 获取用户的所有文章
  postController.index);

app.post('/api/users/:userId/posts',          // 为用户创建文章
  postController.create);

app.get('/api/users/:userId/posts/:id',       // 获取用户的特定文章
  postController.show);

app.put('/api/users/:userId/posts/:id',       // 更新用户的文章
  postController.update);

app.delete('/api/users/:userId/posts/:id',    // 删除用户的文章
  postController.delete);
```

### RESTful 路由设计最佳实践

```javascript
// 使用名词复数形式
app.get('/api/users', ...);           // ✅ 正确
app.get('/api/user', ...);            // ❌ 不推荐

// 使用层级关系表示资源关系
app.get('/api/users/:userId/posts', ...);      // ✅ 正确
app.get('/api/posts?userId=123', ...);         // ❌ 不推荐

// 使用查询字符串进行过滤、排序、分页
app.get('/api/users?role=admin&sort=name&order=asc&page=1&limit=10', ...);

// 使用 HTTP 方法表示操作意图
app.post('/api/users/:id/follow', ...);        // 关注用户
app.delete('/api/users/:id/follow', ...);     // 取消关注
app.post('/api/posts/:id/like', ...);          // 点赞文章
app.delete('/api/posts/:id/like', ...);       // 取消点赞
```

## 路由高级功能

### 路由版本控制

```javascript
// 路径版本控制
app.use('/api/v1/users', v1UserRoutes);
app.use('/api/v2/users', v2UserRoutes);

// 头部版本控制
const versionMiddleware = (req, res, next) => {
  const version = req.headers['api-version'] || 'v1';
  req.apiVersion = version;
  next();
};

app.use(versionMiddleware);

app.get('/api/users', (req, res) => {
  if (req.apiVersion === 'v2') {
    // v2 逻辑
  } else {
    // v1 逻辑
  }
});

// 查询参数版本控制
app.get('/api/users', (req, res) => {
  const version = req.query.version || 'v1';
  // 根据版本处理请求
});
```

### 路由缓存

```javascript
const NodeCache = require('node-cache');
const cache = new NodeCache({ stdTTL: 600 }); // 10分钟缓存

const cacheMiddleware = (req, res, next) => {
  const key = req.originalUrl;
  const cachedData = cache.get(key);
  
  if (cachedData) {
    console.log('Cache hit');
    return res.json(cachedData);
  }
  
  // 存储原始的 json 方法
  const originalJson = res.json.bind(res);
  
  // 覆盖 json 方法
  res.json = function(data) {
    cache.set(key, data);
    return originalJson(data);
  };
  
  next();
};

app.get('/api/products', cacheMiddleware, async (req, res) => {
  const products = await Product.findAll();
  res.json(products);
});
```

### 路由速率限制

```javascript
const rateLimit = require('express-rate-limit');

const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100, // 每个 IP 限制 100 个请求
  message: 'Too many requests from this IP'
});

// 应用到所有路由
app.use('/api', limiter);

// 特定路由的更严格限制
const strictLimiter = rateLimit({
  windowMs: 60 * 60 * 1000, // 1 小时
  max: 5,
  message: 'Too many login attempts'
});

app.post('/api/login', strictLimiter, authController.login);

// 动态速率限制
const dynamicLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: (req) => {
    if (req.user && req.user.premium) {
      return 100; // 高级用户
    }
    return 10;   // 普通用户
  }
});
```

## 路由测试

### 使用 Express Router 进行测试

```javascript
// routes/userRoutes.test.js
const request = require('supertest');
const express = require('express');
const userRoutes = require('./userRoutes');

const app = express();
app.use(express.json());
app.use('/api/users', userRoutes);

describe('User Routes', () => {
  test('GET /api/users should return all users', async () => {
    const response = await request(app)
      .get('/api/users')
      .expect('Content-Type', /json/)
      .expect(200);
    
    expect(response.body).toHaveProperty('users');
    expect(Array.isArray(response.body.users)).toBe(true);
  });

  test('POST /api/users should create a new user', async () => {
    const newUser = {
      name: 'John Doe',
      email: 'john@example.com'
    };

    const response = await request(app)
      .post('/api/users')
      .send(newUser)
      .expect('Content-Type', /json/)
      .expect(201);
    
    expect(response.body).toHaveProperty('id');
    expect(response.body.name).toBe(newUser.name);
  });

  test('GET /api/users/:id should return a specific user', async () => {
    const response = await request(app)
      .get('/api/users/1')
      .expect('Content-Type', /json/)
      .expect(200);
    
    expect(response.body).toHaveProperty('id', '1');
  });
});
```

## 路由性能优化

### 路由缓存和预编译

```javascript
// 使用路由缓存
app.enable('case sensitive routing');  // 大小写敏感
app.enable('strict routing');          // 严格路由

// 避免过多的动态路由
// 不推荐
app.get('/:resource/:id', ...);  // 过于通用

// 推荐
app.get('/users/:id', ...);
app.get('/posts/:id', ...);
app.get('/products/:id', ...);
```

### 路由监控

```javascript
const routeMetrics = {};

app.use((req, res, next) => {
  const start = Date.now();
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const route = req.route ? req.route.path : req.path;
    
    if (!routeMetrics[route]) {
      routeMetrics[route] = { count: 0, totalDuration: 0 };
    }
    
    routeMetrics[route].count++;
    routeMetrics[route].totalDuration += duration;
    
    console.log(`${req.method} ${route}: ${duration}ms`);
  });
  
  next();
});

// 获取路由统计信息
app.get('/api/metrics', (req, res) => {
  const metrics = Object.entries(routeMetrics).map(([route, data]) => ({
    route,
    ...data,
    avgDuration: data.totalDuration / data.count
  }));
  
  res.json(metrics);
});
```

## 常见问题和解决方案

### 问题 1：路由顺序问题

```javascript
// 错误的顺序
app.get('/users/:id', getUserById);
app.get('/users/profile', getProfile); // 永远不会匹配到

// 正确的顺序
app.get('/users/profile', getProfile);
app.get('/users/:id', getUserById);
```

### 问题 2：404 错误处理

```javascript
// 在所有路由之后定义 404 处理
app.use((req, res, next) => {
  res.status(404).json({
    error: 'Not Found',
    path: req.originalUrl,
    method: req.method
  });
});
```

### 问题 3：动态路由匹配失败

```javascript
// 确保路由定义正确
app.get('/api/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  // userId 和 postId 都是字符串
  const userIdNum = parseInt(userId);
  const postIdNum = parseInt(postId);
  // ...
});
```

## 总结

Express 路由系统是构建 Web 应用的基础，通过本篇详细指南，你已经了解了：

- 路由系统的核心概念和基本用法
- 路由参数和查询参数的处理
- 路由模块化和组织结构
- 路由中间件的开发和使用
- RESTful API 路由设计原则
- 路由版本控制和高级功能
- 路由测试和性能优化
- 常见问题的解决方案

良好的路由设计能够显著提高应用的可维护性和可扩展性。在实际项目中，建议遵循 RESTful 原则，合理组织路由结构，使用中间件来处理横切关注点，并为路由编写充分的测试。

掌握 Express 路由系统后，你可以继续深入学习中间件开发、错误处理、安全防护等内容，以构建更加健壮和专业的应用。