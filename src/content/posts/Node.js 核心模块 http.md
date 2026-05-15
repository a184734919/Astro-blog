---
title: Node.js 核心模块 http
published: 2023-01-15
description: 'HTTP 服务器和客户端的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `http` 模块提供了创建 HTTP 服务器和客户端的能力。它是 Node.js 最核心的模块之一，使得使用 JavaScript 构建网络应用变得简单高效。无论是简单的 API 服务还是复杂的 Web 应用，`http` 模块都能胜任。

## 核心概念

### HTTP 请求/响应模型
- 请求（Request）：客户端发送给服务器的数据，包含方法、URL、头部和主体
- 响应（Response）：服务器返回给客户端的数据，包含状态码、头部和主体

### 事件驱动
`http` 模块基于事件驱动架构，通过监听事件来处理请求和响应。

### 流式处理
请求和响应体是可读可写的流，支持分块传输，适合处理大文件。

## 基本用法

### 创建简单的 HTTP 服务器
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello, World!');
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});
```

### 路由处理
```javascript
const server = http.createServer((req, res) => {
  if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end('<h1>Home Page</h1>');
  } else if (req.url === '/api' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ message: 'API Response' }));
  } else {
    res.writeHead(404, { 'Content-Type': 'text/plain' });
    res.end('Not Found');
  }
});
```

### 发送 HTTP 请求（客户端）
```javascript
const http = require('http');

const options = {
  hostname: 'example.com',
  port: 80,
  path: '/api/data',
  method: 'GET',
};

const req = http.request(options, (res) => {
  let data = '';

  res.on('data', (chunk) => {
    data += chunk;
  });

  res.on('end', () => {
    console.log(JSON.parse(data));
  });
});

req.on('error', (error) => {
  console.error(`Request error: ${error.message}`);
});

req.end();
```

### 处理 POST 请求
```javascript
const server = http.createServer((req, res) => {
  if (req.url === '/submit' && req.method === 'POST') {
    let body = '';

    req.on('data', (chunk) => {
      body += chunk.toString();
    });

    req.on('end', () => {
      try {
        const data = JSON.parse(body);
        console.log('Received data:', data);

        res.writeHead(200, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ success: true }));
      } catch (error) {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Invalid JSON' }));
      }
    });
  }
});
```

## 实际应用

### 静态文件服务器

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');
const { promisify } = require('util');

const readFile = promisify(fs.readFile);

// MIME 类型映射
const mimeTypes = {
  '.html': 'text/html',
  '.css': 'text/css',
  '.js': 'application/javascript',
  '.json': 'application/json',
  '.png': 'image/png',
  '.jpg': 'image/jpg',
  '.gif': 'image/gif',
  '.svg': 'image/svg+xml',
  '.ico': 'image/x-icon',
  '.woff': 'font/woff',
  '.woff2': 'font/woff2',
  '.ttf': 'font/ttf',
  '.eot': 'application/vnd.ms-fontobject',
  '.mp4': 'video/mp4',
  '.webm': 'video/webm',
  '.mp3': 'audio/mpeg',
  '.wav': 'audio/wav',
  '.ogg': 'audio/ogg',
  '.pdf': 'application/pdf',
  '.zip': 'application/zip',
  '.tar': 'application/x-tar',
  '.gz': 'application/gzip'
};

const server = http.createServer(async (req, res) => {
  // 安全检查：防止路径遍历攻击
  if (req.url.includes('..')) {
    res.writeHead(403, { 'Content-Type': 'text/plain' });
    res.end('Forbidden');
    return;
  }

  let filePath = path.join(__dirname, 'public', req.url === '/' ? 'index.html' : req.url);

  try {
    const extname = path.extname(filePath);
    const contentType = mimeTypes[extname] || 'application/octet-stream';

    // 启用 Gzip 压缩
    const acceptEncoding = req.headers['accept-encoding'];
    if (acceptEncoding && acceptEncoding.includes('gzip')) {
      const zlib = require('zlib');
      const raw = await readFile(filePath);
      const compressed = zlib.gzipSync(raw);
      res.writeHead(200, {
        'Content-Type': contentType,
        'Content-Encoding': 'gzip',
        'Cache-Control': 'public, max-age=3600'
      });
      res.end(compressed);
    } else {
      const content = await readFile(filePath);
      res.writeHead(200, {
        'Content-Type': contentType,
        'Cache-Control': 'public, max-age=3600'
      });
      res.end(content);
    }
  } catch (err) {
    if (err.code === 'ENOENT') {
      // 自定义 404 页面
      try {
        const notFoundPage = await readFile(path.join(__dirname, 'public', '404.html'));
        res.writeHead(404, { 'Content-Type': 'text/html' });
        res.end(notFoundPage);
      } catch {
        res.writeHead(404, { 'Content-Type': 'text/plain' });
        res.end('404 Not Found');
      }
    } else {
      res.writeHead(500, { 'Content-Type': 'text/plain' });
      res.end(`Server Error: ${err.message}`);
    }
  }
});

server.listen(8080, () => {
  console.log('静态文件服务器运行在 http://localhost:8080');
});
```

