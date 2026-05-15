---
title: Express 会话管理
published: 2023-03-30
description: 'Cookie 和 Session的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 会话管理概述

会话管理是 Web 应用中识别和跟踪用户状态的关键技术。Express 提供了多种会话管理方式，包括 Cookie、Session 以及现代的 Token 认证。

### 为什么需要会话管理

HTTP 协议是无状态的，无法在请求之间保持状态信息。会话管理允许我们：

- 识别用户身份
- 保持用户登录状态
- 存储用户偏好设置
- 实现购物车等功能
- 追踪用户行为

### 会话管理方案

1. **Cookie**: 客户端存储方案
2. **Session**: 服务器端存储方案
3. **JWT (JSON Web Token)**: 无状态认证方案
4. **混合方案**: 结合多种方案的优势

## Cookie 基础

### Cookie 概念

Cookie 是服务器发送到用户浏览器并保存在浏览器上的一小块数据，它会在浏览器向服务器发送请求时自动携带。

### Cookie 基本属性

```javascript
const cookieOptions = {
  domain: 'example.com',       // Cookie 的域
  path: '/',                    // Cookie 的路径
  expires: new Date(),          // 过期时间
  maxAge: 86400000,            // 最大存活时间（毫秒）
  secure: true,                // 仅 HTTPS 传输
  httpOnly: true,              // 禁止客户端 JavaScript 访问
  sameSite: 'strict',          // 防止 CSRF 攻击
  signed: false                // 是否签名
};
```

### Express Cookie 操作

```javascript
const express = require('express');
const cookieParser = require('cookie-parser');
const app = express();

// 使用 cookie-parser 中间件
app.use(cookieParser('your-secret-key'));

// 设置 Cookie
app.get('/set-cookie', (req, res) => {
  // 基础 Cookie
  res.cookie('user', 'john', {
    maxAge: 900000,
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production'
  });
  
  // 复杂 Cookie 对象
  res.cookie('session', {
    userId: 1,
    username: 'john',
    role: 'user'
  }, {
    maxAge: 86400000,
    httpOnly: true
  });
  
  res.send('Cookie set');
});

// 读取 Cookie
app.get('/read-cookie', (req, res) => {
  const user = req.cookies.user;
  const session = req.cookies.session;
  
  res.json({
    user,
    session
  });
});

// 清除 Cookie
app.get('/clear-cookie', (req, res) => {
  res.clearCookie('user');
  res.clearCookie('session');
  res.send('Cookies cleared');
});

// 签名 Cookie（安全性更高）
app.get('/set-signed-cookie', (req, res) => {
  res.cookie('secure_token', 'abc123', {
    signed: true,
    maxAge: 86400000,
    httpOnly: true
  });
  res.send('Signed cookie set');
});

// 读取签名 Cookie
app.get('/read-signed-cookie', (req, res) => {
  const token = req.signedCookies.secure_token;
  res.json({ token });
});
```

## Session 管理

### Session 概念

Session 是在服务器端存储用户状态信息的机制，通常通过 Cookie 中的 Session ID 来关联用户的具体会话数据。

### 基础 Session 配置

```javascript
const express = require('express');
const session = require('express-session');
const app = express();

// Session 基础配置
app.use(session({
  secret: 'your-secret-key',
  resave: false,                    // 是否每次请求都重新保存 session
  saveUninitialized: false,         // 是否保存未初始化的 session
  cookie: {
    secure: false,                  // 仅 HTTPS
    httpOnly: true,                 // 防止 XSS 攻击
    maxAge: 1000 * 60 * 60 * 24,    // 24 小时
    sameSite: 'lax'                 // CSRF 防护
  },
  name: 'sessionId',                // Cookie 名称
  rolling: true,                    // 每次请求重置过期时间
  store: new session.MemoryStore()  // 存储引擎
}));

// 设置 Session 数据
app.get('/login', (req, res) => {
  req.session.userId = 1;
  req.session.username = 'john';
  req.session.role = 'admin';
  req.session.lastLogin = new Date();
  
  res.send('Session created');
});

// 读取 Session 数据
app.get('/profile', (req, res) => {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'Not logged in' });
  }
  
  res.json({
    userId: req.session.userId,
    username: req.session.username,
    role: req.session.role,
    lastLogin: req.session.lastLogin
  });
});

// 销毁 Session
app.get('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Logout failed' });
    }
    res.send('Logged out');
  });
});

// 重新生成 Session ID（用于提升安全性）
app.post('/sensitive-action', (req, res) => {
  req.session.regenerate((err) => {
    if (err) {
      return res.status(500).json({ error: 'Session regeneration failed' });
    }
    
    req.session.userId = req.session.userId; // 保留必要数据
    res.send('Sensitive action completed');
  });
});
```

