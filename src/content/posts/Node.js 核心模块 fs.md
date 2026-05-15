---
title: Node.js 核心模块 fs
published: 2023-01-08
description: '文件系统操作详解的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## Node.js 文件系统模块

fs 模块提供了文件系统的操作接口，包括文件的读写、目录操作等。它是 Node.js 最常用的核心模块之一，提供了同步和异步两种操作方式。

## 基本概念

### 同步 vs 异步

fs 模块中的大多数方法都有同步和异步两个版本：
- 同步方法：以 `Sync` 结尾，会阻塞执行直到操作完成
- 异步方法：使用回调函数或 Promise，不会阻塞执行

### 错误处理

所有的异步操作都可能产生错误，需要通过回调函数的第一个参数或 catch 块来处理。

## 文件读取

```javascript
const fs = require('fs');
const fsPromises = require('fs/promises');

// 同步读取 - 阻塞方式
const data = fs.readFileSync('file.txt', 'utf8');
console.log(data);

// 异步读取 - 回调方式
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('读取失败:', err);
    return;
  }
  console.log(data);
});

// Promise 方式 - async/await
async function readFile() {
  try {
    const data = await fsPromises.readFile('file.txt', 'utf8');
    console.log(data);
  } catch (err) {
    console.error('读取失败:', err);
  }
}

// 读取二进制文件（如图片）
const imageData = fs.readFileSync('image.png');
console.log('图片大小:', imageData.length);

// 创建可读流 - 处理大文件
const readStream = fs.createReadStream('large-file.txt', { encoding: 'utf8' });

readStream.on('data', (chunk) => {
  console.log('读取到数据块:', chunk);
});

readStream.on('end', () => {
  console.log('文件读取完成');
});

readStream.on('error', (err) => {
  console.error('读取错误:', err);
});
```

## 文件写入

```javascript
// 同步写入
fs.writeFileSync('file.txt', 'Hello World');
console.log('文件已写入');

// 异步写入
fs.writeFile('file.txt', 'Hello World', (err) => {
  if (err) {
    console.error('写入失败:', err);
    return;
  }
  console.log('文件已保存');
});

// Promise 方式
async function writeFile() {
  try {
    await fsPromises.writeFile('file.txt', 'Hello World\n第二行');
    console.log('文件已保存');
  } catch (err) {
    console.error('写入失败:', err);
  }
}

// 追加内容
fs.appendFileSync('file.txt', '\nNew line');
fs.appendFile('file.txt', '\nAnother line', (err) => {
  if (err) throw err;
  console.log('内容已追加');
});

// 写入 JSON 对象
const data = { name: 'John', age: 30 };
fs.writeFileSync('data.json', JSON.stringify(data, null, 2));

// 创建可写流
const writeStream = fs.createWriteStream('output.txt');

writeStream.write('第一行\n');
writeStream.write('第二行\n');
writeStream.end();  // 关闭流

// 管道传输 - 从一个文件复制到另一个
const readStream = fs.createReadStream('source.txt');
const writeStream = fs.createWriteStream('destination.txt');
readStream.pipe(writeStream);
```

## 文件信息与操作

