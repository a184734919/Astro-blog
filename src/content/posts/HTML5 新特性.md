---
title: HTML5 新特性
published: 2024-01-20
description: 'HTML5 新增特性和API的详细介绍和学习笔记'
image: ''
tags: ["HTML5","前端"]
category: '前端基础'
draft: false
lang: 'zh-CN'
---

## 概述

HTML5 是 HTML 的第五个主要版本，于 2014 年正式成为 W3C 推荐标准。它引入了许多新的特性、元素和 API，大大简化了 Web 开发，提升了用户体验和性能。HTML5 不仅仅是一个标记语言，更是一个完整的 Web 应用开发平台。本文将详细介绍 HTML5 的新特性、API 和实际应用。

## 核心概念

### HTML5 的主要特点

1. **语义化标签**：提供更丰富的语义化元素，提高文档的可读性和可访问性
2. **多媒体支持**：原生支持音频和视频，无需 Flash 等插件
3. **表单增强**：新增多种输入类型和验证功能
4. **图形和动画**：Canvas、SVG 等 2D/3D 图形支持
5. **本地存储**：提供多种客户端存储方案
6. **API 丰富**：地理定位、拖拽、文件操作等 API
7. **移动友好**：针对移动设备优化的特性和 API

### 浏览器兼容性

HTML5 特性的浏览器支持程度不同，开发时需要考虑兼容性：

```javascript
// 检测 HTML5 特性支持
function checkHTML5Support() {
  return {
    // 语义化标签
    semanticTags: 'header' in document.createElement('header'),

    // 多媒体
    audio: !!document.createElement('audio').canPlayType,
    video: !!document.createElement('video').canPlayType,

    // Canvas
    canvas: !!document.createElement('canvas').getContext,

    // 本地存储
    localStorage: !!window.localStorage,
    sessionStorage: !!window.sessionStorage,
    indexedDB: !!window.indexedDB,

    // 地理位置
    geolocation: 'geolocation' in navigator,

    // 文件 API
    fileAPI: !!window.File && !!window.FileReader && !!window.FileList,

    // 拖拽
    dragAndDrop: 'draggable' in document.createElement('div'),

    // Web Workers
    webWorkers: !!window.Worker,

    // WebSocket
    webSocket: !!window.WebSocket,

    // 历史管理
    historyAPI: !!(window.history && window.history.pushState)
  };
}

const support = checkHTML5Support();
console.log('HTML5 特性支持情况:', support);
```

## 语义化标签

### 新增语义化元素

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HTML5 语义化标签示例</title>
  <style>
    /* 基础样式 */
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      line-height: 1.6;
      margin: 0;
      padding: 20px;
    }

    /* 布局容器 */
    .container {
      max-width: 1200px;
      margin: 0 auto;
      display: grid;
      grid-template-columns: 200px 1fr 200px;
      gap: 20px;
    }

    /* 页头 */
    header {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 20px;
      text-align: center;
      border-radius: 8px;
      margin-bottom: 20px;
    }

    /* 导航 */
    nav {
      background: #f8f9fa;
      padding: 15px;
      border-radius: 8px;
    }

    nav ul {
      list-style: none;
      padding: 0;
      margin: 0;
    }

    nav a {
      display: block;
      padding: 10px;
      color: #333;
      text-decoration: none;
      border-radius: 4px;
      transition: background 0.3s;
    }

    nav a:hover {
      background: #e9ecef;
    }

    /* 主内容区 */
    main {
      background: white;
      padding: 20px;
      border-radius: 8px;
    }

    article {
      background: #f8f9fa;
      padding: 20px;
      border-radius: 8px;
      margin-bottom: 20px;
    }

    section {
      background: #fff;
      padding: 15px;
      border-radius: 4px;
      margin: 10px 0;
    }

    aside {
      background: #f8f9fa;
      padding: 15px;
      border-radius: 8px;
    }

    /* 侧边栏 */
    aside section {
      background: #e9ecef;
    }

    /* 页脚 */
    footer {
      background: #343a40;
      color: white;
      padding: 20px;
      text-align: center;
      border-radius: 8px;
      margin-top: 20px;
    }

    /* 其他元素 */
    figure {
      margin: 20px 0;
      text-align: center;
    }

    figure img {
      max-width: 100%;
      height: auto;
      border-radius: 8px;
    }

    figcaption {
      font-size: 0.9em;
      color: #666;
      margin-top: 10px;
    }

    mark {
      background: #fff3cd;
      padding: 2px 4px;
      border-radius: 2px;
    }

    time {
      color: #666;
      font-size: 0.9em;
    }

    progress {
      width: 100%;
      height: 20px;
      margin: 10px 0;
    }

    meter {
      width: 100%;
      height: 20px;
      margin: 10px 0;
    }

    /* 响应式设计 */
    @media (max-width: 768px) {
      .container {
        grid-template-columns: 1fr;
      }

      aside {
        display: none;
      }
    }
  </style>