## Session 存储方案

### 1. 内存存储（开发环境）

```javascript
const session = require('express-session');
const MemoryStore = session.MemoryStore;

app.use(session({
  secret: 'dev-secret',
  store: new MemoryStore(),
  cookie: { maxAge: 86400000 }
}));
```

### 2. Redis 存储（生产环境推荐）

```javascript
const RedisStore = require('connect-redis')(session);
const redis = require('redis');

const redisClient = redis.createClient({
  host: 'localhost',
  port: 6379,
  password: 'your-redis-password'
});

redisClient.on('error', (err) => {
  console.error('Redis connection error:', err);
});

app.use(session({
  secret: 'your-secret-key',
  store: new RedisStore({
    client: redisClient,
    prefix: 'sess:',
    ttl: 86400,                    // Session 过期时间（秒）
    disableTouch: false           // 是否禁用刷新过期时间
  }),
  cookie: {
    maxAge: 86400000,
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production'
  }
}));
```

### 3. MongoDB 存储

```javascript
const MongoStore = require('connect-mongo');
const mongoose = require('mongoose');

mongoose.connect('mongodb://localhost:27017/session_db');

app.use(session({
  secret: 'your-secret-key',
  store: MongoStore.create({
    mongoUrl: 'mongodb://localhost:27017/session_db',
    collectionName: 'sessions',
    ttl: 86400,
    autoRemove: 'native'
  }),
  cookie: {
    maxAge: 86400000,
    httpOnly: true
  }
}));
```

### 4. MySQL 存储

```javascript
const MySQLStore = require('express-mysql-session')(session);
const mysql = require('mysql2/promise');

const options = {
  host: 'localhost',
  port: 3306,
  user: 'root',
  password: 'password',
  database: 'session_db',
  createDatabaseTable: true
};

const connection = mysql.createPool(options);

const sessionStore = new MySQLStore(options, connection);

app.use(session({
  secret: 'your-secret-key',
  store: sessionStore,
  cookie: {
    maxAge: 86400000,
    httpOnly: true
  }
}));
```

## 会话认证实现

### 完整的用户认证系统

