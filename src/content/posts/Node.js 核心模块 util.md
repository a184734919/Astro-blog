---
title: Node.js 核心模块 util
published: 2023-02-03
description: '实用工具函数的详细介绍和学习笔记'
image: ''
tags: ["Node.js","核心模块","工具函数"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `util` 模块提供了一系列实用的工具函数，用于处理常见的编程任务。这些函数可以帮助开发者更高效地处理字符串、对象、错误、调试等，是 Node.js 核心模块中最常用、最实用的模块之一。`util` 模块的功能涵盖了从简单的格式化输出到复杂的异步流程控制，是每个 Node.js 开发者都应该掌握的重要模块。

## 核心概念

### 类型检查工具
`util.types` 提供了一套精确的类型检查方法，比原生的 `typeof` 操作符更加准确和详细，可以检查各种 JavaScript 和 Node.js 特定的类型。

### 格式化输出
提供了多种格式化输出的方法，包括 `util.format()` 用于字符串格式化，`util.inspect()` 用于深度对象检查和调试输出。

### 异步流程控制
通过 `util.promisify()` 和 `util.callbackify()` 可以在 Promise 和回调函数之间相互转换，方便异步编程。

### 调试和日志
`util.debuglog()` 提供了条件调试日志功能，只在特定调试模式下输出，帮助开发者进行调试。

### 废弃警告
`util.deprecate()` 用于标记废弃的 API，提供友好的废弃警告，帮助维护代码库的向后兼容性。

### 继承工具
虽然现在推荐使用 ES6 的 class 语法，但 `util.inherits()` 仍然提供了一种传统的对象继承方式。

## 基本用法

### 1. 类型检查 (util.types)

#### 基本类型检查

```javascript
const util = require('util');
const { types } = util;

// 检查原始类型
console.log(types.isAnyArrayBuffer(new ArrayBuffer(8)));      // true
console.log(types.isArrayBuffer(new ArrayBuffer(8)));         // true
console.log(types.isBigUint64Array(new BigUint64Array()));    // true
console.log(types.isBigInt64Array(new BigInt64Array()));      // true

// 数组类型
console.log(types.isArray([1, 2, 3]));                       // true
console.log(types.isArray(new Array()));                      // true

// 异步函数
console.log(types.isAsyncFunction(async () => {}));           // true

// Buffer
console.log(types.isBuffer(Buffer.from('hello')));            // true

// 数据视图
console.log(types.isDataView(new DataView(new ArrayBuffer()))); // true

// 日期对象
console.log(types.isDate(new Date()));                        // true

// Map 和 WeakMap
console.log(types.isMap(new Map()));                          // true
console.log(types.isWeakMap(new WeakMap()));                  // true

// Set 和 WeakSet
console.log(types.isSet(new Set()));                          // true
console.log(types.isWeakSet(new WeakSet()));                  // true

// Promise
console.log(types.isPromise(Promise.resolve()));              // true

// 正则表达式
console.log(types.isRegExp(/test/));                          // true

// 错误对象
console.log(types.isError(new Error('test')));                // true
console.log(types.isError(new TypeError('test')));            // true

// 函数类型
console.log(types.isFunction(() => {}));                      // true
console.log(types.isGeneratorFunction(function*() {}));       // true
console.log(types.isAsyncGeneratorFunction(async function*() {})); // true

// Symbol
console.log(types.isSymbol(Symbol('test')));                  // true

// NaN
console.log(types.isNaN(NaN));                                // true
console.log(types.isNaN(Number('abc')));                     // true

// 原生对象
console.log(types.isNativeError(new Error()));                // true
console.log(types.isNumberObject(new Number(42)));            // true
console.log(types.isStringObject(new String('hello')));      // true
console.log(types.isBooleanObject(new Boolean(true)));        // true
```

#### 复杂类型检查

```javascript
const util = require('util');
const { types } = util;

// 检查 Boxed Primitives
function checkBoxedTypes() {
  console.log('Boxed Primitives:');
  console.log('  Number Object:', types.isNumberObject(new Number(42)));
  console.log('  String Object:', types.isStringObject(new String('hello')));
  console.log('  Boolean Object:', types.isBooleanObject(new Boolean(true)));
  console.log('  BigInt Object:', types.isBigIntObject(Object(BigInt(42n))));
}

// 检查 Shared Memory
function checkSharedMemory() {
  console.log('Shared Memory:');
  console.log('  SharedArrayBuffer:', types.isSharedArrayBuffer(new SharedArrayBuffer(8)));
}

// 检查 Proxy
function checkProxy() {
  console.log('Proxy:');
  const target = {};
  const proxy = new Proxy(target, {});
  console.log('  Proxy:', types.isProxy(proxy));
  console.log('  Regular Object:', types.isProxy(target));
}

// 检查编码相关的类型
function checkEncodingTypes() {
  console.log('Encoding Types:');
  console.log('  TextEncoder:', types.isTextEncoder(new TextEncoder()));
  console.log('  TextDecoder:', types.isTextDecoder(new TextDecoder()));
}

// 检查 Crypto 相关类型
function checkCryptoTypes() {
  const crypto = require('crypto');
  console.log('Crypto Types:');
  console.log('  KeyObject:', types.isKeyObject(crypto.createPublicKey('-----BEGIN PUBLIC KEY-----...')));
  console.log('  CryptoKey:', types.isCryptoKey(null)); // Web Crypto API
}

// 检查 Stream 相关类型
function checkStreamTypes() {
  const stream = require('stream');
  console.log('Stream Types:');
  console.log('  ReadableStream:', types.isReadableStream(new stream.Readable()));
  console.log('  WritableStream:', types.isWritableStream(new stream.Writable()));
  console.log('  DuplexStream:', types.isDuplexStream(new stream.Duplex()));
  console.log('  TransformStream:', types.isTransformStream(new stream.Transform()));
}

// 执行所有检查
checkBoxedTypes();
checkSharedMemory();
checkProxy();
checkEncodingTypes();
checkCryptoTypes();
checkStreamTypes();
```

#### 自定义类型检查工具

```javascript
const util = require('util');
const { types } = util;

class TypeChecker {
  // 检查是否为数字（包括字符串数字）
  static isNumeric(value) {
    return types.isNumber(value) || (types.isString(value) && !isNaN(value));
  }

  // 检查是否为布尔值（包括字符串布尔值）
  static isBoolean(value) {
    return types.isBoolean(value) ||
           (types.isString(value) && ['true', 'false'].includes(value.toLowerCase()));
  }

  // 检查是否为普通对象（非数组、函数等）
  static isPlainObject(value) {
    if (!types.isObject(value) || types.isArray(value)) {
      return false;
    }
    return Object.prototype.toString.call(value) === '[object Object]';
  }

  // 检查是否为可迭代对象
  static isIterable(value) {
    return types.isObject(value) && typeof value[Symbol.iterator] === 'function';
  }

  // 检查是否为 Promise-like 对象
  static isPromiseLike(value) {
    return types.isObject(value) && typeof value.then === 'function';
  }

  // 深度类型检查
  static deepCheck(value) {
    const checks = {
      'Null': value === null,
      'Undefined': value === undefined,
      'Boolean': types.isBoolean(value),
      'Number': types.isNumber(value),
      'String': types.isString(value),
      'Symbol': types.isSymbol(value),
      'BigInt': types.isBigInt(value),
      'Array': types.isArray(value),
      'Object': types.isObject(value) && !types.isArray(value),
      'Function': types.isFunction(value),
      'Promise': types.isPromise(value),
      'Date': types.isDate(value),
      'RegExp': types.isRegExp(value),
      'Error': types.isError(value),
      'Map': types.isMap(value),
      'Set': types.isSet(value),
      'Buffer': types.isBuffer(value)
    };

    const detectedType = Object.keys(checks).find(key => checks[key]);
    return detectedType || 'Unknown';
  }
}

// 使用示例
console.log('isNumeric("123"):', TypeChecker.isNumeric("123")); // true
console.log('isNumeric("abc"):', TypeChecker.isNumeric("abc")); // false
console.log('isBoolean("true"):', TypeChecker.isBoolean("true")); // true
console.log('isPlainObject({}):', TypeChecker.isPlainObject({})); // true
console.log('isPlainObject([]):', TypeChecker.isPlainObject([])); // false
console.log('deepCheck(42):', TypeChecker.deepCheck(42)); // 'Number'
console.log('deepCheck([1,2,3]):', TypeChecker.deepCheck([1, 2, 3])); // 'Array'
console.log('deepCheck(new Date()):', TypeChecker.deepCheck(new Date())); // 'Date'
```

### 2. 格式化输出

#### util.format() 基础用法

```javascript
const util = require('util');

// 基本占位符
console.log(util.format('%s', 'Hello'));              // 'Hello'
console.log(util.format('%d', 42));                   // '42'
console.log(util.format('%i', 42.5));                  // '42' (整数)
console.log(util.format('%f', 42.5));                 // '42.5'
console.log(util.format('%%'));                        // '%'

// JSON 格式化
console.log(util.format('%j', { name: 'John' }));     // '{"name":"John"}'
console.log(util.format('%j', ['a', 'b', 'c']));      // '["a","b","c"]'

// 对象显示
console.log(util.format('%o', { a: 1, b: 2 }));        // '{ a: 1, b: 2 }'
console.log(util.format('%O', { a: 1, b: 2 }));        // '{ a: 1, b: 2 }'

// 复杂格式化
console.log(util.format('%s is %d years old', 'Alice', 30));
// 'Alice is 30 years old'

console.log(util.format('Hello %s, you have %d %s', 'Bob', 5, 'apples'));
// 'Hello Bob, you have 5 apples'

// 多个参数
console.log(util.format('%s %s %s', 'a', 'b', 'c')); // 'a b c'

// 不够的参数
console.log(util.format('%s %s', 'a'));             // 'a %s'

// 额外的参数
console.log(util.format('%s', 'a', 'b', 'c'));      // 'a b c'

// 错误处理
console.log(util.format('%s', new Error('test')));  // 'Error: test'

// 自定义对象
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  [util.inspect.custom]() {
    return `Person: ${this.name}, ${this.age}`;
  }
}

const person = new Person('Alice', 30);
console.log(util.format('%o', person));               // 'Person: Alice, 30'
```

#### util.format() 高级用法

```javascript
const util = require('util');

class AdvancedFormatter {
  // 格式化时间
  static formatTime(date) {
    return util.format('%s', date.toISOString());
  }

  // 格式化文件大小
  static formatFileSize(bytes) {
    const units = ['B', 'KB', 'MB', 'GB'];
    let size = bytes;
    let unitIndex = 0;

    while (size >= 1024 && unitIndex < units.length - 1) {
      size /= 1024;
      unitIndex++;
    }

    return util.format('%.2f %s', size, units[unitIndex]);
  }

  // 格式化进度
  static formatProgress(current, total) {
    const percent = (current / total * 100).toFixed(2);
    return util.format('%.2f%% (%d/%d)', percent, current, total);
  }

  // 格式化表格数据
  static formatTable(data) {
    if (!Array.isArray(data) || data.length === 0) {
      return 'No data';
    }

    const headers = Object.keys(data[0]);
    const headerRow = headers.map(h => util.format('%-15s', h)).join('|');
    const separatorRow = headers.map(() => '-'.repeat(15)).join('+');

    const dataRows = data.map(row => {
      return headers.map(h => util.format('%-15s', row[h])).join('|');
    });

    return [headerRow, separatorRow, ...dataRows].join('\n');
  }

  // 格式化错误信息
  static formatError(error, includeStack = true) {
    const errorInfo = {
      message: error.message,
      name: error.name,
      code: error.code,
      stack: includeStack ? error.stack : undefined
    };

    return util.format('Error: %j', errorInfo);
  }

  // 格式化配置对象
  static formatConfig(config, depth = 2) {
    return util.format('%O', config);
  }
}

// 使用示例
console.log('时间格式化:', AdvancedFormatter.formatTime(new Date()));
console.log('文件大小格式化:', AdvancedFormatter.formatFileSize(1024 * 1024 * 5.7));
console.log('进度格式化:', AdvancedFormatter.formatProgress(750, 1000));

const tableData = [
  { name: 'Alice', age: 30, city: 'New York' },
  { name: 'Bob', age: 25, city: 'London' },
  { name: 'Charlie', age: 35, city: 'Paris' }
];
console.log('表格格式化:\n', AdvancedFormatter.formatTable(tableData));

try {
  JSON.parse('{invalid json}');
} catch (error) {
  console.log('错误格式化:', AdvancedFormatter.formatError(error));
}

const config = {
  server: {
    host: 'localhost',
    port: 3000
  },
  database: {
    url: 'mongodb://localhost:27017/myapp'
  }
};
console.log('配置格式化:', AdvancedFormatter.formatConfig(config));
```

#### util.inspect() 深度对象检查

```javascript
const util = require('util');

// 基本使用
const obj = {
  name: 'Alice',
  age: 30,
  skills: ['JavaScript', 'Node.js', 'React'],
  address: {
    city: 'New York',
    zip: '10001'
  },
  greet: function() {
    return `Hello, I'm ${this.name}`;
  }
};

