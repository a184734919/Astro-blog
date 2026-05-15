---
title: CSS Flexbox 布局
published: 2024-10-17
description: 'Flex 弹性布局详解的详细介绍和学习笔记'
image: ''
tags: ["CSS","布局"]
category: '前端样式'
draft: false
lang: 'zh-CN'
---

## 概述

Flexbox（Flexible Box）是 CSS3 中引入的一种一维布局模型，旨在提供一种更有效的方式来布局、对齐和分配容器中项目的空间，即使它们的大小未知或动态变化。

## 核心概念

- **容器（Flex Container）**：设置了 `display: flex` 的父元素
- **项目（Flex Item）**：容器中的直接子元素
- **主轴（Main Axis）**：容器的主方向，默认水平向右
- **交叉轴（Cross Axis）**：垂直于主轴的方向

## 基本用法

### 容器属性

```css
/* 启用 Flex 布局 */
.container {
  display: flex;
}

/* 设置主轴方向 */
.container {
  flex-direction: row;           /* row | row-reverse | column | column-reverse */
}

/* 换行 */
.container {
  flex-wrap: nowrap;             /* nowrap | wrap | wrap-reverse */
}

/* 主轴对齐 */
.container {
  justify-content: flex-start;   /* flex-start | flex-end | center | space-between | space-around | space-evenly */
}

/* 交叉轴对齐 */
.container {
  align-items: stretch;          /* stretch | flex-start | flex-end | center | baseline */
}

/* 多行对齐 */
.container {
  align-content: flex-start;     /* flex-start | flex-end | center | space-between | space-around | stretch */
}
```

### 项目属性

```css
/* 控制项目的放大比例 */
.item {
  flex-grow: 0;
}

/* 控制项目的缩小比例 */
.item {
  flex-shrink: 1;
}

/* 项目的初始大小 */
.item {
  flex-basis: auto;              /* auto | <length> */
}

/* 简写 */
.item {
  flex: 0 1 auto;                /* flex-grow flex-shrink flex-basis */
}

/* 单个项目对齐 */
.item {
  align-self: auto;              /* auto | flex-start | flex-end | center | baseline | stretch */
}

/* 排序 */
.item {
  order: 0;                      /* 整数，数值越小越靠前 */
}
```

## 实际应用

### 水平垂直居中

```css
.container {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 300px;
}

/* 或使用 margin auto */
.container {
  display: flex;
}

.item {
  margin: auto;
}
```

### 两栏布局

```css
.container {
  display: flex;
}

.sidebar {
  flex: 0 0 250px;               /* 固定宽度 */
}

.main {
  flex: 1;                       /* 占据剩余空间 */
}
```

### 三栏布局

```css
.container {
  display: flex;
}

.left {
  flex: 0 0 200px;
}

.center {
  flex: 1;
}

.right {
  flex: 0 0 200px;
}
```

### 响应式卡片网格

```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
}

.card {
  flex: 0 0 calc(33.333% - 14px);  /* 一行3个 */
}

@media (max-width: 768px) {
  .card {
    flex: 0 0 calc(50% - 10px);    /* 一行2个 */
  }
}

@media (max-width: 480px) {
  .card {
    flex: 0 0 100%;                /* 一行1个 */
  }
}
```

### 导航栏

```css
.navbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 0 20px;
}

.nav-links {
  display: flex;
  gap: 20px;
  align-items: center;
}
```

### 卡片内容布局

```css
.card {
  display: flex;
  flex-direction: column;
}

.card-image {
  flex: 0 0 auto;
}

.card-content {
  flex: 1;                        /* 占据剩余空间 */
}

.card-footer {
  flex: 0 0 auto;
  margin-top: auto;               /* 推到底部 */
}
```

## 注意事项

1. **浏览器兼容性**：现代浏览器都支持 Flexbox，无需添加前缀

2. **性能考虑**：Flexbox 会触发重排，在动画或频繁更新的场景要注意性能

3. **选择场景**：
   - 一维布局使用 Flexbox
   - 二维布局使用 Grid

4. **常见问题**：
   - 使用 `gap` 属性时注意浏览器支持
   - `min-width: 0` 防止项目溢出容器
   - 文本换行可能需要设置 `flex-wrap`

5. **调试技巧**：
   ```css
   .item:nth-child(1) { background: red; }
   .item:nth-child(2) { background: blue; }
   /* 查看项目如何分配空间 */
   ```

6. **垂直居中替代方案**：Flexbox 的 `align-items: center` 比传统的 `line-height` 方法更灵活

## 总结

Flexbox 是现代 CSS 布局的重要工具，特别适合一维布局场景。通过理解容器和项目的属性，可以轻松实现居中、对齐、空间分配等常见布局需求。Flexbox 与 Grid 布局结合使用，可以构建出复杂且响应式的页面布局。