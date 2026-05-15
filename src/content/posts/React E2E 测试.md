---
title: React E2E 测试
published: 2023-10-21
description: 'Cypress 端到端测试的详细介绍和学习笔记'
image: ''
tags: ["React","测试"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

端到端（E2E）测试是测试整个应用从前端到后端的完整流程，模拟真实用户的操作。Cypress 是目前最流行的 E2E 测试框架之一，它提供了快速、可靠、易用的测试体验。

### E2E 测试的价值

- **验证完整流程**：测试整个用户流程，从登录到完成操作
- **跨浏览器测试**：确保应用在不同浏览器中正常工作
- **真实环境测试**：在接近生产环境中测试应用
- **回归测试**：防止新功能破坏现有功能
- **用户视角测试**：从用户角度验证应用行为

### Cypress 特点

- **实时重载**：代码修改后自动重新运行测试
- **时间旅行**：可以看到测试执行的每一步
- **调试友好**：提供丰富的调试工具
- **自动等待**：智能等待元素出现，无需手动等待
- **网络控制**：可以拦截和修改网络请求
- **截图录制**：自动截图和视频录制

---

## 核心概念

### Cypress 架构

```
┌─────────────────────────────────────────┐
│           Cypress Test Runner           │
│  ┌─────────────────────────────────────┐ │
│  │         Test Specification          │ │
│  │  (describe, it, before, after)      │ │
│  └─────────────────────────────────────┘ │
│                  ↓                       │
│  ┌─────────────────────────────────────┐ │
│  │         Cypress Commands            │ │
│  │  (cy.visit, cy.get, cy.click)       │ │
│  └─────────────────────────────────────┘ │
│                  ↓                       │
│  ┌─────────────────────────────────────┐ │
│  │           Browser Proxy              │ │
│  │  (Network Interception, Cookies)    │ │
│  └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
                  ↓
┌─────────────────────────────────────────┐
│             Browser (Chrome)             │
│    (Automated Chromium Browser)          │
└─────────────────────────────────────────┘
```

### 测试生命周期

```javascript
// 测试文件结构
describe('功能描述', () => {
  before(() => {
    // 在所有测试之前运行一次
  });

  beforeEach(() => {
    // 在每个测试之前运行
  });

  afterEach(() => {
    // 在每个测试之后运行
  });

  after(() => {
    // 在所有测试之后运行一次
  });

  it('测试用例 1', () => {
    // 测试代码
  });

  it('测试用例 2', () => {
    // 测试代码
  });
});
```

---

## 基本用法

### 1. 环境配置

```bash
# 安装 Cypress
npm install --save-dev cypress

# 或使用 yarn
yarn add --dev cypress

# 或使用 pnpm
pnpm add --dev cypress

# 初始化 Cypress
npx cypress open
```

```javascript
// cypress.config.js
const { defineConfig } = require('cypress');

module.exports = defineConfig({
  e2e: {
    baseUrl: 'http://localhost:3000',
    specPattern: 'cypress/e2e/**/*.cy.{js,jsx,ts,tsx}',
    supportFile: 'cypress/support/e2e.js',
    viewportWidth: 1280,
    viewportHeight: 720,
    video: true,
    screenshotOnRunFailure: true,
    videosFolder: 'cypress/videos',
    screenshotsFolder: 'cypress/screenshots',
    defaultCommandTimeout: 4000,
    pageLoadTimeout: 60000,
    requestTimeout: 5000,
    responseTimeout: 30000,
    setupNodeEvents(on, config) {
      // 实现节点事件监听器
      return config;
    },
    env: {
      apiUrl: 'https://api.example.com',
      username: 'testuser',
      password: 'testpass'
    }
  }
});
```

```javascript
// cypress/support/e2e.js
// 导入 commands 文件
import './commands';

// 导入全局样式
import './index.css';

// 或者使用 @testing-library/cypress 命令
import '@testing-library/cypress/add-commands';
```

```javascript
// cypress/support/commands.js
// 自定义 Cypress 命令
Cypress.Commands.add('login', (username, password) => {
  cy.visit('/login');
  cy.get('[data-testid="username-input"]').type(username);
  cy.get('[data-testid="password-input"]').type(password);
  cy.get('[data-testid="login-button"]').click();
});

Cypress.Commands.add('logout', () => {
  cy.get('[data-testid="logout-button"]').click();
});

Cypress.Commands.add('selectDropdown', (selector, value) => {
  cy.get(selector).click();
  cy.contains(value).click();
});

Cypress.Commands.add('uploadFile', (selector, filePath) => {
  cy.get(selector).selectFile(filePath);
});

// 声明自定义命令的类型
declare global {
  namespace Cypress {
    interface Chainable {
      login(username: string, password: string): Chainable<Element>;
      logout(): Chainable<Element>;
      selectDropdown(selector: string, value: string): Chainable<Element>;
      uploadFile(selector: string, filePath: string): Chainable<Element>;
    }
  }
}
```

### 2. 基础测试用例

```javascript
// cypress/e2e/basic.cy.js
describe('基础测试', () => {
  beforeEach(() => {
    // 在每个测试前访问首页
    cy.visit('/');
  });

  it('应该加载首页', () => {
    cy.contains('欢迎').should('be.visible');
  });

  it('应该导航到关于页面', () => {
    cy.get('[data-testid="about-link"]').click();
    cy.url().should('include', '/about');
    cy.contains('关于我们').should('be.visible');
  });

  it('应该显示导航菜单', () => {
    cy.get('[data-testid="nav-menu"]').should('be.visible');
    cy.get('[data-testid="nav-menu"] > a').should('have.length.greaterThan', 0);
  });
});
```

### 3. 表单测试

```javascript
// cypress/e2e/forms.cy.js
describe('表单测试', () => {
  beforeEach(() => {
    cy.visit('/contact');
  });

  it('应该提交有效的表单', () => {
    cy.get('[data-testid="name-input"]').type('张三');
    cy.get('[data-testid="email-input"]').type('zhangsan@example.com');
    cy.get('[data-testid="message-input"]').type('这是一条测试消息');
    cy.get('[data-testid="submit-button"]').click();

    cy.contains('提交成功').should('be.visible');
  });

  it('应该验证必填字段', () => {
    // 不填写任何字段直接提交
    cy.get('[data-testid="submit-button"]').click();

    // 应该显示错误信息
    cy.contains('姓名是必填项').should('be.visible');
    cy.contains('邮箱是必填项').should('be.visible');
  });

  it('应该验证邮箱格式', () => {
    cy.get('[data-testid="name-input"]').type('张三');
    cy.get('[data-testid="email-input"]').type('invalid-email');
    cy.get('[data-testid="submit-button"]').click();

    cy.contains('请输入有效的邮箱地址').should('be.visible');
  });

  it('应该重置表单', () => {
    cy.get('[data-testid="name-input"]').type('张三');
    cy.get('[data-testid="email-input"]').type('zhangsan@example.com');
    cy.get('[data-testid="reset-button"]').click();

    cy.get('[data-testid="name-input"]').should('have.value', '');
    cy.get('[data-testid="email-input"]').should('have.value', '');
  });
});
```

### 4. 网络请求测试

```javascript
// cypress/e2e/api.cy.js
describe('API 请求测试', () => {
  beforeEach(() => {
    // 拦截所有 API 请求
    cy.intercept('GET', '/api/users').as('getUsers');
    cy.intercept('POST', '/api/users').as('createUser');
    cy.intercept('DELETE', '/api/users/*').as('deleteUser');
  });

  it('应该加载用户列表', () => {
    cy.visit('/users');

    // 等待 API 请求完成
    cy.wait('@getUsers').then((interception) => {
      expect(interception.response.statusCode).to.eq(200);
      expect(interception.response.body).to.have.property('data');
    });

    // 验证用户列表显示
    cy.get('[data-testid="user-list"]').should('be.visible');
  });

  it('应该创建新用户', () => {
    cy.visit('/users/create');

    cy.get('[data-testid="name-input"]').type('新用户');
    cy.get('[data-testid="email-input"]').type('newuser@example.com');
    cy.get('[data-testid="submit-button"]').click();

    // 等待创建用户的 API 请求
    cy.wait('@createUser').then((interception) => {
      expect(interception.request.method).to.eq('POST');
      expect(interception.request.body).to.include({
        name: '新用户',
        email: 'newuser@example.com'
      });
      expect(interception.response.statusCode).to.eq(201);
    });

    // 验证成功消息
    cy.contains('用户创建成功').should('be.visible');
  });

  it('应该删除用户', () => {
    cy.visit('/users');
    cy.wait('@getUsers');

    // 点击第一个用户的删除按钮
    cy.get('[data-testid="delete-button"]').first().click();

    // 确认删除
    cy.get('[data-testid="confirm-delete"]').click();

    // 等待删除 API 请求
    cy.wait('@deleteUser').then((interception) => {
      expect(interception.response.statusCode).to.eq(204);
    });

    // 验证用户已从列表中移除
    cy.get('[data-testid="user-list"]').should('have.length', 0);
  });
});
```

---

## 实际应用

### 1. 用户认证流程测试

```javascript
// cypress/e2e/authentication.cy.js
describe('用户认证流程', () => {
  const validUser = {
    username: 'testuser',
    password: 'testpass123',
    email: 'test@example.com'
  };

  const invalidUser = {
    username: 'invalid',
    password: 'wrongpass'
  };

  beforeEach(() => {
    cy.visit('/login');
  });

  it('应该成功登录', () => {
    cy.get('[data-testid="username-input"]').type(validUser.username);
    cy.get('[data-testid="password-input"]').type(validUser.password);
    cy.get('[data-testid="login-button"]').click();

    // 验证重定向到仪表板
    cy.url().should('include', '/dashboard');
    cy.contains(`欢迎, ${validUser.username}`).should('be.visible');
  });

  it('应该拒绝无效的登录凭据', () => {
    cy.get('[data-testid="username-input"]').type(invalidUser.username);
    cy.get('[data-testid="password-input"]').type(invalidUser.password);
    cy.get('[data-testid="login-button"]').click();

    // 应该显示错误消息
    cy.contains('用户名或密码错误').should('be.visible');
    cy.url().should('include', '/login');
  });

  it('应该验证必填字段', () => {
    // 不填写任何字段直接登录
    cy.get('[data-testid="login-button"]').click();

    cy.contains('请输入用户名').should('be.visible');
    cy.contains('请输入密码').should('be.visible');
  });

  it('应该支持记住我功能', () => {
    cy.get('[data-testid="username-input"]').type(validUser.username);
    cy.get('[data-testid="password-input"]').type(validUser.password);
    cy.get('[data-testid="remember-me"]').check();
    cy.get('[data-testid="login-button"]').click();

    // 验证 cookie 已设置
    cy.getCookie('remember_token').should('exist');
  });

  it('应该成功注册新用户', () => {
    cy.visit('/register');

    const newUser = {
      username: 'newuser',
      email: 'newuser@example.com',
      password: 'password123',
      confirmPassword: 'password123'
    };

    cy.get('[data-testid="username-input"]').type(newUser.username);
    cy.get('[data-testid="email-input"]').type(newUser.email);
    cy.get('[data-testid="password-input"]').type(newUser.password);
    cy.get('[data-testid="confirm-password-input"]').type(newUser.confirmPassword);
    cy.get('[data-testid="register-button"]').click();

    // 验证注册成功
    cy.contains('注册成功').should('be.visible');
    cy.url().should('include', '/dashboard');
  });

  it('应该验证密码匹配', () => {
    cy.visit('/register');

    cy.get('[data-testid="username-input"]').type('testuser');
    cy.get('[data-testid="email-input"]').type('test@example.com');
    cy.get('[data-testid="password-input"]').type('password123');
    cy.get('[data-testid="confirm-password-input"]').type('different');
    cy.get('[data-testid="register-button"]').click();

    cy.contains('两次输入的密码不一致').should('be.visible');
  });

  it('应该成功登出', () => {
    // 先登录
    cy.login(validUser.username, validUser.password);

    // 然后登出
    cy.get('[data-testid="logout-button"]').click();

    // 验证重定向到登录页
    cy.url().should('include', '/login');
    cy.contains('您已成功登出').should('be.visible');
  });
});
```

### 2. 电商应用测试

```javascript
// cypress/e2e/ecommerce.cy.js
describe('电商应用测试', () => {
  const product = {
    id: 1,
    name: '测试商品',
    price: 99.99,
    description: '这是一个测试商品'
  };

  beforeEach(() => {
    // 拦截商品 API
    cy.intercept('GET', '/api/products').as('getProducts');
    cy.intercept('GET', '/api/products/*').as('getProduct');
    cy.intercept('POST', '/api/cart').as('addToCart');
    cy.intercept('GET', '/api/cart').as('getCart');
  });

  describe('商品浏览', () => {
    it('应该显示商品列表', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      cy.get('[data-testid="product-list"]').should('be.visible');
      cy.get('[data-testid="product-card"]').should('have.length.greaterThan', 0);
    });

    it('应该显示商品详情', () => {
      cy.visit(`/products/${product.id}`);
      cy.wait('@getProduct');

      cy.contains(product.name).should('be.visible');
      cy.contains(`¥${product.price}`).should('be.visible');
      cy.contains(product.description).should('be.visible');
    });

    it('应该支持商品搜索', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      cy.get('[data-testid="search-input"]').type(product.name);
      cy.get('[data-testid="search-button"]').click();

      cy.get('[data-testid="product-card"]').should('have.length', 1);
      cy.contains(product.name).should('be.visible');
    });

    it('应该支持商品筛选', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      cy.get('[data-testid="category-filter"]').click();
      cy.contains('电子产品').click();

      cy.url().should('include', 'category=electronics');
      cy.get('[data-testid="product-card"]').should('have.length.greaterThan', 0);
    });
  });

  describe('购物车功能', () => {
    it('应该添加商品到购物车', () => {
      cy.visit(`/products/${product.id}`);
      cy.wait('@getProduct');

      cy.get('[data-testid="add-to-cart-button"]').click();
      cy.wait('@addToCart');

      cy.contains('已添加到购物车').should('be.visible');

      // 验证购物车图标显示商品数量
      cy.get('[data-testid="cart-badge"]').should('contain', '1');
    });

    it('应该查看购物车', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      // 添加商品到购物车
      cy.get('[data-testid="add-to-cart-button"]').first().click();
      cy.wait('@addToCart');

      // 访问购物车页面
      cy.get('[data-testid="cart-icon"]').click();
      cy.url().should('include', '/cart');
      cy.wait('@getCart');

      // 验证购物车内容
      cy.get('[data-testid="cart-item"]').should('have.length', 1);
      cy.contains(product.name).should('be.visible');
    });

    it('应该从购物车中移除商品', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      // 添加商品到购物车
      cy.get('[data-testid="add-to-cart-button"]').first().click();
      cy.wait('@addToCart');

      // 访问购物车页面
      cy.get('[data-testid="cart-icon"]').click();
      cy.wait('@getCart');

      // 移除商品
      cy.get('[data-testid="remove-item-button"]').click();

      // 验证商品已移除
      cy.get('[data-testid="cart-item"]').should('not.exist');
      cy.contains('购物车为空').should('be.visible');
    });

    it('应该计算购物车总价', () => {
      cy.visit('/products');
      cy.wait('@getProducts');

      // 添加两个商品
      cy.get('[data-testid="add-to-cart-button"]').first().click();
      cy.wait('@addToCart');

      cy.get('[data-testid="add-to-cart-button"]').eq(1).click();
      cy.wait('@addToCart');

      // 访问购物车页面
      cy.get('[data-testid="cart-icon"]').click();
      cy.wait('@getCart');

      // 验证总价计算
      cy.get('[data-testid="cart-total"]').should('be.visible');
    });
  });

  describe('结账流程', () => {
    it('应该完成结账流程', () => {
      // 添加商品到购物车
      cy.visit('/products');
      cy.wait('@getProducts');

      cy.get('[data-testid="add-to-cart-button"]').first().click();
      cy.wait('@addToCart');

      // 访问购物车页面
      cy.get('[data-testid="cart-icon"]').click();
      cy.wait('@getCart');

      // 点击结账按钮
      cy.get('[data-testid="checkout-button"]').click();

      // 填写结账信息
      cy.get('[data-testid="shipping-address"]').type('测试地址');
      cy.get('[data-testid="shipping-city"]').type('测试城市');
      cy.get('[data-testid="shipping-zip"]').type('12345');
      cy.get('[data-testid="shipping-phone"]').type('13800138000');

      // 选择支付方式
      cy.get('[data-testid="payment-method"]').select('credit-card');

      // 填写支付信息
      cy.get('[data-testid="card-number"]').type('4111111111111111');
      cy.get('[data-testid="card-expiry"]').type('12/25');
      cy.get('[data-testid="card-cvc"]').type('123');

      // 提交订单
      cy.get('[data-testid="place-order-button"]').click();

      // 验证订单成功
      cy.contains('订单创建成功').should('be.visible');
      cy.url().should('include', '/order-success');
    });
  });
});
```

### 3. 响应式设计测试

```javascript
// cypress/e2e/responsive.cy.js
describe('响应式设计测试', () => {
  const viewports = [
    {
      name: 'desktop',
      width: 1280,
      height: 720
    },
    {
      name: 'laptop',
      width: 1024,
      height: 768
    },
    {
      name: 'tablet',
      width: 768,
      height: 1024
    },
    {
      name: 'mobile',
      width: 375,
      height: 667
    }
  ];

  viewports.forEach(viewport => {
    describe(`在 ${viewport.name} 设备上`, () => {
      beforeEach(() => {
        cy.viewport(viewport.width, viewport.height);
        cy.visit('/');
      });

      it('应该正确显示导航菜单', () => {
        if (viewport.width < 768) {
          // 移动设备应该有汉堡菜单
          cy.get('[data-testid="mobile-menu-button"]').should('be.visible');
          cy.get('[data-testid="desktop-nav"]').should('not.be.visible');
        } else {
          // 桌面设备应该有完整导航
          cy.get('[data-testid="desktop-nav"]').should('be.visible');
          cy.get('[data-testid="mobile-menu-button"]').should('not.be.visible');
        }
      });

      it('应该正确显示内容区域', () => {
        cy.get('[data-testid="main-content"]').should('be.visible');

        if (viewport.width < 768) {
          // 移动设备内容应该是单列布局
          cy.get('[data-testid="content-grid"]').should('have.css', 'grid-template-columns').and('match', /1fr/);
        } else {
          // 桌面设备内容应该是多列布局
          cy.get('[data-testid="content-grid"]').should('have.css', 'grid-template-columns').and('not.match', /1fr/);
        }
      });

      it('应该正确显示侧边栏', () => {
        if (viewport.width >= 1024) {
          cy.get('[data-testid="sidebar"]').should('be.visible');
        } else {
          cy.get('[data-testid="sidebar"]').should('not.be.visible');
        }
      });
    });
  });

  describe('移动端交互', () => {
    beforeEach(() => {
      cy.viewport('iphone-6');
      cy.visit('/');
    });

    it('应该打开移动菜单', () => {
      cy.get('[data-testid="mobile-menu-button"]').click();
      cy.get('[data-testid="mobile-menu"]').should('be.visible');
    });

    it('应该关闭移动菜单', () => {
      cy.get('[data-testid="mobile-menu-button"]').click();
      cy.get('[data-testid="mobile-menu"]').should('be.visible');

      cy.get('[data-testid="close-menu-button"]').click();
      cy.get('[data-testid="mobile-menu"]').should('not.be.visible');
    });
  });
});
```

### 4. 性能测试

```javascript
// cypress/e2e/performance.cy.js
describe('性能测试', () => {
  it('应该快速加载首页', () => {
    const startTime = Date.now();

    cy.visit('/');

    cy.window().then((win) => {
      const loadTime = Date.now() - startTime;
      expect(loadTime).to.be.lessThan(3000); // 页面应该在3秒内加载完成
    });

    // 验证关键指标
    cy.window().then((win) => {
      const performance = win.performance;
      const timing = performance.timing;

      const pageLoadTime = timing.loadEventEnd - timing.navigationStart;
      const domContentLoadedTime = timing.domContentLoadedEventEnd - timing.navigationStart;

      cy.log(`页面加载时间: ${pageLoadTime}ms`);
      cy.log(`DOM 加载完成时间: ${domContentLoadedTime}ms`);

      expect(pageLoadTime).to.be.lessThan(5000);
      expect(domContentLoadedTime).to.be.lessThan(2000);
    });
  });

  it('应该优化图片加载', () => {
    cy.visit('/');

    cy.get('img').each(($img) => {
      const naturalWidth = $img[0].naturalWidth;
      const displayWidth = $img[0].width;

      // 检查图片是否过大
      if (naturalWidth > displayWidth * 2) {
        cy.log(`图片 ${$img.attr('src')} 过大: ${naturalWidth}px > ${displayWidth}px`);
      }
    });
  });

  it('应该控制网络请求', () => {
    // 记录所有网络请求
    const requests = [];

    cy.intercept('*', (req) => {
      requests.push({
        url: req.url,
        method: req.method,
        timestamp: Date.now()
      });
    });

    cy.visit('/');

    // 等待所有请求完成
    cy.wait(1000);

    // 分析请求
    const slowRequests = requests.filter(req => {
      const duration = Date.now() - req.timestamp;
      return duration > 1000;
    });

    if (slowRequests.length > 0) {
      cy.log(`发现 ${slowRequests.length} 个慢请求`);
      slowRequests.forEach(req => {
        cy.log(`- ${req.method} ${req.url}`);
      });
    }
  });
});
```

### 5. Mock 数据测试

```javascript
// cypress/e2e/mocking.cy.js
describe('Mock 数据测试', () => {
  const mockUsers = [
    { id: 1, name: '张三', email: 'zhangsan@example.com' },
    { id: 2, name: '李四', email: 'lisi@example.com' },
    { id: 3, name: '王五', email: 'wangwu@example.com' }
  ];

  beforeEach(() => {
    // Mock 用户列表 API
    cy.intercept('GET', '/api/users', {
      statusCode: 200,
      body: {
        data: mockUsers,
        total: mockUsers.length
      }
    }).as('getUsers');

    // Mock 创建用户 API
    cy.intercept('POST', '/api/users', {
      statusCode: 201,
      body: {
        data: {
          id: 4,
          name: '新用户',
          email: 'newuser@example.com'
        }
      }
    }).as('createUser');

    // Mock 删除用户 API
    cy.intercept('DELETE', '/api/users/*', {
      statusCode: 204,
      body: {}
    }).as('deleteUser');

    // Mock 错误响应
    cy.intercept('GET', '/api/error', {
      statusCode: 500,
      body: {
        message: '服务器内部错误'
      }
    }).as('serverError');
  });

  it('应该使用 Mock 数据渲染用户列表', () => {
    cy.visit('/users');
    cy.wait('@getUsers');

    cy.get('[data-testid="user-card"]').should('have.length', mockUsers.length);

    mockUsers.forEach(user => {
      cy.contains(user.name).should('be.visible');
      cy.contains(user.email).should('be.visible');
    });
  });

  it('应该使用 Mock 数据创建用户', () => {
    cy.visit('/users/create');

    cy.get('[data-testid="name-input"]').type('新用户');
    cy.get('[data-testid="email-input"]').type('newuser@example.com');
    cy.get('[data-testid="submit-button"]').click();

    cy.wait('@createUser').then((interception) => {
      expect(interception.request.body).to.deep.equal({
        name: '新用户',
        email: 'newuser@example.com'
      });
    });

    cy.contains('用户创建成功').should('be.visible');
  });

  it('应该处理服务器错误', () => {
    cy.intercept('GET', '/api/users', {
      statusCode: 500,
      body: {
        message: '服务器内部错误'
      }
    }).as('getUsers');

    cy.visit('/users');
    cy.wait('@getUsers');

    cy.contains('加载用户列表失败').should('be.visible');
  });

  it('应该使用动态 Mock 数据', () => {
    cy.intercept('GET', '/api/users', (req) => {
      const searchTerm = req.url.includes('search=') ?
        req.url.split('search=')[1] : '';

      let filteredUsers = mockUsers;

      if (searchTerm) {
        filteredUsers = mockUsers.filter(user =>
          user.name.includes(searchTerm) || user.email.includes(searchTerm)
        );
      }

      req.reply({
        statusCode: 200,
        body: {
          data: filteredUsers,
          total: filteredUsers.length
        }
      });
    }).as('getUsers');

    cy.visit('/users?search=张三');
    cy.wait('@getUsers');

    cy.get('[data-testid="user-card"]').should('have.length', 1);
    cy.contains('张三').should('be.visible');
    cy.contains('李四').should('not.exist');
  });
});
```

---

## 注意事项

### 1. 测试最佳实践

```javascript
// ✅ 好的实践：使用 data-testid 选择器
it('应该点击按钮', () => {
  cy.get('[data-testid="submit-button"]').click();
});

// ❌ 不好的实践：使用不稳定的 CSS 选择器
it('应该点击按钮', () => {
  cy.get('.btn-primary:nth-child(2)').click();
});

// ✅ 好的实践：等待特定条件
it('应该等待元素可见', () => {
  cy.get('[data-testid="modal"]').should('be.visible');
});

// ❌ 不好的实践：使用固定等待时间
it('应该等待元素可见', () => {
  cy.wait(1000);
  cy.get('[data-testid="modal"]').should('be.visible');
});
```

### 2. 避免测试第三方服务

```javascript
// ✅ 好的实践：Mock 第三方服务
it('应该使用 Mock 的支付服务', () => {
  cy.intercept('POST', 'https://api.stripe.com/**', {
    statusCode: 200,
    body: { id: 'test_payment_id' }
  }).as('stripePayment');

  cy.get('[data-testid="pay-button"]').click();
  cy.wait('@stripePayment');
});

// ❌ 不好的实践：调用真实的第三方服务
it('应该调用真实的支付服务', () => {
  cy.get('[data-testid="pay-button"]').click();
  // 这会影响测试稳定性和成本
});
```

### 3. 测试隔离

```javascript
// ✅ 好的实践：每个测试独立运行
beforeEach(() => {
  // 清理测试数据
  cy.request('POST', '/api/test/reset');
});

it('应该创建新用户', () => {
  cy.visit('/users/create');
  // 创建用户测试
});

it('应该删除用户', () => {
  // 先创建测试用户
  cy.request('POST', '/api/users', {
    name: 'test',
    email: 'test@example.com'
  }).then(response => {
    const userId = response.body.data.id;

    cy.visit(`/users/${userId}`);
    cy.get('[data-testid="delete-button"]').click();
    // 删除用户测试
  });
});

// ❌ 不好的实践：测试相互依赖
let userId;

it('应该创建用户', () => {
  cy.request('POST', '/api/users', {
    name: 'test',
    email: 'test@example.com'
  }).then(response => {
    userId = response.body.data.id;
  });
});

it('应该删除用户', () => {
  // 依赖上一个测试的结果
  cy.visit(`/users/${userId}`);
  // 如果第一个测试失败，这个测试也会失败
});
```

### 4. 错误处理

```javascript
// ✅ 好的实践：处理可能失败的操作
it('应该处理网络错误', () => {
  cy.intercept('GET', '/api/users', {
    statusCode: 500,
    body: { message: 'Server error' }
  }).as('getUsers');

  cy.visit('/users');
  cy.wait('@getUsers');

  cy.contains('加载失败').should('be.visible');
  cy.get('[data-testid="retry-button"]').should('be.visible');
});

// ✅ 好的实践：使用重试机制
it('应该重试失败的操作', () => {
  let attemptCount = 0;

  cy.intercept('GET', '/api/unstable', (req) => {
    attemptCount++;

    if (attemptCount < 3) {
      req.reply({ statusCode: 500, body: { message: 'Server error' } });
    } else {
      req.reply({ statusCode: 200, body: { data: 'success' } });
    }
  }).as('getUnstableData');

  cy.visit('/unstable');
  cy.wait('@getUnstableData').its('response.statusCode').should('eq', 200);
});
```

---

## 总结

Cypress E2E 测试是保证应用质量的重要手段：

### 核心要点

1. **用户视角**：从用户角度测试完整流程
2. **真实环境**：在接近生产的环境中测试
3. **稳定性**：通过 mock 和重试提高测试稳定性
4. **可维护性**：编写清晰、可维护的测试代码

### 最佳实践

- 使用 data-testid 选择器提高稳定性
- 避免固定等待时间
- 保持测试独立和可重复
- Mock 外部依赖和第三方服务
- 合理使用自定义命令

### 常见场景

- 用户认证和授权流程
- 表单提交和验证
- 数据操作和状态管理
- 响应式设计和跨浏览器测试
- 性能监控和优化
- 错误处理和重试机制

### Cypress 优势

- 实时重载和时间旅行
- 丰富的调试工具
- 智能等待机制
- 网络请求拦截
- 截图和视频录制

通过掌握 Cypress E2E 测试，你可以构建更可靠、更稳定的应用。记住 E2E 测试是测试金字塔的顶端，虽然数量少但价值极高，要专注于关键的用户流程和业务逻辑。