console.log('基本检查:');
console.log(util.inspect(obj));

// 带选项的检查
console.log('带选项的检查:');
console.log(util.inspect(obj, {
  showHidden: false,          // 显示隐藏属性
  depth: 2,                   // 检查深度
  colors: true,               // 使用颜色
  customInspect: true,        // 自定义检查
  showProxy: true,            // 显示代理对象
  maxArrayLength: 100,        // 数组最大显示长度
  maxStringLength: 10000,     // 字符串最大显示长度
  breakLength: 80,            // 断行长度
  compact: true,              // 紧凑格式
  sorted: false,              // 排序属性
  getters: true,              // 显示 getter
  numericSeparator: false    // 数字分隔符
}));

// 循环引用处理
const circularObj = { a: 1 };
circularObj.self = circularObj;

console.log('循环引用:');
console.log(util.inspect(circularObj, { depth: null }));

// 自定义检查
class CustomObject {
  constructor(data) {
    this.data = data;
  }

  [util.inspect.custom](depth, options) {
    return `CustomObject { data: ${util.inspect(this.data, options)} }`;
  }
}

const customObj = new CustomObject({ key: 'value' });
console.log('自定义检查:', util.inspect(customObj));

// 检查函数
function complexFunction(a, b, c) {
  return a + b + c;
}