### RESTful API 服务器

```javascript
const http = require('http');
const { URL } = require('url');
const { promisify } = require('util');

// 内存数据库
const database = {
  users: [
    { id: 1, name: 'Alice', email: 'alice@example.com', age: 25 },
    { id: 2, name: 'Bob', email: 'bob@example.com', age: 30 }
  ],
  posts: [
    { id: 1, userId: 1, title: 'First Post', content: 'Hello World' }
  ]
};

// 请求体解析中间件
function parseBody(req) {
  return new Promise((resolve, reject) => {
    let body = '';

    req.on('data', (chunk) => {
      body += chunk.toString();
    });

    req.on('end', () => {
      try {
        resolve(body ? JSON.parse(body) : {});
      } catch (error) {
        reject(error);
      }
    });

    req.on('error', reject);
  });
}

// CORS 中间件
function addCorsHeaders(res) {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
}

// 验证中间件
function validateUser(user) {
  const errors = [];

  if (!user.name || user.name.trim().length === 0) {
    errors.push('Name is required');
  }

  if (!user.email || !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(user.email)) {
    errors.push('Valid email is required');
  }

  if (user.age && (user.age < 0 || user.age > 150)) {
    errors.push('Age must be between 0 and 150');
  }

  return errors;
}

// 路由处理
const routes = {
  'GET /api/users': getUsers,
  'GET /api/users/:id': getUser,
  'POST /api/users': createUser,
  'PUT /api/users/:id': updateUser,
  'DELETE /api/users/:id': deleteUser,

  'GET /api/posts': getPosts,
  'GET /api/posts/:id': getPost,
  'POST /api/posts': createPost,

  'GET /api/users/:id/posts': getUserPosts
};

async function getUsers(req, res) {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const page = parseInt(url.searchParams.get('page')) || 1;
  const limit = parseInt(url.searchParams.get('limit')) || 10;
  const search = url.searchParams.get('search');

  let users = [...database.users];

  // 搜索过滤
  if (search) {
    users = users.filter(user =>
      user.name.toLowerCase().includes(search.toLowerCase()) ||
      user.email.toLowerCase().includes(search.toLowerCase())
    );
  }

  // 分页
  const startIndex = (page - 1) * limit;
  const endIndex = startIndex + limit;
  const paginatedUsers = users.slice(startIndex, endIndex);

  res.writeHead(200, {
    'Content-Type': 'application/json',
    'X-Total-Count': users.length.toString(),
    'X-Page-Count': Math.ceil(users.length / limit).toString()
  });
  res.end(JSON.stringify({
    success: true,
    data: paginatedUsers,
    pagination: {
      page,
      limit,
      total: users.length,
      pages: Math.ceil(users.length / limit)
    }
  }));
}

async function getUser(req, res) {
  const id = parseInt(req.url.split('/')[3]);
  const user = database.users.find(u => u.id === id);

  if (!user) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'User not found'
    }));
    return;
  }

  // 获取用户的文章
  const userPosts = database.posts.filter(p => p.userId === id);

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    success: true,
    data: {
      ...user,
      posts: userPosts
    }
  }));
}

async function createUser(req, res) {
  try {
    const body = await parseBody(req);

    // 验证数据
    const errors = validateUser(body);
    if (errors.length > 0) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        errors
      }));
      return;
    }

    // 检查邮箱是否已存在
    const emailExists = database.users.some(u => u.email === body.email);
    if (emailExists) {
      res.writeHead(409, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        error: 'Email already exists'
      }));
      return;
    }

    // 创建用户
    const newUser = {
      id: database.users.length + 1,
      name: body.name,
      email: body.email,
      age: body.age || null,
      createdAt: new Date().toISOString()
    };

    database.users.push(newUser);

    res.writeHead(201, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: true,
      data: newUser
    }));
  } catch (error) {
    res.writeHead(400, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'Invalid JSON'
    }));
  }
}

async function updateUser(req, res) {
  const id = parseInt(req.url.split('/')[3]);
  const userIndex = database.users.findIndex(u => u.id === id);

  if (userIndex === -1) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'User not found'
    }));
    return;
  }

  try {
    const body = await parseBody(req);

    // 验证数据
    const errors = validateUser(body);
    if (errors.length > 0) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        errors
      }));
      return;
    }

    // 更新用户
    database.users[userIndex] = {
      ...database.users[userIndex],
      ...body,
      updatedAt: new Date().toISOString()
    };

    res.writeHead(200, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: true,
      data: database.users[userIndex]
    }));
  } catch (error) {
    res.writeHead(400, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'Invalid JSON'
    }));
  }
}

async function deleteUser(req, res) {
  const id = parseInt(req.url.split('/')[3]);
  const userIndex = database.users.findIndex(u => u.id === id);

  if (userIndex === -1) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'User not found'
    }));
    return;
  }

  // 删除用户和相关文章
  database.users.splice(userIndex, 1);
  database.posts = database.posts.filter(p => p.userId !== id);

  res.writeHead(204); // No Content
  res.end();
}

async function getPosts(req, res) {
  const url = new URL(req.url, `http://${req.headers.host}`);
  const userId = url.searchParams.get('userId');

  let posts = [...database.posts];

  if (userId) {
    posts = posts.filter(p => p.userId === parseInt(userId));
  }

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    success: true,
    data: posts
  }));
}

