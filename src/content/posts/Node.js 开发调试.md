---
title: Node.js 开发调试
published: 2023-03-04
description: '调试工具和方法的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

调试是开发过程中不可或缺的环节。Node.js 提供了多种调试工具和方法，从简单的 console.log 到功能强大的调试器，本文将详细介绍各种调试技巧和工具。

## Console 调试

### 基本输出方法

```javascript
console.log('普通日志');
console.error('错误日志');
console.warn('警告日志');
console.info('信息日志');
console.debug('调试日志');
```

### 格式化输出

```javascript
// 占位符
const name = 'Alice';
const age = 25;
console.log('姓名: %s, 年龄: %d', name, age);

// 对象输出
const user = { name: 'Alice', age: 25 };
console.log('用户信息:', user);
console.dir(user, { depth: null, colors: true });

// 表格输出
const users = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 }
];
console.table(users);

// 分组输出
console.group('用户信息');
console.log('姓名:', name);
console.log('年龄:', age);
console.groupEnd();
```

### 性能测量

```javascript
// 计时
console.time('数据处理');
// ... 执行操作
console.timeEnd('数据处理'); // 输出耗时

// 计数
console.count('函数调用');
console.count('函数调用');
console.count('函数调用');
console.countReset('函数调用');

// 堆栈跟踪
console.trace('调用堆栈');
```

### 进度和清屏

```javascript
// 清空控制台
console.clear();

// 进度显示（在终端中）
const readline = require('readline');
readline.cursorTo(process.stdout, 0);
process.stdout.write('处理中...');
```

## 内置调试器

### 启动调试模式

```bash
# 调试模式运行脚本
node debug app.js

# 或使用 inspect
node inspect app.js
```

### 调试命令

```
cont 或 c    # 继续执行
next 或 n    # 单步跳过
step 或 s    # 单步进入
out 或 o     # 单步退出
pause        # 暂停执行
watch('expr') # 监视表达式
unwatch('expr') # 取消监视
backtrace 或 bt # 显示调用栈
list(5)      # 显示源代码
```

### 在代码中设置断点

```javascript
const http = require('http');

debugger; // 程序会在这里暂停

http.createServer((req, res) => {
  debugger; // 设置断点
  res.writeHead(200);
  res.end('Hello World');
}).listen(3000);
```

## Chrome DevTools 调试

### 启动 Chrome DevTools

```bash
# 启用调试
node --inspect app.js

# 或在启动时暂停
node --inspect-brk app.js

# 指定端口
node --inspect=9229 app.js
```

### 连接 DevTools

1. 打开 Chrome 浏览器
2. 访问 `chrome://inspect`
3. 点击 "Open dedicated DevTools for Node"

### 示例代码

```javascript
// app.js
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  const data = fetchData(); // 可以在这里设置断点
  res.json(data);
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});

function fetchData() {
  // 断点可以设置在这里
  return { message: 'Hello World' };
}
```

### DevTools 功能

- **Sources**: 设置断点、查看源代码
- **Console**: 执行代码、查看日志
- **Network**: 监控网络请求
- **Memory**: 内存分析
- **Performance**: 性能分析

## VS Code 调试

### 创建 launch.json

在 `.vscode/launch.json` 中配置：

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "启动程序",
      "skipFiles": ["<node_internals>/**"],
      "program": "${workspaceFolder}/app.js"
    },
    {
      "type": "node",
      "request": "attach",
      "name": "附加到进程",
      "port": 9229
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Mocha 测试",
      "program": "${workspaceFolder}/node_modules/.bin/mocha",
      "args": [
        "-u",
        "tdd",
        "--timeout",
        "999999",
        "--colors",
        "${workspaceFolder}/test"
      ],
      "internalConsoleOptions": "neverOpen"
    }
  ]
}
```

### 调试快捷键

- `F5`: 启动调试
- `F9`: 切换断点
- `F10`: 单步跳过
- `F11`: 单步进入
- `Shift+F11`: 单步退出
- `Shift+F5`: 停止调试

### 调试配置示例

```json
{
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "当前文件",
      "program": "${file}"
    },
    {
      "type": "node",
      "request": "launch",
      "name": "带环境变量",
      "program": "${workspaceFolder}/app.js",
      "env": {
        "NODE_ENV": "development",
        "PORT": "3000"
      }
    },
    {
      "type": "node",
      "request": "launch",
      "name": "Nodemon 启动",
      "runtimeExecutable": "nodemon",
      "program": "${workspaceFolder}/app.js",
      "restart": true,
      "console": "integratedTerminal"
    }
  ]
}
```

## 调试工具库

### debug

```javascript
// 安装
npm install debug

