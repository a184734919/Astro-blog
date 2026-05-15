---
title: Node.js 核心模块 util
published: 2023-02-03
description: '实用工具函数的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `util` 模块提供了一系列实用的工具函数，用于处理常见的编程任务。这些函数可以帮助开发者更高效地处理字符串、对象、错误等，是 Node.js 核心模块中最常用的模块之一。

## 核心概念

`util` 模块主要包含以下几类工具：
- **类型检查**：验证变量类型
- **格式化输出**：将对象转换为可读字符串
- **错误处理**：创建和处理错误对象
- **继承工具**：实现对象继承
- **调试工具**：辅助代码调试

## 基本用法

### 1. 类型检查

#### util.types

```javascript
const util = require('util');

// 检查各种类型
util.types.isDate(new Date());        // true
util.types.isMap(new Map());          // true
util.types.isSet(new Set());          // true
util.types.isRegExp(/test/);          // true
util.types.isArray([]);               // true
util.types.isBigInt(123n);            // true
util.types.isPromise(Promise.resolve()); // true
util.types.isAsyncFunction(async () => {}); // true
```

#### util.isArray() 和 util.isRegExp()（已废弃）

```javascript
// 注意：这些方法已被废弃，推荐使用 typeof 或 Array.isArray()
util.isArray([1, 2, 3]);        // true
util.isRegExp(/test/);          // true
```

### 2. 格式化输出

#### util.format()

```javascript
const util = require('util');

// 基本格式化
util.format('%s:%s', 'foo', 'bar');  // 'foo:bar'

// 格式化数字
util.format('%d', 42.5);              // '42'

// 格式化为 JSON
util.format('%j', { name: 'John' });  // '{"name":"John"}'

// 格式化百分比
util.format('%%');                    // '%'

// 混合使用
util.format('%s has %d apples', 'John', 5);  // 'John has 5 apples'
```

#### util.inspect()

```javascript
const util = require('util');

// 深度检查对象
const obj = {
  name: 'John',
  age: 30,
  skills: ['JavaScript', 'Node.js']
};

console.log(util.inspect(obj, {
  showHidden: false,      // 显示对象的不可枚举属性
  depth: null,            // 递归深度，null 表示无限
  colors: true,           // 使用颜色输出
  compact: false,         // 格式化输出
  getters: true,          // 显示 getter
  maxArrayLength: 100     // 数组最大显示长度
}));

// 简化输出
util.inspect(obj, false, 2, true);  // 简短参数形式
```

### 3. 错误处理

#### util.inspect() 用于错误

```javascript
const util = require('util');

class CustomError extends Error {
  constructor(message, code) {
    super(message);
    this.code = code;
  }
}

const error = new CustomError('Something went wrong', 'ERR_CUSTOM');
console.log(util.inspect(error, { depth: null }));
```

### 4. 继承工具

#### util.inherits()（已废弃）

```javascript
const util = require('util');
const EventEmitter = require('events');

// 创建继承关系
function MyStream() {
  EventEmitter.call(this);
}

util.inherits(MyStream, EventEmitter);

// 现在推荐使用 ES6 类语法
class MyStream extends EventEmitter {
  constructor() {
    super();
  }
}
```

### 5. 回调函数转换

#### util.promisify()

```javascript
const util = require('util');
const fs = require('fs');

// 将回调函数转换为 Promise
const readFile = util.promisify(fs.readFile);

// 使用 Promise 方式读取文件
async function main() {
  try {
    const data = await readFile('example.txt', 'utf8');
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}
```

#### util.callbackify()

```javascript
const util = require('util');

// 将 Promise 函数转换为回调函数
async function asyncFn() {
  return 'Hello World';
}

const callbackFn = util.callbackify(asyncFn);

callbackFn((error, result) => {
  if (error) {
    console.error(error);
  } else {
    console.log(result);  // 'Hello World'
  }
});
```

### 6. 调试工具

#### util.debuglog()

```javascript
const util = require('util');
const debug = util.debuglog('myapp');

// 设置环境变量 NODE_DEBUG=myapp 运行时才会输出
debug('Starting application');
debug('Processing data: %O', { id: 1, name: 'test' });
```

