---
title: Node.js 全局对象
published: 2023-02-17
description: 'global、process、Buffer 等的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 中的全局对象是在任何地方都可以访问的对象和变量，无需显式导入。理解这些全局对象对于编写高效的 Node.js 应用至关重要。本文将详细介绍 global、process、Buffer 等重要的全局对象。

## 核心概念

### global 对象

`global` 是 Node.js 的全局命名空间，类似于浏览器中的 `window` 对象。所有全局变量和函数都是 global 对象的属性。

### process 对象

`process` 对象提供了当前 Node.js 进程的信息和控制能力，可以获取命令行参数、环境变量、进程信息等。

### Buffer 对象

`Buffer` 用于处理二进制数据，是 Node.js 处理文件、网络通信等场景的重要工具。

### 其他全局对象

包括 `console`、`setTimeout`、`setInterval`、`require`、`module`、`exports` 等。

## 基本用法

### global 对象

```javascript
// 访问 global 对象
console.log(global === globalThis); // true

// 在全局作用域声明的变量会成为 global 的属性
global.myGlobalVar = 'Hello World';
console.log(myGlobalVar); // 'Hello World'

// 注意：使用 const/let/var 声明的变量不会成为 global 的属性
const localVar = 'Local';
console.log(global.localVar); // undefined

// 不使用关键字声明的变量会成为 global 的属性
globalVar = 'Global';
console.log(global.globalVar); // 'Global'
```

### process 对象

```javascript
// 获取进程信息
console.log('进程 ID:', process.pid);
console.log('Node.js 版本:', process.version);
console.log('平台:', process.platform);
console.log('架构:', process.arch);

// 获取命令行参数
console.log('命令行参数:', process.argv);
console.log('执行路径:', process.execPath);

// 获取环境变量
console.log('当前工作目录:', process.cwd());
console.log('NODE_ENV:', process.env.NODE_ENV);

// 设置环境变量
process.env.CUSTOM_VAR = 'custom_value';

// 退出进程
process.exit(0); // 正常退出
process.exit(1); // 异常退出
```

### process 事件

```javascript
// 进程退出前事件
process.on('exit', (code) => {
  console.log(`进程即将退出，退出码: ${code}`);
});

// 未捕获异常
process.on('uncaughtException', (err) => {
  console.error('未捕获的异常:', err);
  // 生产环境应该优雅关闭
  process.exit(1);
});

// 未处理的 Promise 拒绝
process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的 Promise 拒绝:', reason);
});

// 信号事件
process.on('SIGTERM', () => {
  console.log('收到 SIGTERM 信号');
  // 执行清理操作
  process.exit(0);
});

process.on('SIGINT', () => {
  console.log('收到 SIGINT 信号 (Ctrl+C)');
  process.exit(0);
});
```

### process 方法

```javascript
// 定时器
console.log('开始');
setTimeout(() => console.log('1秒后执行'), 1000);

const interval = setInterval(() => console.log('每秒执行'), 1000);

// 取消定时器
setTimeout(() => clearInterval(interval), 5000);

// 下一个事件循环
setImmediate(() => console.log('下一个事件循环执行'));

// 内存信息
console.log('内存使用情况:', process.memoryUsage());

// CPU 信息
const startUsage = process.cpuUsage();

// 执行一些操作
for (let i = 0; i < 1000000; i++) {
  Math.sqrt(i);
}

const endUsage = process.cpuUsage(startUsage);
console.log('CPU 使用时间:', endUsage);

// 改变工作目录
process.chdir('/tmp');
console.log('新的工作目录:', process.cwd());

// 发送信号
process.kill(process.pid, 'SIGTERM');
```

### Buffer 对象

```javascript
// 创建 Buffer
const buf1 = Buffer.alloc(10); // 创建指定大小的空 Buffer
const buf2 = Buffer.from('Hello World'); // 从字符串创建
const buf3 = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f]); // 从数组创建

// 访问 Buffer
console.log(buf2.toString()); // 'Hello World'
console.log(buf2.length); // 11
console.log(buf2[0]); // 72 (ASCII 码)

// 修改 Buffer
buf2[0] = 0x68; // 改为小写 'h'
console.log(buf2.toString()); // 'hello World'

// Buffer 拼接
const buf4 = Buffer.from('Hello ');
const buf5 = Buffer.from('World');
const buf6 = Buffer.concat([buf4, buf5]);
console.log(buf6.toString()); // 'Hello World'

// Buffer 切片
const buf7 = Buffer.from('Hello World');
const buf8 = buf7.slice(0, 5);
console.log(buf8.toString()); // 'Hello'

// Buffer 复制
const buf9 = Buffer.alloc(10);
buf2.copy(buf9);
console.log(buf9.toString()); // 'hello Worl'
```