console.log('函数检查:', util.inspect(complexFunction));

// 检查 Map 和 Set
const myMap = new Map([
  ['key1', 'value1'],
  ['key2', 'value2']
]);

const mySet = new Set([1, 2, 3, 3, 4]);

console.log('Map 检查:', util.inspect(myMap));
console.log('Set 检查:', util.inspect(mySet));
```

#### util.inspect() 自定义和高级选项

```javascript
const util = require('util');

class AdvancedInspector {
  // 检查并美化输出
  static prettyPrint(obj, options = {}) {
    const defaultOptions = {
      depth: null,
      colors: false,
      compact: false,
      breakLength: 80,
      ...options
    };

    return util.inspect(obj, defaultOptions);
  }

  // 检查对象结构
  static inspectStructure(obj) {
    const structure = {
      type: Object.prototype.toString.call(obj),
      keys: Object.keys(obj),
      length: obj.length || Object.keys(obj).length,
      hasMethods: Object.keys(obj).some(key => typeof obj[key] === 'function'),
      hasProperties: Object.keys(obj).some(key => typeof obj[key] !== 'function')
    };

    return util.inspect(structure);
  }

  // 比较两个对象的差异
  static compareObjects(obj1, obj2) {
    const keys1 = Object.keys(obj1);
    const keys2 = Object.keys(obj2);
    const allKeys = new Set([...keys1, ...keys2]);

    const differences = [];

    allKeys.forEach(key => {
      if (!keys1.includes(key)) {
        differences.push({ key, type: 'only in obj2', value: obj2[key] });
      } else if (!keys2.includes(key)) {
        differences.push({ key, type: 'only in obj1', value: obj1[key] });
      } else if (obj1[key] !== obj2[key]) {
        differences.push({
          key,
          type: 'different',
          value1: obj1[key],
          value2: obj2[key]
        });
      }
    });

    return differences.length > 0 ? differences : 'No differences found';
  }

  // 检查对象深度
  static getObjectDepth(obj) {
    if (typeof obj !== 'object' || obj === null) {
      return 0;
    }

    let maxDepth = 1;
    for (const value of Object.values(obj)) {
      if (typeof value === 'object' && value !== null) {
        const depth = this.getObjectDepth(value) + 1;
        if (depth > maxDepth) {
          maxDepth = depth;
        }
      }
    }

    return maxDepth;
  }

  // 创建自定义检查器
  static createCustomInspector(options) {
    return {
      inspect: (obj) => {
        return util.inspect(obj, options);
      }
    };
  }
}

// 使用示例
const complexObject = {
  name: 'Test',
  data: {
    nested: {
      deeper: {
        value: 'deep value'
      }
    },
    array: [1, 2, 3, { nested: 'item' }]
  }
};

console.log('美化输出:');
console.log(AdvancedInspector.prettyPrint(complexObject));

console.log('对象结构:');
console.log(AdvancedInspector.inspectStructure(complexObject));

const obj1 = { a: 1, b: 2, c: 3 };
const obj2 = { a: 1, b: 3, d: 4 };
console.log('对象差异:');
console.log(AdvancedInspector.compareObjects(obj1, obj2));

console.log('对象深度:', AdvancedInspector.getObjectDepth(complexObject));

const customInspector = AdvancedInspector.createCustomInspector({
  depth: 3,
  colors: true,
  compact: true
});
console.log('自定义检查:', customInspector.inspect(complexObject));
```

### 3. 异步流程控制

#### util.promisify() 基本用法

```javascript
const util = require('util');
const fs = require('fs');

// 基本转换
const readFile = util.promisify(fs.readFile);

async function basicExample() {
  try {
    const data = await readFile('example.txt', 'utf8');
    console.log('文件内容:', data);
  } catch (error) {
    console.error('读取文件失败:', error);
  }
}

// 转换多个函数
const writeFile = util.promisify(fs.writeFile);
const stat = util.promisify(fs.stat);

async function multipleExample() {
  try {
    // 写入文件
    await writeFile('test.txt', 'Hello, World!');

    // 读取文件
    const content = await readFile('test.txt', 'utf8');
    console.log('文件内容:', content);

    // 获取文件信息
    const stats = await stat('test.txt');
    console.log('文件信息:', stats);
  } catch (error) {
    console.error('操作失败:', error);
  }
}

// 错误处理
async function errorHandling() {
  const readFile = util.promisify(fs.readFile);

  try {
    const data = await readFile('nonexistent.txt', 'utf8');
    console.log(data);
  } catch (error) {
    // Promise 转换后的错误
    console.log('错误码:', error.code);
    console.log('错误信息:', error.message);
    console.log('错误路径:', error.path);
    console.log('错误堆栈:', error.stack);
  }
}

// 并行执行
async function parallelExample() {
  const readFile = util.promisify(fs.readFile);

  try {
    // 并行读取多个文件
    const [file1, file2, file3] = await Promise.all([
      readFile('file1.txt', 'utf8'),
      readFile('file2.txt', 'utf8'),
      readFile('file3.txt', 'utf8')
    ]);

    console.log('文件1:', file1);
    console.log('文件2:', file2);
    console.log('文件3:', file3);
  } catch (error) {
    console.error('并行读取失败:', error);
  }
}

