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

### 命令行工具开发

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');

class CLI {
  constructor() {
    this.commands = new Map();
    this.options = new Map();
  }

  registerCommand(name, handler, description = '') {
    this.commands.set(name, { handler, description });
  }

  registerOption(name, handler, description = '') {
    this.options.set(name, { handler, description });
  }

  async run() {
    const args = process.argv.slice(2);
    const command = args[0];
    const options = this.parseOptions(args.slice(1));

    // 显示帮助信息
    if (command === '--help' || command === '-h' || !command) {
      this.showHelp();
      return;
    }

    // 执行命令
    const commandHandler = this.commands.get(command);
    if (commandHandler) {
      try {
        await commandHandler.handler(options);
      } catch (error) {
        console.error(`执行命令失败: ${error.message}`);
        process.exit(1);
      }
    } else {
      console.error(`未知命令: ${command}`);
      this.showHelp();
      process.exit(1);
    }
  }

  parseOptions(args) {
    const options = {};
    for (let i = 0; i < args.length; i++) {
      const arg = args[i];
      if (arg.startsWith('--')) {
        const key = arg.slice(2);
        if (args[i + 1] && !args[i + 1].startsWith('--')) {
          options[key] = args[i + 1];
          i++;
        } else {
          options[key] = true;
        }
      } else if (arg.startsWith('-')) {
        const key = arg.slice(1);
        options[key] = true;
      }
    }
    return options;
  }

  showHelp() {
    console.log('使用方法: cli [命令] [选项]\n');
    console.log('可用命令:');
    for (const [name, { description }] of this.commands) {
      console.log(`  ${name.padEnd(15)} ${description}`);
    }
    console.log('\n全局选项:');
    console.log('  --help, -h     显示帮助信息');
  }
}

// 使用示例
const cli = new CLI();

// 注册命令
cli.registerCommand('init', async (options) => {
  const projectName = options.name || 'my-project';
  const projectPath = path.join(process.cwd(), projectName);

  console.log(`创建项目: ${projectName}`);

  // 创建项目结构
  const structure = {
    'src': {},
    'tests': {},
    'package.json': JSON.stringify({
      name: projectName,
      version: '1.0.0',
      scripts: {
        start: 'node src/index.js',
        test: 'jest'
      }
    }, null, 2),
    'README.md': `# ${projectName}\n\n项目描述`
  };

  function createStructure(base, structure) {
    Object.entries(structure).forEach(([key, value]) => {
      const itemPath = path.join(base, key);
      if (typeof value === 'object' && !Buffer.isBuffer(value)) {
        fs.mkdirSync(itemPath, { recursive: true });
        createStructure(itemPath, value);
      } else {
        fs.writeFileSync(itemPath, value);
      }
    });
  }

  createStructure(projectPath, structure);
  console.log('项目创建完成！');
}, '初始化新项目');

cli.registerCommand('build', async (options) => {
  console.log('开始构建...');
  const watchMode = options.watch || false;

  if (watchMode) {
    console.log('监听模式已启用');
    // 实现文件监听逻辑
  } else {
    // 实现构建逻辑
    console.log('构建完成！');
  }
}, '构建项目');

cli.registerCommand('serve', async (options) => {
  const port = options.port ? parseInt(options.port) : 3000;
  console.log(`启动服务器，端口: ${port}`);

  // 创建简单的 HTTP 服务器
  const http = require('http');
  const server = http.createServer((req, res) => {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('Hello World!');
  });

  server.listen(port, () => {
    console.log(`服务器运行在 http://localhost:${port}`);
  });
}, '启动开发服务器');

// 运行 CLI
cli.run();
```

### 进程管理工具

```javascript
const process = require('process');

class ProcessManager {
  constructor() {
    this.resources = [];
    this.isShuttingDown = false;
    this.setupErrorHandlers();
  }

