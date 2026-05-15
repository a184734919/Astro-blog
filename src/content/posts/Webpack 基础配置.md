---
title: Webpack 基础配置
published: 2024-06-01
description: 'Webpack 入口和出口的详细介绍和学习笔记'
image: ''
tags: ["Webpack","基础"]
category: '前端工具'
draft: false
lang: 'zh-CN'
---

## Webpack 基础

Webpack 是一个静态模块打包工具，它会递归地构建依赖关系图。

## 基本配置

```javascript
// webpack.config.js
const path = require('path');

module.exports = {
  entry: './src/index.js',
  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'dist')
  },
  module: {
    rules: [
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader']
      },
      {
        test: /\.js$/,
        exclude: /node_modules/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: './src/index.html'
    })
  ]
};
```

## 常用 Loader

- `babel-loader`: 转换 ES6+ 代码
- `css-loader`: 处理 CSS 文件
- `style-loader`: 将 CSS 注入 DOM
- `file-loader`: 处理图片字体等文件

## 常用 Plugin

- `HtmlWebpackPlugin`: 自动生成 HTML
- `MiniCssExtractPlugin`: 提取 CSS 到单独文件
- `DefinePlugin`: 定义环境变量

## 总结

Webpack 是前端工程化的核心工具。