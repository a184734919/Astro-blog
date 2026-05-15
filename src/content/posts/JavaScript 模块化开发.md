---
title: JavaScript 模块化开发
published: 2022-05-21
description: 'ES Module 与 CommonJS 对比的详细介绍和学习笔记'
image: ''
tags: ["JS","模块化"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 模块化是现代前端开发的核心概念，它允许我们将大型代码拆分成独立、可维护的模块。历史上，JavaScript 经历了从全局变量到 CommonJS、再到 ES Module 的演进过程。本文将详细介绍 JavaScript 模块化开发的两种主要方案：CommonJS 和 ES Module。

## 模块化的必要性

### 为什么需要模块化

```javascript
// 没有模块化的问题
// 所有的变量都在全局作用域
var name = '张三';
var age = 25;

// 容易发生命名冲突
var age = 30; // 覆盖了上面的 age

// 依赖关系难以管理
function displayUser() {
  console.log(name + '的年龄是' + age);
}
// displayUser 依赖于 name 和 age，但不明确
```

### 模块化的好处

1. **避免全局污染**：每个模块都有自己的作用域
2. **依赖管理**：明确声明依赖关系
3. **代码复用**：模块可以在多个地方复用
4. **可维护性**：代码按功能分离，易于维护
5. **按需加载**：只加载需要的模块，提高性能

## CommonJS

### CommonJS 概述

CommonJS 是 Node.js 采用的模块系统，在服务器端 JavaScript 开发中广泛使用。

### 导出（exports）

```javascript
// 方式1：使用 module.exports
// math.js
module.exports = {
  add: function(a, b) {
    return a + b;
  },
  subtract: function(a, b) {
    return a - b;
  }
};

// 方式2：直接导出多个内容
// utils.js
exports.getName = function() {
  return '张三';
};

exports.getAge = function() {
  return 25;
};

// 方式3：混合使用
// app.js
const version = '1.0.0';
function getVersion() {
  return version;
}

function setVersion(v) {
  version = v;
}

module.exports = {
  version,
  getVersion,
  setVersion
};
```

### 导入（require）

```javascript
// 导入整个模块
const math = require('./math');
console.log(math.add(1, 2)); // 3

// 导入特定功能（需要模块支持）
const { getName } = require('./utils');
console.log(getName()); // '张三'

// 导入 Node.js 内置模块
const fs = require('fs');
const path = require('path');

// 动态导入
function loadModule(moduleName) {
  const module = require(moduleName);
  return module;
}

const mathModule = loadModule('./math');
```

### CommonJS 的特点

```javascript
// 1. 同步加载
const fs = require('fs'); // 立即加载

// 2. 运行时加载
function lazyLoad() {
  const data = require('./data.json'); // 只在调用时加载
  return data;
}

// 3. 模块被缓存
const module1 = require('./module');
const module2 = require('./module');
console.log(module1 === module2); // true，是同一个实例

// 4. 值的拷贝
// counter.js
let count = 0;
module.exports.count = count;
module.exports.increment = function() {
  count++;
  console.log(count);
};

// main.js
const counter = require('./counter');
console.log(counter.count); // 0
counter.increment(); // 1
console.log(counter.count); // 还是 0，导出的是值的拷贝
```

## ES Module (ESM)

### ES Module 概述

ES Module 是 JavaScript 官方的模块系统，是 ES6（ES2015）引入的标准，现在被现代浏览器和 Node.js 广泛支持。

### 导出（export）

```javascript
// 方式1：命名导出
// math.js
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export const PI = 3.14159;

// 方式2：集中导出
// utils.js
const getName = () => '张三';
const getAge = () => 25;
const getCity = () => '北京';

export {
  getName,
  getAge,
  getCity
};

// 方式3：导出时重命名
export {
  getName as fetchName,
  getAge as fetchAge
};

// 方式4：默认导出
// user.js
export default class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hello, ${this.name}!`;
  }
}

// 或
export default function createUser(name, age) {
  return { name, age };
}

// 方式5：混合导出
// app.js
export const version = '1.0.0';
export function getVersion() {
  return version;
}

export default {
  name: 'My App'
};

// 方式6：重新导出
// index.js
export { add, subtract } from './math';
export { default as User } from './user';
export * from './utils';

// 导入并重新导出
export { fetchName, fetchAge } from './utils';

// 重新导出所有
export * as Utils from './utils';
```

### 导入（import）

```javascript
// 方式1：导入整个模块
import * as math from './math.js';
console.log(math.add(1, 2)); // 3

