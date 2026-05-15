---
title: Express 错误处理
published: 2023-03-19
description: '错误处理中间件的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 错误处理概述

错误处理是任何应用程序的关键组成部分，良好的错误处理可以提高应用的稳定性和用户体验。Express 提供了灵活的错误处理机制，允许开发者捕获和处理应用中发生的各种错误。

### 错误处理的重要性

- 防止应用程序崩溃
- 提供有意义的错误信息
- 改善用户体验
- 便于调试和监控
- 满足安全要求

### Express 错误处理机制

Express 的错误处理依赖于具有四个参数的中间件函数：`(err, req, res, next)`。

## 基础错误处理

### 错误处理中间件

```javascript
// 错误处理中间件的基本结构
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something went wrong!');
});

// 更详细的错误处理
app.use((err, req, res, next) => {
  const statusCode = err.statusCode || 500;
  const message = err.message || 'Internal Server Error';
  
  res.status(statusCode).json({
    error: {
      message: message,
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack,
        details: err.details
      })
    }
  });
});
```

### 自定义错误类

```javascript
// 自定义错误类
class AppError extends Error {
  constructor(message, statusCode = 500, details = null) {
    super(message);
    this.statusCode = statusCode;
    this.details = details;
    this.isOperational = true;
    this.timestamp = new Date().toISOString();
    
    Error.captureStackTrace(this, this.constructor);
  }
}

// 验证错误
class ValidationError extends AppError {
  constructor(message, details) {
    super(message, 400, details);
    this.name = 'ValidationError';
  }
}

// 认证错误
class AuthenticationError extends AppError {
  constructor(message = 'Authentication failed') {
    super(message, 401);
    this.name = 'AuthenticationError';
  }
}

// 授权错误
class AuthorizationError extends AppError {
  constructor(message = 'Access denied') {
    super(message, 403);
    this.name = 'AuthorizationError';
  }
}

// 资源未找到错误
class NotFoundError extends AppError {
  constructor(message = 'Resource not found') {
    super(message, 404);
    this.name = 'NotFoundError';
  }
}

// 冲突错误
class ConflictError extends AppError {
  constructor(message = 'Resource already exists') {
    super(message, 409);
    this.name = 'ConflictError';
  }
}

// 使用自定义错误
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    
    if (!user) {
      throw new NotFoundError('User not found');
    }
    
    res.json(user);
  } catch (error) {
    next(error);
  }
});
```

## 错误处理模式

### 1. 同步错误处理

```javascript
// 路由处理函数中的同步错误
app.get('/api/data', (req, res, next) => {
  try {
    // 可能抛出错误的代码
    const data = riskyOperation();
    res.json(data);
  } catch (error) {
    next(error); // 传递给错误处理中间件
  }
});

// 自定义错误
app.get('/api/admin', (req, res, next) => {
  if (req.user.role !== 'admin') {
    next(new AuthorizationError('Admin access required'));
    return;
  }
  res.json({ message: 'Welcome admin' });
});
```

### 2. 异步错误处理

```javascript
// Promise 方式
app.get('/api/users', (req, res, next) => {
  User.findAll()
    .then(users => res.json(users))
    .catch(next);
});

// async/await 方式
app.get('/api/users', async (req, res, next) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (error) {
    next(error);
  }
});

// 包装器函数
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// 使用包装器
app.get('/api/users', asyncHandler(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));
```

### 3. 多层级错误处理

```javascript
// 路由级错误处理
app.get('/api/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    throw new NotFoundError('User not found');
  }
  res.json(user);
}));

// 应用级错误处理
app.use((err, req, res, next) => {
  // 404 错误
  if (err.name === 'NotFoundError') {
    return res.status(404).json({
      error: {
        message: err.message,
        code: 'NOT_FOUND'
      }
    });
  }
  
  // 验证错误
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      error: {
        message: err.message,
        code: 'VALIDATION_ERROR',
        details: err.details
      }
    });
  }
  
  // 默认错误
  next(err);
});

// 最终错误处理
app.use((err, req, res, next) => {
  console.error('Unhandled error:', err);
  
  res.status(err.statusCode || 500).json({
    error: {
      message: err.message || 'Internal Server Error',
      code: 'INTERNAL_ERROR',
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack
      })
    }
  });
});
```

## 常见错误场景处理

### 数据库错误处理