```javascript
const express = require('express');
const session = require('express-session');
const bcrypt = require('bcrypt');
const { body, validationResult } = require('express-validator');

const app = express();

// Session 配置
app.use(session({
  secret: process.env.SESSION_SECRET || 'dev-secret',
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    maxAge: 1000 * 60 * 60 * 24, // 24 小时
    sameSite: 'strict'
  }
}));

// 用户登录
app.post('/api/login', [
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 })
], async (req, res) => {
  // 验证输入
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  const { email, password } = req.body;
  
  try {
    // 查找用户
    const user = await User.findOne({ where: { email } });
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // 验证密码
    const isValidPassword = await bcrypt.compare(password, user.password);
    if (!isValidPassword) {
      return res.status(401).json({ error: 'Invalid credentials' });
    }
    
    // 创建 Session
    req.session.userId = user.id;
    req.session.username = user.username;
    req.session.role = user.role;
    req.session.isLoggedIn = true;
    
    // 记录登录时间
    req.session.lastLogin = new Date();
    await req.session.save();
    
    res.json({
      success: true,
      user: {
        id: user.id,
        username: user.username,
        email: user.email
      }
    });
  } catch (error) {
    console.error('Login error:', error);
    res.status(500).json({ error: 'Login failed' });
  }
});

// 用户注册
app.post('/api/register', [
  body('username').trim().isLength({ min: 3, max: 20 }),
  body('email').isEmail().normalizeEmail(),
  body('password').isLength({ min: 6 })
], async (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(400).json({ errors: errors.array() });
  }
  
  const { username, email, password } = req.body;
  
  try {
    // 检查用户是否已存在
    const existingUser = await User.findOne({ 
      where: { 
        $or: [
          { email },
          { username }
        ]
      }
    });
    
    if (existingUser) {
      return res.status(409).json({ error: 'User already exists' });
    }
    
    // 加密密码
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // 创建用户
    const user = await User.create({
      username,
      email,
      password: hashedPassword,
      role: 'user'
    });
    
    // 自动登录
    req.session.userId = user.id;
    req.session.username = user.username;
    req.session.role = user.role;
    req.session.isLoggedIn = true;
    
    res.status(201).json({
      success: true,
      user: {
        id: user.id,
        username: user.username,
        email: user.email
      }
    });
  } catch (error) {
    console.error('Registration error:', error);
    res.status(500).json({ error: 'Registration failed' });
  }
});

// 获取当前用户信息
app.get('/api/me', (req, res) => {
  if (!req.session.isLoggedIn) {
    return res.status(401).json({ error: 'Not logged in' });
  }
  
  res.json({
    user: {
      id: req.session.userId,
      username: req.session.username,
      role: req.session.role,
      lastLogin: req.session.lastLogin
    }
  });
});

// 用户登出
app.post('/api/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      console.error('Logout error:', err);
      return res.status(500).json({ error: 'Logout failed' });
    }
    
    res.clearCookie('sessionId');
    res.json({ success: true });
  });
});

// 中间件：检查登录状态
const requireLogin = (req, res, next) => {
  if (!req.session.isLoggedIn) {
    return res.status(401).json({ error: 'Authentication required' });
  }
  next();
};

// 中间件：检查角色权限
const requireRole = (...roles) => {
  return (req, res, next) => {
    if (!req.session.isLoggedIn) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    if (!roles.includes(req.session.role)) {
      return res.status(403).json({ error: 'Insufficient permissions' });
    }
    
    next();
  };
};

// 受保护的路由
app.get('/api/profile', requireLogin, async (req, res) => {
  try {
    const user = await User.findByPk(req.session.userId);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    
    res.json({ user });
  } catch (error) {
    console.error('Profile error:', error);
    res.status(500).json({ error: 'Failed to get profile' });
  }
});

// 管理员路由
app.get('/api/admin/users', requireRole('admin'), async (req, res) => {
  try {
    const users = await User.findAll();
    res.json({ users });
  } catch (error) {
    console.error('Admin error:', error);
    res.status(500).json({ error: 'Failed to get users' });
  }
});
```

## 会话安全最佳实践

### 1. 安全的 Cookie 配置

```javascript
app.use(session({
  cookie: {
    httpOnly: true,              // 防止 XSS 攻击
    secure: process.env.NODE_ENV === 'production', // 仅 HTTPS
    sameSite: 'strict',          // 防止 CSRF 攻击
    maxAge: 1000 * 60 * 60 * 24, // 24 小时
    path: '/',
    domain: process.env.COOKIE_DOMAIN
  },
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  rolling: true,                // 滚动过期时间
  name: 'sessionId'             // 自定义 Cookie 名称
}));
```

### 2. Session 安全增强

```javascript
const uuid = require('uuid');

// 生成强 Session ID
const generateSessionId = () => {
  return uuid.v4();
};

// Session 过期策略
app.use(session({
  secret: process.env.SESSION_SECRET,
  store: new RedisStore({
    client: redisClient,
    ttl: 3600,                   // 1 小时过期
    disableTouch: false          // 允许刷新过期时间
  }),
  cookie: {
    maxAge: 3600000,             // 1 小时
    httpOnly: true,
    secure: true,
    sameSite: 'strict'
  },
  genid: generateSessionId,      // 自定义 Session ID 生成
  resave: false,
  saveUninitialized: false
}));

// IP 绑定（可选）
app.use((req, res, next) => {
  if (req.session && req.session.userId) {
    if (!req.session.ipAddress) {
      req.session.ipAddress = req.ip;
    } else if (req.session.ipAddress !== req.ip) {
      // Session 劫持检测
      req.session.destroy();
      return res.status(401).json({ error: 'Session hijacked' });
    }
  }
  next();
});

// User-Agent 验证（可选）
app.use((req, res, next) => {
  if (req.session && req.session.userId) {
    if (!req.session.userAgent) {
      req.session.userAgent = req.headers['user-agent'];
    } else if (req.session.userAgent !== req.headers['user-agent']) {
      req.session.destroy();
      return res.status(401).json({ error: 'Session hijacked' });
    }
  }
  next();
});
```

