---
title: Node.js 入门介绍
published: 2023-01-01
description: 'Node.js 的特点和安装的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 是一个基于 Chrome V8 引擎的 JavaScript 运行时环境，使 JavaScript 能够在服务器端运行。它由 Ryan Dahl 于 2009 年创建，采用事件驱动、非阻塞 I/O 模型，特别适合构建高并发、可扩展的网络应用。

## 核心概念

理解核心概念是掌握任何技术的基础。

### 事件驱动

Node.js 采用事件驱动架构，所有 I/O 操作都是异步的，不会阻塞主线程。当 I/O 操作完成时，会触发相应的事件回调。

### 单线程

Node.js 使用单线程处理请求，通过事件循环机制实现高并发。虽然主线程是单线程的，但底层通过 libuv 实现了线程池来处理 CPU 密集型任务。

### 非阻塞 I/O

所有 I/O 操作（文件读写、网络请求等）都是非阻塞的，使用回调函数或 Promise 处理结果。

### NPM 生态系统

Node.js 自带 npm (Node Package Manager)，是世界上最大的开源库生态系统，拥有超过 150 万个包。

## 基本用法

### 安装 Node.js

```bash
# 官网下载安装
# 访问 https://nodejs.org 下载安装包

# 使用 nvm 安装（推荐）
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
nvm install 20
nvm use 20

# 验证安装
node -v
npm -v
```

### Hello World

```javascript
// 创建 httpServer.js
const http = require('http');

const server = http.createServer((req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('Hello World\n');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});

// 运行
// node httpServer.js
```

### 使用 ES Modules

```javascript
// 使用 import/export 语法（需要 package.json 中设置 "type": "module"）
import http from 'http';

const server = http.createServer((req, res) => {
  res.end('Hello from ES Modules');
});

server.listen(3000);
```

### REPL 交互式环境

```bash
# 启动 REPL
node

# 在 REPL 中
> 1 + 1
2
> const greet = (name) => `Hello, ${name}!`
undefined
> greet('World')
'Hello, World!'
> .exit  # 退出
```

## 实际应用

在实际项目中，我们需要根据具体场景灵活运用。

### 适合使用 Node.js 的场景

- **实时应用**：聊天应用、在线游戏
- **RESTful API 服务**：后端 API 开发
- **单页面应用**：React/Vue 应用的 SSR
- **工具链**：构建工具、代码生成器
- **微服务架构**：轻量级服务组件

### 典型应用案例

- Netflix（流媒体服务）
- PayPal（支付平台）
- LinkedIn（社交网络）
- Uber（出行服务）

### 创建完整的 Web 应用

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');

// MIME 类型映射
const mimeTypes = {
  '.html': 'text/html',
  '.js': 'text/javascript',
  '.css': 'text/css',
  '.json': 'application/json',
  '.png': 'image/png',
  '.jpg': 'image/jpg',
  '.gif': 'image/gif',
  '.svg': 'image/svg+xml',
  '.ico': 'image/x-icon'
};

const server = http.createServer((req, res) => {
  console.log(`${req.method} ${req.url}`);

  // 处理 API 请求
  if (req.url.startsWith('/api/')) {
    handleAPI(req, res);
    return;
  }

  // 处理静态文件
  let filePath = '.' + req.url;
  if (filePath === './') {
    filePath = './index.html';
  }

  const extname = path.extname(filePath);
  const contentType = mimeTypes[extname] || 'application/octet-stream';

  fs.readFile(filePath, (err, content) => {
    if (err) {
      if (err.code === 'ENOENT') {
        fs.readFile('./404.html', (err, content) => {
          res.writeHead(404, { 'Content-Type': 'text/html' });
          res.end(content, 'utf8');
        });
      } else {
        res.writeHead(500);
        res.end(`Server Error: ${err.code}`);
      }
    } else {
      res.writeHead(200, { 'Content-Type': contentType });
      res.end(content, 'utf8');
    }
  });
});

function handleAPI(req, res) {
  // 设置 CORS 头
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');

  if (req.method === 'OPTIONS') {
    res.writeHead(200);
    res.end();
    return;
  }

  // 路由处理
  const url = new URL(req.url, `http://${req.headers.host}`);
  const pathname = url.pathname;

  if (pathname === '/api/users' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify([
      { id: 1, name: 'Alice' },
      { id: 2, name: 'Bob' }
    ]));
  } else if (pathname === '/api/users' && req.method === 'POST') {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const user = JSON.parse(body);
      res.writeHead(201, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ ...user, id: 3 }));
    });
  } else {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
}

const PORT = process.env.PORT || 3000;
server.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}/`);
});
```

### 命令行工具开发

```javascript
#!/usr/bin/env node

const fs = require('fs');
const path = require('path');
const readline = require('readline');

// 创建命令行界面
const rl = readline.createInterface({
  input: process.stdin,
  output: process.stdout
});

// 命令参数解析
const args = process.argv.slice(2);
const command = args[0];

switch (command) {
  case 'init':
    initProject();
    break;
  case 'create':
    createComponent(args[1]);
    break;
  case 'serve':
    startServer(args[1]);
    break;
  default:
    showHelp();
}

function initProject() {
  console.log('初始化项目...');

  const projectStructure = {
    'src': {
      'components': {},
      'styles': {},
      'utils': {}
    },
    'public': {
      'index.html': '<!DOCTYPE html>\n<html>\n<head>\n  <title>My App</title>\n</head>\n<body>\n  <div id="app"></div>\n</body>\n</html>'
    },
    'package.json': JSON.stringify({
      name: 'my-app',
      version: '1.0.0',
      scripts: {
        start: 'node server.js',
        dev: 'nodemon server.js'
      }
    }, null, 2),
    'server.js': `const http = require('http');
