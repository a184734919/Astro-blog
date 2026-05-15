---
title: Express 中间件详解
published: 2023-03-11
description: '中间件的原理和开发的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 中间件概述

中间件是 Express 框架的核心概念之一，它是一个函数，可以访问请求对象、响应对象和应用程序请求-响应循环中的下一个中间件函数。中间件机制使得开发者能够在请求处理的各个阶段插入自定义的处理逻辑。

### 中间件工作原理

```
请求 → 中间件1 → 中间件2 → 路由处理器 → 响应
         ↓         ↓
       处理逻辑   处理逻辑
```

### 中间件函数签名

```javascript
function middleware(req, res, next) {
  // 处理请求
  // 调用 next() 传递控制权给下一个中间件
}
```

### 中间件的职责

- 执行任何代码
- 修改请求和响应对象
- 结束请求-响应循环
- 调用下一个中间件函数

## 中间件类型

### 1. 应用级中间件

应用级中间件绑定到 `app` 对象上，使用 `app.use()` 方法。

```javascript
const express = require('express');
const app = express();

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
    console.log('Development mode middleware');
    next();
  });
}

// 多个中间件
app.use('/api', 
  (req, res, next) => {
    console.log('First middleware');
    next();
  },
  (req, res, next) => {
    console.log('Second middleware');
    next();
  }
);
```

### 2. 路由级中间件

路由级中间件的工作方式与应用级中间件相同，但它是绑定到特定的路由上。

```javascript
// 单个路由中间件
app.get('/users', authenticate, userController.getAllUsers);

// 多个路由中间件
app.get('/users/:id',
  authenticate,
  authorize(['user', 'admin']),
  validateUserParams,
  userController.getUserById
);

// 使用 express.Router
const router = express.Router();

router.use(authenticate); // 路由器级别的中间件

router.get('/profile', (req, res) => {
  res.json({ user: req.user });
});

app.use('/api/users', router);
```

### 3. 错误处理中间件

错误处理中间件有四个参数：`(err, req, res, next)`。

```javascript
// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(err.status || 500).json({
    error: {
      message: err.message || 'Internal Server Error',
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

// 自定义错误类
class AppError extends Error {
  constructor(message, status) {
    super(message);
    this.status = status;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

// 使用自定义错误
app.get('/api/error', (req, res, next) => {
  next(new AppError('Something went wrong', 400));
});
```

### 4. 内置中间件

Express 提供了一些内置的中间件：

```javascript
const express = require('express');
const app = express();

// 解析 JSON 请求体
app.use(express.json());

// 解析 URL 编码的请求体
app.use(express.urlencoded({ extended: true }));

// 解析原始请求体
app.use(express.raw());

// 解析文本请求体
app.use(express.text());

// 静态文件服务
app.use(express.static('public'));

// 自定义静态文件路径
app.use('/static', express.static('public'));
app.use('/assets', express.static('assets'));
app.use('/uploads', express.static('uploads'));

// 设置静态文件缓存时间
app.use('/static', express.static('public', {
  maxAge: '1d',
  etag: true,
  lastModified: true
}));
```

### 5. 第三方中间件

常用的第三方中间件：

```javascript
const express = require('express');
const app = express();

// Cookie 解析
const cookieParser = require('cookie-parser');
app.use(cookieParser());

// 会话管理
const session = require('express-session');
app.use(session({
  secret: 'your-secret-key',
  resave: false,
  saveUninitialized: false,
  cookie: { secure: false }
}));

// 日志记录
const morgan = require('morgan');
app.use(morgan('combined'));

// CORS 支持
const cors = require('cors');
app.use(cors());

// 请求体解析（用于文件上传）
const bodyParser = require('body-parser');
app.use(bodyParser.json());
app.use(bodyParser.urlencoded({ extended: true }));

// 压缩响应
const compression = require('compression');
app.use(compression());

// 安全头部
const helmet = require('helmet');
app.use(helmet());

// 速率限制
const rateLimit = require('express-rate-limit');
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 分钟
  max: 100
});
app.use('/api', limiter);
```