### Buffer 实用方法

```javascript
// 字符串编码转换
const str = '你好世界';
const utf8Buf = Buffer.from(str, 'utf8');
const base64Str = utf8Buf.toString('base64');
console.log(base64Str); // '5L2g5aW95LiW55WM'

// Buffer 转换为 JSON
const buf10 = Buffer.from([0x1, 0x2, 0x3, 0x4]);
console.log(buf10.toJSON()); // { type: 'Buffer', data: [1, 2, 3, 4] }

// 比较 Buffer
const buf11 = Buffer.from('ABC');
const buf12 = Buffer.from('ABCD');
console.log(buf11.compare(buf12)); // -1 (小于)

// 检查是否包含数据
const buf13 = Buffer.from('Hello World');
console.log(buf13.includes('World')); // true
console.log(buf13.indexOf('World')); // 6

// 填充 Buffer
const buf14 = Buffer.alloc(10);
buf14.fill('A');
console.log(buf14.toString()); // 'AAAAAAAAAA'
```

### console 对象

```javascript
// 基本输出
console.log('普通日志');
console.error('错误日志');
console.warn('警告日志');
console.info('信息日志');

// 格式化输出
console.log('姓名: %s, 年龄: %d', '张三', 25);
console.log('对象: %o', { name: '张三', age: 25 });

// 分组输出
console.group('用户信息');
console.log('姓名: 张三');
console.log('年龄: 25');
console.groupEnd();

// 计时
console.time('操作');
for (let i = 0; i < 1000000; i++) {
  Math.sqrt(i);
}
console.timeEnd('操作');

// 计数
console.count('点击');
console.count('点击');
console.count('点击');

// 表格输出
console.table([
  { name: '张三', age: 25 },
  { name: '李四', age: 30 }
]);

// 清空控制台
console.clear();
```

### 定时器函数

```javascript
// setTimeout
const timeoutId = setTimeout(() => {
  console.log('1秒后执行');
}, 1000);

// clearTimeout
clearTimeout(timeoutId);

// setInterval
const intervalId = setInterval(() => {
  console.log('每秒执行');
}, 1000);

// clearInterval
clearInterval(intervalId);

// setImmediate
const immediateId = setImmediate(() => {
  console.log('下一个事件循环执行');
});

// clearImmediate
clearImmediate(immediateId);

// 定时器与微任务队列的执行顺序
console.log('1');

setTimeout(() => console.log('2'), 0);

setImmediate(() => console.log('3'));

Promise.resolve().then(() => console.log('4'));

process.nextTick(() => console.log('5'));

console.log('6');
// 输出顺序: 1, 6, 5, 4, 2, 3 或 1, 6, 5, 4, 3, 2
```

## 实际应用

### 命令行工具

```javascript
#!/usr/bin/env node

// process.argv 的实际应用
const args = process.argv.slice(2);
const command = args[0];

switch (command) {
  case 'init':
    console.log('初始化项目...');
    break;
  case 'build':
    console.log('构建项目...');
    break;
  case 'start':
    const port = args[1] || 3000;
    console.log(`启动服务器，端口: ${port}`);
    break;
  default:
    console.log('使用方法: node app.js [init|build|start] [port]');
    process.exit(1);
}
```

### 配置管理

```javascript
// 使用 process.env 管理配置
const config = {
  port: parseInt(process.env.PORT) || 3000,
  host: process.env.HOST || 'localhost',
  database: {
    host: process.env.DB_HOST || 'localhost',
    port: parseInt(process.env.DB_PORT) || 5432,
    name: process.env.DB_NAME || 'myapp'
  },
  env: process.env.NODE_ENV || 'development'
};

// 根据环境加载不同配置
if (config.env === 'development') {
  console.log('开发模式');
} else if (config.env === 'production') {
  console.log('生产模式');
}
```

### 文件处理