// 顺序执行
async function sequentialExample() {
  const readFile = util.promisify(fs.readFile);

  try {
    // 顺序读取文件
    for (const filename of ['file1.txt', 'file2.txt', 'file3.txt']) {
      const content = await readFile(filename, 'utf8');
      console.log(`${filename}:`, content);
    }
  } catch (error) {
    console.error('顺序读取失败:', error);
  }
}
```

#### util.promisify() 高级用法

```javascript
const util = require('util');

class AsyncTools {
  // 批量转换回调函数为 Promise
  static promisifyAll(target) {
    const result = {};

    for (const key of Object.keys(target)) {
      const value = target[key];
      if (typeof value === 'function' && !key.startsWith('_')) {
        result[key] = util.promisify(value);
      } else {
        result[key] = value;
      }
    }

    return result;
  }

  // 带 retry 的 Promise 包装
  static async withRetry(fn, maxRetries = 3, delay = 1000) {
    let lastError;

    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;
        if (i < maxRetries - 1) {
          await new Promise(resolve => setTimeout(resolve, delay));
          delay *= 2; // 指数退避
        }
      }
    }

    throw lastError;
  }

  // 带超时的 Promise 包装
  static async withTimeout(promise, timeout, timeoutError = new Error('Operation timed out')) {
    let timeoutHandle;
    const timeoutPromise = new Promise((_, reject) => {
      timeoutHandle = setTimeout(() => reject(timeoutError), timeout);
    });

    try {
      return await Promise.race([promise, timeoutPromise]);
    } finally {
      clearTimeout(timeoutHandle);
    }
  }

  // 缓存 Promise 结果
  static cachedPromise(fn, cacheKey = 'default') {
    const cache = new Map();

    return async (...args) => {
      const key = cacheKey + JSON.stringify(args);

      if (cache.has(key)) {
        return cache.get(key);
      }

      const result = await fn(...args);
      cache.set(key, result);
      return result;
    };
  }

  // 带进度报告的 Promise
  static withProgress(fn, progressCallback) {
    return new Promise(async (resolve, reject) => {
      try {
        let lastProgress = 0;

        const wrappedFn = async (...args) => {
          const result = await fn(...args);

          if (result.progress !== undefined && result.progress !== lastProgress) {
            lastProgress = result.progress;
            if (progressCallback) {
              progressCallback(result.progress, result);
            }
          }

          return result;
        };

        const result = await wrappedFn();
        resolve(result);
      } catch (error) {
        reject(error);
      }
    });
  }

  // 批量处理
  static async batch(items, processor, batchSize = 10) {
    const results = [];

    for (let i = 0; i < items.length; i += batchSize) {
      const batch = items.slice(i, i + batchSize);
      const batchResults = await Promise.all(batch.map(item => processor(item)));
      results.push(...batchResults);
    }

    return results;
  }
}

// 使用示例

// 批量转换
const fs = require('fs');
const fsPromises = AsyncTools.promisifyAll(fs);

// 使用转换后的函数
async function usePromisifiedFS() {
  try {
    await fsPromises.writeFile('test.txt', 'Hello');
    const content = await fsPromises.readFile('test.txt', 'utf8');
    console.log('文件内容:', content);
  } catch (error) {
    console.error('操作失败:', error);
  }
}

// 重试机制
async function retryExample() {
  const randomOperation = async () => {
    if (Math.random() > 0.7) {
      throw new Error('Random failure');
    }
    return 'Success!';
  };

  try {
    const result = await AsyncTools.withRetry(randomOperation, 5, 1000);
    console.log('重试结果:', result);
  } catch (error) {
    console.error('重试失败:', error.message);
  }
}

// 超时控制
async function timeoutExample() {
  const slowOperation = async () => {
    await new Promise(resolve => setTimeout(resolve, 5000));
    return 'Done!';
  };

  try {
    const result = await AsyncTools.withTimeout(slowOperation(), 2000);
    console.log('结果:', result);
  } catch (error) {
    console.error('超时错误:', error.message);
  }
}

// 批量处理
async function batchExample() {
  const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
  const processor = async (item) => {
    console.log('处理:', item);
    await new Promise(resolve => setTimeout(resolve, 100));
    return item * 2;
  };

  const results = await AsyncTools.batch(items, processor, 3);
  console.log('批量处理结果:', results);
}
```

#### util.callbackify() 使用

```javascript
const util = require('util');

// 将 Promise 函数转换为回调函数
async function asyncFunction() {
  return 'Hello from async function';
}

const callbackFunction = util.callbackify(asyncFunction);

// 使用回调函数
callbackFunction((error, result) => {
  if (error) {
    console.error('错误:', error);
  } else {
    console.log('结果:', result);
  }
});

// 带参数的转换
async function asyncAdd(a, b) {
  return a + b;
}

const callbackAdd = util.callbackify(asyncAdd);

callbackAdd(5, 3, (error, result) => {
  if (error) {
    console.error('错误:', error);
  } else {
    console.log('5 + 3 =', result);
  }
});

// 错误处理转换
async function asyncError() {
  throw new Error('Something went wrong');
}

const callbackError = util.callbackify(asyncError);

callbackError((error, result) => {
  if (error) {
    console.error('捕获到错误:', error.message);
  } else {
    console.log('结果:', result);
  }
});
```

### 4. 调试和日志

#### util.debuglog() 基本用法

```javascript
const util = require('util');

// 创建调试日志
const debug = util.debuglog('myapp');

// 只在设置了 NODE_DEBUG=myapp 环境变量时才会输出
debug('这是一条调试信息');
debug('用户数据: %O', { name: 'Alice', age: 30 });

// 不同模块的调试日志
const dbDebug = util.debuglog('database');
const apiDebug = util.debuglog('api');

dbDebug('数据库连接成功');
apiDebug('API 请求: GET /users');

// 格式化调试信息
debug('处理用户请求: %s', 'GET /api/users');
debug('性能统计: %d 请求/秒', 150);
debug('系统配置: %j', { timeout: 5000, retries: 3 });

// 条件调试
function conditionalDebug(condition) {
  if (condition) {
    debug('条件满足，执行调试逻辑');
  }
}

conditionalDebug(true);

// 错误调试
function debugError(error) {
  debug('发生错误:', error.message);
  debug('错误堆栈:', error.stack);
}
```

#### util.debuglog() 高级应用

```javascript
const util = require('util');

class DebugLogger {
  constructor(module) {
    this.debug = util.debuglog(module);
    this.module = module;
  }