## 自定义中间件开发

### 基础中间件

```javascript
// 日志中间件
const logger = (req, res, next) => {
  const start = Date.now();
  
  // 响应完成时记录
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.url} - ${res.statusCode} - ${duration}ms`);
  });
  
  next();
};

// 使用中间件
app.use(logger);
```

### 认证中间件

```javascript
const jwt = require('jsonwebtoken');

const authenticate = (req, res, next) => {
  try {
    // 从请求头获取 token
    const authHeader = req.headers.authorization;
    
    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({ error: 'No token provided' });
    }
    
    const token = authHeader.substring(7);
    
    // 验证 token
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // 将用户信息添加到请求对象
    req.user = decoded;
    next();
    
  } catch (error) {
    if (error.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Invalid token' });
    }
    if (error.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    next(error);
  }
};

// 可选认证中间件
const optionalAuth = (req, res, next) => {
  const authHeader = req.headers.authorization;
  
  if (authHeader && authHeader.startsWith('Bearer ')) {
    try {
      const token = authHeader.substring(7);
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = decoded;
    } catch (error) {
      // 忽略 token 错误，继续执行
    }
  }
  
  next();
};
```

### 授权中间件

```javascript
// 基于角色的授权
const authorize = (...allowedRoles) => {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    if (!allowedRoles.includes(req.user.role)) {
      return res.status(403).json({ 
        error: 'Insufficient permissions',
        required: allowedRoles 
      });
    }
    
    next();
  };
};

// 基于权限的授权
const hasPermission = (permission) => {
  return (req, res, next) => {
    if (!req.user || !req.user.permissions || !req.user.permissions.includes(permission)) {
      return res.status(403).json({ 
        error: 'Permission denied',
        required: permission 
      });
    }
    next();
  };
};

// 使用示例
app.get('/api/admin/users', 
  authenticate, 
  authorize('admin'), 
  userController.getAllUsers
);

app.post('/api/posts', 
  authenticate, 
  hasPermission('create:post'), 
  postController.createPost
);
```

### 验证中间件

```javascript
const Joi = require('joi');

// 通用验证中间件
const validate = (schema) => {
  return (req, res, next) => {
    const { error } = schema.validate(req.body, { abortEarly: false });
    
    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));
      
      return res.status(400).json({
        error: 'Validation failed',
        details: errors
      });
    }
    
    next();
  };
};

// 定义验证模式
const userValidationSchema = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/^(?=.*[A-Za-z])(?=.*\d)[A-Za-z\d]{8,}$/).required(),
  age: Joi.number().integer().min(18).max(120).optional(),
  role: Joi.string().valid('user', 'admin', 'moderator').default('user')
});

// 使用验证中间件
app.post('/api/users', 
  validate(userValidationSchema), 
  userController.createUser
);

// 查询参数验证
const queryValidationSchema = Joi.object({
  page: Joi.number().integer().min(1).default(1),
  limit: Joi.number().integer().min(1).max(100).default(10),
  sort: Joi.string().valid('name', 'email', 'createdAt').default('createdAt'),
  order: Joi.string().valid('asc', 'desc').default('asc')
});

const validateQuery = (schema) => {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.query);
    
    if (error) {
      return res.status(400).json({ error: error.details[0].message });
    }
    
    req.query = value; // 使用验证后的值
    next();
  };
};

app.get('/api/users', validateQuery(queryValidationSchema), userController.getUsers);
```

### 缓存中间件

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
  
  console.log('Cache miss');
  
  // 存储原始的 json 方法
  const originalJson = res.json.bind(res);
  
  // 覆盖 json 方法
  res.json = function(data) {
    cache.set(key, data);
    return originalJson(data);
  };
  
  next();
};

// 使用缓存中间件
app.get('/api/products', cacheMiddleware, productController.getProducts);

// 清除缓存
const clearCache = (pattern) => {
  const keys = cache.keys();
  keys.forEach(key => {
    if (key.includes(pattern)) {
      cache.del(key);
    }
  });
};

app.post('/api/products', 
  (req, res, next) => {
    // 在创建产品后清除缓存
    const originalJson = res.json.bind(res);
    res.json = function(data) {
      clearCache('/api/products');
      return originalJson(data);
    };
    next();
  },
  productController.createProduct
);
```

