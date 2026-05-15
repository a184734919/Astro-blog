---
title: Node.js 核心模块 fs
published: 2023-01-08
description: '文件系统操作详解的详细介绍和学习笔记'
image: ''
tags: ["Node.js","模块"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## Node.js 文件系统模块

fs 模块提供了文件系统的操作接口，包括文件的读写、目录操作等。

## 文件读取

```javascript
const fs = require('fs');

// 同步读取
const data = fs.readFileSync('file.txt', 'utf8');
console.log(data);

// 异步读取
fs.readFile('file.txt', 'utf8', (err, data) => {
  if (err) throw err;
  console.log(data);
});

// Promise 方式
const fsPromises = require('fs/promises');
async function readFile() {
  const data = await fsPromises.readFile('file.txt', 'utf8');
  console.log(data);
}
```

## 文件写入

```javascript
// 同步写入
fs.writeFileSync('file.txt', 'Hello World');

// 异步写入
fs.writeFile('file.txt', 'Hello World', (err) => {
  if (err) throw err;
  console.log('文件已保存');
});

// 追加内容
fs.appendFileSync('file.txt', '\nNew line');
```

## 目录操作

```javascript
// 创建目录
fs.mkdirSync('new-dir', { recursive: true });

// 读取目录
const files = fs.readdirSync('.');
console.log(files);

// 删除目录
fs.rmdirSync('empty-dir');
```

## 文件监听

```javascript
fs.watch('file.txt', (eventType, filename) => {
  console.log(`事件类型: ${eventType}`);
  console.log(`文件名: ${filename}`);
});
```

## 总结

fs 模块是 Node.js 核心 API，掌握它对文件操作至关重要。