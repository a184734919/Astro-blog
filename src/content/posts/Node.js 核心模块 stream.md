---
title: Node.js 核心模块 stream
published: 2023-01-23
description: '流处理和数据管道的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Stream（流）是 Node.js 中处理流式数据的抽象接口。Stream 模块提供了一套高效的 API，用于处理读写操作，特别是处理大文件或网络传输时，可以显著提高性能和内存效率。通过流，数据可以分块处理，而不需要一次性加载到内存中。

## 核心概念

### 四种流类型

Node.js 中有四种基本的流类型：

1. **Readable（可读流）** - 用于从数据源读取数据
2. **Writable（可写流）** - 用于向目标写入数据
3. **Duplex（双工流）** - 同时可读可写的流
4. **Transform（转换流）** - 可以修改或转换数据的双工流

### 流的模式

- **流动模式（Flowing Mode）**：数据自动从底层系统读取，并通过事件接口尽快提供给应用程序
- **暂停模式（Paused Mode）**：必须显式调用 `stream.read()` 方法才能读取数据块

### 事件

流模块提供了多个重要事件：

- `data` - 当有数据可读取时触发
- `end` - 当没有更多数据可读取时触发
- `error` - 当发生错误时触发
- `finish` - 当所有数据都已刷新到底层系统时触发
- `close` - 当流及其底层资源关闭时触发

## 基本用法

### 创建可读流

```javascript
const fs = require('fs');

// 方式1：使用 fs.createReadStream
const readableStream = fs.createReadStream('input.txt');

readableStream.on('data', (chunk) => {
  console.log(`收到 ${chunk.length} 字节的数据`);
  console.log(chunk.toString());
});

readableStream.on('end', () => {
  console.log('读取完成');
});

readableStream.on('error', (err) => {
  console.error('读取错误:', err);
});

// 方式2：手动创建可读流
const { Readable } = require('stream');
const customReadable = new Readable({
  read(size) {
    this.push('Hello ');
    this.push('World');
    this.push(null); // 表示结束
  }
});

customReadable.on('data', (chunk) => {
  console.log(chunk.toString());
});
```

### 创建可写流

```javascript
const fs = require('fs');

const writableStream = fs.createWriteStream('output.txt');

writableStream.write('Hello, World!\n');
writableStream.write('这是第二行数据\n');
writableStream.end('结束写入');

writableStream.on('finish', () => {
  console.log('写入完成');
});

writableStream.on('error', (err) => {
  console.error('写入错误:', err);
});

// 手动创建可写流
const { Writable } = require('stream');
const customWritable = new Writable({
  write(chunk, encoding, callback) {
    console.log(`写入数据: ${chunk.toString()}`);
    callback();
  }
});

customWritable.write('测试数据');
customWritable.end();
```

### 使用管道（pipe）

管道是连接可读流和可写流的最简单方法：

```javascript
const fs = require('fs');

const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');

// 使用 pipe 方法
readStream.pipe(writeStream);

// 监听事件
readStream.on('end', () => {
  console.log('复制完成');
});

// 错误处理
readStream.on('error', (err) => {
  console.error('读取错误:', err);
});

writeStream.on('error', (err) => {
  console.error('写入错误:', err);
});
```

### 创建转换流

```javascript
const { Transform } = require('stream');

// 创建一个转换流：将所有文本转为大写
const upperCaseTransform = new Transform({
  transform(chunk, encoding, callback) {
    this.push(chunk.toString().toUpperCase());
    callback();
  }
});

// 使用转换流
const fs = require('fs');
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output-upper.txt');

readStream
  .pipe(upperCaseTransform)
  .pipe(writeStream);

// 更复杂的转换流：压缩数据
const zlib = require('zlib');
const gzip = zlib.createGzip();

const readStream2 = fs.createReadStream('large-file.txt');
const writeStream2 = fs.createWriteStream('large-file.txt.gz');

readStream2
  .pipe(gzip)
  .pipe(writeStream2);
```

## 实际应用

### 文件复制