```javascript
// 数据库连接错误
app.use((err, req, res, next) => {
  if (err.code === 'ECONNREFUSED' || err.code === 'ETIMEDOUT') {
    console.error('Database connection error:', err);
    return res.status(503).json({
      error: {
        message: 'Database connection failed',
        code: 'DATABASE_ERROR'
      }
    });
  }
  next(err);
});

// 唯一约束冲突
app.post('/api/users', asyncHandler(async (req, res) => {
  try {
    const user = await User.create(req.body);
    res.status(201).json(user);
  } catch (error) {
    if (error.code === '23505') { // PostgreSQL 唯一约束错误
      throw new ConflictError('User already exists');
    }
    throw error;
  }
}));

// 外键约束
app.delete('/api/users/:id', asyncHandler(async (req, res) => {
  try {
    await User.findByIdAndDelete(req.params.id);
    res.status(204).send();
  } catch (error) {
    if (error.code === '23503') { // PostgreSQL 外键错误
      throw new ConflictError('Cannot delete user with related records');
    }
    throw error;
  }
}));
```

### 文件上传错误处理

```javascript
const multer = require('multer');
const upload = multer({
  limits: {
    fileSize: 5 * 1024 * 1024 // 5MB
  }
});

app.post('/api/upload', upload.single('file'), (req, res, next) => {
  if (!req.file) {
    return next(new ValidationError('No file uploaded'));
  }
  res.json({ file: req.file });
});

// Multer 错误处理
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    if (err.code === 'LIMIT_FILE_SIZE') {
      return next(new ValidationError('File size exceeds limit'));
    }
    if (err.code === 'LIMIT_FILE_COUNT') {
      return next(new ValidationError('Too many files'));
    }
    if (err.code === 'LIMIT_UNEXPECTED_FILE') {
      return next(new ValidationError('Unexpected file field'));
    }
  }
  next(err);
});
```

### 请求验证错误处理

```javascript
const Joi = require('joi');

const userSchema = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).required()
});

app.post('/api/users', asyncHandler(async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  
  if (error) {
    const details = error.details.map(detail => ({
      field: detail.path.join('.'),
      message: detail.message
    }));
    
    throw new ValidationError('Validation failed', details);
  }
  
  const user = await User.create(value);
  res.status(201).json(user);
}));
```

### API 错误处理

```javascript
// API 错误响应格式
app.use((err, req, res, next) => {
  const errorResponse = {
    success: false,
    error: {
      message: err.message || 'Internal Server Error',
      code: err.code || 'INTERNAL_ERROR',
      ...(err.details && { details: err.details }),
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack
      })
    },
    timestamp: new Date().toISOString(),
    path: req.path
  };
  
  res.status(err.statusCode || 500).json(errorResponse);
});
```

## 错误日志记录

### 详细错误日志

```javascript
const fs = require('fs');
const path = require('path');

// 错误日志文件
const errorLogFile = fs.createWriteStream(
  path.join(__dirname, 'logs', 'errors.log'),
  { flags: 'a' }
);

// 错误日志中间件
const errorLogger = (err, req, res, next) => {
  const errorInfo = {
    timestamp: new Date().toISOString(),
    message: err.message,
    stack: err.stack,
    statusCode: err.statusCode,
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    body: req.body,
    params: req.params,
    query: req.query
  };
  
  // 写入日志文件
  errorLogFile.write(JSON.stringify(errorInfo) + '\n');
  
  // 控制台输出
  console.error('Error:', errorInfo);
  
  next(err);
};

app.use(errorLogger);
```

### 错误监控和告警

```javascript
// 错误统计
const errorStats = {
  total: 0,
  byType: {},
  byStatusCode: {}
};

const errorMonitor = (err, req, res, next) => {
  errorStats.total++;
  
  // 按类型统计
  const errorType = err.name || 'Unknown';
  errorStats.byType[errorType] = (errorStats.byType[errorType] || 0) + 1;
  
  // 按状态码统计
  const statusCode = err.statusCode || 500;
  errorStats.byStatusCode[statusCode] = (errorStats.byStatusCode[statusCode] || 0) + 1;
  
  // 严重错误告警
  if (statusCode >= 500) {
    console.error('Critical error:', err);
    // 发送告警（邮件、Slack 等）
    sendAlert(err, req);
  }
  
  next(err);
};

function sendAlert(err, req) {
  // 实现告警逻辑
  console.log('Alert sent for error:', err.message);
}

// 获取错误统计
app.get('/api/error-stats', (req, res) => {
  res.json(errorStats);
});
```

## 生产环境错误处理

### 环境相关错误处理

```javascript
app.use((err, req, res, next) => {
  // 开发环境：显示详细错误信息
  if (process.env.NODE_ENV === 'development') {
    res.status(err.statusCode || 500).json({
      error: {
        message: err.message,
        stack: err.stack,
        details: err.details
      }
    });
    return;
  }
  
  // 生产环境：隐藏敏感信息
  if (err.isOperational) {
    // 已知错误，返回用户友好的消息
    res.status(err.statusCode).json({
      error: {
        message: err.message,
        code: err.code
      }
    });
  } else {
    // 未知错误，记录日志但返回通用消息
    console.error('Unexpected error:', err);
    res.status(500).json({
      error: {
        message: 'An unexpected error occurred',
        code: 'INTERNAL_ERROR'
      }
    });
  }
});
```