#### util.deprecate()

```javascript
const util = require('util');

// 标记函数为已废弃
function oldFunction() {
  console.log('This is the old function');
}

const deprecated = util.deprecate(
  oldFunction,
  'oldFunction() is deprecated. Use newFunction() instead.'
);

deprecated();  // 会输出废弃警告
```

## 实际应用

### 1. 日志记录

```javascript
const util = require('util');

class Logger {
  constructor() {
    this.debug = util.debuglog('app');
  }

  log(level, message, data) {
    const timestamp = new Date().toISOString();
    const output = {
      timestamp,
      level,
      message,
      ...(data && { data: util.inspect(data, { depth: 2 }) })
    };
    this.debug(util.format('%j', output));
  }
}

const logger = new Logger();
logger.log('info', 'Application started', { port: 3000 });
```

### 2. API 响应格式化

```javascript
const util = require('util');

function formatApiResponse(data, error = null) {
  return {
    success: !error,
    timestamp: new Date().toISOString(),
    ...(error ? { error: util.inspect(error) } : { data })
  };
}

// 使用
try {
  const result = await fetchData();
  const response = formatApiResponse(result);
} catch (error) {
  const response = formatApiResponse(null, error);
}
```

### 3. 配置验证

```javascript
const util = require('util');

function validateConfig(config) {
  const errors = [];

  if (!util.types.isObject(config)) {
    errors.push('Config must be an object');
    return { valid: false, errors };
  }

  if (!util.types.isString(config.name)) {
    errors.push('Config.name must be a string');
  }

  if (!util.types.isNumber(config.port) || config.port < 1 || config.port > 65535) {
    errors.push('Config.port must be a valid port number');
  }

  return {
    valid: errors.length === 0,
    errors
  };
}
```

### 4. 数据转换工具

```javascript
const util = require('util');

class DataConverter {
  static toCamelCase(str) {
    return str.replace(/-([a-z])/g, (_, letter) => letter.toUpperCase());
  }

  static deepInspect(obj) {
    return util.inspect(obj, {
      depth: null,
      colors: false,
      maxArrayLength: Infinity
    });
  }

  static formatSize(bytes) {
    const units = ['B', 'KB', 'MB', 'GB'];
    let size = bytes;
    let unitIndex = 0;

    while (size >= 1024 && unitIndex < units.length - 1) {
      size /= 1024;
      unitIndex++;
    }

    return util.format('%.2f %s', size, units[unitIndex]);
  }
}
```

## 注意事项

1. **废弃方法**：注意 `util.isArray()`、`util.isRegExp()`、`util.inherits()` 等方法已被废弃，应使用原生 JS 语法替代
2. **性能考虑**：`util.inspect()` 对大型对象可能影响性能，在生产环境中应谨慎使用
3. **深度限制**：默认情况下 `util.inspect()` 深度为 2，嵌套深的对象可能无法完整显示
4. **循环引用**：`util.inspect()` 能正确处理循环引用，但自定义实现需要注意
5. **调试日志**：`util.debuglog()` 需要设置 `NODE_DEBUG` 环境变量才会输出

## 最佳实践

1. **类型检查**：优先使用 `util.types` 而非 `typeof`，因为前者更精确
2. **Promise 转换**：对于异步操作，统一使用 `util.promisify()` 转换为 Promise
3. **调试信息**：在开发环境使用 `util.inspect()` 的详细选项，生产环境使用简化版本
4. **错误处理**：使用 `util.inspect()` 捕获完整错误信息，便于问题排查
5. **兼容性**：对于需要支持旧版本 Node.js 的项目，谨慎使用新的 API

## 总结

`util` 模块是 Node.js 中非常实用的工具集合，它提供了类型检查、格式化、错误处理、Promise 转换等功能。合理使用这些工具可以让代码更简洁、更易维护，同时提高开发效率。在实际项目中，可以根据具体需求选择合适的工具函数，并注意性能和兼容性问题。