  /**
   * 设置错误处理器
   */
  setupErrorHandlers() {
    // 未捕获的异常
    process.on('uncaughtException', (err) => {
      this.logError('未捕获的异常', err);
      this.gracefulShutdown(1);
    });

    // 未处理的 Promise 拒绝
    process.on('unhandledRejection', (reason, promise) => {
      this.logError('未处理的 Promise 拒绝', reason);
      this.gracefulShutdown(1);
    });

    // 信号处理
    process.on('SIGTERM', () => {
      console.log('收到 SIGTERM 信号');
      this.gracefulShutdown(0);
    });

    process.on('SIGINT', () => {
      console.log('收到 SIGINT 信号');
      this.gracefulShutdown(0);
    });
  }

  /**
   * 注册需要清理的资源
   */
  registerResource(name, cleanupFn) {
    this.resources.push({ name, cleanupFn });
  }

  /**
   * 记录错误日志
   */
  logError(message, error) {
    const timestamp = new Date().toISOString();
    const logEntry = `[${timestamp}] ${message}: ${error.message}\n${error.stack}\n`;

    console.error(logEntry);

    // 写入错误日志文件
    try {
      fs.appendFileSync('error.log', logEntry);
    } catch (err) {
      console.error('无法写入错误日志:', err);
    }
  }

  /**
   * 优雅关闭
   */
  async gracefulShutdown(exitCode = 0) {
    if (this.isShuttingDown) {
      console.log('已经在关闭进程中...');
      return;
    }

    this.isShuttingDown = true;
    console.log('开始优雅关闭...');

    // 清理所有资源
    for (const resource of this.resources) {
      try {
        console.log(`清理资源: ${resource.name}`);
        await resource.cleanupFn();
      } catch (error) {
        console.error(`清理资源 ${resource.name} 失败:`, error);
      }
    }

    console.log('所有资源已清理完成');
    process.exit(exitCode);
  }

  /**
   * 监控进程状态
   */
  startMonitoring(interval = 60000) {
    this.monitorInterval = setInterval(() => {
      const stats = this.getProcessStats();
      this.logStats(stats);

      // 内存警告
      if (stats.memory.heapUsed > 500 * 1024 * 1024) {
        console.warn('内存使用过高:', this.formatBytes(stats.memory.heapUsed));
      }
    }, interval);
  }

  /**
   * 获取进程统计信息
   */
  getProcessStats() {
    const memory = process.memoryUsage();
    const cpu = process.cpuUsage();

    return {
      pid: process.pid,
      uptime: process.uptime(),
      memory: {
        rss: memory.rss,
        heapTotal: memory.heapTotal,
        heapUsed: memory.heapUsed,
        external: memory.external,
        arrayBuffers: memory.arrayBuffers
      },
      cpu: {
        user: cpu.user,
        system: cpu.system
      },
      platform: process.platform,
      arch: process.arch,
      nodeVersion: process.version
    };
  }

  /**
   * 格式化字节数
   */
  formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
  }

  /**
   * 记录统计信息
   */
  logStats(stats) {
    console.log('进程状态:', {
      PID: stats.pid,
      运行时间: `${Math.floor(stats.uptime)} 秒`,
      内存使用: {
        RSS: this.formatBytes(stats.memory.rss),
        堆总量: this.formatBytes(stats.memory.heapTotal),
        堆使用: this.formatBytes(stats.memory.heapUsed)
      },
      CPU: {
        用户时间: `${stats.cpu.user / 1000} ms`,
        系统时间: `${stats.cpu.system / 1000} ms`
      },
      平台: `${stats.platform} (${stats.arch})`,
      Node版本: stats.nodeVersion
    });
  }

  /**
   * 停止监控
   */
  stopMonitoring() {
    if (this.monitorInterval) {
      clearInterval(this.monitorInterval);
    }
  }
}

// 使用示例
const manager = new ProcessManager();

// 模拟数据库连接
const dbConnection = {
  connect: () => console.log('数据库已连接'),
  disconnect: () => new Promise(resolve => {
    console.log('正在断开数据库连接...');
    setTimeout(() => {
      console.log('数据库连接已断开');
      resolve();
    }, 1000);
  })
};

