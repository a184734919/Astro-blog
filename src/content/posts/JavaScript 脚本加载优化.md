---
title: JavaScript 脚本加载优化
published: 2022-12-05
description: 'defer 和 async 的区别的详细介绍和学习笔记'
image: ''
tags: ["性能"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 脚本加载优化是提升网页性能的关键环节。合理使用 `defer` 和 `async` 属性可以控制脚本的加载和执行时机,减少页面阻塞,优化首屏加载速度和用户体验。

## 核心概念

### 默认行为

浏览器解析 HTML 时,遇到 `<script>` 标签会:

1. **暂停 HTML 解析**
2. **下载脚本文件**
3. **执行脚本代码**
4. **恢复 HTML 解析**

这会导致页面渲染阻塞,用户需要等待脚本加载和执行完成。

### defer 和 async

`defer` 和 `async` 都是 `<script>` 标签的布尔属性,用于改变脚本的加载和执行行为:

- **defer**: 异步下载脚本,延迟执行(HTML 解析完成后)
- **async**: 异步下载脚本,下载完成后立即执行

## 基本用法

### 普通 script 标签(阻塞)

```html
<!DOCTYPE html>
<html>
<head>
  <title>阻塞式脚本</title>
  <!-- 脚本会阻塞 HTML 解析 -->
  <script src="script.js"></script>
</head>
<body>
  <h1>内容</h1>
</body>
</html>
```

执行顺序:
```
1. 开始解析 HTML
2. 遇到 <script> 标签
3. 暂停 HTML 解析,下载 script.js
4. 执行 script.js
5. 恢复 HTML 解析
6. 继续解析剩余 HTML
```

### async 属性(异步加载,立即执行)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Async 脚本</title>
  <!-- 脚本异步下载,下载完成后立即执行 -->
  <script src="analytics.js" async></script>
  <script src="ad.js" async></script>
</head>
<body>
  <h1>内容</h1>
</body>
</html>
```

执行顺序:
```
1. 开始解析 HTML(不阻塞)
2. 异步下载 analytics.js 和 ad.js(并行)
3. 哪个先下载完,就先执行哪个
4. 执行脚本时会暂停 HTML 解析
5. 继续解析 HTML
```

**特点**:
- 不阻塞 HTML 解析
- 下载和执行顺序不确定
- 适合独立脚本(如分析工具、广告)

### defer 属性(异步加载,延迟执行)

```html
<!DOCTYPE html>
<html>
<head>
  <title>Defer 脚本</title>
  <!-- 脚本异步下载,HTML 解析完成后按顺序执行 -->
  <script src="library.js" defer></script>
  <script src="main.js" defer></script>
</head>
<body>
  <h1>内容</h1>
</body>
</html>
```

执行顺序:
```
1. 开始解析 HTML(不阻塞)
2. 异步下载 library.js 和 main.js(并行)
3. HTML 解析完成后,按文档顺序执行
4. 先执行 library.js,再执行 main.js
5. 触发 DOMContentLoaded 事件
```

**特点**:
- 不阻塞 HTML 解析
- 保证执行顺序(按文档顺序)
- 在 DOMContentLoaded 之前执行
- 适合依赖 DOM 的脚本

### 对比示例

```html
<!DOCTYPE html>
<html>
<head>
  <title>对比示例</title>
  <!-- 普通脚本: 阻塞 -->
  <script src="blocking.js"></script>

  <!-- async: 异步加载,谁先下载完谁先执行 -->
  <script src="analytics.js" async></script>
  <script src="social.js" async></script>

  <!-- defer: 异步加载,按顺序执行 -->
  <script src="jquery.js" defer></script>
  <script src="app.js" defer></script>
</head>
<body>
  <h1>页面内容</h1>
</body>
</html>
```

## 实际应用

### 1. 第三方库使用 defer

```html
<!DOCTYPE html>
<html>
<head>
  <title>使用 jQuery</title>
  <!-- 先加载依赖库 -->
  <script src="https://cdn.jsdelivr.net/npm/jquery@3.7.0/dist/jquery.min.js" defer></script>
  <!-- 再加载依赖库的脚本 -->
  <script src="app.js" defer></script>
</head>
<body>
  <button id="clickMe">点击我</button>
</body>
</html>
```

`app.js`:
```javascript
// defer 确保执行时 DOM 已解析,且 jquery 已加载
$(document).ready(function() {
  $('#clickMe').on('click', function() {
    alert('Hello, jQuery!');
  });
});
```

### 2. 独立脚本使用 async

```html
<!DOCTYPE html>
<html>
<head>
  <title>独立脚本</title>
  <!-- 分析工具: 独立执行,不阻塞页面 -->
  <script src="https://www.google-analytics.com/analytics.js" async></script>

  <!-- 广告: 不影响主要内容 -->
  <script src="https://ad-network.com/ad.js" async></script>

  <!-- 热力图: 独立工具 -->
  <script src="https://heatmap.com/tracker.js" async></script>
</head>
<body>
  <h1>主要内容</h1>
</body>
</html>
```

### 3. 关键脚本内联

```html
<!DOCTYPE html>
<html>
<head>
  <title>优化首屏</title>
  <!-- 关键样式内联 -->
  <style>
    body { margin: 0; font-family: Arial; }
    .hero { height: 100vh; background: #f0f0f0; }
  </style>

  <!-- 关键 JS 内联(首屏渲染必需) -->
  <script>
    // 防止 FOUC(Flash of Unstyled Content)
    document.documentElement.style.visibility = 'hidden';
  </script>
</head>
<body>
  <div class="hero">
    <h1>欢迎</h1>
  </div>

  <!-- 非关键脚本延迟加载 -->
  <script src="main.js" defer></script>
  <script src="analytics.js" async></script>
</body>
</html>
```

`main.js`:
```javascript
// 恢复显示
document.documentElement.style.visibility = 'visible';

// 其他功能
document.addEventListener('DOMContentLoaded', function() {
  console.log('DOM 加载完成');
});
```

### 4. 模块化脚本

```html
<!DOCTYPE html>
<html>
<head>
  <title>模块化脚本</title>
  <!-- 模块脚本默认 defer 行为 -->
  <script type="module" src="module1.js"></script>
  <script type="module" src="module2.js"></script>

  <!-- 传统脚本 -->
  <script src="traditional.js" defer></script>
</head>
<body>
  <h1>模块示例</h1>
</body>
</html>
```

`module1.js`:
```javascript
export const greet = (name) => `Hello, ${name}!`;
```

`module2.js`:
```javascript
import { greet } from './module1.js';

console.log(greet('World'));
```

### 5. 按需加载脚本

```html
<!DOCTYPE html>
<html>
<head>
  <title>按需加载</title>
</head>
<body>
  <button id="loadFeature">加载高级功能</button>

  <script defer>
    const button = document.getElementById('loadFeature');

    button.addEventListener('click', async function() {
      // 动态加载脚本
      const script = document.createElement('script');
      script.src = 'advanced-feature.js';
      script.async = true;

      await new Promise((resolve, reject) => {
        script.onload = resolve;
        script.onerror = reject;
        document.head.appendChild(script);
      });

      console.log('高级功能已加载');
    });
  </script>
</body>
</html>
```

### 6. 预加载策略

```html
<!DOCTYPE html>
<html>
<head>
  <title>预加载</title>
  <!-- 预加载重要资源 -->
  <link rel="preload" href="critical.js" as="script">
  <link rel="preload" href="styles.css" as="style">
  <link rel="preload" href="image.jpg" as="image">

  <!-- 预连接到域名 -->
  <link rel="preconnect" href="https://cdn.example.com">
  <link rel="dns-prefetch" href="https://analytics.example.com">

  <!-- 使用预加载的脚本 -->
  <script src="critical.js" defer></script>
</head>
<body>
  <h1>内容</h1>
</body>
</html>
```

### 7. 响应式脚本加载

```html
<!DOCTYPE html>
<html>
<head>
  <title>响应式加载</title>
</head>
<body>
  <script defer>
    // 根据设备加载不同脚本
    if (window.innerWidth > 768) {
      loadScript('desktop-features.js');
    } else {
      loadScript('mobile-features.js');
    }

    // 检测功能支持
    if ('IntersectionObserver' in window) {
      loadScript('lazy-load.js');
    }

    function loadScript(src) {
      const script = document.createElement('script');
      script.src = src;
      script.defer = true;
      document.head.appendChild(script);
    }
  </script>
</body>
</html>
```

## 注意事项

### 1. 依赖关系

```html
<!-- ❌ 错误: async 不保证顺序 -->
<script src="jquery.js" async></script>
<script src="app.js" async></script>

<!-- ✅ 正确: defer 保证顺序 -->
<script src="jquery.js" defer></script>
<script src="app.js" defer></script>

<!-- ✅ 正确: async 只用于独立脚本 -->
<script src="analytics.js" async></script>
<script src="ad.js" async></script>
```

### 2. DOM 访问

```html
<!-- ✅ defer 可以直接访问 DOM -->
<script defer>
  document.getElementById('header').style.color = 'red';
</script>
<header id="header">标题</header>

<!-- ❌ 普通 script 需要等待 DOMContentLoaded -->
<script>
  // 可能会报错,因为 DOM 可能还没解析
  // document.getElementById('header'); // null
</script>
<header id="header">标题</header>

<!-- ✅ 使用 DOMContentLoaded -->
<script>
  document.addEventListener('DOMContentLoaded', function() {
    document.getElementById('header');
  });
</script>
```

### 3. 混合使用

```html
<script src="jquery.js" defer></script>  <!-- 在 DOMContentLoaded 前执行 -->
<script src="analytics.js" async></script> <!-- 不确定执行时机 -->

<script defer>
  document.addEventListener('DOMContentLoaded', function() {
    console.log('DOM 加载完成');
  });
</script>
```

### 4. 内联脚本

```html
<!-- ❌ async/defer 不影响内联脚本 -->
<script async>
  console.log('这个还是会阻塞');
</script>

<!-- ✅ 内联脚本不推荐使用 async/defer -->
<script>
  console.log('直接执行');
</script>
```

### 5. 性能监控

```javascript
// 监控脚本加载性能
window.addEventListener('load', function() {
  const resources = performance.getEntriesByType('resource');

  resources.forEach(resource => {
    if (resource.initiatorType === 'script') {
      console.log(`${resource.name}:`, {
        加载时间: resource.duration,
        DNS时间: resource.domainLookupEnd - resource.domainLookupStart,
        TCP时间: resource.connectEnd - resource.connectStart,
        TTFB: resource.responseStart - resource.requestStart
      });
    }
  });
});
```

### 6. 浏览器兼容性

```javascript
// 检测 async/defer 支持
const supportsAsync = 'async' in document.createElement('script');
const supportsDefer = 'defer' in document.createElement('script');

if (!supportsAsync || !supportsDefer) {
  // 回退方案: 使用 DOMContentLoaded
  document.addEventListener('DOMContentLoaded', function() {
    // 加载脚本
  });
}
```

### 7. 加载顺序总结

| 场景 | 脚本 | 执行时机 | 是否阻塞解析 |
|------|------|----------|--------------|
| 默认 | `<script>` | 立即执行 | ✅ 阻塞 |
| 默认 | `<script async>` | 下载后立即 | ❌ 不阻塞 |
| 默认 | `<script defer>` | DOM 解析后 | ❌ 不阻塞 |
| 模块 | `<script type="module">` | DOM 解析后 | ❌ 不阻塞 |
| 动态 | `createElement('script')` | 下载后立即 | ❌ 不阻塞 |

## 总结

### 选择指南

**使用 defer 当:**
- 脚本依赖 DOM
- 脚本之间有依赖关系
- 需要保证执行顺序
- 不立即执行

**使用 async 当:**
- 脚本独立执行
- 脚本不依赖其他脚本
- 脚本不被其他脚本依赖
- 想要尽早执行

**不使用 defer/async 当:**
- 脚本必须在特定位置立即执行
- 关键首屏 CSS/JS(考虑内联)
- 极少数特殊情况

### 最佳实践

1. **默认使用 defer**: 大多数脚本应该使用 `defer`
2. **独立脚本用 async**: 分析工具、广告等
3. **关键脚本内联**: 首屏必需的代码
4. **避免阻塞**: 减少普通 script 标签
5. **按需加载**: 非关键功能延迟加载
6. **监控性能**: 跟踪脚本加载时间
7. **预加载资源**: 使用 `<link rel="preload">`

通过合理使用 `defer` 和 `async`,可以显著提升页面加载性能和用户体验。