</head>
<body>
  <!-- 页头：网站或页面的主要头部信息 -->
  <header>
    <h1>HTML5 语义化标签示例</h1>
    <p>使用语义化标签构建结构清晰的网页</p>
  </header>

  <!-- 导航：页面内的导航链接 -->
  <nav>
    <ul>
      <li><a href="#home">首页</a></li>
      <li><a href="#articles">文章</a></li>
      <li><a href="#about">关于</a></li>
      <li><a href="#contact">联系</a></li>
    </ul>
  </nav>

  <div class="container">
    <!-- 主内容区：页面的主要内容 -->
    <main>
      <!-- 文章：独立的内容单元 -->
      <article id="home">
        <header>
          <h2>HTML5 语义化标签的优势</h2>
          <time datetime="2024-01-20">2024年1月20日</time>
        </header>

        <section>
          <h3>提高可读性</h3>
          <p>语义化标签让代码更易读，<mark>开发者</mark>能快速理解页面结构。</p>
        </section>

        <section>
          <h3>提升可访问性</h3>
          <p>屏幕阅读器等辅助技术能更好地解析语义化内容。</p>
        </section>

        <section>
          <h3>有利于SEO</h3>
          <p>搜索引擎能更好地理解页面内容，提高搜索排名。</p>
        </section>

        <figure>
          <img src="https://via.placeholder.com/600x300?text=Semantic+HTML" alt="语义化HTML示意图">
          <figcaption>HTML5 语义化标签结构图</figcaption>
        </figure>

        <footer>
          <p>作者：张三 | 分类：前端开发</p>
        </footer>
      </article>

      <article id="articles">
        <header>
          <h2>更多文章</h2>
        </header>

        <section>
          <h3>CSS3 新特性</h3>
          <p>CSS3 提供了更强大的样式控制能力...</p>
          <footer>
            <time datetime="2024-01-15">2024年1月15日</time>
          </footer>
        </section>

        <section>
          <h3>JavaScript ES6+ 特性</h3>
          <p>现代 JavaScript 提供了更简洁的语法...</p>
          <footer>
            <time datetime="2024-01-10">2024年1月10日</time>
          </footer>
        </section>
      </article>
    </main>

    <!-- 侧边栏：页面的辅助内容 -->
    <aside>
      <section>
        <h3>热门标签</h3>
        <ul>
          <li><a href="#">HTML5</a></li>
          <li><a href="#">CSS3</a></li>
          <li><a href="#">JavaScript</a></li>
          <li><a href="#">响应式设计</a></li>
        </ul>
      </section>

      <section>
        <h3>进度演示</h3>
        <label for="progress1">下载进度：</label>
        <progress id="progress1" value="70" max="100">70%</progress>

        <label for="meter1">磁盘使用：</label>
        <meter id="meter1" value="0.7" min="0" max="1" low="0.3" high="0.7" optimum="0.5">70%</meter>
      </section>
    </aside>

    <!-- 侧边栏：广告、友情链接等 -->
    <aside>
      <section>
        <h3>友情链接</h3>
        <ul>
          <li><a href="#">MDN Web Docs</a></li>
          <li><a href="#">W3Schools</a></li>
          <li><a href="#">Stack Overflow</a></li>
        </ul>
      </section>

      <section>
        <h3>关于作者</h3>
        <p>全栈开发者，专注于 Web 前端技术...</p>
      </section>
    </aside>
  </div>

  <!-- 页脚：页面的页脚信息 -->
  <footer>
    <p>&copy; 2024 HTML5 示例网站. 保留所有权利。</p>
    <p>使用语义化标签构建 | 遵循 HTML5 标准</p>
  </footer>
</body>
</html>
```

### 文档大纲和可访问性

```javascript
// 使用 HTML5 构建文档大纲
function createDocumentOutline() {
  const outline = [];
  const sections = document.querySelectorAll('section, article, nav, aside');

  sections.forEach(section => {
    const heading = section.querySelector('h1, h2, h3, h4, h5, h6');
    if (heading) {
      outline.push({
        level: parseInt(heading.tagName[1]),
        text: heading.textContent,
        element: section
      });
    }
  });

  return outline;
}