```javascript
const fs = require('fs');

function copyFile(source, destination) {
  return new Promise((resolve, reject) => {
    const readStream = fs.createReadStream(source);
    const writeStream = fs.createWriteStream(destination);

    readStream.pipe(writeStream);

    writeStream.on('finish', resolve);
    writeStream.on('error', reject);
    readStream.on('error', reject);
  });
}

// 使用
copyFile('source.txt', 'destination.txt')
  .then(() => console.log('文件复制成功'))
  .catch(err => console.error('复制失败:', err));
```

### HTTP 服务器文件流式传输

```javascript
const http = require('http');
const fs = require('fs');

const server = http.createServer((req, res) => {
  if (req.url === '/large-file') {
    const fileStream = fs.createReadStream('large-video.mp4');
    fileStream.pipe(res);

    fileStream.on('error', (err) => {
      res.statusCode = 500;
      res.end('服务器错误');
    });
  } else {
    res.end('访问 /large-file 下载文件');
  }
});

server.listen(3000, () => {
  console.log('服务器运行在 http://localhost:3000');
});
```

### CSV 文件处理

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// 将 CSV 转换为 JSON 数组
const csvToJson = new Transform({
  objectMode: true,
  transform(chunk, encoding, callback) {
    const lines = chunk.toString().split('\n').filter(line => line.trim());

    lines.forEach(line => {
      const [id, name, age] = line.split(',');
      if (id && name && age) {
        this.push({ id, name, age: parseInt(age) });
      }
    });

    callback();
  }
});

// 使用示例
const readStream = fs.createReadStream('data.csv');
const writeStream = fs.createWriteStream('output.json');

let results = [];

csvToJson.on('data', (obj) => {
  results.push(obj);
});

csvToJson.on('end', () => {
  writeStream.write(JSON.stringify(results, null, 2));
  writeStream.end();
});

readStream.pipe(csvToJson);
```

### 实时日志处理

```javascript
const fs = require('fs');
const { Transform } = require('stream');

// 过滤错误日志
const errorFilter = new Transform({
  transform(chunk, encoding, callback) {
    const line = chunk.toString();
    if (line.includes('[ERROR]')) {
      this.push(line);
    }
    callback();
  }
});

// 实时监控日志文件
const logStream = fs.createReadStream('app.log');
const errorLog = fs.createWriteStream('errors.log');

logStream.pipe(errorFilter).pipe(errorLog);

console.log('开始监控日志文件...');
```

## 注意事项

1. **内存管理**：流的主要优势是节省内存，但如果不正确使用流（如一次性读取大文件），仍然可能导致内存问题。

2. **错误处理**：始终为流添加错误监听器，未处理的错误事件会导致进程崩溃。

3. **背压（Backpressure）**：当写入速度慢于读取速度时，需要处理背压。`pipe()` 方法会自动处理背压。

4. **流的状态**：注意区分流动模式和暂停模式，正确使用 `pause()` 和 `resume()` 方法。

5. **资源清理**：使用 `stream.destroy()` 方法手动关闭流，特别是在处理大量流时避免资源泄漏。

6. **编码问题**：处理文本流时，注意字符编码，使用 `setEncoding()` 方法指定正确的编码格式。

```javascript
// 正确设置编码
const readableStream = fs.createReadStream('text.txt');
readableStream.setEncoding('utf8');

readableStream.on('data', (chunk) => {
  // chunk 现在是字符串而不是 Buffer
  console.log(chunk);
});
```

7. **流式处理大文件**：对于非常大的文件，使用流可以避免内存溢出，但要注意处理进度和错误恢复。

## 总结

Node.js 的 Stream 模块提供了强大的数据流处理能力，是处理大文件、网络传输和数据管道的核心工具。通过理解四种流类型和正确使用管道机制，可以构建高效、可扩展的应用程序。在实际项目中，合理使用流可以显著提高性能和减少内存消耗，特别是在处理 I/O 密集型任务时。掌握流的使用是 Node.js 开发者的重要技能之一。