---
title: Express 会话管理
published: 2023-03-30
description: 'Cookie 和 Session的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## Express 框架简介

Express 是 Node.js 平台上最流行的 Web 框架，它提供了丰富的 HTTP 工具和方法，让开发 Web 应用变得更加简单。

## 快速开始

```javascript
const express = require('express');
const app = express();

// 中间件
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// 路由
app.get('/', (req, res) => {
  res.send('Hello World!');
});

app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

app.post('/api/users', (req, res) => {
  const user = req.body;
  res.status(201).json(user);
});

// 启动服务器
app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

## 中间件机制

```javascript
// 应用级中间件
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next();
});

// 路由级中间件
app.get('/user/:id', (req, res, next) => {
  if (req.params.id === '0') next('route');
  else next();
}, (req, res) => {
  res.send('regular');
});

// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  res.status(500).send('Something broke!');
});
```

## 路由模块化

```javascript
// routes/users.js
const router = express.Router();

router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.post('/', (req, res) => {
  res.status(201).json(req.body);
});

module.exports = router;

// app.js
const usersRouter = require('./routes/users');
app.use('/api/users', usersRouter);
```

## 总结

Express 简单灵活，是构建 Node.js Web 应用的首选框架。