### 3. 会话限流

```javascript
const rateLimit = require('express-rate-limit');

// 登录限流
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,      // 15 分钟
  max: 5,                        // 最多 5 次尝试
  message: 'Too many login attempts',
  skipSuccessfulRequests: true   // 成功的请求不计入限制
});

app.post('/api/login', loginLimiter, (req, res) => {
  // 登录逻辑
});

// Session 限流
const sessionLimiter = rateLimit({
  windowMs: 60 * 1000,           // 1 分钟
  max: 100,                      // 最多 100 个请求
  keyGenerator: (req) => {
    return req.session.id || 'anonymous';
  }
});

app.use('/api', sessionLimiter);
```

## 会话管理和监控

### 1. 会话统计

```javascript
// 获取活动会话数量
app.get('/api/sessions/stats', requireRole('admin'), async (req, res) => {
  try {
    const stats = {
      totalSessions: 0,
      activeUsers: new Set(),
      sessionsByRole: {}
    };
    
    // Redis 存储示例
    const sessionKeys = await redisClient.keys('sess:*');
    stats.totalSessions = sessionKeys.length;
    
    for (const key of sessionKeys) {
      const sessionData = await redisClient.get(key);
      if (sessionData) {
        const session = JSON.parse(sessionData);
        if (session.userId) {
          stats.activeUsers.add(session.userId);
        }
        if (session.role) {
          stats.sessionsByRole[session.role] = 
            (stats.sessionsByRole[session.role] || 0) + 1;
        }
      }
    }
    
    stats.activeUsers = stats.activeUsers.size;
    res.json(stats);
  } catch (error) {
    console.error('Session stats error:', error);
    res.status(500).json({ error: 'Failed to get session stats' });
  }
});
```

### 2. 会话管理接口

```javascript
// 撤销指定用户的会话
app.post('/api/admin/sessions/revoke/:userId', 
  requireRole('admin'), 
  async (req, res) => {
    try {
      const { userId } = req.params;
      
      // 删除用户的所有会话
      const sessionKeys = await redisClient.keys('sess:*');
      
      for (const key of sessionKeys) {
        const sessionData = await redisClient.get(key);
        if (sessionData) {
          const session = JSON.parse(sessionData);
          if (session.userId === parseInt(userId)) {
            await redisClient.del(key);
          }
        }
      }
      
      res.json({ success: true, message: 'Sessions revoked' });
    } catch (error) {
      console.error('Session revocation error:', error);
      res.status(500).json({ error: 'Failed to revoke sessions' });
    }
  }
);

// 获取会话信息
app.get('/api/admin/sessions/:sessionId', 
  requireRole('admin'), 
  async (req, res) => {
    try {
      const { sessionId } = req.params;
      const sessionData = await redisClient.get(`sess:${sessionId}`);
      
      if (!sessionData) {
        return res.status(404).json({ error: 'Session not found' });
      }
      
      const session = JSON.parse(sessionData);
      
      // 返回敏感信息以外的数据
      const safeSession = {
        id: sessionId,
        userId: session.userId,
        username: session.username,
        role: session.role,
        lastLogin: session.lastLogin,
        createdAt: session.cookie.expires
      };
      
      res.json({ session: safeSession });
    } catch (error) {
      console.error('Session info error:', error);
      res.status(500).json({ error: 'Failed to get session info' });
    }
  }
);
```

## 常见问题和解决方案

### 问题 1：Session 丢失

```javascript
// 问题：会话意外丢失
// 原因：Session 存储配置错误或服务器重启

// 解决方案 1：使用持久化存储
app.use(session({
  store: new RedisStore({
    client: redisClient,
    ttl: 86400
  }),
  // ...
}));

// 解决方案 2：检查 Cookie 配置
app.use(session({
  cookie: {
    maxAge: 1000 * 60 * 60 * 24, // 确保过期时间合理
    httpOnly: true,
    secure: false // 开发环境设为 false
  }
  // ...
}));
```

### 问题 2：会话劫持