// 为可访问性增强页面
function enhanceAccessibility() {
  // 为所有图片添加 alt 属性
  document.querySelectorAll('img:not([alt])').forEach(img => {
    img.alt = '图片描述';
  });

  // 为表单元素添加标签
  document.querySelectorAll('input:not([aria-label]):not([id])').forEach(input => {
    const label = document.createElement('label');
    label.textContent = input.placeholder || '输入框';
    label.htmlFor = `input-${Date.now()}`;
    input.id = label.htmlFor;
    input.parentNode.insertBefore(label, input);
  });

  // 添加 ARIA 属性
  const nav = document.querySelector('nav');
  if (nav && !nav.getAttribute('aria-label')) {
    nav.setAttribute('aria-label', '主导航');
  }

  const main = document.querySelector('main');
  if (main && !main.getAttribute('role')) {
    main.setAttribute('role', 'main');
  }
}
```

## 表单增强

### 新增输入类型

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HTML5 表单新特性</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 20px auto;
      padding: 20px;
    }

    form {
      background: #f8f9fa;
      padding: 30px;
      border-radius: 8px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    .form-group {
      margin-bottom: 20px;
    }

    label {
      display: block;
      margin-bottom: 8px;
      font-weight: 500;
      color: #333;
    }

    input,
    select,
    textarea {
      width: 100%;
      padding: 10px;
      border: 2px solid #ddd;
      border-radius: 4px;
      font-size: 16px;
      box-sizing: border-box;
      transition: border-color 0.3s;
    }

    input:focus,
    select:focus,
    textarea:focus {
      outline: none;
      border-color: #667eea;
    }

    input:valid {
      border-color: #28a745;
    }

    input:invalid {
      border-color: #dc3545;
    }

    .error-message {
      color: #dc3545;
      font-size: 14px;
      margin-top: 5px;
      display: none;
    }

    input:invalid + .error-message {
      display: block;
    }

    .range-container {
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .range-value {
      font-weight: bold;
      min-width: 50px;
      text-align: center;
    }

    .color-preview {
      width: 50px;
      height: 50px;
      border-radius: 4px;
      border: 2px solid #ddd;
      margin-top: 10px;
    }

    button {
      background: #667eea;
      color: white;
      padding: 12px 24px;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      font-size: 16px;
      transition: background 0.3s;
    }

    button:hover {
      background: #5568d3;
    }

    .output {
      margin-top: 20px;
      padding: 15px;
      background: white;
      border-radius: 4px;
      border: 2px solid #ddd;
    }

    @media (max-width: 600px) {
      form {
        padding: 20px;
      }

      .form-group {
        margin-bottom: 15px;
      }
    }
  </style>
</head>
<body>
  <h1>HTML5 表单新特性演示</h1>

  <form id="enhancedForm">
    <!-- 1. email 类型：邮箱验证 -->
    <div class="form-group">
      <label for="email">邮箱地址：</label>
      <input
        type="email"
        id="email"
        name="email"
        required
        placeholder="your@example.com"
        autocomplete="email"
      >
      <span class="error-message">请输入有效的邮箱地址</span>
    </div>

    <!-- 2. tel 类型：电话号码 -->
    <div class="form-group">
      <label for="phone">电话号码：</label>
      <input
        type="tel"
        id="phone"
        name="phone"
        pattern="[0-9]{11}"
        required
        placeholder="11位手机号码"
        autocomplete="tel"
      >
      <span class="error-message">请输入11位手机号码</span>
    </div>

    <!-- 3. url 类型：网址 -->
    <div class="form-group">
      <label for="website">个人网站：</label>
      <input
        type="url"
        id="website"
        name="website"
        placeholder="https://example.com"
        autocomplete="url"
      >
      <span class="error-message">请输入有效的网址</span>
    </div>

    <!-- 4. number 类型：数字输入 -->
    <div class="form-group">
      <label for="age">年龄：</label>
      <input
        type="number"
        id="age"
        name="age"
        min="18"
        max="100"
        step="1"
        required
        placeholder="18-100"
      >
      <span class="error-message">年龄必须在18-100之间</span>
    </div>

    <!-- 5. range 类型：滑块 -->
    <div class="form-group">
      <label for="volume">音量：</label>
      <div class="range-container">
        <input
          type="range"
          id="volume"
          name="volume"
          min="0"
          max="100"
          value="50"
        >
        <span class="range-value" id="volumeValue">50</span>
      </div>
    </div>

    <!-- 6. date 类型：日期选择 -->
    <div class="form-group">
      <label for="birthday">出生日期：</label>
      <input
        type="date"
        id="birthday"
        name="birthday"
        required
      >
      <span class="error-message">请选择出生日期</span>
    </div>

    <!-- 7. time 类型：时间选择 -->
    <div class="form-group">
      <label for="meetingTime">会议时间：</label>
      <input
        type="time"
        id="meetingTime"
        name="meetingTime"
        required
      >
      <span class="error-message">请选择会议时间</span>
    </div>

    <!-- 8. datetime-local 类型：本地日期时间 -->
    <div class="form-group">
      <label for="eventDateTime">活动时间：</label>
      <input
        type="datetime-local"
        id="eventDateTime"
        name="eventDateTime"
        required
      >
      <span class="error-message">请选择活动时间</span>
    </div>

    <!-- 9. month 类型：月份选择 -->
    <div class="form-group">
      <label for="creditCardMonth">信用卡有效期：</label>
      <input
        type="month"
        id="creditCardMonth"
        name="creditCardMonth"
        required
      >
      <span class="error-message">请选择有效期月份</span>
    </div>

    <!-- 10. week 类型：周选择 -->
    <div class="form-group">
      <label for="weekNumber">周数：</label>
      <input
        type="week"
        id="weekNumber"
        name="weekNumber"
      >
    </div>

    <!-- 11. color 类型：颜色选择器 -->
    <div class="form-group">
      <label for="themeColor">主题颜色：</label>
      <input
        type="color"
        id="themeColor"
        name="themeColor"
        value="#667eea"
      >
      <div class="color-preview" id="colorPreview" style="background-color: #667eea;"></div>
    </div>

    <!-- 12. search 类型：搜索框 -->
    <div class="form-group">
      <label for="search">搜索：</label>
      <input
        type="search"
        id="search"
        name="search"
        placeholder="搜索内容..."
        autocomplete="off"
      >
    </div>

    <!-- 新增属性演示 -->
    <div class="form-group">
      <label for="username">用户名：</label>
      <input
        type="text"
        id="username"
        name="username"
        required
        pattern="[a-zA-Z0-9_]{3,20}"
        placeholder="3-20个字母、数字或下划线"
        autocomplete="username"
        title="用户名必须是3-20个字母、数字或下划线"
      >
      <span class="error-message">用户名必须是3-20个字母、数字或下划线</span>
    </div>

    <div class="form-group">
      <label for="password">密码：</label>
      <input
        type="password"
        id="password"
        name="password"
        required
        minlength="8"
        maxlength="20"
        autocomplete="new-password"
        placeholder="8-20个字符"
      >
      <span class="error-message">密码必须是8-20个字符</span>
    </div>

    <div class="form-group">
      <label for="confirmPassword">确认密码：</label>
      <input
        type="password"
        id="confirmPassword"
        name="confirmPassword"
        required
        autocomplete="new-password"
        placeholder="再次输入密码"
      >
      <span class="error-message">两次输入的密码不一致</span>
    </div>

    <div class="form-group">
      <label for="websiteUrl">个人博客：</label>
      <input
        type="url"
        id="websiteUrl"
        name="websiteUrl"
        placeholder="https://example.com"
        list="websiteList"
      >
      <datalist id="websiteList">
        <option value="https://github.com">
        <option value="https://stackoverflow.com">
        <option value="https://developer.mozilla.org">
      </datalist>
    </div>

    <div class="form-group">
      <label for="country">国家/地区：</label>
      <select id="country" name="country" required>
        <option value="">请选择</option>
        <option value="CN">中国</option>
        <option value="US">美国</option>
        <option value="UK">英国</option>
        <option value="JP">日本</option>
      </select>
      <span class="error-message">请选择国家/地区</span>
    </div>

    <div class="form-group">
      <label for="comments">备注：</label>
      <textarea
        id="comments"
        name="comments"
        rows="4"
        cols="50"
        placeholder="请输入备注信息..."
      ></textarea>
    </div>

    <button type="submit">提交表单</button>
  </form>

  <div class="output" id="formOutput">
    <h3>表单提交结果：</h3>
    <p>填写并提交表单后，结果将显示在这里。</p>
  </div>

  <script>
    // 范围滑块值显示
    const volumeInput = document.getElementById('volume');
    const volumeValue = document.getElementById('volumeValue');

    volumeInput.addEventListener('input', function() {
      volumeValue.textContent = this.value;
    });

    // 颜色选择器预览
    const colorInput = document.getElementById('themeColor');
    const colorPreview = document.getElementById('colorPreview');

    colorInput.addEventListener('input', function() {
      colorPreview.style.backgroundColor = this.value;
    });

    // 表单验证和提交
    const form = document.getElementById('enhancedForm');
    const formOutput = document.getElementById('formOutput');

    form.addEventListener('submit', function(e) {
      e.preventDefault();

      // 自定义验证：确认密码
      const password = document.getElementById('password').value;
      const confirmPassword = document.getElementById('confirmPassword').value;

      if (password !== confirmPassword) {
        alert('两次输入的密码不一致！');
        return;
      }

      // 收集表单数据
      const formData = new FormData(form);
      const formDataObj = {};

      formData.forEach((value, key) => {
        formDataObj[key] = value;
      });

      // 显示结果
      formOutput.innerHTML = `
        <h3>表单提交结果：</h3>
        <pre>${JSON.stringify(formDataObj, null, 2)}</pre>
      `;

      // 检查表单有效性
      if (form.checkValidity()) {
        console.log('表单验证通过');
        console.log('表单数据:', formDataObj);
      } else {
        console.log('表单验证失败');
      }
    });

    // 实时验证反馈
    const inputs = form.querySelectorAll('input, select, textarea');
    inputs.forEach(input => {
      input.addEventListener('input', function() {
        if (this.checkValidity()) {
          this.classList.remove('invalid');
          this.classList.add('valid');
        } else {
          this.classList.remove('valid');
          this.classList.add('invalid');
        }
      });

      input.addEventListener('blur', function() {
        if (!this.checkValidity()) {
          this.classList.add('invalid');
          this.reportValidity();
        }
      });
    });
  </script>
</body>
</html>
```