  // 不同级别的日志
  debug(message, ...args) {
    this.debug(message, ...args);
  }

  info(message, ...args) {
    this.debug(`[INFO] ${message}`, ...args);
  }

  warn(message, ...args) {
    this.debug(`[WARN] ${message}`, ...args);
  }

  error(message, ...args) {
    this.debug(`[ERROR] ${message}`, ...args);
  }

  // 性能监控
  measureTime(label, fn) {
    const start = Date.now();
    this.debug(`[START] ${label}`);

    return fn().finally(() => {
      const duration = Date.now() - start;
      this.debug(`[END] ${label} - 耗时: ${duration}ms`);
    });
  }

  // 追踪函数调用
  traceFunction(fn) {
    return (...args) => {
      this.debug(`[TRACE] 调用函数: ${fn.name}`, args);
      const result = fn(...args);
      this.debug(`[TRACE] 函数返回:`, result);
      return result;
    };
  }

  // 条件调试
  conditionalDebug(condition, message, ...args) {
    if (condition) {
      this.debug(message, ...args);
    }
  }
}

// 使用示例
const logger = new DebugLogger('app');

// 基本使用
logger.debug('应用程序启动');
logger.info('用户登录成功', { userId: 123, username: 'Alice' });
logger.warn('内存使用率较高', { usage: '85%' });
logger.error('数据库连接失败', new Error('Connection refused'));

// 性能监控
async function measureExample() {
  return logger.measureTime('数据处理', async () => {
    await new Promise(resolve => setTimeout(resolve, 1000));
    return '处理完成';
  });
}

measureExample();

// 函数追踪
function add(a, b) {
  return a + b;
}

const tracedAdd = logger.traceFunction(add);
console.log(tracedAdd(5, 3));

// 条件调试
logger.conditionalDebug(process.env.NODE_ENV === 'development', '开发环境特定逻辑');
```

#### util.deprecate() 废弃警告

```javascript
const util = require('util');

// 基本使用
function oldFunction() {
  console.log('这是旧函数');
}

const deprecatedFunction = util.deprecate(
  oldFunction,
  'oldFunction() is deprecated. Use newFunction() instead.'
);

console.log('调用废弃函数:');
deprecatedFunction();

// 类方法废弃
class OldClass {
  constructor() {
    this.value = 0;
  }

  oldMethod() {
    this.value++;
    return this.value;
  }
}

OldClass.prototype.oldMethod = util.deprecate(
  OldClass.prototype.oldMethod,
  'OldClass.oldMethod() is deprecated. Use newMethod() instead.'
);

const oldInstance = new OldClass();
oldInstance.oldMethod();

// 条件废弃
function experimentalFunction() {
  console.log('实验性功能');
}

const conditionallyDeprecated = util.deprecate(
  experimentalFunction,
  'experimentalFunction() is experimental and may change.',
  'DEP_EXPERIMENTAL'
);

conditionallyDeprecated();

// 自定义废弃处理
class DeprecationManager {
  constructor() {
    this.deprecations = new Map();
    this.callbacks = [];
  }

  deprecate(fn, message, code) {
    const wrapped = util.deprecate(fn, message, code);
    const key = `${fn.name || 'anonymous'}_${code}`;

    return (...args) => {
      this.recordDeprecation(key, message, code);
      return wrapped(...args);
    };
  }

  recordDeprecation(key, message, code) {
    if (!this.deprecations.has(key)) {
      this.deprecations.set(key, {
        message,
        code,
        count: 0,
        firstSeen: Date.now()
      });
    }

    const info = this.deprecations.get(key);
    info.count++;

    // 通知回调
    this.callbacks.forEach(callback => {
      callback(info);
    });
  }

  onDeprecation(callback) {
    this.callbacks.push(callback);
  }

  getDeprecationReport() {
    return Array.from(this.deprecations.values());
  }
}

// 使用自定义废弃管理器
const deprecationManager = new DeprecationManager();

deprecationManager.onDeprecation((info) => {
  console.warn(`[DEPRECATION] ${info.code}: ${info.message} (已调用 ${info.count} 次)`);
});

function deprecatedWithManager() {
  console.log('需要更新的函数');
}

const managedDeprecated = deprecationManager.deprecate(
  deprecatedWithManager,
  'This function will be removed in next version.',
  'DEP001'
);

managedDeprecated();
managedDeprecated();

console.log('废弃报告:', deprecationManager.getDeprecationReport());
```

### 5. 其他实用工具

#### 继承工具 util.inherits()

```javascript
const util = require('util');
const EventEmitter = require('events');

// 传统继承方式
function MyEmitter() {
  EventEmitter.call(this);
}

util.inherits(MyEmitter, EventEmitter);

MyEmitter.prototype.greet = function(name) {
  this.emit('greet', `Hello, ${name}!`);
};

// 现代推荐方式
class ModernEmitter extends EventEmitter {
  greet(name) {
    this.emit('greet', `Hello, ${name}!`);
  }
}

// 使用示例
const oldEmitter = new MyEmitter();
oldEmitter.on('greet', (message) => {
  console.log('收到问候:', message);
});

oldEmitter.greet('Alice');

const modernEmitter = new ModernEmitter();
modernEmitter.on('greet', (message) => {
  console.log('收到问候:', message);
});

modernEmitter.greet('Bob');
```

#### util.toUSVString()

```javascript
const util = require('util');

// 处理包含替代对的字符串
const stringWithSurrogates = 'Hello \uD83D\uDE00 World!'; // 包含emoji

console.log('原始字符串:', stringWithSurrogates);
console.log('USV 字符串:', util.toUSVString(stringWithSurrogates));

// 处理无效的 UTF-16 序列
const invalidUTF16 = 'Hello \uD800 World!'; // 无效的代理对

console.log('无效 UTF-16:', invalidUTF16);
console.log('USV 字符串:', util.toUSVString(invalidUTF16));

// 在错误处理中的应用
class ValidationError extends Error {
  constructor(message) {
    super(util.toUSVString(message));
    this.name = 'ValidationError';
  }
}

const error = new ValidationError('Invalid input: \uD800');
console.log('错误消息:', error.message);
```

#### util.parseEnv()

```javascript
const util = require('util');

// 环境变量解析（Node.js 16+）
const envString = `
NODE_ENV=development
PORT=3000
DATABASE_URL=mongodb://localhost:27017/myapp
DEBUG=*
`;

// 注意：这个函数在较新的 Node.js 版本中可用
if (util.parseEnv) {
  const env = util.parseEnv(envString);
  console.log('解析的环境变量:', env);
}

