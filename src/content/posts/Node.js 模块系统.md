---
title: Node.js 模块系统
published: 2023-01-04
description: 'CommonJS 模块规范的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 模块系统是 Node.js 的核心特性之一，它提供了一种组织代码和封装功能的方式。Node.js 使用 CommonJS 模块规范，每个文件都被视为一个独立的模块，具有自己的作用域。

## 核心概念

理解核心概念是掌握任何技术的基础。

### 模块与作用域

每个 Node.js 模块都有独立的作用域，变量不会泄露到全局。模块之间通过 `require()` 和 `module.exports` 进行通信。

### CommonJS 规范

CommonJS 是 Node.js 采用的模块规范，主要特点：
- 使用 `require()` 导入模块
- 使用 `module.exports` 或 `exports` 导出模块
- 同步加载模块
- 运行时加载

### ES Modules

Node.js 从 v12.17.0 起原生支持 ES Modules，使用 `import/export` 语法。

## 基本用法

### 导出模块

```javascript
// utils.js - 使用 module.exports 导出
function add(a, b) {
  return a + b;
}

function multiply(a, b) {
  return a * b;
}

// 导出单个值
module.exports = add;

// 或导出多个值
module.exports = {
  add,
  multiply,
  PI: 3.14159
};

// 或使用 exports（注意：exports 是 module.exports 的引用）
exports.add = add;
exports.multiply = multiply;

// 错误示例：不要直接给 exports 赋值
// exports = { add, multiply };  // 这样导出会失败
```

```javascript
// math.js - 使用 ES Modules 语法
// 需要在 package.json 中设置 "type": "module"

export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export default function multiply(a, b) {
  return a * b;
}
```

### 导入模块

```javascript
// app.js - 使用 require 导入 CommonJS 模块
const math = require('./utils');

// 使用导出的函数
console.log(math.add(2, 3));        // 5
console.log(math.multiply(2, 3));   // 6
console.log(math.PI);               // 3.14159

// 导入整个模块
const fs = require('fs');

// 解构导入
const { readFile, writeFile } = require('fs');
```

```javascript
// app.js - 使用 import 导入 ES Modules
import math, { add, PI } from './math.js';
import { readFile } from 'fs/promises';

console.log(add(2, 3));    // 5
console.log(PI);           // 3.14159
console.log(math(2, 3));   // 6
```

### 模块分类

```javascript
// 1. 核心模块（内置模块）
const http = require('http');      // HTTP 服务器
const fs = require('fs');          // 文件系统
const path = require('path');      // 路径处理
const os = require('os');          // 操作系统信息

// 2. 文件模块（自定义模块）
const utils = require('./utils');        // 相对路径
const config = require('./config.json'); // JSON 文件

// 3. 第三方模块（npm 安装的模块）
const express = require('express');
const lodash = require('lodash');
```

### 模块查找机制

```javascript
// Node.js 按以下顺序查找模块：

// 1. 核心模块 - 优先查找
require('http');  // 立即找到核心模块

// 2. 相对路径模块（./ 或 ../）
require('./utils');  // 查找当前目录下的 utils.js、utils.json、utils.node

// 3. 绝对路径模块
require('/path/to/module');

// 4. node_modules 目录
require('express');  // 查找当前目录的 node_modules
                      // 如果没有，向上级目录查找，直到根目录
```

## 实际应用

在实际项目中，我们需要根据具体场景灵活运用。

### 模块缓存机制

```javascript
// counter.js
let count = 0;
function increment() {
  count++;
  return count;
}
module.exports = { increment };

// app.js
const counter1 = require('./counter');
const counter2 = require('./counter');  // 使用缓存，不会重新加载

console.log(counter1.increment());  // 1
console.log(counter2.increment());  // 2（共享同一个模块实例）

// 查看缓存模块
console.log(require.cache);
```

### 循环依赖处理

```javascript
// a.js
const b = require('./b');
console.log('a.js loaded', b.getA());

module.exports = {
  name: 'Module A',
  getB: () => b
};

// b.js
const a = require('./a');
console.log('b.js loaded');

module.exports = {
  name: 'Module B',
  getA: () => a
};

// 运行结果：
// b.js loaded
// a.js loaded Module B

// Node.js 使用部分加载解决循环依赖
// 当 b.js 加载时，a.exports 还未完全导出，只能获取到部分对象
```

### 动态导入

```javascript
// 使用 require() 动态导入
function loadModule(moduleName) {
  const module = require(moduleName);
  return module;
}

// 使用 import() 动态导入（ES Modules）
async function loadConfig() {
  const config = await import('./config.js');
  console.log(config.default);
}
```

### 条件导出

```javascript
// package.json 配置
{
  "name": "my-package",
  "exports": {
    ".": {
      "import": "./index.mjs",
      "require": "./index.js",
      "default": "./index.js"
    },
    "./feature": {
      "import": "./feature.mjs",
      "require": "./feature.js"
    }
  }
}

// 使用
const myPackage = require('my-package');
const feature = require('my-package/feature');
```

## 注意事项

1. 注意边界情况
   - 避免循环依赖，如果无法避免，确保导出的函数不依赖其他模块
   - 注意 exports 和 module.exports 的区别
   - 文件扩展名可以省略，但建议明确指定

2. 考虑性能影响
   - 模块会被缓存，避免在循环中重复 require
   - 使用条件加载减少不必要的模块加载
   - ES Modules 的 tree-shaking 可以减少打包体积

3. 遵循最佳实践
   - 保持模块职责单一（Single Responsibility Principle）
   - 导出时使用解构赋值提高可读性
   - 为公共 API 添加 JSDoc 注释
   - 使用 ES Modules 时在 package.json 中明确指定 "type": "module"

```javascript
// 推荐的模块导出方式
/**
 * 计算两个数的和
 * @param {number} a - 第一个数
 * @param {number} b - 第二个数
 * @returns {number} 两数之和
 */
function add(a, b) {
  return a + b;
}

/**
 * 计算两个数的乘积
 */
function multiply(a, b) {
  return a * b;
}

module.exports = {
  add,
  multiply
};
```

## 总结

通过本文的学习，相信你已经对 Node.js 模块系统有了更深入的理解。模块系统是 Node.js 组织代码的基础，合理使用模块可以提高代码的可维护性和复用性。掌握 CommonJS 和 ES Modules 两种模块规范，理解模块加载机制和缓存策略，能够帮助你更好地组织和管理 Node.js 项目。