---
title: JavaScript Web Storage
published: 2022-12-31
description: 'localStorage 和 sessionStorage的详细介绍和学习笔记'
image: ''
tags: ["存储"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

Web Storage API 提供了一种机制，通过浏览器以键值对的形式安全存储和访问数据。它包括两种存储方式：`localStorage` 和 `sessionStorage`，比传统的 Cookie 更大、更安全、更易用。

## 什么是 Web Storage

Web Storage 是 HTML5 提供的客户端存储技术，允许在浏览器中存储键值对数据。它解决了 Cookie 存储容量小、每个请求都会携带等问题。

### 存储类型对比

| 特性 | localStorage | sessionStorage | Cookie |
|------|-------------|----------------|--------|
| 存储容量 | ~5-10MB | ~5-10MB | ~4KB |
| 生命周期 | 永久（手动删除） | 会话结束（关闭标签页） | 可设置过期时间 |
| 作用域 | 同源策略 | 同源 + 同标签页 | 同源 + 可设置路径和域 |
| 与服务器通信 | 不参与 | 不参与 | 每次请求都会携带 |
| API 易用性 | 简单 | 简单 | 复杂

## localStorage 详解

### 基本概念

`localStorage` 用于持久化存储数据，除非主动删除，否则数据永远不会过期。数据存储在用户的浏览器中。

### 基本用法

```javascript
// 存储数据
localStorage.setItem('username', '张三');
localStorage.setItem('age', '25');

// 读取数据
const username = localStorage.getItem('username'); // '张三'
const age = localStorage.getItem('age'); // '25'

// 删除指定数据
localStorage.removeItem('username');

// 清空所有数据
localStorage.clear();

// 获取存储的数据数量
console.log(localStorage.length); // 1

// 获取指定索引的键名
console.log(localStorage.key(0)); // 'age'

// 遍历所有数据
for (let i = 0; i < localStorage.length; i++) {
  const key = localStorage.key(i);
  const value = localStorage.getItem(key);
  console.log(`${key}: ${value}`);
}
```

### 存储对象和数组

由于 localStorage 只能存储字符串，需要使用 JSON 转换：

```javascript
// 存储对象
const user = {
  id: 1,
  name: '张三',
  email: 'zhangsan@example.com'
};

localStorage.setItem('user', JSON.stringify(user));

// 读取对象
const storedUser = JSON.parse(localStorage.getItem('user'));
console.log(storedUser.name); // '张三'

// 存储数组
const todos = [
  { id: 1, text: '学习 JavaScript', done: false },
  { id: 2, text: '完成项目', done: true }
];

localStorage.setItem('todos', JSON.stringify(todos));

// 读取数组
const storedTodos = JSON.parse(localStorage.getItem('todos'));
console.log(storedTodos[0].text); // '学习 JavaScript'

// 更新数组中的项
todos[0].done = true;
localStorage.setItem('todos', JSON.stringify(todos));
```

### 安全存储

```javascript
// 封装安全的 localStorage 操作
const safeStorage = {
  // 安全存储
  set(key, value) {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (error) {
      console.error('存储失败:', error);
      return false;
    }
  },

  // 安全读取
  get(key, defaultValue = null) {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    } catch (error) {
      console.error('读取失败:', error);
      return defaultValue;
    }
  },

  // 安全删除
  remove(key) {
    try {
      localStorage.removeItem(key);
      return true;
    } catch (error) {
      console.error('删除失败:', error);
      return false;
    }
  },

  // 安全清空
  clear() {
    try {
      localStorage.clear();
      return true;
    } catch (error) {
      console.error('清空失败:', error);
      return false;
    }
  }
};

// 使用
safeStorage.set('user', { name: '张三' });
const user = safeStorage.get('user');
```

## sessionStorage 详解

### 基本概念

`sessionStorage` 用于存储一个会话的数据，数据只在当前标签页有效，关闭标签页后数据会被清除。

### 基本用法

```javascript
// 存储数据
sessionStorage.setItem('tabId', '12345');
sessionStorage.setItem('pageViews', '1');

// 读取数据
const tabId = sessionStorage.getItem('tabId'); // '12345'
const pageViews = sessionStorage.getItem('pageViews'); // '1'

// 更新数据
let views = parseInt(sessionStorage.getItem('pageViews') || '0');
sessionStorage.setItem('pageViews', (views + 1).toString());

// 删除数据
sessionStorage.removeItem('tabId');

// 清空当前会话数据
sessionStorage.clear();

// 检查存储状态
console.log(sessionStorage.length); // 数据项数量
console.log(sessionStorage.key(0)); // 获取键名
```

### 与 localStorage 的区别

```javascript
// localStorage 和 sessionStorage 的 API 完全相同
// 但存储的生命周期不同

// localStorage 示例
localStorage.setItem('persist', '这个数据会永久保存');
// 刷新页面、关闭浏览器后重新打开，数据仍然存在

// sessionStorage 示例
sessionStorage.setItem('temp', '这个数据只在当前标签页有效');
// 关闭标签页后数据会被清除
// 但刷新页面或在同域名的新标签页中打开链接，数据仍然存在

// 跨标签页测试
// 在标签页 A 中存储
localStorage.setItem('shared', '所有标签页都能访问');
sessionStorage.setItem('private', '只有当前标签页能访问');

// 在标签页 B 中读取
console.log(localStorage.getItem('shared')); // '所有标签页都能访问'
console.log(sessionStorage.getItem('private')); // null
```

## localStorage 和 sessionStorage 的区别

| 特性 | localStorage | sessionStorage |
|------|-------------|----------------|
| **生命周期** | 永久存储，除非手动删除 | 当前标签页会话结束即清除 |
| **作用域** | 同源的所有标签页共享 | 仅在当前标签页有效 |
| **存储位置** | 磁盘 | 内存 |
| **访问方式** | 所有同源页面可访问 | 只有创建它的标签页可访问 |
| **刷新页面** | 数据保留 | 数据保留 |
| **关闭标签页** | 数据保留 | 数据清除 |
| **关闭浏览器** | 数据保留 | 数据清除 |

```javascript
// 验证区别的示例代码

// 在控制台运行以下代码，然后在新标签页中读取

// localStorage 测试
localStorage.setItem('localTest', Date.now().toString());
console.log('localStorage 已设置');

// sessionStorage 测试
sessionStorage.setItem('sessionTest', Date.now().toString());
console.log('sessionStorage 已设置');

// 新标签页中运行
console.log(localStorage.getItem('localTest')); // 有值
console.log(sessionStorage.getItem('sessionTest')); // null
```

## 存储事件监听

### storage 事件

当 localStorage 或 sessionStorage 发生变化时，会触发 `storage` 事件。注意：**只有同源的其他页面才会收到事件**，当前页面不会收到。

```javascript
// 监听存储变化
window.addEventListener('storage', (event) => {
  console.log('存储发生变化');
  console.log('键:', event.key); // 变化的键名
  console.log('旧值:', event.oldValue); // 变化前的值
  console.log('新值:', event.newValue); // 变化后的值
  console.log('存储类型:', event.storageArea); // localStorage 或 sessionStorage
  console.log('URL:', event.url); // 触发变化的页面 URL
});

// 测试：在标签页 A 中运行监听器
// 然后在标签页 B 中修改 localStorage
localStorage.setItem('test', 'changed');
// 标签页 A 会收到事件
```

### 实际应用：多标签页数据同步

```javascript
// 在主应用中监听存储变化
class TabManager {
  constructor() {
    this.currentTabId = this.generateTabId();
    sessionStorage.setItem('currentTabId', this.currentTabId);

    this.setupStorageListener();
  }

  generateTabId() {
    return 'tab_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }

  setupStorageListener() {
    window.addEventListener('storage', (event) => {
      if (event.key === 'activeTab') {
        console.log(`标签页 ${event.newValue} 被激活`);
        this.handleTabSwitch(event.newValue);
      }

      if (event.key === 'sharedData') {
        console.log('共享数据更新:', event.newValue);
        this.handleDataUpdate(JSON.parse(event.newValue));
      }
    });
  }

  setActiveTab() {
    localStorage.setItem('activeTab', this.currentTabId);
  }

  shareData(data) {
    localStorage.setItem('sharedData', JSON.stringify({
      tabId: this.currentTabId,
      timestamp: Date.now(),
      data
    }));
  }

  handleTabSwitch(activeTabId) {
    if (activeTabId !== this.currentTabId) {
      console.log('其他标签页被激活，暂停某些操作');
      // 可以在这里暂停定时器、动画等
    }
  }

  handleDataUpdate(update) {
    if (update.tabId !== this.currentTabId) {
      console.log('接收其他标签页的数据:', update.data);
      // 处理数据更新
    }
  }
}

// 使用
const tabManager = new TabManager();
tabManager.setActiveTab();
tabManager.shareData({ message: 'Hello from this tab' });
```

## 实际应用场景

### 1. 用户偏好设置

```javascript
// 保存用户偏好
const userPreferences = {
  theme: 'dark',
  language: 'zh-CN',
  fontSize: 16,
  notifications: true
};

// 保存偏好设置
function savePreferences(prefs) {
  localStorage.setItem('preferences', JSON.stringify(prefs));
}

// 读取偏好设置
function loadPreferences() {
  const stored = localStorage.getItem('preferences');
  return stored ? JSON.parse(stored) : userPreferences;
}

// 应用偏好设置
function applyPreferences() {
  const prefs = loadPreferences();
  document.documentElement.setAttribute('data-theme', prefs.theme);
  document.documentElement.lang = prefs.language;
  document.body.style.fontSize = `${prefs.fontSize}px`;

  if (prefs.notifications) {
    enableNotifications();
  }
}

// 切换主题
function toggleTheme() {
  const prefs = loadPreferences();
  prefs.theme = prefs.theme === 'light' ? 'dark' : 'light';
  savePreferences(prefs);
  applyPreferences();
}
```

### 2. 购物车功能

```javascript
class ShoppingCart {
  constructor() {
    this.items = this.loadItems();
  }

  // 加载购物车
  loadItems() {
    const stored = localStorage.getItem('cart');
    return stored ? JSON.parse(stored) : [];
  }

  // 保存购物车
  saveItems() {
    localStorage.setItem('cart', JSON.stringify(this.items));
  }

  // 添加商品
  addItem(product) {
    const existingItem = this.items.find(item => item.id === product.id);

    if (existingItem) {
      existingItem.quantity += 1;
    } else {
      this.items.push({
        id: product.id,
        name: product.name,
        price: product.price,
        quantity: 1
      });
    }

    this.saveItems();
    this.notifyChange();
  }

  // 移除商品
  removeItem(productId) {
    this.items = this.items.filter(item => item.id !== productId);
    this.saveItems();
    this.notifyChange();
  }

  // 更新数量
  updateQuantity(productId, quantity) {
    const item = this.items.find(item => item.id === productId);

    if (item) {
      if (quantity <= 0) {
        this.removeItem(productId);
      } else {
        item.quantity = quantity;
        this.saveItems();
        this.notifyChange();
      }
    }
  }

  // 计算总价
  getTotalPrice() {
    return this.items.reduce((total, item) => {
      return total + (item.price * item.quantity);
    }, 0);
  }

  // 获取商品数量
  getItemCount() {
    return this.items.reduce((count, item) => count + item.quantity, 0);
  }

  // 清空购物车
  clear() {
    this.items = [];
    this.saveItems();
    this.notifyChange();
  }

  // 通知变化
  notifyChange() {
    // 可以派发自定义事件或触发回调
    window.dispatchEvent(new CustomEvent('cart-change', {
      detail: {
        items: this.items,
        count: this.getItemCount(),
        total: this.getTotalPrice()
      }
    }));
  }
}

// 使用
const cart = new ShoppingCart();

cart.addItem({ id: 1, name: '商品1', price: 99 });
cart.addItem({ id: 2, name: '商品2', price: 199 });

console.log(cart.getTotalPrice()); // 298
console.log(cart.getItemCount()); // 2

// 监听购物车变化
window.addEventListener('cart-change', (event) => {
  console.log('购物车已更新:', event.detail);
});
```

### 3. 表单自动保存

```javascript
class FormAutoSave {
  constructor(formId, storageKey = 'form-data') {
    this.form = document.getElementById(formId);
    this.storageKey = storageKey;
    this.inputs = Array.from(this.form.elements);
    this.debounceTimer = null;

    this.init();
  }

  init() {
    // 恢复数据
    this.restoreData();

    // 监听输入变化
    this.inputs.forEach(input => {
      if (input.name) {
        input.addEventListener('input', () => this.handleInput());
        input.addEventListener('change', () => this.handleInput());
      }
    });

    // 表单提交后清除数据
    this.form.addEventListener('submit', () => {
      this.clearData();
    });
  }

  handleInput() {
    // 防抖处理
    clearTimeout(this.debounceTimer);
    this.debounceTimer = setTimeout(() => {
      this.saveData();
    }, 500);
  }

  saveData() {
    const formData = {};

    this.inputs.forEach(input => {
      if (input.name && !input.disabled) {
        if (input.type === 'checkbox') {
          formData[input.name] = input.checked;
        } else if (input.type === 'radio') {
          if (input.checked) {
            formData[input.name] = input.value;
          }
        } else if (input.type === 'select-multiple') {
          formData[input.name] = Array.from(input.selectedOptions).map(opt => opt.value);
        } else {
          formData[input.name] = input.value;
        }
      }
    });

    localStorage.setItem(this.storageKey, JSON.stringify(formData));
    console.log('表单数据已自动保存');
  }

  restoreData() {
    const stored = localStorage.getItem(this.storageKey);

    if (stored) {
      const formData = JSON.parse(stored);

      this.inputs.forEach(input => {
        if (input.name && formData.hasOwnProperty(input.name) && !input.disabled) {
          if (input.type === 'checkbox') {
            input.checked = formData[input.name];
          } else if (input.type === 'radio') {
            input.checked = input.value === formData[input.name];
          } else if (input.type === 'select-multiple') {
            Array.from(input.options).forEach(opt => {
              opt.selected = formData[input.name].includes(opt.value);
            });
          } else {
            input.value = formData[input.name];
          }
        }
      });

      console.log('表单数据已恢复');
    }
  }

  clearData() {
    localStorage.removeItem(this.storageKey);
    console.log('表单数据已清除');
  }
}

// 使用
const autoSave = new FormAutoSave('my-form', 'user-profile-form');
```

### 4. 离线数据缓存

```javascript
class OfflineCache {
  constructor(cacheName = 'offline-data') {
    this.cacheName = cacheName;
  }

  // 缓存数据
  async cacheData(key, data, ttl = 3600000) {
    const cacheItem = {
      data,
      timestamp: Date.now(),
      ttl
    };

    localStorage.setItem(`${this.cacheName}-${key}`, JSON.stringify(cacheItem));
  }

  // 获取缓存数据
  async getCachedData(key) {
    const cached = localStorage.getItem(`${this.cacheName}-${key}`);

    if (!cached) {
      return null;
    }

    const cacheItem = JSON.parse(cached);
    const isExpired = Date.now() - cacheItem.timestamp > cacheItem.ttl;

    if (isExpired) {
      localStorage.removeItem(`${this.cacheName}-${key}`);
      return null;
    }

    return cacheItem.data;
  }

  // 获取数据，优先使用缓存
  async getData(key, fetcher, ttl = 3600000) {
    // 先尝试从缓存获取
    const cached = await this.getCachedData(key);

    if (cached) {
      console.log('使用缓存数据');
      return cached;
    }

    // 缓存未命中，获取新数据
    console.log('获取新数据');
    const data = await fetcher();

    // 缓存数据
    await this.cacheData(key, data, ttl);

    return data;
  }

  // 清除缓存
  clearCache(key = null) {
    if (key) {
      localStorage.removeItem(`${this.cacheName}-${key}`);
    } else {
      // 清除所有相关缓存
      const keys = Object.keys(localStorage);
      keys.forEach(k => {
        if (k.startsWith(this.cacheName)) {
          localStorage.removeItem(k);
        }
      });
    }
  }

  // 获取缓存大小
  getCacheSize() {
    let size = 0;
    const keys = Object.keys(localStorage);

    keys.forEach(key => {
      if (key.startsWith(this.cacheName)) {
        size += localStorage.getItem(key).length;
      }
    });

    return size;
  }
}

// 使用
const cache = new OfflineCache('api-cache');

async function fetchUser(userId) {
  return cache.getData(
    `user-${userId}`,
    async () => {
      const response = await fetch(`/api/users/${userId}`);
      return response.json();
    },
    60000 // 缓存 1 分钟
  );
}

// 使用
fetchUser(1).then(user => {
  console.log(user);
});
```

### 5. 会话状态管理

```javascript
class SessionManager {
  constructor() {
    this.sessionKey = 'app-session';
    this.session = this.loadSession();
    this.activityTimer = null;
    this.sessionTimeout = 30 * 60 * 1000; // 30 分钟

    this.init();
  }

  init() {
    // 恢复会话
    this.restoreSession();

    // 监听用户活动
    this.trackActivity();

    // 定期保存会话
    setInterval(() => this.saveSession(), 60000);

    // 页面卸载时保存会话
    window.addEventListener('beforeunload', () => {
      this.saveSession();
    });
  }

  loadSession() {
    const stored = sessionStorage.getItem(this.sessionKey);
    return stored ? JSON.parse(stored) : this.createSession();
  }

  createSession() {
    return {
      id: this.generateSessionId(),
      startTime: Date.now(),
      lastActivity: Date.now(),
      pageViews: 0,
      data: {}
    };
  }

  generateSessionId() {
    return 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2, 9);
  }

  saveSession() {
    this.session.lastActivity = Date.now();
    sessionStorage.setItem(this.sessionKey, JSON.stringify(this.session));
  }

  restoreSession() {
    // 检查会话是否过期
    const isExpired = Date.now() - this.session.lastActivity > this.sessionTimeout;

    if (isExpired) {
      console.log('会话已过期，创建新会话');
      this.session = this.createSession();
    } else {
      console.log('会话已恢复:', this.session.id);
    }
  }

  trackActivity() {
    const events = ['click', 'keypress', 'scroll', 'mousemove'];

    events.forEach(event => {
      document.addEventListener(event, () => {
        clearTimeout(this.activityTimer);
        this.activityTimer = setTimeout(() => {
          this.saveSession();
        }, 1000);
      }, { passive: true });
    });

    this.incrementPageViews();
  }

  incrementPageViews() {
    this.session.pageViews++;
    this.saveSession();
  }

  setSessionData(key, value) {
    this.session.data[key] = value;
    this.saveSession();
  }

  getSessionData(key) {
    return this.session.data[key];
  }

  getSessionInfo() {
    return {
      id: this.session.id,
      duration: Date.now() - this.session.startTime,
      pageViews: this.session.pageViews,
      isActive: Date.now() - this.session.lastActivity < this.sessionTimeout
    };
  }

  destroySession() {
    sessionStorage.removeItem(this.sessionKey);
    this.session = this.createSession();
  }
}

// 使用
const sessionManager = new SessionManager();

sessionManager.setSessionData('user', { id: 1, name: '张三' });
const user = sessionManager.getSessionData('user');

console.log('会话信息:', sessionManager.getSessionInfo());
```

## 注意事项

### 1. 存储容量限制

```javascript
// 检查存储容量
function checkStorageCapacity(storage = localStorage) {
  const testKey = 'test-key';
  let data = 'x';

  try {
    // 逐步增加数据量
    while (true) {
      storage.setItem(testKey, data);
      data += data;
    }
  } catch (error) {
    storage.removeItem(testKey);

    // 计算容量
    const sizeKB = (data.length / 1024).toFixed(2);
    console.log(`存储容量约为: ${sizeKB} KB`);
    return parseFloat(sizeKB);
  }
}

// 使用
checkStorageCapacity(localStorage);
checkStorageCapacity(sessionStorage);
```

### 2. 数据安全性

```javascript
// 不要存储敏感信息
const badPractice = {
  // ❌ 不要存储密码
  password: 'user123',

  // ❌ 不要存储 Token
  authToken: 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...',

  // ❌ 不要存储信用卡信息
  creditCard: '4111-1111-1111-1111'
};

localStorage.setItem('sensitive', JSON.stringify(badPractice));

// ✅ 可以存储非敏感信息
const goodPractice = {
  theme: 'dark',
  language: 'zh-CN',
  preferences: { fontSize: 16 }
};

localStorage.setItem('preferences', JSON.stringify(goodPractice));

// 敏感信息应该使用 HttpOnly Cookie
// 或在内存中管理（sessionStorage + HTTPS）
```

### 3. 同步存储问题

```javascript
// localStorage 和 sessionStorage 都是同步 API
// 大量数据可能会阻塞主线程

// ❌ 不好的做法：存储大量数据
const hugeData = new Array(1000000).fill('x').join('');
localStorage.setItem('huge', hugeData); // 可能会卡顿

// ✅ 好的做法：分批存储或使用 IndexedDB
function batchStore(key, data, chunkSize = 100000) {
  const chunks = [];

  for (let i = 0; i < data.length; i += chunkSize) {
    chunks.push(data.slice(i, i + chunkSize));
  }

  chunks.forEach((chunk, index) => {
    localStorage.setItem(`${key}_${index}`, chunk);
  });

  localStorage.setItem(`${key}_meta`, JSON.stringify({
    chunks: chunks.length,
    chunkSize
  }));
}

// 或使用异步的 IndexedDB
```

### 4. 浏览器兼容性

```javascript
// 检查浏览器支持
function isStorageSupported(storage) {
  try {
    const testKey = '__storage_test__';
    storage.setItem(testKey, 'test');
    storage.removeItem(testKey);
    return true;
  } catch (error) {
    return false;
  }
}

// 检查 localStorage 支持
const localStorageSupported = isStorageSupported(localStorage);
const sessionStorageSupported = isStorageSupported(sessionStorage);

if (!localStorageSupported) {
  console.error('浏览器不支持 localStorage');
  // 降级方案：使用 Cookie 或内存存储
}

// 检查是否在隐私模式
function isPrivateMode() {
  try {
    localStorage.setItem('test', 'test');
    localStorage.removeItem('test');
    return false;
  } catch (error) {
    return true;
  }
}

if (isPrivateMode()) {
  console.warn('当前处于隐私模式，存储功能受限');
}
```

### 5. 数据类型处理

```javascript
// localStorage 只能存储字符串
// 需要正确的序列化和反序列化

// ❌ 错误做法
const obj = { name: '张三', age: 25 };
localStorage.setItem('user', obj); // 存储为 "[object Object]"
localStorage.getItem('user'); // "[object Object]"

// ✅ 正确做法：使用 JSON
localStorage.setItem('user', JSON.stringify(obj));
JSON.parse(localStorage.getItem('user')); // { name: '张三', age: 25 }

// 封装类型处理
const typedStorage = {
  set(key, value) {
    const item = {
      value,
      type: typeof value,
      timestamp: Date.now()
    };
    localStorage.setItem(key, JSON.stringify(item));
  },

  get(key) {
    const item = localStorage.getItem(key);
    if (!item) return null;

    const parsed = JSON.parse(item);
    return parsed.value;
  }
};

// 使用
typedStorage.set('number', 123);
typedStorage.set('boolean', true);
typedStorage.set('object', { a: 1 });

console.log(typedStorage.get('number')); // 123
console.log(typedStorage.get('boolean')); // true
console.log(typedStorage.get('object')); // { a: 1 }
```

### 6. 清理和过期管理

```javascript
class ExpiringStorage {
  constructor(storage = localStorage, prefix = 'exp_') {
    this.storage = storage;
    this.prefix = prefix;
  }

  // 设置带过期时间的数据
  setItem(key, value, ttl = 3600000) {
    const item = {
      value,
      expiresAt: Date.now() + ttl
    };

    this.storage.setItem(this.prefix + key, JSON.stringify(item));
  }

  // 获取数据，自动检查过期
  getItem(key) {
    const item = this.storage.getItem(this.prefix + key);

    if (!item) {
      return null;
    }

    const parsed = JSON.parse(item);

    // 检查是否过期
    if (Date.now() > parsed.expiresAt) {
      this.removeItem(key);
      return null;
    }

    return parsed.value;
  }

  removeItem(key) {
    this.storage.removeItem(this.prefix + key);
  }

  // 清理所有过期数据
  cleanExpired() {
    const now = Date.now();
    const keys = Object.keys(this.storage);

    keys.forEach(key => {
      if (key.startsWith(this.prefix)) {
        try {
          const item = JSON.parse(this.storage.getItem(key));

          if (now > item.expiresAt) {
            this.storage.removeItem(key);
          }
        } catch (error) {
          // 解析失败，删除该项
          this.storage.removeItem(key);
        }
      }
    });
  }

  // 获取所有键
  keys() {
    return Object.keys(this.storage)
      .filter(key => key.startsWith(this.prefix))
      .map(key => key.slice(this.prefix.length));
  }
}

// 使用
const expiringStorage = new ExpiringStorage();

// 设置 5 分钟后过期
expiringStorage.setItem('user', { name: '张三' }, 5 * 60 * 1000);

// 获取数据（自动检查过期）
const user = expiringStorage.getItem('user');

// 清理过期数据
expiringStorage.cleanExpired();

// 定期清理
setInterval(() => {
  expiringStorage.cleanExpired();
}, 60000); // 每分钟清理一次
```

## 最佳实践

### 1. 使用命名空间

```javascript
// 避免键名冲突，使用命名空间
const AppStorage = {
  USER_PREFIX: 'user_',
  PREF_PREFIX: 'pref_',
  CACHE_PREFIX: 'cache_',

  // 用户相关
  setUser(key, value) {
    localStorage.setItem(this.USER_PREFIX + key, JSON.stringify(value));
  },

  getUser(key) {
    const item = localStorage.getItem(this.USER_PREFIX + key);
    return item ? JSON.parse(item) : null;
  },

  // 偏好设置
  setPref(key, value) {
    localStorage.setItem(this.PREF_PREFIX + key, JSON.stringify(value));
  },

  getPref(key) {
    const item = localStorage.getItem(this.PREF_PREFIX + key);
    return item ? JSON.parse(item) : null;
  },

  // 缓存
  setCache(key, value, ttl = 3600000) {
    const item = { value, expiresAt: Date.now() + ttl };
    localStorage.setItem(this.CACHE_PREFIX + key, JSON.stringify(item));
  },

  getCache(key) {
    const item = localStorage.getItem(this.CACHE_PREFIX + key);
    if (!item) return null;

    const parsed = JSON.parse(item);

    if (Date.now() > parsed.expiresAt) {
      localStorage.removeItem(this.CACHE_PREFIX + key);
      return null;
    }

    return parsed.value;
  }
};
```

### 2. 错误处理

```javascript
// 完善的错误处理
const SafeStorage = {
  tryOperation(operation, fallback = null) {
    try {
      return operation();
    } catch (error) {
      console.error('Storage operation failed:', error);

      // 检查具体错误类型
      if (error.name === 'QuotaExceededError') {
        console.error('存储空间已满');
        this.cleanOldItems();
      }

      return fallback;
    }
  },

  setItem(key, value) {
    return this.tryOperation(() => {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    }, false);
  },

  getItem(key, defaultValue = null) {
    return this.tryOperation(() => {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : defaultValue;
    }, defaultValue);
  },

  removeItem(key) {
    return this.tryOperation(() => {
      localStorage.removeItem(key);
      return true;
    }, false);
  },

  // 清理旧数据
  cleanOldItems() {
    const keys = Object.keys(localStorage);
    const now = Date.now();

    keys.forEach(key => {
      try {
        const item = JSON.parse(localStorage.getItem(key));

        if (item.expiresAt && now > item.expiresAt) {
          localStorage.removeItem(key);
        }
      } catch (error) {
        // 无法解析的项目，可能需要处理
      }
    });
  }
};
```

### 3. 版本管理

```javascript
// 支持数据结构版本升级
class VersionedStorage {
  constructor(key, version = 1) {
    this.key = key;
    this.currentVersion = version;
  }

  save(data) {
    const item = {
      version: this.currentVersion,
      data,
      timestamp: Date.now()
    };

    localStorage.setItem(this.key, JSON.stringify(item));
  }

  load() {
    const item = localStorage.getItem(this.key);

    if (!item) {
      return null;
    }

    const parsed = JSON.parse(item);

    // 检查版本并迁移
    if (parsed.version < this.currentVersion) {
      return this.migrate(parsed);
    }

    return parsed.data;
  }

  migrate(item) {
    console.log(`迁移数据从版本 ${item.version} 到 ${this.currentVersion}`);

    let data = item.data;

    // 版本迁移逻辑
    for (let v = item.version; v < this.currentVersion; v++) {
      data = this.migrateToVersion(data, v + 1);
    }

    // 保存迁移后的数据
    this.save(data);

    return data;
  }

  migrateToVersion(data, targetVersion) {
    switch (targetVersion) {
      case 2:
        // 从版本 1 迁移到版本 2
        return { ...data, migrated: true };

      case 3:
        // 从版本 2 迁移到版本 3
        return { ...data, newField: 'default' };

      default:
        return data;
    }
  }
}

// 使用
const userStorage = new VersionedStorage('user-data', 3);

userStorage.save({ name: '张三' });
const user = userStorage.load();
```

### 4. 性能优化

```javascript
// 批量操作减少 I/O
class BatchStorage {
  constructor() {
    this.pendingWrites = new Map();
    this.flushTimer = null;
    this.batchSize = 10;
  }

  set(key, value) {
    this.pendingWrites.set(key, value);

    if (this.pendingWrites.size >= this.batchSize) {
      this.flush();
    } else {
      this.scheduleFlush();
    }
  }

  scheduleFlush() {
    if (!this.flushTimer) {
      this.flushTimer = setTimeout(() => this.flush(), 100);
    }
  }

  flush() {
    clearTimeout(this.flushTimer);
    this.flushTimer = null;

    this.pendingWrites.forEach((value, key) => {
      localStorage.setItem(key, JSON.stringify(value));
    });

    this.pendingWrites.clear();
  }

  // 确保所有数据都已写入
  async ensureFlushed() {
    if (this.flushTimer) {
      await new Promise(resolve => {
        this.flushTimer = setTimeout(() => {
          this.flush();
          resolve();
        }, 0);
      });
    }
  }
}

// 使用
const batchStorage = new BatchStorage();

batchStorage.set('key1', 'value1');
batchStorage.set('key2', 'value2');
batchStorage.set('key3', 'value3');
// 数据会在适当时机批量写入
```

## 总结

JavaScript Web Storage 提供了强大而简单的客户端存储能力：

- **localStorage**：持久化存储，适合保存用户偏好、缓存数据等
- **sessionStorage**：会话存储，适合临时数据、表单自动保存等
- **容量充足**：通常 5-10MB，远大于 Cookie
- **API 简单**：使用键值对存储，易于理解和使用
- **同源策略**：只在同一域名下共享数据

在实际开发中，要注意：
- 不要存储敏感信息
- 正确处理数据类型（JSON 序列化）
- 管理存储容量和过期时间
- 处理浏览器兼容性
- 使用命名空间避免键名冲突

合理使用 Web Storage 可以显著提升应用的用户体验，实现离线缓存、数据持久化等功能。对于更复杂的存储需求，可以考虑使用 IndexedDB。