// 自定义环境变量解析器
class EnvParser {
  static parse(envString) {
    const lines = envString.split('\n');
    const result = {};

    for (const line of lines) {
      // 跳过注释和空行
      const trimmedLine = line.trim();
      if (!trimmedLine || trimmedLine.startsWith('#')) {
        continue;
      }

      // 解析键值对
      const equalIndex = trimmedLine.indexOf('=');
      if (equalIndex === -1) continue;

      const key = trimmedLine.substring(0, equalIndex).trim();
      const value = trimmedLine.substring(equalIndex + 1).trim();

      // 处理引号
      if ((value.startsWith('"') && value.endsWith('"')) ||
          (value.startsWith("'") && value.endsWith("'"))) {
        result[key] = value.slice(1, -1);
      } else {
        result[key] = value;
      }
    }

    return result;
  }

  static loadEnvFile(filename = '.env') {
    const fs = require('fs');
    const path = require('path');

    const envPath = path.resolve(process.cwd(), filename);
    try {
      const envString = fs.readFileSync(envPath, 'utf8');
      const env = this.parse(envString);

      // 合并到 process.env
      Object.assign(process.env, env);

      return env;
    } catch (error) {
      console.error('加载环境文件失败:', error.message);
      return {};
    }
  }
}

// 使用示例
const customEnv = EnvParser.parse(envString);
console.log('自定义解析的环境变量:', customEnv);
```

## 实际应用

### 1. 高级日志系统

```javascript
const util = require('util');
const fs = require('fs');
const path = require('path');

class AdvancedLogger {
  constructor(options = {}) {
    this.options = {
      level: options.level || 'info',
      format: options.format || 'text',
      colors: options.colors !== false,
      timestamp: options.timestamp !== false,
      file: options.file || null,
      maxSize: options.maxSize || 10 * 1024 * 1024, // 10MB
      maxFiles: options.maxFiles || 5,
      ...options
    };

    this.levels = {
      error: 0,
      warn: 1,
      info: 2,
      debug: 3
    };

    this.currentLevel = this.levels[this.options.level] || this.levels.info;

    // 初始化文件日志
    if (this.options.file) {
      this.initFileLogging();
    }

    // 创建不同级别的日志器
    this.debugLog = util.debuglog('app');
  }

  initFileLogging() {
    try {
      // 检查文件大小，如果超过则轮转
      if (fs.existsSync(this.options.file)) {
        const stats = fs.statSync(this.options.file);
        if (stats.size > this.options.maxSize) {
          this.rotateLogFile();
        }
      }

      // 确保目录存在
      const dir = path.dirname(this.options.file);
      if (!fs.existsSync(dir)) {
        fs.mkdirSync(dir, { recursive: true });
      }
    } catch (error) {
      console.error('文件日志初始化失败:', error);
    }
  }

  rotateLogFile() {
    for (let i = this.options.maxFiles - 1; i >= 1; i--) {
      const oldFile = i === 1
        ? this.options.file
        : `${this.options.file}.${i - 1}`;
      const newFile = `${this.options.file}.${i}`;

      if (fs.existsSync(oldFile)) {
        if (fs.existsSync(newFile)) {
          fs.unlinkSync(newFile);
        }
        fs.renameSync(oldFile, newFile);
      }
    }
  }

  formatMessage(level, message, meta = {}) {
    const parts = [];

    if (this.options.timestamp) {
      parts.push(`[${new Date().toISOString()}]`);
    }

    parts.push(`[${level.toUpperCase()}]`);

    if (meta.module) {
      parts.push(`[${meta.module}]`);
    }

    parts.push(message);

    if (Object.keys(meta).length > 0) {
      const cleanMeta = { ...meta };
      delete cleanMeta.module;

      if (this.options.format === 'json') {
        parts.push(util.format('%j', cleanMeta));
      } else {
        parts.push(util.format('%O', cleanMeta));
      }
    }

    return parts.join(' ');
  }

  shouldLog(level) {
    return this.levels[level] <= this.currentLevel;
  }

  writeToFile(message) {
    if (!this.options.file) return;

    try {
      fs.appendFileSync(this.options.file, message + '\n', 'utf8');
    } catch (error) {
      console.error('写入日志文件失败:', error);
    }
  }

  log(level, message, meta = {}) {
    if (!this.shouldLog(level)) return;

    const formattedMessage = this.formatMessage(level, message, meta);

    // 输出到控制台
    if (level === 'error') {
      console.error(formattedMessage);
    } else if (level === 'warn') {
      console.warn(formattedMessage);
    } else {
      console.log(formattedMessage);
    }

    // 输出到文件
    this.writeToFile(formattedMessage);

    // 输出到调试日志
    if (level === 'debug') {
      this.debugLog(formattedMessage);
    }
  }

  error(message, meta = {}) {
    this.log('error', message, meta);
  }

  warn(message, meta = {}) {
    this.log('warn', message, meta);
  }

  info(message, meta = {}) {
    this.log('info', message, meta);
  }

  debug(message, meta = {}) {
    this.log('debug', message, meta);
  }

  // 上下文日志
  context(context) {
    const logger = this;

    return {
      error: (message, meta = {}) => logger.error(message, { ...meta, ...context }),
      warn: (message, meta = {}) => logger.warn(message, { ...meta, ...context }),
      info: (message, meta = {}) => logger.info(message, { ...meta, ...context }),
      debug: (message, meta = {}) => logger.debug(message, { ...meta, ...context })
    };
  }
}

// 使用示例
const logger = new AdvancedLogger({
  level: 'debug',
  file: './logs/app.log',
  format: 'text',
  colors: true
});

logger.info('应用程序启动', { version: '1.0.0', env: 'development' });

// 带上下文的日志
const dbLogger = logger.context({ module: 'database' });
dbLogger.info('连接数据库', { host: 'localhost', port: 27017 });

dbLogger.error('连接失败', new Error('Connection refused'));

// 模块化日志
const apiLogger = logger.context({ module: 'api' });
apiLogger.debug('处理请求', { method: 'GET', path: '/users', userId: 123 });
```

### 2. 配置验证工具

```javascript
const util = require('util');