async function getPost(req, res) {
  const id = parseInt(req.url.split('/')[3]);
  const post = database.posts.find(p => p.id === id);

  if (!post) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'Post not found'
    }));
    return;
  }

  // 获取作者信息
  const author = database.users.find(u => u.id === post.userId);

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    success: true,
    data: {
      ...post,
      author: {
        id: author.id,
        name: author.name,
        email: author.email
      }
    }
  }));
}

async function createPost(req, res) {
  try {
    const body = await parseBody(req);

    if (!body.title || !body.content || !body.userId) {
      res.writeHead(400, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        error: 'Title, content, and userId are required'
      }));
      return;
    }

    // 验证用户是否存在
    const userExists = database.users.some(u => u.id === body.userId);
    if (!userExists) {
      res.writeHead(404, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        error: 'User not found'
      }));
      return;
    }

    const newPost = {
      id: database.posts.length + 1,
      userId: body.userId,
      title: body.title,
      content: body.content,
      createdAt: new Date().toISOString()
    };

    database.posts.push(newPost);

    res.writeHead(201, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: true,
      data: newPost
    }));
  } catch (error) {
    res.writeHead(400, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'Invalid JSON'
    }));
  }
}

async function getUserPosts(req, res) {
  const id = parseInt(req.url.split('/')[3]);
  const user = database.users.find(u => u.id === id);

  if (!user) {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'User not found'
    }));
    return;
  }

  const userPosts = database.posts.filter(p => p.userId === id);

  res.writeHead(200, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify({
    success: true,
    data: {
      user: {
        id: user.id,
        name: user.name,
        email: user.email
      },
      posts: userPosts
    }
  }));
}