// 使用
const debug = require('debug')('myapp:server');

debug('服务器启动');
debug('请求数据: %o', { id: 1, name: 'Alice' });

// 启用调试输出
// Linux/Mac: DEBUG=myapp:* node app.js
// Windows: set DEBUG=myapp:* && node app.js
```

### util.inspect

```javascript
const util = require('util');

const obj = {
  name: 'Alice',
  age: 25,
  hobbies: ['reading', 'coding']
};

// 深度检查
console.log(util.inspect(obj, {
  depth: null,      // 深度
  colors: true,     // 颜色
  compact: false,   // 紧凑
  showHidden: true  // 显示隐藏属性
}));
```

### util.format

```javascript
const util = require('util');

const message = util.format('Hello %s, you are %d years old', 'Alice', 25);
console.log(message);
```

## 日志管理

### Winston

```javascript
// 安装
npm install winston

// 使用
const winston = require('winston');

const logger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' })
  ]
});

// 开发环境添加控制台输出
if (process.env.NODE_ENV !== 'production') {
  logger.add(new winston.transports.Console({
    format: winston.format.simple()
  }));
}

// 使用
logger.info('服务器启动');
logger.error('发生错误', { error: err });
logger.warn('警告信息');
logger.debug('调试信息');
```

### Pino

```javascript
// 安装
npm install pino pino-pretty

// 使用
const pino = require('pino');
const logger = pino({
  transport: {
    target: 'pino-pretty',
    options: {
      colorize: true
    }
  }
});

logger.info({ msg: '服务器启动', port: 3000 });
logger.error({ err });
```

## 性能分析

### CPU 分析

```bash
# 使用 --prof 选项
node --prof app.js

# 生成分析报告
node --prof-process isolate-*.log > profile.txt
```

### 内存分析

```bash
# 生成内存快照
node --heapsnapshot-signal=SIGUSR2 app.js

# 发送信号生成快照
kill -USR2 <pid>

# 使用 Chrome DevTools 分析快照
```

### v8-profiler

```javascript
// 安装
npm install v8-profiler-next

// 使用
const v8 = require('v8-profiler-next');
const fs = require('fs');

// 开始性能分析
v8.startProfiling('CPU Profile');

// ... 执行代码

// 停止性能分析
const profile = v8.stopProfiling('CPU Profile');
profile.export()
  .pipe(fs.createWriteStream('profile.cpuprofile'))
  .on('finish', () => profile.delete());
```

## 错误追踪

### 错误堆栈

```javascript
function a() {
  b();
}

function b() {
  c();
}

function c() {
  const err = new Error('出错了');
  console.log(err.stack);
  // 或
  console.trace();
}

a();
```

### 获取调用栈

```javascript
const { stackTrace } = require('node:console');

function getCaller() {
  const stack = stackTrace();
  return stack[1]; // 获取调用者信息
}

function foo() {
  const caller = getCaller();
  console.log('调用者:', caller.getFunctionName());
}

foo();
```

## 调试技巧

### 条件断点

在代码中设置条件：

```javascript
if (process.env.DEBUG === 'true') {
  debugger;
}
```

### 环境变量

```javascript
const DEBUG = process.env.DEBUG === 'true';

function debugLog(...args) {
  if (DEBUG) {
    console.log(...args);
  }
}

debugLog('调试信息');
```

### 日志级别

```javascript
const LOG_LEVELS = {
  ERROR: 0,
  WARN: 1,
  INFO: 2,
  DEBUG: 3
};

const currentLevel = LOG_LEVELS[process.env.LOG_LEVEL || 'INFO'];