// 注册资源
manager.registerResource('数据库连接', () => dbConnection.disconnect());

// 模拟 HTTP 服务器
const http = require('http');
const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World!');
});

manager.registerResource('HTTP 服务器', () => new Promise((resolve) => {
  server.close(() => {
    console.log('HTTP 服务器已关闭');
    resolve();
  });
}));

// 启动监控
manager.startMonitoring();

// 模拟启动服务
dbConnection.connect();
server.listen(3000, () => {
  console.log('服务器运行在 http://localhost:3000');
});

// 测试错误处理
setTimeout(() => {
  // 模拟错误
  // throw new Error('测试错误');
}, 5000);
```

### 环境配置管理

```javascript
const process = require('process');
const fs = require('fs');
const path = require('path');

class ConfigManager {
  constructor() {
    this.config = {};
    this.loadConfig();
  }

  /**
   * 加载配置
   */
  loadConfig() {
    // 加载环境变量
    this.config = {
      // 应用配置
      app: {
        name: process.env.APP_NAME || 'my-app',
        version: process.env.APP_VERSION || '1.0.0',
        env: process.env.NODE_ENV || 'development',
        debug: process.env.DEBUG === 'true'
      },

      // 服务器配置
      server: {
        host: process.env.HOST || 'localhost',
        port: parseInt(process.env.PORT) || 3000,
        ssl: process.env.SSL === 'true',
        sslCert: process.env.SSL_CERT,
        sslKey: process.env.SSL_KEY
      },

      // 数据库配置
      database: {
        host: process.env.DB_HOST || 'localhost',
        port: parseInt(process.env.DB_PORT) || 5432,
        name: process.env.DB_NAME || 'myapp',
        user: process.env.DB_USER || 'user',
        password: process.env.DB_PASSWORD || '',
        ssl: process.env.DB_SSL === 'true'
      },

      // 日志配置
      logging: {
        level: process.env.LOG_LEVEL || 'info',
        file: process.env.LOG_FILE || 'app.log',
        maxSize: parseInt(process.env.LOG_MAX_SIZE) || 10485760, // 10MB
        maxFiles: parseInt(process.env.LOG_MAX_FILES) || 5
      },

      // 缓存配置
      cache: {
        enabled: process.env.CACHE_ENABLED === 'true',
        ttl: parseInt(process.env.CACHE_TTL) || 3600,
        maxSize: parseInt(process.env.CACHE_MAX_SIZE) || 1000
      },

      // 安全配置
      security: {
        jwtSecret: process.env.JWT_SECRET || 'change-me-in-production',
        jwtExpiresIn: process.env.JWT_EXPIRES_IN || '24h',
        bcryptRounds: parseInt(process.env.BCRYPT_ROUNDS) || 10,
        corsOrigins: process.env.CORS_ORIGINS ? process.env.CORS_ORIGINS.split(',') : ['*']
      }
    };

    // 加载配置文件（如果存在）
    this.loadConfigFile();
  }

  /**
   * 加载配置文件
   */
  loadConfigFile() {
    const configFile = path.join(process.cwd(), 'config.json');

    if (fs.existsSync(configFile)) {
      try {
        const fileConfig = JSON.parse(fs.readFileSync(configFile, 'utf8'));
        this.config = this.mergeConfig(this.config, fileConfig);
      } catch (error) {
        console.error('加载配置文件失败:', error);
      }
    }
  }