### 性能监控中间件

```javascript
const performanceMonitor = (req, res, next) => {
  const start = Date.now();
  
  // 监控内存使用
  const memoryUsage = process.memoryUsage();
  const startMemory = memoryUsage.heapUsed;
  
  res.on('finish', () => {
    const duration = Date.now() - start;
    const endMemory = process.memoryUsage().heapUsed;
    const memoryDelta = endMemory - startMemory;
    
    const performanceData = {
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: `${duration}ms`,
      memoryDelta: `${(memoryDelta / 1024 / 1024).toFixed(2)}MB`,
      timestamp: new Date().toISOString()
    };
    
    console.log('Performance:', JSON.stringify(performanceData));
    
    // 记录慢请求
    if (duration > 1000) {
      console.warn('Slow request:', performanceData);
    }
  });
  
  next();
};

app.use(performanceMonitor);
```

### 请求日志中间件

```javascript
const fs = require('fs');
const path = require('path');

const accessLogStream = fs.createWriteStream(
  path.join(__dirname, 'access.log'),
  { flags: 'a' }
);

const requestLogger = (req, res, next) => {
  const logData = {
    timestamp: new Date().toISOString(),
    method: req.method,
    url: req.url,
    ip: req.ip,
    userAgent: req.headers['user-agent'],
    contentType: req.headers['content-type']
  };
  
  accessLogStream.write(JSON.stringify(logData) + '\n');
  next();
};

app.use(requestLogger);
```

## 中间件最佳实践

### 1. 中间件组织

```javascript
// middlewares/index.js
const authenticate = require('./auth');
const authorize = require('./authorize');
const validate = require('./validation');
const errorHandler = require('./errorHandler');
const logger = require('./logger');
const rateLimiter = require('./rateLimiter');

module.exports = {
  authenticate,
  authorize,
  validate,
  errorHandler,
  logger,
  rateLimiter
};

// 使用
const { authenticate, authorize, errorHandler } = require('./middlewares');

app.get('/api/users', 
  authenticate, 
  authorize('admin'), 
  userController.getUsers
);

app.use(errorHandler);
```

### 2. 中间件配置

```javascript
// 可配置的中间件工厂
const createLogger = (options = {}) => {
  const {
    logFile = 'access.log',
    enabled = true,
    includeBody = false
  } = options;
  
  return (req, res, next) => {
    if (!enabled) {
      return next();
    }
    
    const logData = {
      timestamp: new Date().toISOString(),
      method: req.method,
      url: req.url
    };
    
    if (includeBody) {
      logData.body = req.body;
    }
    
    console.log(JSON.stringify(logData));
    next();
  };
};

// 使用配置的中间件
app.use(createLogger({
  enabled: process.env.NODE_ENV === 'production',
  includeBody: false
}));
```

### 3. 中间件错误处理

```javascript
// 异步中间件错误处理
const asyncHandler = (fn) => {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
};

// 使用异步中间件
app.get('/api/users', 
  asyncHandler(async (req, res) => {
    const users = await User.findAll();
    res.json(users);
  })
);

// 错误包装器
const wrapMiddleware = (middleware) => {
  return (req, res, next) => {
    try {
      middleware(req, res, next);
    } catch (error) {
      next(error);
    }
  };
};
```

### 4. 中间件条件执行

```javascript
// 条件中间件
const conditionalMiddleware = (condition, middleware) => {
  return (req, res, next) => {
    if (condition(req, res)) {
      middleware(req, res, next);
    } else {
      next();
    }
  };
};

// 使用
const requiresAuth = conditionalMiddleware(
  (req) => req.path.startsWith('/api/protected'),
  authenticate
);

app.use(requiresAuth);

// 基于环境的中间件
const devOnly = (middleware) => {
  return (req, res, next) => {
    if (process.env.NODE_ENV === 'development') {
      middleware(req, res, next);
    } else {
      next();
    }
  };
};

app.use(devOnly(logger));
```