class ConfigValidator {
  static validate(config, schema) {
    const errors = [];

    for (const [key, rules] of Object.entries(schema)) {
      const value = config[key];

      // 检查必填字段
      if (rules.required && value === undefined) {
        errors.push(`${key} is required`);
        continue;
      }

      // 跳过可选字段
      if (value === undefined && !rules.required) {
        continue;
      }

      // 类型检查
      if (rules.type && !this.checkType(value, rules.type)) {
        errors.push(`${key} must be ${rules.type}`);
        continue;
      }

      // 枚举检查
      if (rules.enum && !rules.enum.includes(value)) {
        errors.push(`${key} must be one of: ${rules.enum.join(', ')}`);
      }

      // 数值范围检查
      if (rules.min !== undefined && value < rules.min) {
        errors.push(`${key} must be >= ${rules.min}`);
      }

      if (rules.max !== undefined && value > rules.max) {
        errors.push(`${key} must be <= ${rules.max}`);
      }

      // 字符串长度检查
      if (rules.minLength && value.length < rules.minLength) {
        errors.push(`${key} must have at least ${rules.minLength} characters`);
      }

      if (rules.maxLength && value.length > rules.maxLength) {
        errors.push(`${key} must have at most ${rules.maxLength} characters`);
      }

      // 正则表达式检查
      if (rules.pattern && !rules.pattern.test(value)) {
        errors.push(`${key} does not match the required pattern`);
      }

      // 自定义验证器
      if (rules.validator && !rules.validator(value)) {
        errors.push(`${key} failed custom validation`);
      }
    }

    return {
      valid: errors.length === 0,
      errors
    };
  }

  static checkType(value, expectedType) {
    const { types } = util;

    switch (expectedType) {
      case 'string':
        return types.isString(value);
      case 'number':
        return types.isNumber(value);
      case 'boolean':
        return types.isBoolean(value);
      case 'array':
        return types.isArray(value);
      case 'object':
        return types.isObject(value);
      case 'function':
        return types.isFunction(value);
      case 'date':
        return types.isDate(value);
      case 'regexp':
        return types.isRegExp(value);
      default:
        return true;
    }
  }

  static mergeDefaults(config, schema) {
    const merged = { ...config };

    for (const [key, rules] of Object.entries(schema)) {
      if (merged[key] === undefined && rules.default !== undefined) {
        merged[key] = rules.default;
      }
    }

    return merged;
  }
}

// 使用示例
const configSchema = {
  port: {
    type: 'number',
    required: true,
    min: 1,
    max: 65535
  },
  host: {
    type: 'string',
    required: true
  },
  database: {
    type: 'object',
    required: true,
    validator: (value) => {
      return value.url && value.name;
    }
  },
  logLevel: {
    type: 'string',
    enum: ['error', 'warn', 'info', 'debug'],
    default: 'info'
  },
  features: {
    type: 'array',
    default: []
  }
};

const userConfig = {
  port: 3000,
  host: 'localhost',
  database: {
    url: 'mongodb://localhost:27017',
    name: 'myapp'
  },
  logLevel: 'debug'
};

// 验证配置
const validation = ConfigValidator.validate(userConfig, configSchema);
console.log('配置验证:', validation);

// 合并默认值
const mergedConfig = ConfigValidator.mergeDefaults(userConfig, configSchema);
console.log('合并后的配置:', mergedConfig);
```

### 3. 异步任务管理器

```javascript
const util = require('util');

class AsyncTaskManager {
  constructor(concurrency = 5) {
    this.concurrency = concurrency;
    this.queue = [];
    this.running = 0;
    this.results = new Map();
  }

  async add(taskFn, taskId) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        fn: taskFn,
        id: taskId || Date.now() + Math.random(),
        resolve,
        reject
      });

      this.process();
    });
  }

  async process() {
    if (this.running >= this.concurrency || this.queue.length === 0) {
      return;
    }

    const task = this.queue.shift();
    this.running++;

    try {
      const result = await task.fn();
      this.results.set(task.id, { status: 'completed', result });
      task.resolve(result);
    } catch (error) {
      this.results.set(task.id, { status: 'failed', error });
      task.reject(error);
    } finally {
      this.running--;
      this.process();
    }
  }

  async waitAll() {
    while (this.running > 0 || this.queue.length > 0) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }

  getResults() {
    return Object.fromEntries(this.results);
  }

  async runParallel(tasks) {
    const promises = tasks.map(task => this.add(task.fn, task.id));
    return Promise.all(promises);
  }

  async runSequential(tasks) {
    const results = [];
    for (const task of tasks) {
      try {
        const result = await this.add(task.fn, task.id);
        results.push({ id: task.id, status: 'success', result });
      } catch (error) {
        results.push({ id: task.id, status: 'error', error });
      }
    }
    return results;
  }
}

// 使用示例
const taskManager = new AsyncTaskManager(3);

// 添加任务
const task1 = taskManager.add(async () => {
  await new Promise(resolve => setTimeout(resolve, 1000));
  return 'Task 1 completed';
}, 'task1');

const task2 = taskManager.add(async () => {
  await new Promise(resolve => setTimeout(resolve, 1500));
  return 'Task 2 completed';
}, 'task2');

const task3 = taskManager.add(async () => {
  await new Promise(resolve => setTimeout(resolve, 800));
  return 'Task 3 completed';
}, 'task3');

// 等待所有任务完成
Promise.all([task1, task2, task3])
  .then(results => {
    console.log('所有任务完成:', results);
    console.log('任务管理器结果:', taskManager.getResults());
  })
  .catch(error => {
    console.error('任务执行失败:', error);
  });

// 并行运行多个任务
async function parallelExample() {
  const tasks = [
    { id: 'parallel1', fn: async () => { await new Promise(resolve => setTimeout(resolve, 500)); return 'P1'; }},
    { id: 'parallel2', fn: async () => { await new Promise(resolve => setTimeout(resolve, 600)); return 'P2'; }},
    { id: 'parallel3', fn: async () => { await new Promise(resolve => setTimeout(resolve, 400)); return 'P3'; }}
  ];

  const manager = new AsyncTaskManager(2);
  const results = await manager.runParallel(tasks);
  console.log('并行任务结果:', results);
}
```

### 4. 错误处理增强工具

```javascript
const util = require('util');

class ErrorHandler {
  static formatError(error) {
    if (!error) return null;

    const errorInfo = {
      name: error.name,
      message: error.message,
      code: error.code,
      stack: error.stack,
      timestamp: new Date().toISOString()
    };

    // 处理特殊错误类型
    if (util.types.isNativeError(error)) {
      errorInfo.type = 'NativeError';
    }

    // 如果是系统错误
    if (error.syscall || error.errno) {
      errorInfo.syscall = error.syscall;
      errorInfo.errno = error.errno;
      errorInfo.path = error.path;
    }

    // 如果是验证错误
    if (error.errors) {
      errorInfo.validationErrors = error.errors;
    }

    return errorInfo;
  }