// 方式2：导入特定功能
import { add, subtract } from './math.js';
console.log(add(1, 2)); // 3

// 方式3：导入时重命名
import { add as sum, subtract as diff } from './math.js';
console.log(sum(1, 2)); // 3

// 方式4：导入默认导出
import User from './user.js';
const user = new User('张三', 25);

// 方式5：混合导入
import React, { useState, useEffect } from 'react';

// 方式6：只导入模块（用于副作用）
import './styles.css';
import './polyfill.js';

// 方式7：动态导入
async function loadModule() {
  const module = await import('./math.js');
  console.log(module.add(1, 2));
}

// 按需加载
document.getElementById('load').addEventListener('click', async () => {
  const { heavyFunction } = await import('./heavy.js');
  heavyFunction();
});
```

### ES Module 的特点

```javascript
// 1. 静态分析
// import 语句必须放在模块顶部
import { something } from './module.js'; // 正确
function load() {
  // import { something } from './module.js'; // 错误：不能在函数内使用
}

// 2. 编译时加载
// 模块依赖在编译时就确定了

// 3. 模块是单例
// module.js
export let count = 0;
export function increment() {
  count++;
}

// main.js
import { count, increment } from './module.js';
console.log(count); // 0
increment();
console.log(count); // 1

// 4. 值的引用
// 导出的是值的引用，不是拷贝
// counter.js
let count = 0;
export function getCount() {
  return count;
}

export function increment() {
  count++;
}

// main.js
import { getCount, increment } from './counter.js';
console.log(getCount()); // 0
increment();
console.log(getCount()); // 1

// 5. this 指向 undefined
// module.js
console.log(this); // undefined

// 6. 顶层代码在严格模式下执行
// module.js
var a = 1; // 不会创建全局变量
```

## CommonJS vs ES Module 对比

### 语法对比

```javascript
// CommonJS
// 导出
module.exports = {
  add: (a, b) => a + b,
  version: '1.0.0'
};

// 导入
const math = require('./math');

// ES Module
// 导出
export const add = (a, b) => a + b;
export const version = '1.0.0';

// 导入
import { add, version } from './math.js';
```

### 运行机制对比

```javascript
// CommonJS - 运行时加载
// 在代码运行时加载模块
const data = require('./data'); // 运行时执行

// ES Module - 编译时加载
// 在编译时就确定了模块依赖
import { data } from './data.js'; // 编译时分析
```

### 值的导出对比

```javascript
// CommonJS - 值的拷贝
// counter.js
let counter = 0;

function increment() {
  counter++;
  console.log(counter);
}

module.exports = {
  counter,
  increment
};

// main.js
const { counter, increment } = require('./counter');
console.log(counter); // 0
increment(); // 1
console.log(counter); // 还是 0（导出的是拷贝）

// ES Module - 值的引用
// counter.js
let counter = 0;

export function getCounter() {
  return counter;
}

export function increment() {
  counter++;
}

// main.js
import { getCounter, increment } from './counter.js';
console.log(getCounter()); // 0
increment();
console.log(getCounter()); // 1（导出的是引用）
```

### 循环依赖处理对比

```javascript
// CommonJS - 可以处理但需要小心
// a.js
const b = require('./b');
module.exports = {
  name: 'a',
  b
};

// b.js
const a = require('./a');
module.exports = {
  name: 'b',
  a: a.name // 只能访问已导出的属性
};

// main.js
const a = require('./a');
console.log(a); // { name: 'a', b: { name: 'b', a: 'a' } }

// ES Module - 更好的支持
// a.js
import { b } from './b.js';
export const name = 'a';
export const b = b;

// b.js
import { name } from './a.js';
export const name = 'b';
export const a = name; // 可以正常访问

// main.js
import { a } from './a.js';
console.log(a); // 正确的循环依赖
```

### 性能对比

```javascript
// CommonJS - 同步加载
// 适合服务器环境，但不适合浏览器
const fs = require('fs'); // 同步加载，可能阻塞
const data = fs.readFileSync('data.json');

// ES Module - 异步加载
// 适合浏览器，非阻塞
import { data } from './data.js'; // 异步加载
```

### 互操作性

```javascript
// CommonJS 导入 ES Module
// 在 Node.js 中（需要实验性支持或使用 .mjs 扩展名）
const { add } = require('./es-module.mjs');