### 全局错误处理

```javascript
// 未捕获的 Promise 拒绝
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
  // 优雅关闭
  gracefulShutdown();
});

// 未捕获的异常
process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  // 优雅关闭
  gracefulShutdown();
});

// 优雅关闭
function gracefulShutdown() {
  console.log('Gracefully shutting down...');
  server.close(() => {
    console.log('Server closed');
    process.exit(1);
  });
  
  // 强制关闭超时
  setTimeout(() => {
    console.error('Forced shutdown');
    process.exit(1);
  }, 10000);
}
```

## 404 错误处理

### 自定义 404 处理

```javascript
// 404 错误处理（必须在所有路由之后）
app.use((req, res, next) => {
  const error = new NotFoundError(`Route ${req.originalUrl} not found`);
  next(error);
});

// 或者直接响应
app.use((req, res) => {
  res.status(404).json({
    error: {
      message: 'Not Found',
      code: 'NOT_FOUND',
      path: req.originalUrl,
      method: req.method
    }
  });
});

// API 和前端不同的 404 处理
app.use('/api', (req, res) => {
  res.status(404).json({
    error: {
      message: 'API endpoint not found',
      code: 'API_NOT_FOUND',
      path: req.originalUrl
    }
  });
});

app.use((req, res) => {
  res.status(404).sendFile(path.join(__dirname, 'public', '404.html'));
});
```

## 错误处理最佳实践

### 1. 错误分类

```javascript
// 运营错误（可预期）
class OperationalError extends AppError {
  constructor(message, statusCode = 500) {
    super(message, statusCode);
    this.isOperational = true;
  }
}

// 编程错误（不可预期）
class ProgrammingError extends AppError {
  constructor(message) {
    super(message, 500);
    this.isOperational = false;
  }
}

// 错误处理中间件
app.use((err, req, res, next) => {
  if (err.isOperational) {
    // 返回用户友好的消息
    return res.status(err.statusCode).json({
      error: {
        message: err.message,
        code: err.code
      }
    });
  }
  
  // 编程错误需要记录详细信息
  console.error('Programming error:', err);
  res.status(500).json({
    error: {
      message: 'Internal Server Error',
      code: 'INTERNAL_ERROR'
    }
  });
});
```

### 2. 错误恢复策略

```javascript
// 重试机制
const retryOperation = async (operation, maxRetries = 3, delay = 1000) => {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }
      console.log(`Retry attempt ${attempt}/${maxRetries}`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
};

// 使用重试
app.get('/api/external-data', asyncHandler(async (req, res) => {
  const data = await retryOperation(() => 
    fetchExternalData(req.params.id)
  );
  res.json(data);
}));
```

### 3. 错误处理链

```javascript
// 错误处理中间件链
app.use((err, req, res, next) => {
  // 记录错误
  console.error(err);
  
  // 发送错误监控
  if (!err.isOperational) {
    sendErrorToMonitoring(err, req);
  }
  
  next(err);
});

app.use((err, req, res, next) => {
  // 根据错误类型处理
  if (err.name === 'ValidationError') {
    return handleValidationError(err, res);
  }
  
  if (err.name === 'AuthenticationError') {
    return handleAuthError(err, res);
  }
  
  next(err);
});

app.use((err, req, res, next) => {
  // 最终错误处理
  res.status(err.statusCode || 500).json({
    error: {
      message: err.message || 'Internal Server Error',
      code: err.code || 'INTERNAL_ERROR'
    }
  });
});
```

## 错误处理测试

### 错误处理测试

```javascript
// 错误处理测试示例
describe('Error Handling', () => {
  test('should handle 404 errors', async () => {
    const response = await request(app)
      .get('/nonexistent-route')
      .expect(404);
    
    expect(response.body.error).toBeDefined();
    expect(response.body.error.code).toBe('NOT_FOUND');
  });
  
  test('should handle validation errors', async () => {
    const response = await request(app)
      .post('/api/users')
      .send({ name: 'John' }) // 缺少必填字段
      .expect(400);
    
    expect(response.body.error.code).toBe('VALIDATION_ERROR');
    expect(response.body.error.details).toBeDefined();
  });
  
  test('should handle authentication errors', async () => {
    const response = await request(app)
      .get('/api/protected')
      .expect(401);
    
    expect(response.body.error.code).toBe('AUTHENTICATION_ERROR');
  });
  
  test('should handle server errors', async () => {
    // 模拟服务器错误
    jest.spyOn(User, 'findAll').mockRejectedValue(new Error('Database error'));
    
    const response = await request(app)
      .get('/api/users')
      .expect(500);
    
    expect(response.body.error.code).toBe('INTERNAL_ERROR');
  });
});
```