  static createErrorClass(name) {
    return class extends Error {
      constructor(message, options = {}) {
        super(message);
        this.name = name;
        this.code = options.code;
        this.status = options.status;
        this.details = options.details;

        if (options.cause) {
          this.cause = options.cause;
        }
      }

      toJSON() {
        return {
          name: this.name,
          message: this.message,
          code: this.code,
          status: this.status,
          details: this.details,
          cause: this.cause ? this.cause.message : undefined
        };
      }
    };
  }

  static async withErrorHandling(fn, errorHandler) {
    try {
      return await fn();
    } catch (error) {
      if (errorHandler) {
        return errorHandler(error);
      }
      throw error;
    }
  }

  static async withRetry(fn, maxRetries = 3, delay = 1000) {
    let lastError;

    for (let i = 0; i < maxRetries; i++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;
        if (i < maxRetries - 1) {
          await new Promise(resolve => setTimeout(resolve, delay));
          delay *= 2;
        }
      }
    }

    throw lastError;
  }

  static async withFallback(primaryFn, fallbackFn) {
    try {
      return await primaryFn();
    } catch (primaryError) {
      try {
        const fallbackResult = await fallbackFn(primaryError);
        return fallbackResult;
      } catch (fallbackError) {
        console.error('主函数和后备函数都失败了');
        throw primaryError;
      }
    }
  }
}

// 创建自定义错误类
const ValidationError = ErrorHandler.createErrorClass('ValidationError');
const DatabaseError = ErrorHandler.createErrorClass('DatabaseError');
const NetworkError = ErrorHandler.createErrorClass('NetworkError');

// 使用示例
async function errorHandlingExample() {
  // 自定义错误
  const validationError = new ValidationError('Invalid email format', {
    code: 'INVALID_EMAIL',
    status: 400,
    details: { field: 'email', value: 'invalid' }
  });

  console.log('自定义错误:', util.inspect(validationError, { depth: null }));
  console.log('错误 JSON:', validationError.toJSON());

  // 错误格式化
  try {
    throw new Error('Something went wrong');
  } catch (error) {
    const formattedError = ErrorHandler.formatError(error);
    console.log('格式化错误:', util.inspect(formattedError, { depth: null }));
  }

  // 错误处理包装
  const riskyOperation = async () => {
    if (Math.random() > 0.5) {
      throw new Error('Random failure');
    }
    return 'Success';
  };

  const result = await ErrorHandler.withErrorHandling(riskyOperation, (error) => {
    console.error('操作失败:', error.message);
    return 'Fallback result';
  });

  console.log('结果:', result);
}

errorHandlingExample();
```

## 注意事项

### 1. 性能考虑

```javascript
const util = require('util');

// 避免在生产环境中过度使用 util.inspect()
class PerformanceOptimizations {
  static safeInspect(obj, production = true) {
    if (production) {
      // 生产环境使用简化输出
      return util.inspect(obj, {
        depth: 2,
        maxArrayLength: 10,
        maxStringLength: 100,
        compact: true
      });
    } else {
      // 开发环境使用详细输出
      return util.inspect(obj, {
        depth: null,
        colors: true,
        showHidden: true
      });
    }
  }

  // 延迟格式化
  static lazyInspect(obj) {
    let cached = null;

    return () => {
      if (!cached) {
        cached = util.inspect(obj);
      }
      return cached;
    };
  }

  // 批量格式化
  static batchInspect(objects) {
    return objects.map(obj => ({
      original: obj,
      inspected: util.inspect(obj, { compact: true })
    }));
  }
}
```

### 2. 内存管理

```javascript
const util = require('util');

class MemoryEfficientInspector {
  static inspectLargeObject(largeObj, options = {}) {
    const result = [];

    // 分块处理大对象
    const chunks = Object.entries(largeObj);
    const chunkSize = options.chunkSize || 100;

    for (let i = 0; i < chunks.length; i += chunkSize) {
      const chunk = chunks.slice(i, i + chunkSize);
      const chunkObj = Object.fromEntries(chunk);
      result.push(util.inspect(chunkObj, { ...options, compact: true }));

      // 释放内存
      chunk.length = 0;
    }

    return result.join('\n');
  }

  // 流式检查
  static *streamInspect(obj, bufferSize = 1000) {
    const entries = Object.entries(obj);
    let buffer = [];

    for (const [key, value] of entries) {
      buffer.push(`${key}: ${util.inspect(value)}`);

      if (buffer.length >= bufferSize) {
        yield buffer.join('\n');
        buffer = [];
      }
    }

    if (buffer.length > 0) {
      yield buffer.join('\n');
    }
  }
}
```

### 3. 向后兼容性

```javascript
const util = require('util');

class CompatibilityWrapper {
  static checkAPIAvailability() {
    return {
      parseEnv: typeof util.parseEnv === 'function',
      toUSVString: typeof util.toUSVString === 'function',
      types: !!util.types
    };
  }

  // 提供兼容的实现
  static toUSVStringFallback(str) {
    if (typeof util.toUSVString === 'function') {
      return util.toUSVString(str);
    }

    // 简化的替代实现
    return str.replace(/[\uD800-\uDFFF]/g, '�');
  }

  // 兼容的类型检查
  static isDate(value) {
    if (util.types && util.types.isDate) {
      return util.types.isDate(value);
    }

    return Object.prototype.toString.call(value) === '[object Date]';
  }
}
```

## 总结

Node.js 的 `util` 模块提供了丰富而实用的工具函数，是日常开发中不可或缺的助手。通过本文的学习，我们掌握了：

1. **精确的类型检查**：`util.types` 提供了比原生 `typeof` 更准确的类型检查方法
2. **强大的格式化输出**：`util.format()` 和 `util.inspect()` 用于字符串格式化和对象调试
3. **异步流程控制**：`util.promisify()` 和 `util.callbackify()` 实现回调函数和 Promise 的相互转换
4. **调试和日志工具**：`util.debuglog()` 和 `util.deprecate()` 用于条件调试和 API 废弃警告
5. **实用工具函数**：继承、字符串处理、错误处理等各种辅助功能

在实际开发中，合理使用 `util` 模块可以：
- 提高代码的可读性和可维护性
- 简化异步编程的复杂度
- 增强错误处理和调试能力
- 实现跨版本的兼容性
- 优化性能和内存使用

建议开发者深入理解这些工具函数的特性和最佳实践，在实际项目中灵活运用，以提升开发效率和代码质量。同时，要关注 Node.js 版本的更新，及时掌握新增的工具函数和废弃的功能，确保代码的现代化和可维护性。