// ES Module 导入 CommonJS
// 在 Node.js 中
import { default as math } from './commonjs.cjs';
console.log(math.add(1, 2));

// 导入默认导出
import math from './commonjs.cjs';
console.log(math.default);
```

## 实际应用

### 使用 ES Module 构建应用

```javascript
// api.js - API 模块
const API_BASE = 'https://api.example.com';

export async function fetchUser(id) {
  const response = await fetch(`${API_BASE}/users/${id}`);
  if (!response.ok) {
    throw new Error(`HTTP 错误: ${response.status}`);
  }
  return response.json();
}

export async function fetchPosts(userId) {
  const response = await fetch(`${API_BASE}/users/${userId}/posts`);
  if (!response.ok) {
    throw new Error(`HTTP 错误: ${response.status}`);
  }
  return response.json();
}

export async function createPost(userId, data) {
  const response = await fetch(`${API_BASE}/users/${userId}/posts`, {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(data)
  });
  return response.json();
}

// utils.js - 工具函数模块
export function formatDate(date) {
  const d = new Date(date);
  return d.toLocaleDateString('zh-CN');
}

export function formatNumber(num) {
  return num.toLocaleString('zh-CN');
}

export function debounce(fn, delay) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => fn.apply(this, args), delay);
  };
}

// validator.js - 验证模块
export function validateEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

export function validatePhone(phone) {
  const regex = /^1[3-9]\d{9}$/;
  return regex.test(phone);
}

export function validateRequired(value) {
  return value !== null && value !== undefined && value !== '';
}

// main.js - 主应用模块
import { fetchUser, fetchPosts, createPost } from './api.js';
import { formatDate, formatNumber } from './utils.js';
import { validateEmail, validateRequired } from './validator.js';

async function displayUserPage(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(userId);

    console.log(`用户: ${user.name}`);
    console.log(`注册时间: ${formatDate(user.createdAt)}`);
    console.log(`文章数: ${formatNumber(posts.length)}`);

    return { user, posts };
  } catch (error) {
    console.error('加载失败:', error);
    throw error;
  }
}

// 导出 API
export { displayUserPage };

// index.js - 入口文件
import { displayUserPage } from './main.js';

displayUserPage(1)
  .then(() => console.log('加载完成'))
  .catch(error => console.error('错误:', error));
```

### 使用 CommonJS 构建应用

```javascript
// database.js - 数据库模块
const Database = require('better-sqlite3');
const db = new Database('app.db');

module.exports = {
  db,
  query: (sql, params = []) => {
    const stmt = db.prepare(sql);
    return stmt.all(...params);
  },
  run: (sql, params = []) => {
    const stmt = db.prepare(sql);
    return stmt.run(...params);
  }
};

// user-service.js - 用户服务模块
const { db } = require('./database.js');

class UserService {
  getUserById(id) {
    const user = db.prepare('SELECT * FROM users WHERE id = ?').get(id);
    return user;
  }

  createUser(userData) {
    const result = db.prepare(`
      INSERT INTO users (name, email, age)
      VALUES (?, ?, ?)
    `).run(userData.name, userData.email, userData.age);

    return this.getUserById(result.lastInsertRowid);
  }

  updateUser(id, updates) {
    const fields = Object.keys(updates);
    const values = Object.values(updates);
    const setClause = fields.map(f => `${f} = ?`).join(', ');

    const result = db.prepare(`
      UPDATE users
      SET ${setClause}
      WHERE id = ?
    `).run(...values, id);

    return this.getUserById(id);
  }

  deleteUser(id) {
    const result = db.prepare('DELETE FROM users WHERE id = ?').run(id);
    return result.changes > 0;
  }
}

module.exports = new UserService();

// auth-service.js - 认证服务模块
const crypto = require('crypto');

class AuthService {
  hashPassword(password) {
    return crypto
      .createHash('sha256')
      .update(password)
      .digest('hex');
  }

  verifyPassword(password, hash) {
    const hashToVerify = this.hashPassword(password);
    return hashToVerify === hash;
  }

  generateToken(user) {
    const payload = {
      id: user.id,
      name: user.name,
      timestamp: Date.now()
    };

    return Buffer.from(JSON.stringify(payload)).toString('base64');
  }

  verifyToken(token) {
    try {
      const payload = Buffer.from(token, 'base64').toString('utf-8');
      return JSON.parse(payload);
    } catch (error) {
      return null;
    }
  }
}

module.exports = new AuthService();

// controllers/user-controller.js - 用户控制器模块
const userService = require('./user-service');
const authService = require('./auth-service');