```javascript
// 获取文件信息（同步）
const stats = fs.statSync('file.txt');
console.log('文件大小:', stats.size);           // 文件大小（字节）
console.log('创建时间:', stats.birthtime);      // 创建时间
console.log('修改时间:', stats.mtime);          // 修改时间
console.log('访问时间:', stats.atime);          // 访问时间
console.log('是否是文件:', stats.isFile());     // true/false
console.log('是否是目录:', stats.isDirectory()); // true/false

// 获取文件信息（异步）
fs.stat('file.txt', (err, stats) => {
  if (err) throw err;
  console.log(stats);
});

// Promise 方式
async function getFileInfo() {
  const stats = await fsPromises.stat('file.txt');
  console.log(stats);
}

// 检查文件是否存在
fs.access('file.txt', fs.constants.F_OK, (err) => {
  if (err) {
    console.log('文件不存在');
  } else {
    console.log('文件存在');
  }
});

// 检查文件权限
fs.access('file.txt', fs.constants.R_OK, (err) => {
  if (err) {
    console.log('没有读取权限');
  } else {
    console.log('有读取权限');
  }
});

// 删除文件
fs.unlinkSync('file.txt');  // 同步
fs.unlink('file.txt', (err) => {  // 异步
  if (err) throw err;
  console.log('文件已删除');
});

// 重命名文件
fs.renameSync('old.txt', 'new.txt');
fs.rename('old.txt', 'new.txt', (err) => {
  if (err) throw err;
  console.log('文件已重命名');
});

// 复制文件
fs.copyFileSync('source.txt', 'copy.txt');
fs.copyFile('source.txt', 'copy.txt', (err) => {
  if (err) throw err;
  console.log('文件已复制');
});
```

## 目录操作

```javascript
// 创建目录
fs.mkdirSync('new-dir');  // 创建单层目录

// 创建多层目录（递归）
fs.mkdirSync('path/to/deep/dir', { recursive: true });

// 读取目录内容（同步）
const files = fs.readdirSync('.');
console.log(files);  // ['file1.txt', 'dir1', 'dir2']

// 读取目录内容（异步）
fs.readdir('.', (err, files) => {
  if (err) throw err;
  console.log(files);
});

// 读取目录详细信息
async function readDirWithStats() {
  const files = await fsPromises.readdir('.');
  for (const file of files) {
    const filePath = path.join('.', file);
    const stats = await fsPromises.stat(filePath);
    console.log(`${file} - ${stats.isDirectory() ? '目录' : '文件'}`);
  }
}

// 删除空目录
fs.rmdirSync('empty-dir');
fs.rmdir('empty-dir', (err) => {
  if (err) throw err;
  console.log('目录已删除');
});

// 递归删除目录（Node.js 14.14.0+）
fs.rmSync('non-empty-dir', { recursive: true, force: true });

// 获取当前工作目录
console.log('当前目录:', process.cwd());

// 改变当前工作目录
process.chdir('/path/to/directory');
```

## 文件路径处理

```javascript
const path = require('path');

// 路径拼接
const fullPath = path.join('folder', 'subfolder', 'file.txt');
console.log(fullPath);  // folder/subfolder/file.txt

// 解析路径
const parsed = path.parse('/home/user/docs/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/home/user/docs',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// 获取文件名
console.log(path.basename('/path/to/file.txt'));  // file.txt
console.log(path.basename('/path/to/file.txt', '.txt'));  // file

// 获取目录名
console.log(path.dirname('/path/to/file.txt'));  // /path/to

// 获取扩展名
console.log(path.extname('/path/to/file.txt'));  // .txt

// 规范化路径
console.log(path.normalize('path/./to/../file.txt'));  // path/file.txt

// 判断绝对路径
console.log(path.isAbsolute('/path/to/file'));  // true
console.log(path.isAbsolute('./file'));         // false

// 获取相对路径
console.log(path.relative('/home/user/dir', '/home/user/other/dir'));  // ../../other/dir
```

## 文件监听

```javascript
// 监听文件变化
fs.watch('file.txt', (eventType, filename) => {
  console.log(`事件类型: ${eventType}`);
  console.log(`文件名: ${filename}`);
});

// 监听目录变化
fs.watch('.', { recursive: true }, (eventType, filename) => {
  console.log(`目录变化: ${eventType}, 文件: ${filename}`);
});

// 使用 watchFile（轮询方式，跨平台兼容）
fs.watchFile('file.txt', (curr, prev) => {
  console.log('文件修改时间变化');
  console.log('当前修改时间:', curr.mtime);
  console.log('之前修改时间:', prev.mtime);
});

// 取消监听
const watcher = fs.watch('file.txt', (eventType, filename) => {
  console.log(`变化: ${eventType}`);
});

// 5秒后停止监听
setTimeout(() => {
  watcher.close();
  console.log('监听已停止');
}, 5000);
```

