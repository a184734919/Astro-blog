---
title: Express 模板引擎
published: 2023-03-22
description: 'EJS、Pug 模板使用的详细介绍和学习笔记'
image: ''
tags: ["Node.js","Express"]
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 模板引擎概述

模板引擎是生成 HTML 的工具，它允许你在模板中使用变量、逻辑和控制结构，从而动态生成内容。Express 支持多种模板引擎，包括 EJS、Pug、Handlebars 等。

### 为什么使用模板引擎

- 分离业务逻辑和表现层
- 提高代码的可维护性
- 支持模板继承和复用
- 提供动态内容生成能力
- 改善开发体验

### 常见模板引擎对比

| 模板引擎 | 语法 | 特点 | 适用场景 |
|---------|------|------|---------|
| EJS | HTML-like | 简单直观，学习成本低 | 快速开发、小型项目 |
| Pug | 缩进 | 代码简洁，但需要学习 | 大型项目、专业前端 |
| Handlebars | {{ }} | 逻辑分离，安全 | 大型项目、团队开发 |
| Mustache | {{ }} | 无逻辑模板 | 静态内容较多 |

## EJS 模板引擎

### 安装和配置

```javascript
// 安装 EJS
// npm install ejs

const express = require('express');
const app = express();

// 设置模板引擎和模板目录
app.set('view engine', 'ejs');
app.set('views', './views');

// 提供静态文件
app.use(express.static('public'));

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### EJS 基础语法

```html
<!-- 变量 -->
<h1><%= title %></h1>
<p>Welcome, <%= user.name %></p>

<!-- 原始输出（不转义 HTML） -->
<div><%- content %></div>

<!-- JavaScript 表达式 -->
<p>Total: <%= items.length %></p>
<p>Price: <%= formatPrice(item.price) %></p>

<!-- 条件语句 -->
<% if (user.isLoggedIn) { %>
  <p>Welcome back, <%= user.name %>!</p>
<% } else { %>
  <p>Please <a href="/login">login</a>.</p>
<% } %>

<!-- 循环语句 -->
<ul>
<% items.forEach(function(item) { %>
  <li><%= item.name %> - <%= item.price %></li>
<% }); %>
</ul>

<!-- 包含其他模板 -->
<%- include('header', { title: 'Home' }) %>

<div class="content">
  <p>Main content here</p>
</div>

<%- include('footer') %>
```

### EJS 高级用法

```html
<!-- 自定义分隔符 -->
<% ejs.open = '{{'; ejs.close = '}}'; %>
<p>{{ variable }}</p>

<!-- 带布局的模板 -->
<%- include('layouts/main', {
  title: 'Product Page',
  content: `
    <div class="product">
      <h2><%= product.name %></h2>
      <p>Price: $<%= product.price %></p>
      <p>Description: <%= product.description %></p>
    </div>
  `
}) %>

<!-- 条件渲染 -->
<% if (products && products.length > 0) { %>
  <div class="products">
    <% products.forEach(function(product) { %>
      <div class="product-card">
        <h3><%= product.name %></h3>
        <p>$<%= product.price %></p>
      </div>
    <% }); %>
  </div>
<% } else { %>
  <p>No products available</p>
<% } %>

<!-- 循环带索引 -->
<% for (let i = 0; i < items.length; i++) { %>
  <div class="item <%= i === 0 ? 'first' : '' %>">
    <span><%= i + 1 %>. <%= items[i].name %></span>
  </div>
<% } %>

<!-- 对象遍历 -->
<% Object.keys(user).forEach(function(key) { %>
  <p><%= key %>: <%= user[key] %></p>
<% }); %>

<!-- 过滤数据 -->
<% const activeItems = items.filter(item => item.active); %>
<% if (activeItems.length > 0) { %>
  <ul>
    <% activeItems.forEach(function(item) { %>
      <li><%= item.name %></li>
    <% }); %>
  </ul>
<% } %>
```

### EJS 布局系统

```html
<!-- views/layouts/main.ejs -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title><%= title %> - My App</title>
  <link rel="stylesheet" href="/css/style.css">
</head>
<body>
  <header>
    <%- include('../partials/header') %>
  </header>

  <main>
    <%- content %>
  </main>

  <footer>
    <%- include('../partials/footer') %>
  </footer>

  <script src="/js/main.js"></script>