### 表单验证 API

```javascript
// HTML5 表单验证 API
const form = document.getElementById('myForm');
const input = document.getElementById('username');

// 1. checkValidity() - 检查表单元素是否有效
function validateInput(input) {
  if (input.checkValidity()) {
    console.log('输入有效');
    return true;
  } else {
    console.log('输入无效');
    input.reportValidity(); // 显示浏览器默认验证消息
    return false;
  }
}

// 2. setCustomValidity() - 设置自定义验证消息
function customValidate(input) {
  if (input.value.length < 3) {
    input.setCustomValidity('至少需要3个字符');
  } else {
    input.setCustomValidity(''); // 清除自定义验证消息
  }
}

// 3. validity 属性 - 访问验证状态
function checkValidityStatus(input) {
  const validity = input.validity;

  console.log('总体有效性:', validity.valid);
  console.log('是否为空:', validity.valueMissing);
  console.log('类型不匹配:', validity.typeMismatch);
  console.log('格式不匹配:', validity.patternMismatch);
  console.log('过长:', validity.tooLong);
  console.log('过短:', validity.tooShort);
  console.log('超出范围:', validity.rangeOverflow);
  console.log('低于范围:', validity.rangeUnderflow);
  console.log('不匹配步骤:', validity.stepMismatch);
  console.log('自定义验证失败:', validity.customError);
}

// 4. 表单提交事件
form.addEventListener('submit', function(e) {
  e.preventDefault();

  if (form.checkValidity()) {
    // 表单验证通过
    const formData = new FormData(form);
    console.log('表单数据:', Object.fromEntries(formData));
  } else {
    // 表单验证失败
    form.reportValidity();
  }
});

// 5. 实时验证
input.addEventListener('input', function() {
  customValidate(this);
  if (this.checkValidity()) {
    this.classList.remove('invalid');
    this.classList.add('valid');
  } else {
    this.classList.remove('valid');
    this.classList.add('invalid');
  }
});

// 6. 约束验证 API
const constraints = {
  username: {
    presence: true,
    length: { minimum: 3, maximum: 20 }
  },
  email: {
    presence: true,
    email: true
  },
  age: {
    presence: true,
    numericality: { greaterThanOrEqualTo: 18, lessThanOrEqualTo: 100 }
  }
};

function validateConstraints(data, constraints) {
  const errors = {};

  Object.keys(constraints).forEach(field => {
    const constraint = constraints[field];
    const value = data[field];

    if (constraint.presence && !value) {
      errors[field] = `${field} 是必填项`;
      return;
    }

    if (constraint.length) {
      if (value.length < constraint.length.minimum) {
        errors[field] = `${field} 至少需要 ${constraint.length.minimum} 个字符`;
      }
      if (value.length > constraint.length.maximum) {
        errors[field] = `${field} 最多 ${constraint.length.maximum} 个字符`;
      }
    }

    if (constraint.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
      errors[field] = `${field} 格式不正确`;
    }

    if (constraint.numericality) {
      const num = parseInt(value);
      if (constraint.numericality.greaterThanOrEqualTo && num < constraint.numericality.greaterThanOrEqualTo) {
        errors[field] = `${field} 必须大于等于 ${constraint.numericality.greaterThanOrEqualTo}`;
      }
      if (constraint.numericality.lessThanOrEqualTo && num > constraint.numericality.lessThanOrEqualTo) {
        errors[field] = `${field} 必须小于等于 ${constraint.numericality.lessThanOrEqualTo}`;
      }
    }
  });

  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}
```

## 多媒体支持

