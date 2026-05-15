---
title: CSS 变量详解
published: 2024-09-20
description: 'CSS 变量的使用的详细介绍和学习笔记'
image: ''
tags: ["CSS","样式"]
category: '前端样式'
draft: false
lang: 'zh-CN'
---

## 概述

CSS 变量（Custom Properties），也称为 CSS 自定义属性，是一种在 CSS 中定义和重用值的机制。它们使得样式管理更加灵活和可维护，特别适合主题切换、设计系统等场景。

## 核心概念

- **声明**：使用 `--` 前缀声明自定义属性
- **作用域**：变量具有继承性，在声明元素的子元素中可用
- **计算**：变量值会在运行时计算，可以使用 `var()` 函数引用
- **回退值**：可以为变量提供回退值

## 基本用法

### 声明和使用

```css
/* 声明变量 */
:root {
  --primary-color: #3498db;
  --secondary-color: #2ecc71;
  --font-size-base: 16px;
  --spacing-unit: 8px;
  --border-radius: 4px;
}

/* 使用变量 */
.button {
  background-color: var(--primary-color);
  font-size: var(--font-size-base);
  padding: calc(var(--spacing-unit) * 2);
  border-radius: var(--border-radius);
}
```

### 作用域和继承

```css
/* 全局作用域 */
:root {
  --global-color: #333;
}

/* 局部作用域 */
.card {
  --card-bg: #fff;
  background-color: var(--card-bg);
}

.card:hover {
  --card-bg: #f5f5f5;  /* 局部变量优先级更高 */
  background-color: var(--card-bg);
}
```

### 变量回退

```css
/* 提供回退值 */
.button {
  color: var(--text-color, #333);
  background: var(--bg-color, var(--fallback-bg, #fff));
}
```

### 动态变量

```css
/* 通过 JS 修改变量 */
.element {
  --dynamic-color: blue;
  color: var(--dynamic-color);
}

/* JavaScript */
document.documentElement.style.setProperty('--dynamic-color', 'red');
```

## 实际应用

### 主题切换

```css
/* 定义主题变量 */
:root {
  --bg-color: #ffffff;
  --text-color: #333333;
  --primary-color: #3498db;
  --border-color: #e0e0e0;
}

[data-theme="dark"] {
  --bg-color: #1a1a1a;
  --text-color: #ffffff;
  --primary-color: #5dade2;
  --border-color: #333333;
}

/* 使用变量 */
body {
  background-color: var(--bg-color);
  color: var(--text-color);
}

.button {
  background-color: var(--primary-color);
  color: var(--bg-color);
  border: 1px solid var(--border-color);
}
```

### 设计系统

```css
/* 设计系统变量 */
:root {
  /* 颜色 */
  --color-primary: #3498db;
  --color-secondary: #2ecc71;
  --color-success: #10b981;
  --color-warning: #f59e0b;
  --color-error: #ef4444;

  /* 间距 */
  --spacing-xs: 4px;
  --spacing-sm: 8px;
  --spacing-md: 16px;
  --spacing-lg: 24px;
  --spacing-xl: 32px;

  /* 字体 */
  --font-size-xs: 12px;
  --font-size-sm: 14px;
  --font-size-base: 16px;
  --font-size-lg: 18px;
  --font-size-xl: 20px;

  /* 阴影 */
  --shadow-sm: 0 1px 2px rgba(0, 0, 0, 0.05);
  --shadow-md: 0 4px 6px rgba(0, 0, 0, 0.1);
  --shadow-lg: 0 10px 15px rgba(0, 0, 0, 0.15);
}

/* 使用设计系统 */
.card {
  padding: var(--spacing-lg);
  border-radius: var(--border-radius-md);
  box-shadow: var(--shadow-md);
}

h1 {
  font-size: var(--font-size-xl);
  margin-bottom: var(--spacing-md);
}
```

### 响应式设计

```css
:root {
  --container-width: 1200px;
  --font-size-base: 16px;
  --spacing: 16px;
}

@media (max-width: 768px) {
  :root {
    --container-width: 100%;
    --font-size-base: 14px;
    --spacing: 12px;
  }
}

.container {
  max-width: var(--container-width);
  padding: var(--spacing);
}

body {
  font-size: var(--font-size-base);
}
```

### 组件样式

```css
/* 组件变量 */
.card {
  --card-bg: #fff;
  --card-padding: 16px;
  --card-radius: 8px;
  --card-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.card {
  background: var(--card-bg);
  padding: var(--card-padding);
  border-radius: var(--card-radius);
  box-shadow: var(--card-shadow);
}

/* 可以覆盖组件变量 */
.card.dark {
  --card-bg: #333;
  --card-shadow: 0 2px 4px rgba(0, 0, 0, 0.5);
}
```

## 注意事项

1. **浏览器兼容性**：现代浏览器都支持，IE11 不支持

2. **命名规范**：
   - 使用 `--` 前缀
   - 使用 kebab-case 命名
   - 语义化命名

3. **性能考虑**：
   - 变量具有继承性，过度使用可能影响性能
   - 在 CSS 动画中使用变量可能影响性能

4. **限制**：
   - 不能用作媒体查询值
   - 不能用作 @import 值
   - 变量值必须有效

5. **最佳实践**：
   - 在 `:root` 中定义全局变量
   - 组件内部定义局部变量
   - 提供合理的回退值
   - 使用语义化命名

6. **调试技巧**：
   ```css
   /* 调试未定义的变量 */
   .debug {
     background: var(--undefined-color, red);
   }
   ```

## 总结

CSS 变量是一种强大的样式管理工具，特别适合主题切换、设计系统和组件化开发。通过合理使用 CSS 变量，可以提高代码的可维护性和可重用性，使得样式管理更加灵活和高效。