</body>
</html>
```

```html
<!-- views/home.ejs -->
<%
  const content = `
    <h1>Welcome to My App</h1>
    <p>This is the home page.</p>
    <div class="features">
      <div class="feature">
        <h3>Fast</h3>
        <p>Lightning fast performance</p>
      </div>
      <div class="feature">
        <h3>Secure</h3>
        <p>Enterprise-grade security</p>
      </div>
    </div>
  `;
%>

<%- include('layouts/main', {
  title: 'Home',
  content: content
}) %>
```

### EJS 实际应用示例

```javascript
const express = require('express');
const app = express();

app.set('view engine', 'ejs');

// 首页
app.get('/', (req, res) => {
  res.render('home', {
    title: 'Home',
    user: req.session.user || null,
    message: 'Welcome to our platform!'
  });
});

// 产品列表
app.get('/products', async (req, res) => {
  try {
    const products = await Product.findAll();
    res.render('products/index', {
      title: 'Products',
      products: products,
      user: req.session.user
    });
  } catch (error) {
    res.status(500).render('error', {
      title: 'Error',
      error: 'Failed to load products'
    });
  }
});

// 产品详情
app.get('/products/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id);
    if (!product) {
      return res.status(404).render('error', {
        title: 'Not Found',
        error: 'Product not found'
      });
    }
    
    res.render('products/show', {
      title: product.name,
      product: product,
      user: req.session.user
    });
  } catch (error) {
    res.status(500).render('error', {
      title: 'Error',
      error: 'Failed to load product'
    });
  }
});

// 用户个人资料
app.get('/profile', requireLogin, async (req, res) => {
  try {
    const user = await User.findById(req.session.userId);
    res.render('users/profile', {
      title: 'Profile',
      user: user,
      isOwnProfile: true
    });
  } catch (error) {
    res.status(500).render('error', {
      title: 'Error',
      error: 'Failed to load profile'
    });
  }
});
```

## Pug 模板引擎

### 安装和配置

```javascript
// 安装 Pug
// npm install pug

const express = require('express');
const app = express();

// 设置模板引擎
app.set('view engine', 'pug');
app.set('views', './views');

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Pug 基础语法

```jade
// 变量
h1= title
p Welcome, #{user.name}

// 原始输出
div!= content

// 属性
a(href="/login", class="btn btn-primary") Login
input(type="text", name="username", placeholder="Username", required)

// 样式和类
.button.primary Submit
#main-container
  .content
    p Content here

// 条件语句
if user.isLoggedIn
  p Welcome back, #{user.name}!
else
  p Please 
    a(href="/login") login
    | .

// 循环
ul
  each item in items
    li= item.name + ' - ' + item.price

// 包含模板
include header

div.content
  p Main content

include footer
```

### Pug 高级语法

```jade
// 混合（Mixins）
mixin productCard(product)
  .product-card
    h2.product-name= product.name
    p.product-price $#{product.price}
    p.product-description= product.description
    button.btn Buy Now

// 使用混合
- products.forEach(product)
  +productCard(product)

// 带参数的混合
mixin alert(type, message)
  .alert(class="alert-#{type}")
    = message

+alert('success', 'Operation successful')
+alert('error', 'Operation failed')

// 条件渲染
if products && products.length > 0
  .products
    each product in products
      .product= product.name
else
  p No products available

// 带索引的循环
each item, index in items
  .item(class=index === 0 ? 'first' : '')
    span #{index + 1}. #{item.name}

// 对象遍历
each value, key in user
  p #{key}: #{value}

// 代码块
- const fullName = user.firstName + ' ' + user.lastName;
- const formattedPrice = '$' + product.price.toFixed(2);

p User: #{fullName}
p Price: #{formattedPrice}

// 脚本和样式
script.
  console.log('Page loaded');
  const userData = !{JSON.stringify(user)};

style.
  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
```

### Pug 模板继承

```jade
// views/layouts/main.pug
doctype html
html
  head
    meta(charset='UTF-8')
    meta(name='viewport' content='width=device-width, initial-scale=1.0')
    title #{title} - My App
    link(rel='stylesheet' href='/css/style.css')
    block styles
    
  body
    header
      include partials/header
    
    main
      block content
    
    footer
      include partials/footer
    
    script(src='/js/main.js')
    block scripts
```