  /**
   * 合并配置
   */
  mergeConfig(target, source) {
    const result = { ...target };

    for (const key in source) {
      if (typeof source[key] === 'object' && !Array.isArray(source[key])) {
        result[key] = this.mergeConfig(target[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    }

    return result;
  }

  /**
   * 获取配置值
   */
  get(path, defaultValue = null) {
    const keys = path.split('.');
    let value = this.config;

    for (const key of keys) {
      if (value && typeof value === 'object' && key in value) {
        value = value[key];
      } else {
        return defaultValue;
      }
    }

    return value;
  }

  /**
   * 设置配置值
   */
  set(path, value) {
    const keys = path.split('.');
    let current = this.config;

    for (let i = 0; i < keys.length - 1; i++) {
      const key = keys[i];
      if (!(key in current) || typeof current[key] !== 'object') {
        current[key] = {};
      }
      current = current[key];
    }

    current[keys[keys.length - 1]] = value;
  }

  /**
   * 检查是否为开发环境
   */
  isDevelopment() {
    return this.config.app.env === 'development';
  }

  /**
   * 检查是否为生产环境
   */
  isProduction() {
    return this.config.app.env === 'production';
  }

  /**
   * 检查是否为测试环境
   */
  isTest() {
    return this.config.app.env === 'test';
  }

  /**
   * 验证必需的配置
   */
  validate() {
    const required = [
      'app.name',
      'server.host',
      'server.port',
      'database.host',
      'database.name'
    ];

    const missing = [];

    for (const path of required) {
      if (!this.get(path)) {
        missing.push(path);
      }
    }

    if (missing.length > 0) {
      throw new Error(`缺少必需的配置: ${missing.join(', ')}`);
    }

    // 生产环境额外验证
    if (this.isProduction()) {
      const productionRequired = [
        'security.jwtSecret',
        'database.password'
      ];

      for (const path of productionRequired) {
        if (!this.get(path) || this.get(path) === 'change-me-in-production') {
          missing.push(path);
        }
      }

      if (missing.length > 0) {
        throw new Error(`生产环境缺少必需的配置: ${missing.join(', ')}`);
      }
    }
  }

  /**
   * 导出配置
   */
  export() {
    // 隐藏敏感信息
    const exported = JSON.parse(JSON.stringify(this.config));

    if (exported.security && exported.security.jwtSecret) {
      exported.security.jwtSecret = '***';
    }

    if (exported.database && exported.database.password) {
      exported.database.password = '***';
    }

    return exported;
  }

  /**
   * 打印配置摘要
   */
  printSummary() {
    console.log('配置摘要:');
    console.log(`  应用名称: ${this.config.app.name}`);
    console.log(`  应用版本: ${this.config.app.version}`);
    console.log(`  运行环境: ${this.config.app.env}`);
    console.log(`  调试模式: ${this.config.app.debug}`);
    console.log(`  服务器: ${this.config.server.host}:${this.config.server.port}`);
    console.log(`  数据库: ${this.config.database.host}:${this.config.database.port}/${this.config.database.name}`);
    console.log(`  日志级别: ${this.config.logging.level}`);
  }
}

// 使用示例
const config = new ConfigManager();

// 验证配置
try {
  config.validate();
} catch (error) {
  console.error('配置验证失败:', error.message);
  process.exit(1);
}

// 打印配置摘要
config.printSummary();

// 使用配置
console.log('数据库连接:', {
  host: config.get('database.host'),
  port: config.get('database.port'),
  name: config.get('database.name')
});

// 根据环境执行不同逻辑
if (config.isDevelopment()) {
  console.log('开发模式：启用详细日志');
} else if (config.isProduction()) {
  console.log('生产模式：启用性能优化');
}
```

### 性能监控工具

```javascript
const process = require('process');

class PerformanceMonitor {
  constructor() {
    this.metrics = [];
    this.startupTime = Date.now();
    this.cpuBaseline = process.cpuUsage();
  }

  /**
   * 记录指标
   */
  recordMetric(name, value, tags = {}) {
    const metric = {
      timestamp: Date.now(),
      name,
      value,
      tags
    };
    this.metrics.push(metric);

    // 保持最近1000条记录
    if (this.metrics.length > 1000) {
      this.metrics.shift();
    }

    return metric;
  }

  /**
   * 获取内存快照
   */
  getMemorySnapshot() {
    const memory = process.memoryUsage();

    return {
      timestamp: Date.now(),
      rss: memory.rss,                    // 驻留集大小
      heapTotal: memory.heapTotal,        // 堆总量
      heapUsed: memory.heapUsed,          // 堆使用量
      external: memory.external,          // 外部内存
      arrayBuffers: memory.arrayBuffers,  // ArrayBuffer 内存
      heapUsagePercent: (memory.heapUsed / memory.heapTotal * 100).toFixed(2)
    };
  }

  /**
   * 获取CPU使用率
   */
  getCpuUsage() {
    const currentUsage = process.cpuUsage(this.cpuBaseline);

    return {
      timestamp: Date.now(),
      user: currentUsage.user,      // 用户 CPU 时间（微秒）
      system: currentUsage.system,  // 系统 CPU 时间（微秒）
      total: currentUsage.user + currentUsage.system
    };
  }

  /**
   * 获取进程信息
   */
  getProcessInfo() {
    return {
      timestamp: Date.now(),
      pid: process.pid,
      ppid: process.ppid,
      uptime: process.uptime(),
      uptimeFormatted: this.formatUptime(process.uptime()),
      platform: process.platform,
      arch: process.arch,
      nodeVersion: process.version,
      execPath: process.execPath,
      execArgv: process.execArgv,
      argv: process.argv,
      cwd: process.cwd()
    };
  }

  /**
   * 获取事件循环延迟
   */
  getEventLoopDelay() {
    return new Promise((resolve) => {
      const start = process.hrtime.bigint();

      setImmediate(() => {
        const end = process.hrtime.bigint();
        const delay = Number(end - start) / 1000000; // 转换为毫秒
        resolve(delay);
      });
    });
  }

  /**
   * 运行性能测试
   */
  async runPerformanceTest(fn, iterations = 1000) {
    const results = [];

    // 预热
    for (let i = 0; i < 100; i++) {
      await fn();
    }

    // 正式测试
    for (let i = 0; i < iterations; i++) {
      const start = process.hrtime.bigint();
      await fn();
      const end = process.hrtime.bigint();
      const duration = Number(end - start) / 1000000; // 毫秒
      results.push(duration);
    }

    // 计算统计信息
    const sorted = results.sort((a, b) => a - b);
    const sum = results.reduce((a, b) => a + b, 0);
    const avg = sum / results.length;
    const min = sorted[0];
    const max = sorted[sorted.length - 1];
    const median = sorted[Math.floor(sorted.length / 2)];
    const p95 = sorted[Math.floor(sorted.length * 0.95)];
    const p99 = sorted[Math.floor(sorted.length * 0.99)];

    return {
      iterations,
      avg: avg.toFixed(3),
      min: min.toFixed(3),
      max: max.toFixed(3),
      median: median.toFixed(3),
      p95: p95.toFixed(3),
      p99: p99.toFixed(3)
    };
  }

  /**
   * 内存泄漏检测
   */
  async detectMemoryLeaks(checkInterval = 5000, checks = 5) {
    console.log('开始内存泄漏检测...');

    const snapshots = [];

    for (let i = 0; i < checks; i++) {
      const snapshot = this.getMemorySnapshot();
      snapshots.push(snapshot);
      console.log(`检查 ${i + 1}/${checks}:`, {
        heapUsed: this.formatBytes(snapshot.heapUsed),
        heapTotal: this.formatBytes(snapshot.heapTotal),
        heapUsagePercent: snapshot.heapUsagePercent + '%'
      });

      await new Promise(resolve => setTimeout(resolve, checkInterval));
    }

    // 分析趋势
    const firstSnapshot = snapshots[0];
    const lastSnapshot = snapshots[snapshots.length - 1];
    const growth = lastSnapshot.heapUsed - firstSnapshot.heapUsed;
    const growthPercent = ((growth / firstSnapshot.heapUsed) * 100).toFixed(2);

    console.log('\n内存分析结果:');
    console.log(`初始堆使用: ${this.formatBytes(firstSnapshot.heapUsed)}`);
    console.log(`最终堆使用: ${this.formatBytes(lastSnapshot.heapUsed)}`);
    console.log(`增长量: ${this.formatBytes(growth)} (${growthPercent}%)`);

    if (growth > 10 * 1024 * 1024) { // 增长超过 10MB
      console.warn('⚠️  检测到可能的内存泄漏！');
      return true;
    } else {
      console.log('✅ 未检测到内存泄漏');
      return false;
    }
  }

  /**
   * 生成性能报告
   */
  generateReport() {
    const report = {
      timestamp: Date.now(),
      startupTime: Date.now() - this.startupTime,
      process: this.getProcessInfo(),
      memory: this.getMemorySnapshot(),
      cpu: this.getCpuUsage(),
      metrics: this.metrics.slice(-10) // 最近10条指标
    };

    return report;
  }

  /**
   * 格式化运行时间
   */
  formatUptime(seconds) {
    const days = Math.floor(seconds / 86400);
    const hours = Math.floor((seconds % 86400) / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    const secs = Math.floor(seconds % 60);

    const parts = [];
    if (days > 0) parts.push(`${days}天`);
    if (hours > 0) parts.push(`${hours}小时`);
    if (minutes > 0) parts.push(`${minutes}分钟`);
    if (secs > 0 || parts.length === 0) parts.push(`${secs}秒`);

    return parts.join(' ');
  }

  /**
   * 格式化字节数
   */
  formatBytes(bytes) {
    if (bytes === 0) return '0 Bytes';
    const k = 1024;
    const sizes = ['Bytes', 'KB', 'MB', 'GB'];
    const i = Math.floor(Math.log(bytes) / Math.log(k));
    return Math.round(bytes / Math.pow(k, i) * 100) / 100 + ' ' + sizes[i];
  }
}

// 使用示例
const monitor = new PerformanceMonitor();

// 记录自定义指标
monitor.recordMetric('request.count', 1, { endpoint: '/api/users' });
monitor.recordMetric('request.duration', 123, { endpoint: '/api/users' });

// 获取内存快照
const memorySnapshot = monitor.getMemorySnapshot();
console.log('内存快照:', memorySnapshot);

// 获取进程信息
const processInfo = monitor.getProcessInfo();
console.log('进程信息:', processInfo);

// 运行性能测试
async function testFunction() {
  // 模拟一些工作
  let result = 0;
  for (let i = 0; i < 1000; i++) {
    result += Math.sqrt(i);
  }
  return result;
}

const performanceResults = await monitor.runPerformanceTest(testFunction);
console.log('性能测试结果:', performanceResults);

// 生成报告
const report = monitor.generateReport();
console.log('性能报告:', report);

// 检测内存泄漏
// await monitor.detectMemoryLeaks(2000, 3);
```

### 文件处理工具

```javascript
const fs = require('fs');
const path = require('path');
const process = require('process');

class FileProcessor {
  constructor(options = {}) {
    this.concurrency = options.concurrency || 5;
    this.retryAttempts = options.retryAttempts || 3;
    this.retryDelay = options.retryDelay || 1000;
  }

  /**
   * 批量处理文件
   */
  async processFiles(directory, processor, options = {}) {
    const {
      recursive = false,
      pattern = '*',
      maxFiles = Infinity
    } = options;

    const files = await this.findFiles(directory, pattern, recursive);
    const limitedFiles = files.slice(0, maxFiles);

    console.log(`找到 ${files.length} 个文件，处理 ${limitedFiles.length} 个`);

    const results = await this.processWithConcurrency(limitedFiles, async (filePath) => {
      return await this.withRetry(() => processor(filePath));
    });

    return results;
  }

  /**
   * 查找文件
   */
  async findFiles(directory, pattern = '*', recursive = false) {
    const files = [];

    async function search(dir) {
      const entries = await fs.promises.readdir(dir, { withFileTypes: true });

      for (const entry of entries) {
        const fullPath = path.join(dir, entry.name);

        if (entry.isDirectory() && recursive) {
          await search(fullPath);
        } else if (entry.isFile() && this.matchesPattern(entry.name, pattern)) {
          files.push(fullPath);
        }
      }
    }

    await search(directory);
    return files;
  }

  /**
   * 匹配文件模式
   */
  matchesPattern(filename, pattern) {
    if (pattern === '*') return true;

    const regex = new RegExp(
      '^' + pattern.replace(/\*/g, '.*').replace(/\?/g, '.') + '$'
    );

    return regex.test(filename);
  }

  /**
   * 并发处理
   */
  async processWithConcurrency(items, processor) {
    const results = [];
    const executing = [];

    for (const item of items) {
      const promise = processor(item).then(result => {
        executing.splice(executing.indexOf(promise), 1);
        return result;
      });

      results.push(promise);
      executing.push(promise);

      if (executing.length >= this.concurrency) {
        await Promise.race(executing);
      }
    }

    return Promise.all(results);
  }

  /**
   * 带重试的操作
   */
  async withRetry(fn) {
    let lastError;

    for (let attempt = 1; attempt <= this.retryAttempts; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;
        console.warn(`尝试 ${attempt}/${this.retryAttempts} 失败: ${error.message}`);

        if (attempt < this.retryAttempts) {
          await new Promise(resolve => setTimeout(resolve, this.retryDelay));
        }
      }
    }

    throw lastError;
  }

  /**
   * 监控文件大小
   */
  async watchFileSize(filePath, callback, interval = 1000) {
    let previousSize = 0;

    const checkSize = async () => {
      try {
        const stats = await fs.promises.stat(filePath);
        const currentSize = stats.size;

        if (currentSize !== previousSize) {
          callback({
            filePath,
            previousSize,
            currentSize,
            change: currentSize - previousSize
          });
          previousSize = currentSize;
        }
      } catch (error) {
        console.error('检查文件大小失败:', error);
      }
    };

    const intervalId = setInterval(checkSize, interval);

    // 返回停止函数
    return () => clearInterval(intervalId);
  }

  /**
   * 流式处理大文件
   */
  async processLargeFile(filePath, processor, chunkSize = 1024 * 1024) {
    const stats = await fs.promises.stat(filePath);
    const fileSize = stats.size;
    let processedBytes = 0;

    return new Promise((resolve, reject) => {
      const stream = fs.createReadStream(filePath, { highWaterMark: chunkSize });

      stream.on('data', async (chunk) => {
        try {
          await processor(chunk, processedBytes, fileSize);
          processedBytes += chunk.length;

          const progress = (processedBytes / fileSize * 100).toFixed(2);
          process.stdout.write(`\r处理进度: ${progress}%`);
        } catch (error) {
          stream.destroy();
          reject(error);
        }
      });

      stream.on('end', () => {
        process.stdout.write('\r处理完成!\n');
        resolve({ processedBytes, fileSize });
      });

      stream.on('error', reject);
    });
  }
}

// 使用示例
const processor = new FileProcessor({
  concurrency: 3,
  retryAttempts: 2
});

// 批量处理文件
const results = await processor.processFiles(
  './documents',
  async (filePath) => {
    console.log(`处理文件: ${path.basename(filePath)}`);
    const content = await fs.promises.readFile(filePath, 'utf8');
    // 处理文件内容
    return { filePath, size: content.length };
  },
  {
    recursive: true,
    pattern: '*.txt',
    maxFiles: 100
  }
);

console.log(`处理完成，共 ${results.length} 个文件`);

// 监控文件大小
const stopWatching = await processor.watchFileSize(
  './growing-file.txt',
  (info) => {
    console.log(`文件大小变化: ${info.change} 字节 (总计: ${info.currentSize} 字节)`);
  }
);

// 5秒后停止监听
setTimeout(() => {
  stopWatching();
  console.log('停止监听文件大小');
}, 5000);

// 流式处理大文件
await processor.processLargeFile(
  './large-file.txt',
  async (chunk, offset, totalSize) => {
    // 处理数据块
    console.log(`处理块: ${offset}-${offset + chunk.length}/${totalSize}`);
  },
  1024 * 1024 // 1MB 块大小
);
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