// 主服务器
const server = http.createServer(async (req, res) => {
  addCorsHeaders(res);

  // 处理 OPTIONS 请求
  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    res.end();
    return;
  }

  const url = new URL(req.url, `http://${req.headers.host}`);
  const pathname = url.pathname;
  const method = req.method;
  const routeKey = `${method} ${pathname}`;

  // 查找路由
  let handler = null;
  let params = {};

  // 精确匹配
  if (routes[routeKey]) {
    handler = routes[routeKey];
  } else {
    // 参数化路由匹配
    for (const route in routes) {
      const [routeMethod, routePath] = route.split(' ');
      if (routeMethod !== method) continue;

      const routeParts = routePath.split('/');
      const pathParts = pathname.split('/');

      if (routeParts.length !== pathParts.length) continue;

      let match = true;
      for (let i = 0; i < routeParts.length; i++) {
        if (routeParts[i].startsWith(':')) {
          params[routeParts[i].slice(1)] = pathParts[i];
        } else if (routeParts[i] !== pathParts[i]) {
          match = false;
          break;
        }
      }

      if (match) {
        handler = routes[route];
        break;
      }
    }
  }

  if (handler) {
    try {
      req.params = params;
      await handler(req, res);
    } catch (error) {
      console.error('Handler error:', error);
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: false,
        error: 'Internal server error'
      }));
    }
  } else {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({
      success: false,
      error: 'Route not found'
    }));
  }
});

server.listen(3000, () => {
  console.log('RESTful API 服务器运行在 http://localhost:3000');
  console.log('可用端点:');
  console.log('  GET    /api/users');
  console.log('  GET    /api/users/:id');
  console.log('  POST   /api/users');
  console.log('  PUT    /api/users/:id');
  console.log('  DELETE /api/users/:id');
  console.log('  GET    /api/posts');
  console.log('  GET    /api/posts/:id');
  console.log('  POST   /api/posts');
  console.log('  GET    /api/users/:id/posts');
});
```

### 文件上传处理

```javascript
const http = require('http');
const fs = require('fs');
const path = require('path');
const { promisify } = require('util');

const mkdir = promisify(fs.mkdir);
const writeFile = promisify(fs.writeFile);
const access = promisify(fs.access);

// 上传目录
const UPLOAD_DIR = path.join(__dirname, 'uploads');

// 确保上传目录存在
async function ensureUploadDir() {
  try {
    await access(UPLOAD_DIR);
  } catch {
    await mkdir(UPLOAD_DIR, { recursive: true });
  }
}

// 解析 multipart/form-data
function parseMultipartForm(req) {
  return new Promise((resolve, reject) => {
    const chunks = [];
    const boundary = req.headers['content-type'].split('boundary=')[1];

    req.on('data', (chunk) => chunks.push(chunk));

    req.on('end', () => {
      try {
        const buffer = Buffer.concat(chunks);
        const parts = [];

        let start = 0;
        while (true) {
          const boundaryIndex = buffer.indexOf(`--${boundary}`, start);
          if (boundaryIndex === -1) break;

          const nextBoundaryIndex = buffer.indexOf(`--${boundary}`, boundaryIndex + boundary.length + 2);
          if (nextBoundaryIndex === -1) break;

          const partData = buffer.slice(boundaryIndex + boundary.length + 2, nextBoundaryIndex - 2);
          const headerEnd = partData.indexOf('\r\n\r\n');

          if (headerEnd !== -1) {
            const headers = partData.slice(0, headerEnd).toString();
            const content = partData.slice(headerEnd + 4);

            const nameMatch = headers.match(/name="([^"]+)"/);
            const filenameMatch = headers.match(/filename="([^"]+)"/);
            const contentTypeMatch = headers.match(/Content-Type:\s*([^\r\n]+)/);

            parts.push({
              name: nameMatch ? nameMatch[1] : null,
              filename: filenameMatch ? filenameMatch[1] : null,
              contentType: contentTypeMatch ? contentTypeMatch[1].trim() : null,
              data: content
            });
          }

          start = nextBoundaryIndex;
        }

        resolve(parts);
      } catch (error) {
        reject(error);
      }
    });

    req.on('error', reject);
  });
}