```jade
// views/home.pug
extends layouts/main

block content
  h1 Welcome to My App
  p This is the home page.
  
  .features
    .feature
      h3 Fast
      p Lightning fast performance
    .feature
      h3 Secure
      p Enterprise-grade security

block styles
  style.
    .features {
      display: flex;
      gap: 20px;
    }
    
    .feature {
      flex: 1;
      padding: 20px;
      border: 1px solid #ddd;
    }
```

### Pug 实际应用示例

```jade
// views/products/index.pug
extends layouts/main

block content
  h1 Products
  
  .filters
    form(action='/products', method='GET')
      input(type='text', name='search', placeholder='Search products...')
      button(type='submit') Search
  
  if products.length > 0
    .products-grid
      each product in products
        .product-card
          img.product-image(src=product.image, alt=product.name)
          h2.product-name= product.name
          p.product-price $#{product.price}
          p.product-description= product.description
          .product-actions
            a.btn.btn-primary(href='/products/#{product.id}') View Details
            a.btn.btn-secondary(href='/products/#{product.id}/edit') Edit
  else
    .empty-state
      h2 No Products Found
      p Try adjusting your search filters or browse all products.
```

```jade
// views/users/profile.pug
extends layouts/main

block content
  .profile-container
    .profile-header
      img.profile-avatar(src=user.avatar || '/default-avatar.png', alt='Profile')
      h1= user.username
      p.profile-email= user.email
      p.profile-role= user.role
    
    .profile-sections
      .profile-section
        h2 Personal Information
        .info-grid
          .info-item
            label Name
            span= user.name
          .info-item
            label Email
            span= user.email
          .info-item
            label Role
            span= user.role
      
      .profile-section
        h2 Account Settings
        form(action='/users/#{user.id}', method='POST')
          .form-group
            label(for='name') Name
            input#name(type='text', name='name', value=user.name)
          
          .form-group
            label(for='email') Email
            input#email(type='email', name='email', value=user.email)
          
          button.btn.btn-primary(type='submit') Update Profile
    
    .profile-actions
      a.btn.btn-secondary(href='/users/#{user.id}/change-password') Change Password
      if user.role === 'admin'
        a.btn.btn-danger(href='/admin/users') Admin Panel
```

## Handlebars 模板引擎

### 安装和配置

```javascript
// 安装 Handlebars
// npm install express-handlebars

const express = require('express');
const exphbs = require('express-handlebars');

const app = express();

// 配置 Handlebars
app.engine('handlebars', exphbs.engine({
  defaultLayout: 'main',
  layoutsDir: 'views/layouts/',
  partialsDir: 'views/partials/',
  helpers: {
    formatDate: function(date) {
      return new Date(date).toLocaleDateString();
    },
    eq: function(a, b) {
      return a === b;
    }
  }
}));

app.set('view engine', 'handlebars');

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

### Handlebars 基础语法

```html
<!-- 变量 -->
<h1>{{title}}</h1>
<p>Welcome, {{user.name}}</p>

<!-- 原始输出 -->
<div>{{{content}}}</div>

