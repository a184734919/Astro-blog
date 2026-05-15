---
title: CSS Grid 布局
published: 2024-10-03
description: 'Grid 网格布局详解的详细介绍和学习笔记'
image: ''
tags: ["CSS","布局"]
category: '前端样式'
draft: false
lang: 'zh-CN'
---

## 概述

CSS Grid Layout（网格布局）是一个二维布局系统，可以同时处理行和列，提供了强大而灵活的布局能力。它特别适合构建复杂的页面布局，如仪表盘、杂志式布局等。

## 核心概念

- **网格容器（Grid Container）**：设置了 `display: grid` 的父元素
- **网格项目（Grid Item）**：容器中的直接子元素
- **网格线（Grid Line）**：构成网格结构的水平和垂直线
- **网格轨道（Grid Track）**：两条相邻网格线之间的空间
- **网格单元格（Grid Cell）**：最小的网格单位
- **网格区域（Grid Area）**：由四条网格线围成的矩形区域

## 基本用法

### 容器属性

```css
/* 启用 Grid 布局 */
.container {
  display: grid;
}

/* 定义行列 */
.container {
  grid-template-columns: 1fr 2fr 1fr;    /* 三列，比例为 1:2:1 */
  grid-template-rows: auto 1fr auto;     /* 三行 */
}

/* 使用 repeat() 函数 */
.container {
  grid-template-columns: repeat(3, 1fr); /* 重复 3 次 1fr */
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr)); /* 自适应列 */
}

/* 间距 */
.container {
  gap: 20px;                             /* 行列间距 */
  gap: 20px 10px;                        /* 行间距20px，列间距10px */
}

/* 区域命名 */
.container {
  grid-template-areas:
    "header header header"
    "sidebar main main"
    "footer footer footer";
}

/* 对齐 */
.container {
  justify-content: center;               /* 水平对齐整个网格 */
  align-content: center;                 /* 垂直对齐整个网格 */
}
```

### 项目属性

```css
/* 指定位置 */
.item {
  grid-column: 1 / 3;                    /* 从第1条线到第3条线 */
  grid-row: 2 / 4;                       /* 从第2条线到第4条线 */
}

/* 使用 span */
.item {
  grid-column: 1 / span 2;               /* 从第1列开始，跨越2列 */
  grid-row: span 2;                      /* 跨越2行 */
}

/* 使用简写 */
.item {
  grid-area: 1 / 1 / 3 / 3;             /* row-start / col-start / row-end / col-end */
}

/* 命名区域 */
.item {
  grid-area: header;
}

/* 单元格对齐 */
.item {
  justify-self: center;                  /* 水平对齐 */
  align-self: center;                    /* 垂直对齐 */
  place-self: center center;             /* 简写 */
}
```

## 实际应用

### 经典圣杯布局

```css
.container {
  display: grid;
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  min-height: 100vh;
  gap: 20px;
}

.header { grid-area: header; }
.sidebar { grid-area: sidebar; }
.main { grid-area: main; }
.aside { grid-area: aside; }
.footer { grid-area: footer; }
```

### 响应式卡片网格

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 20px;
}

/* 自适应，根据容器宽度自动调整列数 */
```

### 仪表盘布局

```css
.dashboard {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  grid-template-rows: auto auto 1fr;
  gap: 20px;
  min-height: 100vh;
}

.sidebar {
  grid-column: 1 / 3;
  grid-row: 1 / 4;
}

.header {
  grid-column: 3 / 13;
  grid-row: 1;
}

.stats {
  grid-column: 3 / 7;
  grid-row: 2;
}

.charts {
  grid-column: 7 / 13;
  grid-row: 2;
}

.content {
  grid-column: 3 / 13;
  grid-row: 3;
}
```

### 图片画廊

```css
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  grid-auto-rows: 200px;
  gap: 10px;
}

.gallery-item {
  overflow: hidden;
}

/* 跨越多个单元格 */
.gallery-item.wide {
  grid-column: span 2;
}

.gallery-item.tall {
  grid-row: span 2;
}

.gallery-item.large {
  grid-column: span 2;
  grid-row: span 2;
}
```

### 杂志式布局

```css
.magazine {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-template-rows: auto;
  gap: 20px;
}

.article-featured {
  grid-column: 1 / 3;
  grid-row: 1 / 3;
}

.article-regular {
  grid-column: 3 / 5;
}

.article-list {
  grid-column: 1 / 5;
}
```

## 注意事项

1. **与 Flexbox 配合**：Grid 适合整体布局，Flexbox 适合组件内部布局

2. **浏览器兼容性**：现代浏览器都支持，IE11 需要使用 `-ms-` 前缀

3. **性能优化**：
   - 避免过度嵌套
   - 使用 `minmax()` 实现响应式
   - 合理使用 `fr` 单位

4. **常见问题**：
   - 使用 `grid-auto-flow: dense` 填补空隙
   - `min-height: 0` 防止项目溢出
   - 使用 `@supports` 检测浏览器支持

5. **调试技巧**：
   ```css
   .container {
     background: linear-gradient(90deg, #f00 1px, transparent 1px),
                 linear-gradient(#f00 1px, transparent 1px);
     background-size: 50px 50px;
   }
   ```

6. **命名约定**：使用语义化的网格区域名称提高可读性

## 总结

CSS Grid 提供了强大的二维布局能力，特别适合构建复杂的页面布局。通过合理使用轨道定义、区域命名和响应式技巧，可以创建出灵活且易于维护的布局。掌握 Grid 与 Flexbox 的配合使用，可以应对各种布局需求。