const fs = require('fs');
const path = require('path');

const server = http.createServer((req, res) => {
  let filePath = path.join(__dirname, 'public', req.url === '/' ? 'index.html' : req.url);

  fs.readFile(filePath, (err, content) => {
    if (err) {
      res.writeHead(404);
      res.end('File not found');
    } else {
      res.writeHead(200);
      res.end(content);
    }
  });
});

server.listen(3000, () => console.log('Server running on port 3000'));
`
  };

  function createStructure(base, structure) {
    Object.keys(structure).forEach(key => {
      const item = structure[key];
      const itemPath = path.join(base, key);

      if (typeof item === 'object' && !Buffer.isBuffer(item)) {
        fs.mkdirSync(itemPath, { recursive: true });
        createStructure(itemPath, item);
      } else {
        fs.writeFileSync(itemPath, item);
      }
    });
  }

  createStructure('.', projectStructure);
  console.log('项目初始化完成！');
}

function createComponent(name) {
  if (!name) {
    console.error('请提供组件名称');
    process.exit(1);
  }

  const componentPath = `src/components/${name}.js`;
  const componentContent = `class ${name} {
  constructor() {
    this.element = document.createElement('div');
    this.element.className = '${name.toLowerCase()}';
  }

  render() {
    this.element.innerHTML = \`
      <h1>${name} Component</h1>
    \`;
    return this.element;
  }
}

export default ${name};
`;

  fs.writeFileSync(componentPath, componentContent);
  console.log(`组件 ${name} 已创建: ${componentPath}`);
}

function startServer(port = 3000) {
  console.log(`启动开发服务器，端口: ${port}`);
  // 这里可以集成开发服务器逻辑
}

function showHelp() {
  console.log(`
使用方法:
  node app.js init          - 初始化项目
  node app.js create <name> - 创建组件
  node app.js serve [port]  - 启动服务器
  `);
}
```

### 文件处理工具

```javascript
const fs = require('fs').promises;
const path = require('path');

class FileProcessor {
  constructor(inputDir, outputDir) {
    this.inputDir = inputDir;
    this.outputDir = outputDir;
  }

  async processAll() {
    try {
      await fs.mkdir(this.outputDir, { recursive: true });
      const files = await fs.readdir(this.inputDir);

      for (const file of files) {
        const inputPath = path.join(this.inputDir, file);
        const outputPath = path.join(this.outputDir, file);

        const stats = await fs.stat(inputPath);

        if (stats.isFile() && file.endsWith('.txt')) {
          await this.processFile(inputPath, outputPath);
        }
      }

      console.log('所有文件处理完成');
    } catch (error) {
      console.error('处理失败:', error);
    }
  }

  async processFile(inputPath, outputPath) {
    try {
      const content = await fs.readFile(inputPath, 'utf8');
      const processed = this.transformContent(content);
      await fs.writeFile(outputPath, processed);
      console.log(`已处理: ${path.basename(inputPath)}`);
    } catch (error) {
      console.error(`处理文件失败 ${inputPath}:`, error);
    }
  }

  transformContent(content) {
    // 示例：转换大写
    return content.toUpperCase();
  }
}

// 使用示例
const processor = new FileProcessor('./input', './output');
processor.processAll();
```

### 数据处理管道

```javascript
const { Transform, pipeline } = require('stream');
const fs = require('fs');

// 创建转换流
class DataTransform extends Transform {
  constructor(options) {
    super(options);
    this.count = 0;
  }

  _transform(chunk, encoding, callback) {
    try {
      const data = JSON.parse(chunk.toString());
      this.count++;

      // 数据转换
      const transformed = {
        id: this.count,
        timestamp: new Date().toISOString(),
        data: data
      };

      this.push(JSON.stringify(transformed) + '\n');
      callback();
    } catch (error) {
      callback(error);
    }
  }

  _flush(callback) {
    console.log(`总共处理了 ${this.count} 条记录`);
    callback();
  }
}

// 使用管道处理数据
const readStream = fs.createReadStream('input.jsonl');
const writeStream = fs.createWriteStream('output.jsonl');
const transform = new DataTransform();

pipeline(
  readStream,
  transform,
  writeStream,
  (err) => {
    if (err) {
      console.error('管道处理失败:', err);
    } else {
      console.log('数据处理完成');
    }
  }
);
```

## 注意事项

1. 注意边界情况
   - 避免在单线程中执行 CPU 密集型任务
   - 合理使用 Worker Threads 处理计算密集型操作

2. 考虑性能影响
   - 避免内存泄漏（及时清理事件监听器）
   - 使用缓存优化重复计算
   - 合理设置集群模式利用多核 CPU

3. 遵循最佳实践
   - 使用异步编程避免阻塞
   - 做好错误处理和日志记录
   - 使用 process.env 管理环境变量
   - 遵守 Semantic Versioning 版本管理

## 总结

通过本文的学习，相信你已经对 Node.js 有了更深入的理解。Node.js 是一个强大的 JavaScript 运行时环境，特别适合构建高性能、可扩展的后端应用。掌握事件驱动、非阻塞 I/O 等核心概念，结合丰富的 npm 生态系统，能够极大地提升开发效率。