<!-- 条件语句 -->
{{#if user.isLoggedIn}}
  <p>Welcome back, {{user.name}}!</p>
{{else}}
  <p>Please <a href="/login">login</a>.</p>
{{/if}}

<!-- 循环 -->
<ul>
  {{#each items}}
    <li>{{this.name}} - {{this.price}}</li>
  {{/each}}
</ul>

<!-- 对象属性 -->
<p>{{user.profile.name}}</p>

<!-- 自定义 Helper -->
<p>{{formatDate date}}</p>

<!-- with 语句 -->
{{#with user}}
  <p>{{name}}</p>
  <p>{{email}}</p>
{{/with}}
```

### Handlebars 高级用法

```html
<!-- 自定义 Helper -->
<h1>Handlebars Helpers</h1>

<!-- 格式化日期 -->
<p>{{formatDate createdAt}}</p>

<!-- 条件判断 -->
{{#if (eq user.role 'admin')}}
  <p>Admin access granted</p>
{{/if}}

<!-- 数学运算 -->
<p>Total: {{add quantity 1}}</p>
<p>Discount: {{subtract price 10}}</p>

<!-- 字符串操作 -->
<p>{{uppercase name}}</p>
<p>{{substring description 0 100}}...</p>

<!-- 数组操作 -->
<p>{{arrayLength items}}</p>

<!-- 路径参数 -->
<a href="/products/{{id}}">View Product</a>

<!-- unless 语句 -->
{{#unless user}}
  <p>Not logged in</p>
{{/unless}}

<!-- 嵌套循环 -->
{{#each categories}}
  <h3>{{name}}</h3>
  <ul>
    {{#each items}}
      <li>{{name}}</li>
    {{/each}}
  </ul>
{{/each}}

<!-- else 块 -->
{{#if products}}
  <div class="products">
    {{#each products}}
      <div class="product">{{name}}</div>
    {{/each}}
  </div>
{{else}}
  <p>No products available</p>
{{/if}}
```

### Handlebars 布局系统

```html
<!-- views/layouts/main.handlebars -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{{title}} - My App</title>
  <link rel="stylesheet" href="/css/style.css">
  {{{_sections.styles}}}
</head>
<body>
  <header>
    {{> header}}
  </header>

  <main>
    {{{body}}}
  </main>

  <footer>
    {{> footer}}
  </footer>

  <script src="/js/main.js"></script>
  {{{_sections.scripts}}}
</body>
</html>
```

```html
<!-- views/home.handlebars -->
<h1>Welcome to My App</h1>
<p>This is the home page.</p>

<div class="features">
  {{#each features}}
    <div class="feature">
      <h3>{{title}}</h3>
      <p>{{description}}</p>
    </div>
  {{/each}}
</div>

{{#section 'styles'}}
  <style>
    .features {
      display: flex;
      gap: 20px;
    }
    .feature {
      flex: 1;
      padding: 20px;
      border: 1px solid #ddd;
    }
  </style>
{{/section}}
```

## 模板引擎最佳实践

### 1. 组织模板结构

```
views/
├── layouts/
│   ├── main.ejs
│   ├── admin.ejs
│   └── email.ejs
├── partials/
│   ├── header.ejs
│   ├── footer.ejs
│   ├── navigation.ejs
│   └── forms/
│       ├── login.ejs
│       └── signup.ejs
├── users/
│   ├── profile.ejs
│   ├── edit.ejs
│   └── list.ejs
├── products/
│   ├── index.ejs
│   ├── show.ejs
│   ├── create.ejs
│   └── edit.ejs
└── error.ejs
```

### 2. 使用全局变量和 Helper

```javascript
// 为所有模板提供全局变量
app.use((req, res, next) => {
  res.locals.currentUser = req.session.user;
  res.locals.flashMessages = req.session.flash || {};
  res.locals.path = req.path;
  res.locals.query = req.query;
  next();
});

// 自定义 Helper 函数
app.locals.formatDate = (date) => {
  return new Date(date).toLocaleDateString();
};

app.locals.formatCurrency = (amount) => {
  return new Intl.NumberFormat('en-US', {
    style: 'currency',
    currency: 'USD'
  }).format(amount);
};

app.locals.truncate = (str, length) => {
  if (str.length <= length) return str;
  return str.substring(0, length) + '...';
};
```

### 3. 错误处理页面

```html
<!-- views/error.ejs -->
<div class="error-page">
  <h1><%= error.status || 500 %></h1>
  <h2><%= error.message %></h2>
  
  <% if (process.env.NODE_ENV === 'development' && error.stack) { %>
    <pre><%= error.stack %></pre>
  <% } %>
  
  <a href="/" class="btn">Go Home</a>
</div>
```

```javascript
// 404 错误处理
app.use((req, res, next) => {
  res.status(404).render('error', {
    title: 'Not Found',
    error: {
      status: 404,
      message: 'The page you requested was not found'
    }
  });
});

// 错误处理中间件
app.use((err, req, res, next) => {
  console.error(err.stack);
  
  res.status(err.status || 500).render('error', {
    title: 'Error',
    error: {
      status: err.status || 500,
      message: err.message || 'Internal Server Error',
      stack: process.env.NODE_ENV === 'development' ? err.stack : undefined
    }
  });
});
```

### 4. 缓存优化

```javascript
// 开发环境禁用缓存
app.set('view cache', process.env.NODE_ENV !== 'development');

// 生产环境启用缓存
if (process.env.NODE_ENV === 'production') {
  app.set('view cache', true);
  app.use(express.static('public', {
    maxAge: '1d',
    etag: true
  }));
}
```

### 5. 国际化支持

```javascript
const i18n = require('i18n');

i18n.configure({
  locales: ['en', 'zh', 'es'],
  defaultLocale: 'en',
  directory: './locales',
  registerGlobal: true,
  syncFiles: false
});

app.use(i18n.init);

// 在路由中使用
app.get('/', (req, res) => {
  res.render('home', {
    title: req.__('Home')
  });
});
```

```html
<!-- 在模板中使用翻译 -->
<h1>{{__('Welcome')}}</h1>
<p>{{__('description')}}</p>

```javascript
// locales/en.json
{
  "Welcome": "Welcome",
  "description": "This is a description"
}

// locales/zh.json
{
  "Welcome": "欢迎",
  "description": "这是一个描述"
}
```

## 模板引擎性能优化

### 1. 预编译模板

```javascript
// EJS 预编译
const ejs = require('ejs');
const fs = require('fs');

// 预编译模板文件
const template = fs.readFileSync('./views/home.ejs', 'utf-8');
const compiledTemplate = ejs.compile(template);

// 使用预编译的模板
app.get('/', (req, res) => {
  const html = compiledTemplate({
    title: 'Home',
    user: req.session.user
  });
  res.send(html);
});
```

### 2. 模板片段缓存

```javascript
const NodeCache = require('node-cache');
const templateCache = new NodeCache({ stdTTL: 3600 });

app.get('/products', async (req, res) => {
  const cacheKey = 'products-list';
  
  let templateHtml = templateCache.get(cacheKey);
  
  if (!templateHtml) {
    const products = await Product.findAll();
    templateHtml = res.render('products/list', { products }, (err, html) => {
      if (err) return next(err);
      templateCache.set(cacheKey, html);
      res.send(html);
    });
  } else {
    res.send(templateHtml);
  }
});
```

### 3. 静态资源优化

```javascript
const compression = require('compression');
const helmet = require('helmet');

// 启用压缩
app.use(compression());

// 安全头部
app.use(helmet());

// 静态资源缓存
app.use(express.static('public', {
  maxAge: '1y',
  etag: true,
  lastModified: true,
  setHeaders: (res, path) => {
    if (path.endsWith('.html')) {
      res.setHeader('Cache-Control', 'no-cache');
    }
  }
}));
```

## 常见问题和解决方案

### 问题 1：模板文件未找到

```javascript
// 问题：Error: Failed to lookup view "home"

// 解决方案：检查视图目录设置
app.set('views', path.join(__dirname, 'views'));

// 或使用绝对路径
app.set('views', '/absolute/path/to/views');
```

### 问题 2：变量未定义

```html
<!-- 问题：模板中变量显示为 undefined -->

<!-- 解决方案：提供默认值 -->
<% if (typeof user !== 'undefined' && user) { %>
  <p>Welcome, <%= user.name %></p>
<% } %>

<!-- 或使用默认值 -->
<p>Welcome, <%= (user && user.name) || 'Guest' %></p>
```

### 问题 3：模板渲染性能

```javascript
// 问题：模板渲染速度慢

// 解决方案：启用缓存
app.set('view cache', true);

// 预编译模板
const compiledTemplates = {
  home: ejs.compile(fs.readFileSync('views/home.ejs', 'utf-8')),
  products: ejs.compile(fs.readFileSync('views/products.ejs', 'utf-8'))
};

app.get('/', (req, res) => {
  const html = compiledTemplates.home({
    title: 'Home',
    user: req.session.user
  });
  res.send(html);
});
```

## 总结

Express 模板引擎提供了强大的动态内容生成能力。通过本篇详细指南，你已经了解了：

- 模板引擎的基本概念和选择
- EJS 的语法和高级用法
- Pug 的模板系统和继承机制
- Handlebars 的 Helper 和布局
- 模板引擎的实际应用
- 模板组织结构最佳实践
- 错误处理和国际化
- 性能优化技巧
- 常见问题的解决方案

在实际项目中，建议：

1. 根据项目需求选择合适的模板引擎
2. 建立清晰的模板组织结构
3. 使用布局和模板继承提高复用性
4. 实施适当的缓存策略
5. 遵循安全最佳实践
6. 考虑国际化支持

掌握 Express 模板引擎能够帮助你构建更加用户友好、易于维护的 Web 应用界面。