// 生成唯一文件名
function generateFilename(originalName) {
  const timestamp = Date.now();
  const random = Math.random().toString(36).substring(2, 8);
  const ext = path.extname(originalName);
  const baseName = path.basename(originalName, ext);
  return `${baseName}-${timestamp}-${random}${ext}`;
}

// 文件类型验证
const ALLOWED_TYPES = [
  'image/jpeg',
  'image/png',
  'image/gif',
  'application/pdf',
  'text/plain'
];

function validateFileType(contentType) {
  return ALLOWED_TYPES.includes(contentType);
}

const server = http.createServer(async (req, res) => {
  // CORS
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS');

  if (req.method === 'OPTIONS') {
    res.writeHead(204);
    res.end();
    return;
  }

  // 文件上传处理
  if (req.url === '/upload' && req.method === 'POST') {
    try {
      await ensureUploadDir();

      if (!req.headers['content-type']?.includes('multipart/form-data')) {
        res.writeHead(400, { 'Content-Type': 'application/json' });
        res.end(JSON.stringify({ error: 'Content-Type must be multipart/form-data' }));
        return;
      }

      const parts = await parseMultipartForm(req);
      const uploadedFiles = [];

      for (const part of parts) {
        if (part.filename) {
          // 验证文件类型
          if (!validateFileType(part.contentType)) {
            res.writeHead(400, { 'Content-Type': 'application/json' });
            res.end(JSON.stringify({
              error: `File type ${part.contentType} not allowed`,
              allowedTypes: ALLOWED_TYPES
            }));
            return;
          }

          // 生成文件名并保存
          const filename = generateFilename(part.filename);
          const filepath = path.join(UPLOAD_DIR, filename);

          await writeFile(filepath, part.data);

          uploadedFiles.push({
            originalName: part.filename,
            filename: filename,
            size: part.data.length,
            contentType: part.contentType,
            url: `/files/${filename}`
          });
        }
      }

      res.writeHead(200, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({
        success: true,
        files: uploadedFiles
      }));

    } catch (error) {
      console.error('Upload error:', error);
      res.writeHead(500, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'Upload failed' }));
    }
  }
  // 文件下载
  else if (req.url.startsWith('/files/') && req.method === 'GET') {
    const filename = req.url.replace('/files/', '');
    const filepath = path.join(UPLOAD_DIR, filename);

    try {
      const fileData = await fs.promises.readFile(filepath);

      // 设置内容类型
      const ext = path.extname(filename);
      const contentTypes = {
        '.jpg': 'image/jpeg',
        '.jpeg': 'image/jpeg',
        '.png': 'image/png',
        '.gif': 'image/gif',
        '.pdf': 'application/pdf',
        '.txt': 'text/plain'
      };

      const contentType = contentTypes[ext] || 'application/octet-stream';

      res.writeHead(200, {
        'Content-Type': contentType,
        'Content-Disposition': `attachment; filename="${filename}"`,
        'Content-Length': fileData.length
      });
      res.end(fileData);

    } catch (error) {
      res.writeHead(404, { 'Content-Type': 'application/json' });
      res.end(JSON.stringify({ error: 'File not found' }));
    }
  }
  // 上传页面
  else if (req.url === '/' && req.method === 'GET') {
    res.writeHead(200, { 'Content-Type': 'text/html' });
    res.end(`
      <!DOCTYPE html>
      <html>
      <head>
        <title>文件上传</title>
        <style>
          body { font-family: Arial, sans-serif; max-width: 600px; margin: 50px auto; padding: 20px; }
          h1 { color: #333; }
          .upload-form { border: 2px dashed #ccc; padding: 20px; border-radius: 5px; }
          input[type="file"] { margin: 10px 0; }
          button { background: #007bff; color: white; border: none; padding: 10px 20px; border-radius: 5px; cursor: pointer; }
          button:hover { background: #0056b3; }
          #result { margin-top: 20px; }
          .file-item { padding: 10px; margin: 5px 0; background: #f8f9fa; border-radius: 3px; }
        </style>
      </head>
      <body>
        <h1>文件上传</h1>
        <div class="upload-form">
          <form id="uploadForm">
            <input type="file" id="fileInput" multiple>
            <button type="submit">上传</button>
          </form>
        </div>
        <div id="result"></div>

        <script>
          document.getElementById('uploadForm').addEventListener('submit', async (e) => {
            e.preventDefault();
            const fileInput = document.getElementById('fileInput');
            const formData = new FormData();

            for (const file of fileInput.files) {
              formData.append('files', file);
            }

            try {
              const response = await fetch('/upload', {
                method: 'POST',
                body: formData
              });

              const result = await response.json();

              const resultDiv = document.getElementById('result');
              if (result.success) {
                resultDiv.innerHTML = result.files.map(file => `
                  <div class="file-item">
                    <strong>${file.originalName}</strong><br>
                    大小: ${file.size} 字节<br>
                    <a href="${file.url}" download>下载</a>
                  </div>
                `).join('');
              } else {
                resultDiv.innerHTML = `<p style="color: red;">${result.error}</p>`;
              }
            } catch (error) {
              document.getElementById('result').innerHTML = `<p style="color: red;">上传失败</p>`;
            }
          });
        </script>
      </body>
      </html>
    `);
  }
  else {
    res.writeHead(404, { 'Content-Type': 'application/json' });
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
});

