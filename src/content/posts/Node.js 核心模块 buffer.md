---
title: Node.js 核心模块 buffer
published: 2023-01-26
description: '二进制数据处理的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Buffer 是 Node.js 中用于处理二进制数据的全局对象。由于 JavaScript 最初没有处理二进制数据的能力，Node.js 提供了 Buffer 类来处理 TCP 流或文件系统操作等需要处理二进制数据的场景。Buffer 类在 Node.js v6.0 之后已成为全局对象，无需 `require('buffer')` 即可使用。

## 核心概念

### Buffer 的特点

1. **固定大小**：一旦创建，Buffer 的大小就不能改变
2. **内存分配**：Buffer 在 V8 堆内存之外分配内存
3. **类数组**：Buffer 可以像数组一样访问和操作
4. **全局可用**：Buffer 是全局对象，可以直接使用

### 字符编码

Buffer 支持多种字符编码：

- `utf8` - 多字节编码的 Unicode 字符（默认）
- `utf16le` - 2 或 4 字节，小端序编码的 Unicode 字符
- `latin1` - 单字节编码
- `base64` - Base64 编码
- `hex` - 十六进制编码
- `ascii` - 7 位 ASCII 数据

### 内存视图

Buffer 提供了对原始内存的视图，可以高效地操作二进制数据。

## 基本用法

### 创建 Buffer

```javascript
// 方式1：从字符串创建
const buf1 = Buffer.from('Hello World');
console.log(buf1); // <Buffer 48 65 6c 6c 6f 20 57 6f 72 6c 64>

// 指定编码
const buf2 = Buffer.from('你好世界', 'utf8');

// 方式2：指定大小创建（初始化为零）
const buf3 = Buffer.alloc(10);
console.log(buf3); // <Buffer 00 00 00 00 00 00 00 00 00 00>

// 方式3：不安全的方式（不推荐）
const buf4 = Buffer.allocUnsafe(10); // 可能包含旧数据

// 方式4：从数组创建
const buf5 = Buffer.from([0x48, 0x65, 0x6c, 0x6c, 0x6f]);

// 方式5：从 Buffer 创建
const buf6 = Buffer.from(buf1);
```

### 读取和写入数据

```javascript
const buf = Buffer.alloc(10);

// 写入数据
buf.write('Hello');

// 读取数据
console.log(buf.toString('utf8', 0, 5)); // 'Hello'

// 读取单个字节
console.log(buf[0]); // 72 ('H' 的 ASCII 码)

// 写入数值
buf.writeUInt8(255, 5); // 写入 8 位无符号整数
buf.writeInt16BE(1000, 6); // 写入 16 位有符号整数（大端序）

// 读取数值
console.log(buf.readUInt8(5)); // 255
console.log(buf.readInt16BE(6)); // 1000
```

### Buffer 转换

```javascript
const buf = Buffer.from('Hello World');

// 转换为字符串
const str = buf.toString('utf8');
console.log(str); // 'Hello World'

// 转换为 JSON
const json = buf.toJSON();
console.log(json);

// 转换为十六进制
const hex = buf.toString('hex');
console.log(hex); // '48656c6c6f20576f726c64'

// 转换为 Base64
const base64 = buf.toString('base64');
console.log(base64); // 'SGVsbG8gV29ybGQ='

// 从十六进制创建 Buffer
const bufFromHex = Buffer.from('48656c6c6f', 'hex');
console.log(bufFromHex.toString()); // 'Hello'

// 从 Base64 创建 Buffer
const bufFromBase64 = Buffer.from('SGVsbG8=', 'base64');
console.log(bufFromBase64.toString()); // 'Hello'
```

### Buffer 的常用方法