```javascript
const fs = require('fs');
const path = require('path');

// 读取文件内容
const filePath = path.join(__dirname, 'example.txt');
const buffer = fs.readFileSync(filePath);

// 转换为字符串
const content = buffer.toString('utf8');
console.log(content);

// 处理二进制数据
const imageBuffer = fs.readFileSync('image.png');
const base64Image = imageBuffer.toString('base64');

// 在 HTML 中使用
const html = `<img src="data:image/png;base64,${base64Image}">`;
```

### 进程监控

```javascript
// 监控进程状态
setInterval(() => {
  const memory = process.memoryUsage();
  const cpu = process.cpuUsage();

  console.log({
    memory: {
      rss: `${Math.round(memory.rss / 1024 / 1024)} MB`,
      heapTotal: `${Math.round(memory.heapTotal / 1024 / 1024)} MB`,
      heapUsed: `${Math.round(memory.heapUsed / 1024 / 1024)} MB`
    },
    uptime: `${Math.round(process.uptime())} s`
  });
}, 60000);

// 监听进程信号
process.on('SIGUSR2', () => {
  console.log('收到 SIGUSR2 信号，重新加载配置');
  // 重新加载配置逻辑
});
```

### 错误处理

```javascript
// 全局错误处理
process.on('uncaughtException', (err) => {
  console.error('未捕获的异常:', err);
  // 记录错误日志
  fs.appendFileSync('error.log', `${new Date()}: ${err.stack}\n`);
  // 优雅关闭
  gracefulShutdown();
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的 Promise 拒绝:', reason);
  fs.appendFileSync('error.log', `${new Date()}: ${reason}\n`);
});

function gracefulShutdown() {
  console.log('开始优雅关闭...');
  // 关闭数据库连接
  // 清理资源
  // 保存状态
  process.exit(1);
}
```

## 注意事项

### 1. 避免污染 global 对象

```javascript
// ❌ 错误：污染全局命名空间
global.myHelper = function() { /* ... */ };

// ✅ 正确：使用模块
module.exports.myHelper = function() { /* ... */ };
```

### 2. process.exit 的正确使用

```javascript
// ❌ 错误：直接退出可能丢失数据
process.exit(1);

// ✅ 正确：先清理资源再退出
function gracefulExit(code) {
  server.close(() => {
    console.log('服务器已关闭');
    database.disconnect(() => {
      process.exit(code);
    });
  });

  // 超时强制退出
  setTimeout(() => {
    console.log('强制退出');
    process.exit(code);
  }, 10000);
}
```

### 3. Buffer 的安全性

```javascript
// ❌ 错误：使用已废弃的构造函数
const buf = new Buffer(10);

// ✅ 正确：使用推荐的方法
const buf1 = Buffer.alloc(10); // 初始化为零
const buf2 = Buffer.allocUnsafe(10); // 更快但可能包含旧数据
const buf3 = Buffer.from('Hello'); // 从数据创建
```

### 4. 内存泄漏检测

```javascript
// 定期检查内存使用
setInterval(() => {
  const usage = process.memoryUsage();
  const heapUsed = usage.heapUsed / 1024 / 1024;

  if (heapUsed > 500) {
    console.warn('内存使用过高:', heapUsed.toFixed(2), 'MB');
    // 触发垃圾回收（仅开发环境）
    if (process.env.NODE_ENV === 'development') {
      global.gc && global.gc();
    }
  }
}, 30000);
```

### 5. 异步操作的顺序

```javascript
// 理解事件循环的执行顺序
console.log('1'); // 同步

process.nextTick(() => console.log('2')); // 微任务 1

Promise.resolve().then(() => console.log('3')); // 微任务 2

setTimeout(() => console.log('4'), 0); // 宏任务 1

setImmediate(() => console.log('5')); // 宏任务 2

console.log('6'); // 同步

// 输出顺序: 1, 6, 2, 3, 4/5 (4 和 5 的顺序不确定)
```

## 总结

Node.js 全局对象是构建 Node.js 应用的基础工具，掌握这些对象对于编写高效、稳定的应用至关重要：

- **global**: 全局命名空间，避免污染
- **process**: 进程控制、环境信息、事件处理
- **Buffer**: 二进制数据处理，文件和网络通信
- **console**: 调试和日志输出
- **定时器**: 异步调度和时间控制

合理使用这些全局对象，可以帮助我们更好地控制 Node.js 应用的行为和性能。