### 音频和视频

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HTML5 多媒体支持</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 20px auto;
      padding: 20px;
    }

    .media-container {
      background: #f8f9fa;
      padding: 30px;
      border-radius: 8px;
      margin-bottom: 30px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    h2 {
      color: #333;
      margin-bottom: 20px;
    }

    audio, video {
      width: 100%;
      border-radius: 4px;
    }

    .controls {
      margin-top: 15px;
      display: flex;
      gap: 10px;
      flex-wrap: wrap;
    }

    button {
      padding: 8px 16px;
      background: #667eea;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      transition: background 0.3s;
    }

    button:hover {
      background: #5568d3;
    }

    button:disabled {
      background: #ccc;
      cursor: not-allowed;
    }

    .video-info {
      margin-top: 15px;
      padding: 15px;
      background: white;
      border-radius: 4px;
    }

    .video-info p {
      margin: 5px 0;
    }

    .volume-control {
      display: flex;
      align-items: center;
      gap: 10px;
      margin-top: 10px;
    }

    .volume-control input {
      flex: 1;
    }

    .playback-rate {
      margin-top: 10px;
    }

    .playback-rate select {
      padding: 5px;
      border-radius: 4px;
    }

    @media (max-width: 600px) {
      .media-container {
        padding: 20px;
      }

      .controls {
        flex-direction: column;
      }
    }
  </style>
</head>
<body>
  <h1>HTML5 多媒体支持演示</h1>

  <!-- 音频播放器 -->
  <div class="media-container">
    <h2>音频播放器</h2>
    <audio id="audioPlayer" controls>
      <source src="audio.mp3" type="audio/mpeg">
      <source src="audio.ogg" type="audio/ogg">
      <source src="audio.wav" type="audio/wav">
      您的浏览器不支持 HTML5 音频播放。
    </audio>

    <div class="controls">
      <button id="audioPlay">播放</button>
      <button id="audioPause">暂停</button>
      <button id="audioStop">停止</button>
      <button id="audioVolumeUp">音量+</button>
      <button id="audioVolumeDown">音量-</button>
    </div>

    <div class="volume-control">
      <label for="audioVolume">音量：</label>
      <input type="range" id="audioVolume" min="0" max="1" step="0.1" value="1">
      <span id="audioVolumeValue">100%</span>
    </div>
  </div>

  <!-- 视频播放器 -->
  <div class="media-container">
    <h2>视频播放器</h2>
    <video id="videoPlayer" width="640" height="360" poster="poster.jpg" controls>
      <source src="video.mp4" type="video/mp4">
      <source src="video.webm" type="video/webm">
      <source src="video.ogg" type="video/ogg">
      <track kind="subtitles" src="subtitles_zh.vtt" srclang="zh" label="中文字幕">
      <track kind="subtitles" src="subtitles_en.vtt" srclang="en" label="English Subtitles">
      您的浏览器不支持 HTML5 视频播放。
    </video>

    <div class="controls">
      <button id="videoPlay">播放</button>
      <button id="videoPause">暂停</button>
      <button id="videoStop">停止</button>
      <button id="videoMute">静音</button>
      <button id="videoFullscreen">全屏</button>
    </div>

    <div class="volume-control">
      <label for="videoVolume">音量：</label>
      <input type="range" id="videoVolume" min="0" max="1" step="0.1" value="1">
      <span id="videoVolumeValue">100%</span>
    </div>

    <div class="playback-rate">
      <label for="playbackRate">播放速度：</label>
      <select id="playbackRate">
        <option value="0.5">0.5x</option>
        <option value="0.75">0.75x</option>
        <option value="1" selected>1x</option>
        <option value="1.25">1.25x</option>
        <option value="1.5">1.5x</option>
        <option value="2">2x</option>
      </select>
    </div>

    <div class="video-info">
      <h3>视频信息</h3>
      <p>当前时间: <span id="currentTime">0:00</span></p>
      <p>总时长: <span id="duration">0:00</span></p>
      <p>播放状态: <span id="playbackState">未播放</span></p>
      <p>网络状态: <span id="networkState">未知</span></p>
    </div>
  </div>

  <script>
    // 音频播放器控制
    const audio = document.getElementById('audioPlayer');
    const audioPlay = document.getElementById('audioPlay');
    const audioPause = document.getElementById('audioPause');
    const audioStop = document.getElementById('audioStop');
    const audioVolumeUp = document.getElementById('audioVolumeUp');
    const audioVolumeDown = document.getElementById('audioVolumeDown');
    const audioVolume = document.getElementById('audioVolume');
    const audioVolumeValue = document.getElementById('audioVolumeValue');

    audioPlay.addEventListener('click', () => audio.play());
    audioPause.addEventListener('click', () => audio.pause());
    audioStop.addEventListener('click', () => {
      audio.pause();
      audio.currentTime = 0;
    });

    audioVolumeUp.addEventListener('click', () => {
      audio.volume = Math.min(1, audio.volume + 0.1);
      audioVolume.value = audio.volume;
      audioVolumeValue.textContent = Math.round(audio.volume * 100) + '%';
    });

    audioVolumeDown.addEventListener('click', () => {
      audio.volume = Math.max(0, audio.volume - 0.1);
      audioVolume.value = audio.volume;
      audioVolumeValue.textContent = Math.round(audio.volume * 100) + '%';
    });

    audioVolume.addEventListener('input', (e) => {
      audio.volume = parseFloat(e.target.value);
      audioVolumeValue.textContent = Math.round(audio.volume * 100) + '%';
    });

    // 视频播放器控制
    const video = document.getElementById('videoPlayer');
    const videoPlay = document.getElementById('videoPlay');
    const videoPause = document.getElementById('videoPause');
    const videoStop = document.getElementById('videoStop');
    const videoMute = document.getElementById('videoMute');
    const videoFullscreen = document.getElementById('videoFullscreen');
    const videoVolume = document.getElementById('videoVolume');
    const videoVolumeValue = document.getElementById('videoVolumeValue');
    const playbackRate = document.getElementById('playbackRate');

    const currentTime = document.getElementById('currentTime');
    const duration = document.getElementById('duration');
    const playbackState = document.getElementById('playbackState');
    const networkState = document.getElementById('networkState');

    // 格式化时间
    function formatTime(seconds) {
      const mins = Math.floor(seconds / 60);
      const secs = Math.floor(seconds % 60);
      return `${mins}:${secs.toString().padStart(2, '0')}`;
    }

    // 视频控制事件
    videoPlay.addEventListener('click', () => video.play());
    videoPause.addEventListener('click', () => video.pause());
    videoStop.addEventListener('click', () => {
      video.pause();
      video.currentTime = 0;
    });
    videoMute.addEventListener('click', () => video.muted = !video.muted);
    videoFullscreen.addEventListener('click', () => {
      if (video.requestFullscreen) {
        video.requestFullscreen();
      } else if (video.webkitRequestFullscreen) {
        video.webkitRequestFullscreen();
      }
    });

    videoVolume.addEventListener('input', (e) => {
      video.volume = parseFloat(e.target.value);
      videoVolumeValue.textContent = Math.round(video.volume * 100) + '%';
    });

    playbackRate.addEventListener('change', (e) => {
      video.playbackRate = parseFloat(e.target.value);
    });

    // 视频事件监听
    video.addEventListener('timeupdate', () => {
      currentTime.textContent = formatTime(video.currentTime);
    });

    video.addEventListener('loadedmetadata', () => {
      duration.textContent = formatTime(video.duration);
    });

    video.addEventListener('play', () => {
      playbackState.textContent = '播放中';
      videoPlay.disabled = true;
      videoPause.disabled = false;
    });

    video.addEventListener('pause', () => {
      playbackState.textContent = '已暂停';
      videoPlay.disabled = false;
      videoPause.disabled = true;
    });

    video.addEventListener('ended', () => {
      playbackState.textContent = '播放结束';
      videoPlay.disabled = false;
      videoPause.disabled = true;
    });

    video.addEventListener('waiting', () => {
      playbackState.textContent = '缓冲中...';
    });

    // 网络状态监听
    const networkStates = ['NETWORK_EMPTY', 'NETWORK_IDLE', 'NETWORK_LOADING', 'NETWORK_NO_SOURCE'];
    video.addEventListener('networkstatechange', () => {
      networkState.textContent = networkStates[video.networkState];
    });

    // 检测浏览器支持
    function checkMediaSupport() {
      const audioSupport = !!document.createElement('audio').canPlayType;
      const videoSupport = !!document.createElement('video').canPlayType;

      console.log('音频支持:', audioSupport);
      console.log('视频支持:', videoSupport);

      if (audioSupport) {
        console.log('支持的音频格式:');
        console.log('MP3:', !!document.createElement('audio').canPlayType('audio/mpeg'));
        console.log('OGG:', !!document.createElement('audio').canPlayType('audio/ogg'));
        console.log('WAV:', !!document.createElement('audio').canPlayType('audio/wav'));
      }

      if (videoSupport) {
        console.log('支持的视频格式:');
        console.log('MP4:', !!document.createElement('video').canPlayType('video/mp4'));
        console.log('WebM:', !!document.createElement('video').canPlayType('video/webm'));
        console.log('OGG:', !!document.createElement('video').canPlayType('video/ogg'));
      }
    }

    checkMediaSupport();
  </script>
</body>
</html>
```

## 本地存储

### LocalStorage 和 SessionStorage

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>HTML5 本地存储</title>
  <style>
    body {
      font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
      line-height: 1.6;
      max-width: 800px;
      margin: 20px auto;
      padding: 20px;
    }

    .storage-container {
      background: #f8f9fa;
      padding: 30px;
      border-radius: 8px;
      margin-bottom: 30px;
      box-shadow: 0 2px 4px rgba(0,0,0,0.1);
    }

    h2 {
      color: #333;
      margin-bottom: 20px;
    }

    .form-group {
      margin-bottom: 15px;
    }

    label {
      display: block;
      margin-bottom: 5px;
      font-weight: 500;
    }

    input, textarea {
      width: 100%;
      padding: 10px;
      border: 2px solid #ddd;
      border-radius: 4px;
      box-sizing: border-box;
    }

    button {
      padding: 10px 20px;
      background: #667eea;
      color: white;
      border: none;
      border-radius: 4px;
      cursor: pointer;
      margin-right: 10px;
      transition: background 0.3s;
    }

    button:hover {
      background: #5568d3;
    }

    button.danger {
      background: #dc3545;
    }

    button.danger:hover {
      background: #c82333;
    }

    .storage-data {
      margin-top: 20px;
      padding: 15px;
      background: white;
      border-radius: 4px;
      border: 1px solid #ddd;
    }

    .storage-item {
      padding: 10px;
      border-bottom: 1px solid #eee;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .storage-item:last-child {
      border-bottom: none;
    }

    .storage-key {
      font-weight: bold;
      color: #667eea;
    }

    .storage-value {
      margin-left: 10px;
      color: #333;
    }

    .empty-message {
      color: #999;
      text-align: center;
      padding: 20px;
    }

    .toast {
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 15px 25px;
      background: #28a745;
      color: white;
      border-radius: 4px;
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
      opacity: 0;
      transition: opacity 0.3s;
      z-index: 1000;
    }

    .toast.show {
      opacity: 1;
    }

    .stats {
      display: flex;
      gap: 20px;
      margin-bottom: 20px;
    }

    .stat-item {
      flex: 1;
      padding: 15px;
      background: white;
      border-radius: 4px;
      text-align: center;
    }

    .stat-value {
      font-size: 24px;
      font-weight: bold;
      color: #667eea;
    }

    .stat-label {
      font-size: 14px;
      color: #666;
      margin-top: 5px;
    }
  </style>
</head>
<body>
  <h1>HTML5 本地存储演示</h1>

  <!-- LocalStorage -->
  <div class="storage-container">
    <h2>LocalStorage（本地存储）</h2>
    <div class="stats">
      <div class="stat-item">
        <div class="stat-value" id="localStorageCount">0</div>
        <div class="stat-label">存储项数量</div>
      </div>
      <div class="stat-item">
        <div class="stat-value" id="localStorageSize">0</div>
        <div class="stat-label">存储大小（字节）</div>
      </div>
    </div>

    <div class="form-group">
      <label for="localStorageKey">键名：</label>
      <input type="text" id="localStorageKey" placeholder="输入键名">
    </div>

    <div class="form-group">
      <label for="localStorageValue">值：</label>
      <textarea id="localStorageValue" rows="3" placeholder="输入值"></textarea>
    </div>

    <button onclick="saveToLocalStorage()">保存</button>
    <button onclick="clearLocalStorage()" class="danger">清空所有</button>

    <div class="storage-data" id="localStorageData">
      <p class="empty-message">暂无存储数据</p>
    </div>
  </div>

  <!-- SessionStorage -->
  <div class="storage-container">
    <h2>SessionStorage（会话存储）</h2>
    <div class="stats">
      <div class="stat-item">
        <div class="stat-value" id="sessionStorageCount">0</div>
        <div class="stat-label">存储项数量</div>
      </div>
      <div class="stat-item">
        <div class="stat-value" id="sessionStorageSize">0</div>
        <div class="stat-label">存储大小（字节）</div>
      </div>
    </div>

    <div class="form-group">
      <label for="sessionStorageKey">键名：</label>
      <input type="text" id="sessionStorageKey" placeholder="输入键名">
    </div>

    <div class="form-group">
      <label for="sessionStorageValue">值：</label>
      <textarea id="sessionStorageValue" rows="3" placeholder="输入值"></textarea>
    </div>

    <button onclick="saveToSessionStorage()">保存</button>
    <button onclick="clearSessionStorage()" class="danger">清空所有</button>

    <div class="storage-data" id="sessionStorageData">
      <p class="empty-message">暂无存储数据</p>
    </div>
  </div>

  <!-- 高级用法 -->
  <div class="storage-container">
    <h2>高级用法</h2>

    <div class="form-group">
      <label>用户偏好设置：</label>
      <div>
        <label>
          <input type="checkbox" id="darkMode" onchange="savePreferences()"> 暗黑模式
        </label>
        <label>
          <input type="checkbox" id="notifications" onchange="savePreferences()"> 启用通知
        </label>
      </div>
    </div>

    <div class="form-group">
      <label for="language">语言：</label>
      <select id="language" onchange="savePreferences()">
        <option value="zh-CN">中文</option>
        <option value="en-US">English</option>
        <option value="ja-JP">日本語</option>
      </select>
    </div>

    <button onclick="loadPreferences()">加载偏好设置</button>
    <button onclick="resetPreferences()" class="danger">重置偏好设置</button>
  </div>

  <div class="toast" id="toast">操作成功！</div>

  <script>
    // 检查浏览器支持
    function checkStorageSupport() {
      try {
        localStorage.setItem('test', 'test');
        localStorage.removeItem('test');
        return true;
      } catch (e) {
        console.error('LocalStorage 不支持或已禁用:', e);
        return false;
      }
    }

    if (!checkStorageSupport()) {
      alert('您的浏览器不支持本地存储功能！');
    }

    // LocalStorage 操作
    function saveToLocalStorage() {
      const key = document.getElementById('localStorageKey').value.trim();
      const value = document.getElementById('localStorageValue').value.trim();

      if (!key) {
        alert('请输入键名！');
        return;
      }

      try {
        localStorage.setItem(key, value);
        showToast('保存成功！');
        updateLocalStorageDisplay();

        // 清空输入框
        document.getElementById('localStorageKey').value = '';
        document.getElementById('localStorageValue').value = '';
      } catch (e) {
        if (e.name === 'QuotaExceededError') {
          alert('存储空间已满！');
        } else {
          alert('保存失败：' + e.message);
        }
      }
    }

    function clearLocalStorage() {
      if (confirm('确定要清空所有 LocalStorage 数据吗？')) {
        localStorage.clear();
        showToast('已清空所有数据！');
        updateLocalStorageDisplay();
      }
    }

    function deleteFromLocalStorage(key) {
      if (confirm(`确定要删除 "${key}" 吗？`)) {
        localStorage.removeItem(key);
        showToast('删除成功！');
        updateLocalStorageDisplay();
      }
    }

    function updateLocalStorageDisplay() {
      const container = document.getElementById('localStorageData');
      const countEl = document.getElementById('localStorageCount');
      const sizeEl = document.getElementById('localStorageSize');

      // 更新统计信息
      const count = localStorage.length;
      let size = 0;

      for (let i = 0; i < count; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);
        size += (key + value).length;
      }

      countEl.textContent = count;
      sizeEl.textContent = size;

      // 更新数据显示
      if (count === 0) {
        container.innerHTML = '<p class="empty-message">暂无存储数据</p>';
        return;
      }

      let html = '';
      for (let i = 0; i < count; i++) {
        const key = localStorage.key(i);
        const value = localStorage.getItem(key);

        html += `
          <div class="storage-item">
            <div>
              <span class="storage-key">${escapeHtml(key)}</span>:
              <span class="storage-value">${escapeHtml(value)}</span>
            </div>
            <button onclick="deleteFromLocalStorage('${escapeHtml(key)}')" class="danger" style="padding: 5px 10px;">删除</button>
          </div>
        `;
      }

      container.innerHTML = html;
    }

    // SessionStorage 操作
    function saveToSessionStorage() {
      const key = document.getElementById('sessionStorageKey').value.trim();
      const value = document.getElementById('sessionStorageValue').value.trim();

      if (!key) {
        alert('请输入键名！');
        return;
      }

      try {
        sessionStorage.setItem(key, value);
        showToast('保存成功！');
        updateSessionStorageDisplay();

        document.getElementById('sessionStorageKey').value = '';
        document.getElementById('sessionStorageValue').value = '';
      } catch (e) {
        alert('保存失败：' + e.message);
      }
    }

    function clearSessionStorage() {
      if (confirm('确定要清空所有 SessionStorage 数据吗？')) {
        sessionStorage.clear();
        showToast('已清空所有数据！');
        updateSessionStorageDisplay();
      }
    }

    function deleteFromSessionStorage(key) {
      if (confirm(`确定要删除 "${key}" 吗？`)) {
        sessionStorage.removeItem(key);
        showToast('删除成功！');
        updateSessionStorageDisplay();
      }
    }

    function updateSessionStorageDisplay() {
      const container = document.getElementById('sessionStorageData');
      const countEl = document.getElementById('sessionStorageCount');
      const sizeEl = document.getElementById('sessionStorageSize');

      const count = sessionStorage.length;
      let size = 0;

      for (let i = 0; i < count; i++) {
        const key = sessionStorage.key(i);
        const value = sessionStorage.getItem(key);
        size += (key + value).length;
      }

      countEl.textContent = count;
      sizeEl.textContent = size;

      if (count === 0) {
        container.innerHTML = '<p class="empty-message">暂无存储数据</p>';
        return;
      }

      let html = '';
      for (let i = 0; i < count; i++) {
        const key = sessionStorage.key(i);
        const value = sessionStorage.getItem(key);

        html += `
          <div class="storage-item">
            <div>
              <span class="storage-key">${escapeHtml(key)}</span>:
              <span class="storage-value">${escapeHtml(value)}</span>
            </div>
            <button onclick="deleteFromSessionStorage('${escapeHtml(key)}')" class="danger" style="padding: 5px 10px;">删除</button>
          </div>
        `;
      }

      container.innerHTML = html;
    }

    // 偏好设置管理
    function savePreferences() {
      const preferences = {
        darkMode: document.getElementById('darkMode').checked,
        notifications: document.getElementById('notifications').checked,
        language: document.getElementById('language').value,
        lastUpdated: new Date().toISOString()
      };

      try {
        localStorage.setItem('userPreferences', JSON.stringify(preferences));
        showToast('偏好设置已保存！');
      } catch (e) {
        alert('保存失败：' + e.message);
      }
    }

    function loadPreferences() {
      try {
        const stored = localStorage.getItem('userPreferences');
        if (stored) {
          const preferences = JSON.parse(stored);

          document.getElementById('darkMode').checked = preferences.darkMode || false;
          document.getElementById('notifications').checked = preferences.notifications || false;
          document.getElementById('language').value = preferences.language || 'zh-CN';

          showToast('偏好设置已加载！');
          console.log('加载的偏好设置:', preferences);
        } else {
          alert('没有找到保存的偏好设置！');
        }
      } catch (e) {
        alert('加载失败：' + e.message);
      }
    }

    function resetPreferences() {
      if (confirm('确定要重置所有偏好设置吗？')) {
        localStorage.removeItem('userPreferences');
        document.getElementById('darkMode').checked = false;
        document.getElementById('notifications').checked = false;
        document.getElementById('language').value = 'zh-CN';
        showToast('偏好设置已重置！');
      }
    }

    // 工具函数
    function showToast(message) {
      const toast = document.getElementById('toast');
      toast.textContent = message;
      toast.classList.add('show');

      setTimeout(() => {
        toast.classList.remove('show');
      }, 3000);
    }

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.textContent = text;
      return div.innerHTML;
    }

    // 监听存储变化（跨标签页同步）
    window.addEventListener('storage', (e) => {
      console.log('存储发生变化:', {
        key: e.key,
        oldValue: e.oldValue,
        newValue: e.newValue,
        url: e.url,
        storageArea: e.storageArea
      });

      if (e.storageArea === localStorage) {
        updateLocalStorageDisplay();
      } else if (e.storageArea === sessionStorage) {
        updateSessionStorageDisplay();
      }
    });

    // 初始化显示
    updateLocalStorageDisplay();
    updateSessionStorageDisplay();
    loadPreferences();
  </script>
</body>
</html>
```

## 注意事项

### 1. 浏览器兼容性

```javascript
// HTML5 特性检测和 polyfill
function detectHTML5Features() {
  const features = {
    // 语义化标签
    semanticTags: 'header' in document.createElement('header'),

    // 多媒体
    audio: !!document.createElement('audio').canPlayType,
    video: !!document.createElement('video').canPlayType,

    // Canvas
    canvas: !!document.createElement('canvas').getContext,

    // SVG
    svg: !!document.createElementNS && !!document.createElementNS('http://www.w3.org/2000/svg', 'svg').createSVGRect,

    // 存储
    localStorage: (function() {
      try {
        return !!localStorage;
      } catch (e) {
        return false;
      }
    })(),

    // 地理位置
    geolocation: 'geolocation' in navigator,

    // 文件 API
    fileAPI: !!(window.File && window.FileReader && window.FileList && window.Blob),

    // 拖拽
    dragAndDrop: 'draggable' in document.createElement('div'),

    // 历史管理
    historyAPI: !!(window.history && window.history.pushState && window.history.replaceState),

    // Web Workers
    webWorkers: !!window.Worker,

    // WebSocket
    webSocket: !!window.WebSocket,

    // 通知
    notifications: 'Notification' in window,

    // 触摸事件
    touchEvents: 'ontouchstart' in window
  };

  return features;
}

// 为不支持的浏览器提供 fallback
function provideFallbacks(features) {
  if (!features.semanticTags) {
    console.log('语义化标签不支持，使用 polyfill');
    // 可以使用 html5shiv 等库
  }

  if (!features.localStorage) {
    console.log('localStorage 不支持，使用 cookie 替代');
    // 实现基于 cookie 的存储方案
  }

  if (!features.geolocation) {
    console.log('地理位置不支持，使用 IP 定位替代方案');
  }
}

const features = detectHTML5Features();
console.log('HTML5 特性支持情况:', features);
provideFallbacks(features);
```

### 2. 性能优化

```javascript
// 多媒体资源优化
function optimizeMedia() {
  // 懒加载图片
  const images = document.querySelectorAll('img[data-src]');
  const imageObserver = new IntersectionObserver((entries, observer) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const img = entry.target;
        img.src = img.dataset.src;
        observer.unobserve(img);
      }
    });
  });

  images.forEach(img => imageObserver.observe(img));

  // 预加载关键资源
  const preloadLinks = document.createElement('link');
  preloadLinks.rel = 'preload';
  preloadLinks.href = 'critical.css';
  preloadLinks.as = 'style';
  document.head.appendChild(preloadLinks);
}

// 存储优化
function optimizeStorage() {
  // 定期清理过期数据
  function cleanExpiredData() {
    const now = Date.now();
    const keys = Object.keys(localStorage);

    keys.forEach(key => {
      try {
        const data = JSON.parse(localStorage.getItem(key));
        if (data.expiry && data.expiry < now) {
          localStorage.removeItem(key);
        }
      } catch (e) {
        // 忽略解析错误
      }
    });
  }

  // 压缩存储数据
  function compressData(data) {
    const json = JSON.stringify(data);
    // 可以使用 LZW 等算法压缩
    return json;
  }

  // 设置定期清理
  setInterval(cleanExpiredData, 3600000); // 每小时清理一次
}
```

### 3. 安全性考虑

```javascript
// 防止 XSS 攻击
function escapeHtml(unsafe) {
  return unsafe
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&#039;");
}

// 存储敏感数据时加密
function encryptData(data, key) {
  // 使用 Web Crypto API 进行加密
  // 这里简化为 base64 编码，实际应用中应使用真正的加密算法
  return btoa(encodeURIComponent(JSON.stringify(data)));
}

function decryptData(encryptedData, key) {
  try {
    return JSON.parse(decodeURIComponent(atob(encryptedData)));
  } catch (e) {
    console.error('解密失败:', e);
    return null;
  }
}

// 安全的表单处理
function sanitizeFormInput(input) {
  // 防止 XSS 和 SQL 注入
  let sanitized = input.trim();

  // 移除潜在的恶意脚本
  sanitized = sanitized.replace(/<script\b[^<]*(?:(?!<\/script>)<[^<]*)*<\/script>/gi, '');

  // 转义 HTML 特殊字符
  sanitized = escapeHtml(sanitized);

  return sanitized;
}

// 验证用户输入
function validateInput(input, type) {
  const patterns = {
    email: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    phone: /^1[3-9]\d{9}$/,
    url: /^https?:\/\/[^\s/$.?#].[^\s]*$/,
    number: /^\d+$/,
    alnum: /^[a-zA-Z0-9]+$/
  };

  if (patterns[type]) {
    return patterns[type].test(input);
  }

  return false;
}
```

## 总结

HTML5 为 Web 开发带来了革命性的变化，提供了丰富的特性和 API，使开发者能够创建更强大、更现代化的 Web 应用。

核心要点：

1. **语义化标签**：使用 `<header>`, `<nav>`, `<main>`, `<article>`, `<section>`, `<aside>`, `<footer>` 等语义化标签
2. **表单增强**：新增多种输入类型、验证功能和属性
3. **多媒体支持**：原生 `<audio>` 和 `<video>` 标签，无需插件
4. **本地存储**：`localStorage` 和 `sessionStorage` 提供客户端存储方案
5. **Canvas API**：2D 绘图和动画能力
6. **地理位置 API**：获取用户位置信息
7. **拖拽 API**：原生拖拽功能
8. **历史管理 API**：更好的页面导航控制

记住：
- 始终考虑浏览器兼容性，必要时提供 polyfill
- 注意性能优化，特别是在处理多媒体和大量数据时
- 重视安全性，防止 XSS 和其他攻击
- 合理使用 HTML5 特性，避免过度设计
- 持续学习新的 Web API 和最佳实践

通过掌握 HTML5 的新特性，你将能够创建更现代、更强大、更用户友好的 Web 应用。HTML5 不仅仅是一组新标签，更是一个完整的 Web 开发平台，为现代 Web 应用开发提供了坚实基础。