```javascript
const buf1 = Buffer.from('Hello');
const buf2 = Buffer.from('World');

// 拼接 Buffer
const buf3 = Buffer.concat([buf1, buf2, Buffer.from('!')]);
console.log(buf3.toString()); // 'HelloWorld!'

// 比较两个 Buffer
const result = Buffer.compare(buf1, buf2);
console.log(result); // 返回比较结果

// 检查 Buffer
console.log(buf1.equals(buf2)); // false

// 复制 Buffer
const buf4 = Buffer.alloc(5);
buf1.copy(buf4);
console.log(buf4.toString()); // 'Hello'

// 切片 Buffer
const buf5 = Buffer.from('Node.js Buffer');
const slice = buf5.slice(0, 4);
console.log(slice.toString()); // 'Node'

// 获取长度
console.log(buf1.length); // 5

// 填充 Buffer
const buf6 = Buffer.alloc(10);
buf6.fill(0); // 填充零
buf6.fill(0x41, 0, 5); // 填充 'A' 到前 5 个位置
console.log(buf6.toString()); // 'AAAAA\0\0\0\0\0'
```

## 实际应用

### 文件读取和写入

```javascript
const fs = require('fs');

// 读取文件为 Buffer
const fileBuffer = fs.readFileSync('image.png');
console.log('文件大小:', fileBuffer.length);

// 写入 Buffer 到文件
fs.writeFileSync('copy.png', fileBuffer);

// 异步读取
fs.readFile('data.txt', (err, data) => {
  if (err) {
    console.error('读取失败:', err);
    return;
  }

  console.log('文件内容:', data.toString('utf8'));
  console.log('文件大小:', data.length);
  console.log('前 10 字节:', data.slice(0, 10).toString('hex'));
});
```

### 网络数据传输

```javascript
const net = require('net');

// 创建 TCP 服务器
const server = net.createServer((socket) => {
  socket.on('data', (data) => {
    console.log('接收到数据:', data.toString());
    console.log('数据长度:', data.length);

    // 处理二进制数据
    const header = data.slice(0, 4);
    const body = data.slice(4);

    console.log('头部:', header.toString('hex'));
    console.log('内容:', body.toString());

    // 回显数据
    socket.write(Buffer.from('Echo: '));
    socket.write(data);
  });
});

server.listen(3000, () => {
  console.log('TCP 服务器运行在端口 3000');
});
```

### 图片处理

```javascript
const fs = require('fs');
const http = require('http');

// 读取图片
const imageBuffer = fs.readFileSync('photo.jpg');

// 创建 HTTP 服务器提供图片
const server = http.createServer((req, res) => {
  if (req.url === '/image.jpg') {
    res.writeHead(200, {
      'Content-Type': 'image/jpeg',
      'Content-Length': imageBuffer.length
    });
    res.end(imageBuffer);
  } else {
    res.writeHead(200, { 'Content-Type': 'text/plain' });
    res.end('访问 /image.jpg 查看图片');
  }
});

server.listen(3000, () => {
  console.log('服务器运行在 http://localhost:3000');
});

// 图片压缩示例
const zlib = require('zlib');

fs.readFile('large-image.png', (err, data) => {
  if (err) throw err;

  zlib.gzip(data, (err, compressed) => {
    if (err) throw err;

    console.log('原始大小:', data.length);
    console.log('压缩后大小:', compressed.length);
    console.log('压缩率:', ((1 - compressed.length / data.length) * 100).toFixed(2) + '%');

    fs.writeFile('image.png.gz', compressed, (err) => {
      if (err) throw err;
      console.log('图片已压缩并保存');
    });
  });
});
```

### 二进制协议解析

```javascript
// 模拟解析二进制数据包
function parsePacket(buffer) {
  const packet = {
    version: buffer.readUInt8(0),
    type: buffer.readUInt8(1),
    length: buffer.readUInt16BE(2),
    payload: buffer.slice(4, 4 + buffer.readUInt16BE(2))
  };

  return packet;
}

// 创建数据包
function createPacket(version, type, payload) {
  const length = payload.length;
  const buffer = Buffer.alloc(4 + length);

  buffer.writeUInt8(version, 0);
  buffer.writeUInt8(type, 1);
  buffer.writeUInt16BE(length, 2);
  payload.copy(buffer, 4);

  return buffer;
}

// 使用示例
const payload = Buffer.from('Hello, this is payload data');
const packet = createPacket(1, 2, payload);

console.log('数据包:', packet.toString('hex'));
const parsed = parsePacket(packet);
console.log('解析结果:', parsed);
console.log('Payload:', parsed.payload.toString());
```