## 完整的中间件应用示例

### Express 应用配置

```javascript
const express = require('express');
const { 
  authenticate, 
  authorize, 
  validate, 
  errorHandler,
  logger,
  rateLimiter
} = require('./middlewares');
const userValidationSchema = require('./validations/user');
const productValidationSchema = require('./validations/product');

const app = express();

// 基础中间件
app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use(express.static('public'));

// 日志中间件
app.use(logger);

// CORS
app.use(cors());

// 安全头部
app.use(helmet());

// 速率限制
app.use('/api', rateLimiter);

// API 路由
app.use('/api/users', 
  authenticate,
  userRoutes
);

app.use('/api/products',
  authenticate,
  productRoutes
);

// 管理员路由
app.use('/api/admin',
  authenticate,
  authorize('admin'),
  adminRoutes
);

// 错误处理
app.use(errorHandler);

module.exports = app;
```

## 中间件测试

### 中间件单元测试

```javascript
// middlewares/auth.test.js
const { authenticate } = require('./auth');
const jwt = require('jsonwebtoken');

describe('Authentication Middleware', () => {
  let req, res, next;
  
  beforeEach(() => {
    req = {
      headers: {}
    };
    res = {
      status: jest.fn().mockReturnThis(),
      json: jest.fn().mockReturnThis()
    };
    next = jest.fn();
  });
  
  test('should pass with valid token', () => {
    const token = jwt.sign({ id: 1, role: 'user' }, 'secret');
    req.headers.authorization = `Bearer ${token}`;
    
    authenticate(req, res, next);
    
    expect(next).toHaveBeenCalled();
    expect(req.user).toBeDefined();
    expect(req.user.id).toBe(1);
  });
  
  test('should fail with no token', () => {
    authenticate(req, res, next);
    
    expect(res.status).toHaveBeenCalledWith(401);
    expect(next).not.toHaveBeenCalled();
  });
  
  test('should fail with invalid token', () => {
    req.headers.authorization = 'Bearer invalid-token';
    
    authenticate(req, res, next);
    
    expect(res.status).toHaveBeenCalledWith(401);
  });
});
```

## 常见问题和解决方案

### 问题 1：中间件执行顺序

```javascript
// 确保中间件的正确顺序
app.use(express.json());              // 1. 解析请求体
app.use(logger);                      // 2. 记录日志
app.use(authenticate);                // 3. 认证
app.use('/api', routes);              // 4. 路由
app.use(errorHandler);                // 5. 错误处理（必须在最后）
```

### 问题 2：忘记调用 next()

```javascript
// 错误示例
app.use((req, res) => {
  console.log('Request received');
  // 忘记调用 next()，请求会挂起
});

// 正确示例
app.use((req, res, next) => {
  console.log('Request received');
  next(); // 必须调用 next()
});
```

### 问题 3：异步中间件错误处理

```javascript
// 错误示例
app.get('/api/users', async (req, res) => {
  const users = await User.findAll();
  res.json(users);
  // 如果 Promise 被拒绝，错误不会被捕获
});

// 正确示例
app.get('/api/users', asyncHandler(async (req, res) => {
  const users = await User.findAll();
  res.json(users);
}));
```

## 总结

Express 中间件系统是其最强大和灵活的特性之一，通过本篇详细指南，你已经了解了：

- 中间件的工作原理和核心概念
- 不同类型中间件的特点和使用场景
- 如何开发自定义中间件
- 常用的中间件模式和最佳实践
- 中间件的错误处理和异步支持
- 中间件的测试方法
- 常见问题的解决方案

中间件机制让开发者能够在请求处理的各个阶段插入自定义逻辑，实现关注点分离和代码复用。合理使用中间件可以显著提高代码的可维护性和可扩展性。

在实际项目中，建议将中间件按功能组织，遵循单一职责原则，为中间件编写充分的测试，并注意中间件的执行顺序和错误处理。掌握中间件开发是成为 Express 专家的重要一步。