## 实际应用示例

### 日志文件管理

```javascript
const fs = require('fs');
const path = require('path');

class Logger {
  constructor(logDir = './logs') {
    this.logDir = logDir;
    this.ensureLogDir();
  }

  ensureLogDir() {
    if (!fs.existsSync(this.logDir)) {
      fs.mkdirSync(this.logDir, { recursive: true });
    }
  }

  log(message) {
    const timestamp = new Date().toISOString();
    const logMessage = `[${timestamp}] ${message}\n`;
    const date = new Date().toISOString().split('T')[0];
    const logFile = path.join(this.logDir, `${date}.log`);

    fs.appendFileSync(logFile, logMessage);
  }
}

const logger = new Logger();
logger.log('系统启动');
logger.log('用户登录');
```

### 配置文件管理

```javascript
const fs = require('fs');
const path = require('path');

class ConfigManager {
  constructor(configFile = './config.json') {
    this.configFile = configFile;
    this.config = this.load();
  }

  load() {
    try {
      const data = fs.readFileSync(this.configFile, 'utf8');
      return JSON.parse(data);
    } catch (err) {
      // 如果文件不存在，返回默认配置
      return {
        port: 3000,
        database: {
          host: 'localhost',
          port: 5432
        }
      };
    }
  }

  save() {
    fs.writeFileSync(
      this.configFile,
      JSON.stringify(this.config, null, 2)
    );
  }

  get(key) {
    return this.config[key];
  }

  set(key, value) {
    this.config[key] = value;
    this.save();
  }
}
```

### 批量文件处理

```javascript
const fs = require('fs');
const path = require('path');

async function processFiles(directory, callback) {
  const files = await fs.promises.readdir(directory);

  for (const file of files) {
    const filePath = path.join(directory, file);
    const stats = await fs.promises.stat(filePath);

    if (stats.isFile()) {
      const content = await fs.promises.readFile(filePath, 'utf8');
      await callback(file, content);
    }
  }
}

// 使用示例：批量替换文件中的内容
processFiles('./documents', async (filename, content) => {
  const newContent = content.replace(/oldText/g, 'newText');
  await fs.promises.writeFile(`./processed/${filename}`, newContent);
  console.log(`已处理: ${filename}`);
});
```

## 注意事项

1. **性能考虑**
   - 大文件操作优先使用流式处理
   - 避免在循环中使用同步操作
   - 使用 `fs.promises` 的 async/await 模式更易维护

2. **错误处理**
   - 总是处理回调函数中的错误
   - 使用 try-catch 包裹同步操作和 async/await
   - 检查文件存在性后再操作

3. **安全性**
   - 验证用户输入的路径，防止路径遍历攻击
   - 使用相对路径而不是绝对路径
   - 检查文件权限后再执行操作

```javascript
// 安全的路径检查
const path = require('path');
const fs = require('fs');

function safeReadFile(userPath) {
  // 规范化路径
  const normalizedPath = path.normalize(userPath);

  // 检查是否包含 .. 防止路径遍历
  if (normalizedPath.includes('..')) {
    throw new Error('非法路径');
  }

  // 检查是否在允许的目录内
  const allowedDir = '/safe/directory';
  if (!normalizedPath.startsWith(allowedDir)) {
    throw new Error('访问被拒绝');
  }

  return fs.readFileSync(normalizedPath, 'utf8');
}
```

4. **跨平台兼容性**
   - 使用 `path` 模块处理路径，不要硬编码路径分隔符
   - 注意文件系统大小写敏感性
   - 测试在不同操作系统上的行为

## 总结

fs 模块是 Node.js 核心 API，掌握它对文件操作至关重要。本文介绍了文件读写、目录操作、文件监听等常用功能，以及实际应用场景和最佳实践。记住合理使用异步操作，做好错误处理，注意安全性，就能充分发挥 fs 模块的能力。