function log(level, ...args) {
  if (LOG_LEVELS[level] <= currentLevel) {
    console[level.toLowerCase()](...args);
  }
}

log('DEBUG', '调试信息');
log('INFO', '普通信息');
log('WARN', '警告信息');
log('ERROR', '错误信息');
```

## 异步代码调试

### Promise 调试

```javascript
// 使用 then/catch
fetchData()
  .then(data => {
    debugger; // 断点
    console.log('数据:', data);
  })
  .catch(err => {
    console.error('错误:', err);
  });

// 使用 async/await
async function main() {
  try {
    const data = await fetchData();
    debugger; // 断点
    console.log('数据:', data);
  } catch (err) {
    console.error('错误:', err);
  }
}
```

### 事件循环调试

```javascript
const process = require('process');

// 监听未处理的 Promise rejection
process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的 Promise rejection:', reason);
});

// 监听未捕获的异常
process.on('uncaughtException', (err) => {
  console.error('未捕获的异常:', err);
});
```

## 实际应用

### Express 应用调试

```javascript
const express = require('express');
const debug = require('debug')('myapp:server');
const morgan = require('morgan');

const app = express();

// 请求日志
app.use(morgan('dev'));

app.get('/', (req, res) => {
  debug('处理根路径请求');
  res.send('Hello World');
});

// 错误处理
app.use((err, req, res, next) => {
  debug('错误:', err);
  res.status(500).send('Internal Server Error');
});

app.listen(3000, () => {
  debug('服务器启动在端口 3000');
});
```

### 数据库操作调试

```javascript
const { Pool } = require('pg');
const debug = require('debug')('myapp:db');

const pool = new Pool({
  connectionString: 'postgres://localhost/mydb'
});

async function query(sql, params) {
  debug('执行查询:', sql, params);

  try {
    const result = await pool.query(sql, params);
    debug('查询结果:', result.rows.length, '行');
    return result.rows;
  } catch (err) {
    debug('查询失败:', err);
    throw err;
  }
}
```

## 最佳实践

### 1. 使用适当的日志级别

```javascript
logger.error('严重错误');
logger.warn('警告');
logger.info('重要信息');
logger.debug('调试信息');
```

### 2. 生产环境移除调试代码

```javascript
const isProduction = process.env.NODE_ENV === 'production';

if (!isProduction) {
  app.use(morgan('dev'));
  app.use((req, res, next) => {
    debug('请求:', req.method, req.url);
    next();
  });
}
```

### 3. 敏感信息脱敏

```javascript
function sanitize(obj) {
  const sanitized = { ...obj };
  if (sanitized.password) {
    sanitized.password = '******';
  }
  return sanitized;
}

console.log('用户信息:', sanitize(user));
```

### 4. 使用结构化日志

```javascript
logger.info('用户登录', {
  userId: user.id,
  ip: req.ip,
  timestamp: new Date().toISOString()
});
```

### 5. 定期清理日志

```javascript
const fs = require('fs');
const path = require('path');

function cleanOldLogs(logDir, maxAge) {
  const files = fs.readdirSync(logDir);
  const now = Date.now();

  files.forEach(file => {
    const filePath = path.join(logDir, file);
    const stat = fs.statSync(filePath);

    if (now - stat.mtimeMs > maxAge) {
      fs.unlinkSync(filePath);
      console.log('删除旧日志:', file);
    }
  });
}

// 每天清理 7 天前的日志
cleanOldLogs('./logs', 7 * 24 * 60 * 60 * 1000);
```

## 总结

Node.js 开发调试涉及多个方面：

1. **基础调试**: console.log、内置调试器
2. **IDE 调试**: VS Code、Chrome DevTools
3. **日志管理**: debug、winston、pino
4. **性能分析**: CPU 分析、内存分析
5. **错误追踪**: 堆栈分析、异步调试
6. **最佳实践**: 适当的日志级别、环境区分、信息脱敏

掌握这些调试技巧能够帮助开发者快速定位和解决问题，提高开发效率和代码质量。选择合适的调试工具和方法，根据实际情况灵活运用。