class UserController {
  async getUser(req, res) {
    try {
      const { id } = req.params;
      const user = userService.getUserById(id);

      if (!user) {
        return res.status(404).json({ error: '用户不存在' });
      }

      res.json({ success: true, data: user });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  async createUser(req, res) {
    try {
      const userData = req.body;
      userData.password = authService.hashPassword(userData.password);

      const user = userService.createUser(userData);

      const token = authService.generateToken(user);

      res.status(201).json({
        success: true,
        data: { user, token }
      });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }

  async updateUser(req, res) {
    try {
      const { id } = req.params;
      const updates = req.body;

      const user = userService.updateUser(id, updates);

      res.json({ success: true, data: user });
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
}

module.exports = new UserController();

// app.js - 应用主文件
const express = require('express');
const userController = require('./controllers/user-controller');

const app = express();

app.use(express.json());

// 用户路由
app.get('/api/users/:id', userController.getUser.bind(userController));
app.post('/api/users', userController.createUser.bind(userController));
app.put('/api/users/:id', userController.updateUser.bind(userController));

// 导出应用
module.exports = app;
```

## 模块打包工具

### Webpack 配置

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  mode: 'development',
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: {
          loader: 'babel-loader',
          options: {
            presets: ['@babel/preset-env']
          }
        }
      }
    ]
  },
  resolve: {
    extensions: ['.js', '.json'],
    alias: {
      '@': path.resolve(__dirname, 'src'),
      '@utils': path.resolve(__dirname, 'src/utils')
    }
  }
};
```

### Vite 配置

```javascript
// vite.config.js
import { defineConfig } from 'vite';
import path from 'path';

export default defineConfig({
  resolve: {
    alias: {
      '@': path.resolve(__dirname, './src'),
      '@components': path.resolve(__dirname, './src/components'),
      '@utils': path.resolve(__dirname, './src/utils')
    }
  },
  build: {
    rollupOptions: {
      output: {
        manualChunks: {
          'vendor': ['vue', 'vue-router'],
          'utils': ['lodash', 'axios']
        }
      }
    }
  }
});
```

## 注意事项

1. **文件扩展名**：ES Module 通常需要 `.js` 或 `.mjs` 扩展名，Node.js 的 CommonJS 使用 `.cjs`。

2. **静态导入**：ES Module 的 import 语句必须在文件顶部，不能在条件语句或函数中使用。

3. **循环依赖**：两种模块系统都支持循环依赖，但 ES Module 的处理更优雅。

4. **兼容性**：ES Module 在现代浏览器中广泛支持，旧浏览器需要使用打包工具或 Polyfill。

5. **this 指向**：ES Module 的顶层 this 是 undefined，而 CommonJS 的 this 是 module.exports。

6. **动态导入**：使用 `import()` 可以动态加载模块，返回 Promise。

7. **默认导出**：CommonJS 的 module.exports 和 ES Module 的 default 导出有细微差别。

8. **性能考虑**：ES Module 的静态分析可以实现 tree-shaking，减少打包体积。

## 最佳实践

1. **使用 ES Module**：在新项目中优先使用 ES Module，它更现代化且标准化。

2. **模块化设计**：每个模块只负责一个功能，保持模块的单一职责。

3. **明确导出**：使用命名导出而不是全部导出，提高代码可读性。

4. **避免循环依赖**：设计模块结构时尽量避免循环依赖。

5. **使用别名**：配置路径别名，简化导入路径。

6. **TypeScript 集成**：使用 TypeScript 获得更好的类型安全。

7. **文档注释**：为导出的函数和类添加文档注释。

8. **测试覆盖**：为每个模块编写单元测试。

## 总结

JavaScript 模块化经历了从 CommonJS 到 ES Module 的演进：

**CommonJS**：
- Node.js 的原生模块系统
- 运行时加载，同步执行
- 值的拷贝导出
- module.exports/require 语法
- 适合服务器端开发

**ES Module**：
- JavaScript 官方标准
- 编译时加载，支持静态分析
- 值的引用导出
- export/import 语法
- 支持浏览器和 Node.js
- 支持 tree-shaking

选择模块系统时，考虑以下因素：
- 目标环境（浏览器或 Node.js）
- 是否需要静态分析
- 性能要求
- 团队技术栈

在现代前端开发中，推荐使用 ES Module，配合打包工具（如 Webpack、Vite）可以获得最佳的开发体验和性能。