### 数据加密

```javascript
const crypto = require('crypto');

// 生成密钥
const key = crypto.randomBytes(32); // 256 位密钥
const iv = crypto.randomBytes(16);  // 初始化向量

// 加密函数
function encrypt(text, key, iv) {
  const cipher = crypto.createCipheriv('aes-256-cbc', key, iv);
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}

// 解密函数
function decrypt(encrypted, key, iv) {
  const decipher = crypto.createDecipheriv('aes-256-cbc', key, iv);
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}

// 使用示例
const plaintext = '这是要加密的秘密信息';
const encrypted = encrypt(plaintext, key, iv);
const decrypted = decrypt(encrypted, key, iv);

console.log('原文:', plaintext);
console.log('密文:', encrypted);
console.log('解密:', decrypted);
```

## 注意事项

1. **内存泄漏**：Buffer 在 V8 堆外分配内存，如果不及时释放可能导致内存泄漏。在使用完大 Buffer 后，可以将其长度设为 0。

```javascript
// 清空 Buffer
buf.fill(0);
// 或
buf.length = 0;
```

2. **编码处理**：处理多字节字符时要注意编码问题，避免字符截断导致乱码。

```javascript
// 错误示例：可能截断多字节字符
const buf = Buffer.from('你好世界');
const slice = buf.slice(0, 5); // 可能截断字符

// 正确做法：使用字符串操作
const str = '你好世界';
const sliceStr = str.substring(0, 2);
```

3. **大小端序**：在处理多字节数值时，要注意大小端序的选择。

```javascript
const buf = Buffer.alloc(4);
buf.writeUInt32BE(0x12345678, 0); // 大端序
buf.writeUInt32LE(0x12345678, 0); // 小端序
```

4. **性能考虑**：频繁的 Buffer 操作可能影响性能，尽量批量处理数据。

```javascript
// 不推荐：频繁创建小 Buffer
for (let i = 0; i < 1000; i++) {
  const buf = Buffer.alloc(10);
  // ...
}

// 推荐：重用 Buffer
const buf = Buffer.alloc(10);
for (let i = 0; i < 1000; i++) {
  buf.fill(0);
  // ...
}
```

5. **Buffer.from() vs Buffer.alloc()**：`Buffer.allocUnsafe()` 更快但不安全，只在确定会立即覆盖所有数据时使用。

6. **数组索引访问**：访问超出范围的索引返回 undefined，而不会抛出错误。

```javascript
const buf = Buffer.from('hello');
console.log(buf[10]); // undefined
```

7. **Buffer.slice() 的共享**：Buffer.slice() 返回的视图与原 Buffer 共享内存。

```javascript
const buf1 = Buffer.from('hello');
const buf2 = buf1.slice(0, 3);
buf1[0] = 0x48; // 修改 buf1 会影响 buf2
console.log(buf2.toString()); // 'Hello'
```

如需要独立的副本，使用 `Buffer.from()`：

```javascript
const buf1 = Buffer.from('hello');
const buf2 = Buffer.from(buf1.slice(0, 3));
buf1[0] = 0x48; // 修改 buf1 不会影响 buf2
console.log(buf2.toString()); // 'hel'
```

## 总结

Buffer 是 Node.js 处理二进制数据的核心工具，提供了高效的内存操作和丰富的 API。通过掌握 Buffer 的创建、转换和操作方法，可以有效地处理文件 I/O、网络传输和数据加密等场景。在实际开发中，正确使用 Buffer 可以显著提高性能，特别是在处理大量数据时。理解 Buffer 的工作原理和注意事项，对于编写高效、可靠的 Node.js 应用程序至关重要。