---
title: Node.js 核心模块 path
published: 2023-01-12
description: '路径处理方法详解的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的 `path` 模块提供了一系列用于处理和转换文件路径的工具方法。由于不同操作系统（Windows、macOS、Linux）的路径分隔符和路径规范存在差异，`path` 模块能够自动适配这些差异，确保跨平台兼容性。

## 核心概念

### 路径分隔符
- Windows 使用反斜杠 `\`
- Unix 系统使用正斜杠 `/`

### 路段与路径分段
路径可以拆分为多个路段（segments），如 `/home/user/project` 包含三个路段。

### 规范化路径
将路径转换为标准格式，处理 `.`、`..` 以及多余的分隔符。

## 基本用法

```javascript
const path = require('path');

// 1. path.join() - 拼接路径片段
const fullPath = path.join('/foo', 'bar', 'baz/asdf', 'quux', '..');
console.log(fullPath); // '/foo/bar/baz/asdf'

// 2. path.resolve() - 解析为绝对路径
const absolutePath = path.resolve('src', 'assets', 'image.png');
console.log(absolutePath); // /current/working/dir/src/assets/image.png

// 3. path.basename() - 获取文件名
console.log(path.basename('/foo/bar/baz.txt')); // 'baz.txt'
console.log(path.basename('/foo/bar/baz.txt', '.txt')); // 'baz'

// 4. path.dirname() - 获取目录名
console.log(path.dirname('/foo/bar/baz.txt')); // '/foo/bar'

// 5. path.extname() - 获取扩展名
console.log(path.extname('/foo/bar/baz.txt')); // '.txt'

// 6. path.parse() - 解析路径对象
const parsed = path.parse('/home/user/dir/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/home/user/dir',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }
```

## 实际应用

### 读取配置文件
```javascript
const path = require('path');
const fs = require('fs');

const configPath = path.join(__dirname, 'config', 'settings.json');
const config = JSON.parse(fs.readFileSync(configPath, 'utf8'));
```

### 构建静态资源路径
```javascript
const getAssetPath = (filename) => {
  return path.join(process.cwd(), 'public', 'assets', filename);
};

const imagePath = getAssetPath('logo.png');
```

### 跨平台路径处理
```javascript
// 自动适配不同操作系统的分隔符
const filePath = path.join('data', 'uploads', 'document.pdf');
// Windows: 'data\\uploads\\document.pdf'
// Unix: 'data/uploads/document.pdf'
```

## 注意事项

1. **路径分隔符差异**：不要硬编码 `/` 或 `\`，始终使用 `path.join()` 或 `path.resolve()`
2. **相对路径陷阱**：`path.resolve()` 会基于当前工作目录解析，而 `path.join()` 只是简单拼接
3. **规范化处理**：使用 `path.normalize()` 处理包含 `.` 或 `..` 的路径
4. **路径安全性**：处理用户输入的路径时，使用 `path.normalize()` 防止路径遍历攻击

## 总结

通过本文的学习，相信你已经对 Node.js 核心模块 path 有了更深入的理解。`path` 模块虽然简单，但在文件操作、路径处理等场景中至关重要，掌握它能够让你的代码更加健壮和跨平台兼容。