ensureUploadDir().then(() => {
  server.listen(3000, () => {
    console.log('文件服务器运行在 http://localhost:3000');
  });
});
```

### HTTPS 服务器

```javascript
const https = require('https');
const fs = require('fs');
const path = require('path');

// SSL 证书配置（开发环境使用自签名证书）
const options = {
  key: fs.readFileSync(path.join(__dirname, 'certs', 'server.key')),
  cert: fs.readFileSync(path.join(__dirname, 'certs', 'server.cert')),
  ca: fs.readFileSync(path.join(__dirname, 'certs', 'ca.cert')),

  // 安全配置
  minVersion: 'TLSv1.2',
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384',
    'ECDHE-ECDSA-CHACHA20-POLY1305',
    'ECDHE-RSA-CHACHA20-POLY1305'
  ].join(':'),
  honorCipherOrder: true
};

const server = https.createServer(options, (req, res) => {
  res.writeHead(200, { 'Content-Type': 'text/plain' });
  res.end('HTTPS Server running!');
});

server.listen(443, () => {
  console.log('HTTPS 服务器运行在端口 443');
});

// HTTP 到 HTTPS 重定向
const http = require('http');
const httpServer = http.createServer((req, res) => {
  res.writeHead(301, {
    'Location': `https://${req.headers.host}${req.url}`
  });
  res.end();
});

httpServer.listen(80, () => {
  console.log('HTTP 重定向服务器运行在端口 80');
});
```

## 注意事项

1. **安全性**：始终对用户输入进行验证和清理，防止 XSS、注入攻击
2. **错误处理**：妥善处理各种错误情况，避免服务器崩溃
3. **性能优化**：合理使用流式处理，避免将大文件全部读入内存
4. **HTTP 状态码**：正确使用状态码，如 200、404、500 等
5. **头部设置**：正确设置 Content-Type、Content-Length 等响应头
6. **HTTPS**：生产环境建议使用 `https` 模块确保数据传输安全

## 总结

通过本文的学习，相信你已经对 Node.js 核心模块 http 有了更深入的理解。`http` 模块是构建网络应用的基础，掌握它的使用对于 Node.js 开发至关重要。在实际项目中，许多流行的框架如 Express.js 都是基于 `http` 模块构建的，理解底层原理有助于更好地使用这些框架。