```javascript
// 问题：会话被劫持
// 原因：Session ID 可预测或传输不安全

// 解决方案 1：使用强加密
app.use(session({
  secret: crypto.randomBytes(32).toString('hex'),
  genid: (req) => {
    return uuid.v4();
  }
  // ...
}));

// 解决方案 2：启用 HTTPS 和安全 Cookie
app.use(session({
  cookie: {
    secure: process.env.NODE_ENV === 'production',
    httpOnly: true,
    sameSite: 'strict'
  }
  // ...
}));
```

### 问题 3：Session 存储

```javascript
// 问题：大量 Session 占用内存
// 原因：没有清理过期 Session

// 解决方案：配置自动清理
app.use(session({
  store: new RedisStore({
    client: redisClient,
    ttl: 3600,                   // 1 小时过期
    disableTouch: true,          // 禁用刷新，强制过期
    scanCount: 1000              // 扫描数量
  })
  // ...
}));

// 定期清理 Session
const cleanupSessions = async () => {
  try {
    await sessionStore.clearExpiredSessions();
    console.log('Expired sessions cleaned');
  } catch (error) {
    console.error('Cleanup error:', error);
  }
};

// 每小时清理一次
setInterval(cleanupSessions, 3600000);
```

## 会话管理最佳实践

### 1. 环境相关配置

```javascript
const sessionConfig = {
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,
    sameSite: 'strict'
  }
};

if (process.env.NODE_ENV === 'production') {
  sessionConfig.cookie.secure = true;
  sessionConfig.cookie.domain = process.env.COOKIE_DOMAIN;
  sessionConfig.store = new RedisStore({
    client: redisClient,
    ttl: 3600
  });
} else {
  sessionConfig.cookie.secure = false;
  sessionConfig.store = new session.MemoryStore();
}

app.use(session(sessionConfig));
```

### 2. 会话生命周期管理

```javascript
// 用户活动时更新 Session
app.use((req, res, next) => {
  if (req.session && req.session.userId) {
    req.session.lastActivity = Date.now();
  }
  next();
});

// 清理不活跃 Session
const cleanInactiveSessions = async () => {
  const inactiveThreshold = Date.now() - (30 * 24 * 60 * 60 * 1000); // 30 天
  
  try {
    const sessionKeys = await redisClient.keys('sess:*');
    
    for (const key of sessionKeys) {
      const sessionData = await redisClient.get(key);
      if (sessionData) {
        const session = JSON.parse(sessionData);
        if (session.lastActivity && session.lastActivity < inactiveThreshold) {
          await redisClient.del(key);
          console.log('Cleaned inactive session:', key);
        }
      }
    }
  } catch (error) {
    console.error('Cleanup error:', error);
  }
};

// 每天清理一次
setInterval(cleanInactiveSessions, 24 * 60 * 60 * 1000);
```

### 3. 监控和告警

```javascript
// Session 监控
const monitorSessions = async () => {
  try {
    const totalSessions = await redisClient.dbsize();
    const activeUsers = await getActiveUsersCount();
    
    console.log('Session stats:', {
      totalSessions,
      activeUsers,
      timestamp: new Date().toISOString()
    });
    
    // 告警
    if (totalSessions > 10000) {
      sendAlert('High session count', { totalSessions });
    }
  } catch (error) {
    console.error('Monitoring error:', error);
  }
};

// 每 5 分钟监控一次
setInterval(monitorSessions, 5 * 60 * 1000);
```

## 总结

Express 会话管理是构建安全、用户友好的 Web 应用的关键。通过本篇详细指南，你已经了解了：

- 会话管理的重要性和基本概念
- Cookie 的使用和安全配置
- Session 的原理和实现方式
- 多种 Session 存储方案（内存、Redis、MongoDB、MySQL）
- 完整的用户认证系统实现
- 会话安全最佳实践
- 会话管理和监控方法
- 常见问题的解决方案

在实际项目中，建议：

1. 根据应用规模选择合适的 Session 存储方案
2. 始终使用安全配置（httpOnly、secure、sameSite）
3. 实现会话监控和清理机制
4. 采用多层安全防护（加密、限流、验证）
5. 定期审计会话管理安全性

掌握 Express 会话管理能够帮助你构建更加安全、可靠的用户认证系统，为用户提供更好的体验。