## 常见问题和解决方案

### 问题 1：异步错误未捕获

```javascript
// 错误示例
app.get('/api/users', async (req, res) => {
  const users = await User.findAll();
  res.json(users);
  // 如果 Promise 被拒绝，错误不会被捕获
});

// 解决方案 1：使用 try-catch
app.get('/api/users', async (req, res, next) => {
  try {
    const users = await User.findAll();
    res.json(users);
  } catch (error) {
    next(error);
  }
});

// 解决方案 2：使用包装器
app.get('/api/users', asyncHandler(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));
```

### 问题 2：错误信息泄露

```javascript
// 错误示例：开发环境配置用于生产
app.use((err, req, res, next) => {
  res.status(500).json({
    error: {
      message: err.message,
      stack: err.stack
    }
  });
});

// 正确示例：根据环境返回不同信息
app.use((err, req, res, next) => {
  if (process.env.NODE_ENV === 'production') {
    res.status(500).json({
      error: {
        message: 'Internal Server Error'
      }
    });
  } else {
    res.status(500).json({
      error: {
        message: err.message,
        stack: err.stack
      }
    });
  }
});
```

### 问题 3：404 错误处理位置错误

```javascript
// 错误示例：404 处理在路由之前
app.use((req, res, next) => {
  res.status(404).send('Not Found');
});

app.get('/api/users', userController.getUsers);
// 永远不会被访问到

// 正确示例：404 处理在所有路由之后
app.get('/api/users', userController.getUsers);
app.use((req, res, next) => {
  res.status(404).send('Not Found');
});
```

## 完整的错误处理配置

```javascript
const express = require('express');
const { asyncHandler } = require('./utils/asyncHandler');
const { 
  ValidationError, 
  AuthenticationError,
  NotFoundError,
  AppError 
} = require('./utils/errors');

const app = express();

// 请求解析
app.use(express.json());

// 请求日志
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 路由
app.get('/api/users', asyncHandler(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));

app.post('/api/users', asyncHandler(async (req, res) => {
  const { error, value } = userSchema.validate(req.body);
  if (error) {
    throw new ValidationError('Validation failed', error.details);
  }
  const user = await User.create(value);
  res.status(201).json(user);
}));

app.get('/api/users/:id', asyncHandler(async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) {
    throw new NotFoundError('User not found');
  }
  res.json(user);
}));

// 404 错误处理
app.use((req, res, next) => {
  next(new NotFoundError(`Route ${req.originalUrl} not found`));
});

// 错误日志记录
app.use((err, req, res, next) => {
  console.error('Error:', {
    message: err.message,
    stack: err.stack,
    statusCode: err.statusCode,
    path: req.path,
    method: req.method
  });
  next(err);
});

// 错误处理
app.use((err, req, res, next) => {
  // 运营错误
  if (err.isOperational) {
    return res.status(err.statusCode || 500).json({
      success: false,
      error: {
        message: err.message,
        code: err.code || 'OPERATIONAL_ERROR',
        ...(err.details && { details: err.details })
      }
    });
  }
  
  // 未预期的错误
  console.error('Unexpected error:', err);
  
  res.status(500).json({
    success: false,
    error: {
      message: 'Internal Server Error',
      code: 'INTERNAL_ERROR',
      ...(process.env.NODE_ENV === 'development' && {
        stack: err.stack
      })
    }
  });
});

// 全局错误处理
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection:', reason);
  gracefulShutdown();
});

process.on('uncaughtException', (error) => {
  console.error('Uncaught Exception:', error);
  gracefulShutdown();
});

module.exports = app;
```

## 总结

Express 错误处理是构建稳定应用的关键，通过本篇详细指南，你已经了解了：

- 错误处理的核心概念和重要性
- 自定义错误类的设计和使用
- 同步和异步错误处理模式
- 常见错误场景的处理方法
- 错误日志记录和监控
- 生产环境下的错误处理策略
- 404 错误的处理方法
- 错误处理的最佳实践
- 错误处理的测试方法
- 完整的错误处理配置示例

良好的错误处理不仅能防止应用崩溃，还能提供更好的用户体验和调试信息。在实际项目中，建议：

1. 设计清晰的错误类型和分类
2. 实现详细的错误日志记录
3. 根据环境提供不同级别的错误信息
4. 实现错误监控和告警机制
5. 为错误处理编写充分的测试
6. 遵循安全最佳实践，避免敏感信息泄露

掌握 Express 错误处理是构建生产级应用的重要技能，能够显著提高应用的稳定性和可靠性。