---
title: JavaScript 异步编程总结
published: 2022-04-25
description: '回调、Promise、async/await 对比的详细介绍和学习笔记'
image: ''
tags: ["JS基础","异步"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 是单线程语言，但支持异步编程。异步编程让我们可以处理耗时操作（如网络请求、文件操作）而不会阻塞主线程。本文将总结 JavaScript 异步编程的三种主要方式：回调函数、Promise 和 async/await，并对比它们的优缺点和适用场景。

## 回调函数

### 基本概念

回调函数是最早的异步编程方式，将一个函数作为参数传递给另一个函数，在操作完成后执行。

### 回调函数示例

```javascript
// 基本回调
function fetchData(callback) {
  setTimeout(() => {
    const data = { id: 1, name: '张三' };
    callback(data);
  }, 1000);
}

fetchData(function(data) {
  console.log(data); // { id: 1, name: '张三' }
});

// 带错误处理的回调
function fetchDataWithError(callback) {
  setTimeout(() => {
    const success = true;
    if (success) {
      callback(null, { id: 1, name: '张三' });
    } else {
      callback(new Error('获取数据失败'));
    }
  }, 1000);
}

fetchDataWithError(function(error, data) {
  if (error) {
    console.error('错误:', error);
  } else {
    console.log('数据:', data);
  }
});
```

### 回调地狱

```javascript
// 回调地狱示例
function getUserInfo(userId, callback) {
  fetchUser(userId, function(user) {
    fetchPosts(user.id, function(posts) {
      fetchComments(posts[0].id, function(comments) {
        fetchAuthor(posts[0].authorId, function(author) {
          callback({ user, posts, comments, author });
        });
      });
    });
  });
}

getUserInfo(1, function(result) {
  console.log(result);
});
```

### 回调的优缺点

**优点：**
- 简单直接，易于理解
- 兼容性好，所有浏览器都支持
- 性能开销小

**缺点：**
- 回调地狱导致代码难以维护
- 错误处理复杂
- 难以进行控制流操作
- 代码可读性差

## Promise

### 基本概念

Promise 是 ES6 引入的异步编程解决方案，代表一个异步操作的最终完成或失败。

### Promise 基础示例

```javascript
// 创建 Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve({ id: 1, name: '张三' });
    } else {
      reject(new Error('获取数据失败'));
    }
  }, 1000);
});

// 使用 Promise
promise
  .then(data => console.log(data))
  .catch(error => console.error(error));

// Promise 封装异步函数
function fetchData(url) {
  return new Promise((resolve, reject) => {
    fetch(url)
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP 错误: ${response.status}`);
        }
        return response.json();
      })
      .then(data => resolve(data))
      .catch(error => reject(error));
  });
}

// 使用封装的函数
fetchData('/api/user')
  .then(user => console.log(user))
  .catch(error => console.error(error));
```

### Promise 链式调用

```javascript
// 链式调用替代回调地狱
function getUserInfo(userId) {
  return fetchUser(userId)
    .then(user => {
      return Promise.all([
        user,
        fetchPosts(user.id),
        fetchUserProfile(user.id)
      ]);
    })
    .then(([user, posts, profile]) => {
      return {
        user,
        posts,
        profile
      };
    });
}

getUserInfo(1)
  .then(result => console.log(result))
  .catch(error => console.error(error));
```

### Promise 静态方法

```javascript
// Promise.all - 所有 Promise 都成功才成功
Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
  .then(([user, posts, comments]) => {
    console.log('所有数据加载完成');
  })
  .catch(error => {
    console.error('某个请求失败:', error);
  });

// Promise.race - 返回最先完成的 Promise
Promise.race([
  fetchFromSource1(),
  fetchFromSource2()
])
  .then(result => {
    console.log('最快的数据源响应:', result);
  });

// Promise.allSettled - 等待所有 Promise 完成
Promise.allSettled([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === 'fulfilled') {
        console.log(`请求 ${index} 成功:`, result.value);
      } else {
        console.log(`请求 ${index} 失败:`, result.reason);
      }
    });
  });

// Promise.any - 返回第一个成功的 Promise
Promise.any([
  fetchFromSource1(),
  fetchFromSource2(),
  fetchFromSource3()
])
  .then(result => {
    console.log('第一个成功的数据源:', result);
  })
  .catch(error => {
    console.error('所有数据源都失败');
  });
```

### Promise 的优缺点

**优点：**
- 解决回调地狱，代码更扁平
- 统一的错误处理机制
- 支持链式调用，代码可读性好
- 提供丰富的静态方法处理复杂场景

**缺点：**
- 代码量相对较多
- 仍然需要理解 Promise 的工作原理
- 错误处理可能在链中丢失

## async/await

### 基本概念

async/await 是 ES2017 引入的语法糖，基于 Promise 实现，让异步代码看起来像同步代码。

### async/await 基础示例

```javascript
// async 函数总是返回 Promise
async function fetchData() {
  const response = await fetch('/api/user');
  const data = await response.json();
  return data;
}

// 使用 async 函数
fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// 带错误处理的 async 函数
async function fetchUserWithErrorHandling() {
  try {
    const response = await fetch('/api/user');
    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('获取用户失败:', error);
    throw error;
  }
}
```

### async/await 链式操作

```javascript
// 简洁的异步流程
async function getUserInfo(userId) {
  try {
    const user = await fetchUser(userId);

    const [posts, profile] = await Promise.all([
      fetchPosts(user.id),
      fetchUserProfile(user.id)
    ]);

    return {
      user,
      posts,
      profile
    };
  } catch (error) {
    console.error('获取用户信息失败:', error);
    throw error;
  }
}

// 使用
async function displayUserInfo() {
  try {
    const userInfo = await getUserInfo(1);
    console.log(userInfo);
  } catch (error) {
    console.error('显示用户信息失败:', error);
  }
}
```

### async/await 并行执行

```javascript
// 串行执行（慢）
async function serial() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { user, posts, comments };
}

// 并行执行（快）
async function parallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
}

// 部分失败处理
async function partialFailure() {
  const results = await Promise.allSettled([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  return {
    successful,
    failed,
    successCount: successful.length,
    failureCount: failed.length
  };
}
```

### async/await 的优缺点

**优点：**
- 代码像同步代码，可读性极佳
- 统一的 try-catch 错误处理
- 调试更方便，可以设置断点
- 更容易理解执行流程

**缺点：**
- 需要现代浏览器或转译器支持
- 滥用 await 可能导致性能问题
- 过度串行可能影响性能

## 三种方式对比

### 代码可读性对比

```javascript
// 回调方式
function getUserData(userId, callback) {
  fetchUser(userId, function(user) {
    fetchPosts(user.id, function(posts) {
      fetchComments(posts[0].id, function(comments) {
        callback({ user, posts, comments });
      });
    });
  });
}

// Promise 方式
function getUserData(userId) {
  return fetchUser(userId)
    .then(user => fetchPosts(user.id))
    .then(posts => fetchComments(posts[0].id))
    .then(comments => ({ user, posts, comments }));
}

// async/await 方式
async function getUserData(userId) {
  const user = await fetchUser(userId);
  const posts = await fetchPosts(user.id);
  const comments = await fetchComments(posts[0].id);
  return { user, posts, comments };
}
```

### 错误处理对比

```javascript
// 回调错误处理
function fetchData(callback) {
  setTimeout(() => {
    const success = Math.random() > 0.5;
    if (success) {
      callback(null, { data: 'success' });
    } else {
      callback(new Error('failed'));
    }
  }, 1000);
}

fetchData(function(error, data) {
  if (error) {
    console.error('错误:', error);
  } else {
    console.log('数据:', data);
  }
});

// Promise 错误处理
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = Math.random() > 0.5;
      if (success) {
        resolve({ data: 'success' });
      } else {
        reject(new Error('failed'));
      }
    }, 1000);
  });
}

fetchData()
  .then(data => console.log('数据:', data))
  .catch(error => console.error('错误:', error));

// async/await 错误处理
async function fetchData() {
  const success = Math.random() > 0.5;
  if (success) {
    return { data: 'success' };
  } else {
    throw new Error('failed');
  }
}

async function main() {
  try {
    const data = await fetchData();
    console.log('数据:', data);
  } catch (error) {
    console.error('错误:', error);
  }
}
```

### 性能对比

```javascript
// 回调性能（最佳）
function callbackMethod(items, callback) {
  const results = [];
  let completed = 0;

  items.forEach(item => {
    asyncOperation(item, (error, result) => {
      if (!error) {
        results.push(result);
      }
      completed++;

      if (completed === items.length) {
        callback(null, results);
      }
    });
  });
}

// Promise 性能（良好）
function promiseMethod(items) {
  return Promise.all(items.map(item => asyncOperation(item)));
}

// async/await 性能（依赖实现）
async function asyncMethod(items) {
  const results = [];
  for (const item of items) {
    const result = await asyncOperation(item);
    results.push(result);
  }
  return results;
}

// 正确的 async/await 并行
async function asyncParallelMethod(items) {
  return await Promise.all(items.map(item => asyncOperation(item)));
}
```

## 使用场景指南

### 使用回调函数的场景

1. **传统库和框架**：如 jQuery、Node.js 早期版本
2. **简单的一次性操作**：如事件监听
3. **性能敏感场景**：需要最小化内存开销
4. **向后兼容性要求**：需要支持旧浏览器

```javascript
// 事件监听 - 回调适合
button.addEventListener('click', function(event) {
  console.log('按钮被点击:', event);
});

// Node.js 风格的回调
fs.readFile('file.txt', 'utf8', function(err, data) {
  if (err) {
    console.error('读取文件失败:', err);
    return;
  }
  console.log('文件内容:', data);
});
```

### 使用 Promise 的场景

1. **现代 Web 应用**：需要处理多个异步操作
2. **链式操作**：需要按顺序执行多个异步任务
3. **错误集中处理**：需要统一处理多个可能的错误
4. **Promise API**：使用返回 Promise 的 API

```javascript
// 链式操作
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error));

// Promise 组合
Promise.all([
  fetchUser(),
  fetchPosts(),
  fetchComments()
])
  .then(([user, posts, comments]) => {
    console.log('所有数据加载完成');
  });
```

### 使用 async/await 的场景

1. **复杂的异步流程**：需要清晰的执行顺序
2. **需要条件执行**：根据条件决定是否等待
3. **需要循环处理**：在循环中执行异步操作
4. **错误处理复杂**：需要细粒度的错误处理

```javascript
// 复杂的异步流程
async function processOrder(orderId) {
  try {
    // 第一步：验证订单
    const order = await validateOrder(orderId);
    console.log('订单验证成功');

    // 第二步：检查库存
    const inventory = await checkInventory(order.items);
    if (!inventory.available) {
      throw new Error('库存不足');
    }
    console.log('库存检查通过');

    // 第三步：扣减库存
    await deductInventory(order.items);
    console.log('库存扣减成功');

    // 第四步：创建支付
    const payment = await createPayment(order);
    console.log('支付创建成功');

    // 第五步：发送通知
    await sendNotification(order.userId, 'order_created');
    console.log('通知发送成功');

    return { success: true, orderId: order.id };

  } catch (error) {
    console.error('订单处理失败:', error);

    // 错误恢复
    await rollbackOrder(orderId);
    await sendNotification(order.userId, 'order_failed');

    throw error;
  }
}

// 循环处理
async function processItems(items) {
  const results = [];

  for (const item of items) {
    try {
      const result = await processItem(item);
      results.push({ success: true, item, result });
    } catch (error) {
      results.push({ success: false, item, error: error.message });
    }
  }

  return results;
}
```

## 迁移指南

### 回调转 Promise

```javascript
// 原始回调函数
function fetchDataCallback(callback) {
  setTimeout(() => {
    callback(null, { id: 1, name: '张三' });
  }, 1000);
}

// 转换为 Promise
function fetchDataPromise() {
  return new Promise((resolve, reject) => {
    fetchDataCallback((error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// 通用转换函数
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (error, result) => {
        if (error) {
          reject(error);
        } else {
          resolve(result);
        }
      });
    });
  };
}

// 使用
const fetchUser = promisify(fetchDataCallback);
fetchUser().then(user => console.log(user));
```

### Promise 转 async/await

```javascript
// 原始 Promise 链
function getUserData(userId) {
  return fetchUser(userId)
    .then(user => fetchPosts(user.id))
    .then(posts => fetchComments(posts[0].id))
    .then(comments => ({ user, posts, comments }))
    .catch(error => {
      console.error('获取数据失败:', error);
      throw error;
    });
}

// 转换为 async/await
async function getUserData(userId) {
  try {
    const user = await fetchUser(userId);
    const posts = await fetchPosts(user.id);
    const comments = await fetchComments(posts[0].id);
    return { user, posts, comments };
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error;
  }
}
```

## 最佳实践

### 1. 选择合适的异步方式

```javascript
// 简单事件处理 - 使用回调
element.addEventListener('click', () => {
  console.log('Clicked');
});

// 多步骤异步流程 - 使用 async/await
async function handleUserAction() {
  const user = await getCurrentUser();
  const permissions = await getUserPermissions(user.id);
  const data = await fetchData(permissions);
  renderData(data);
}

// 独立异步操作 - 使用 Promise.all
async function loadDashboard() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  renderDashboard({ user, posts, comments });
}
```

### 2. 错误处理策略

```javascript
// 全局错误处理
window.addEventListener('unhandledrejection', (event) => {
  console.error('未处理的 Promise 拒绝:', event.reason);
  // 发送到错误监控服务
  sendErrorToMonitoring(event.reason);
});

// 分层错误处理
async function robustOperation() {
  try {
    const data = await fetchData();
    return processData(data);
  } catch (error) {
    // 提供降级数据
    console.error('操作失败，使用默认数据:', error);
    return getDefaultData();
  }
}

// 错误边界
async function withFallback(primary, fallback) {
  try {
    return await primary();
  } catch (error) {
    console.log('主要方法失败，尝试备用方法:', error);
    return await fallback();
  }
}

// 使用
const data = await withFallback(
  () => fetchFromPrimary(),
  () => fetchFromBackup()
);
```

### 3. 性能优化

```javascript
// 避免串行等待
async function badPerformance() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { user, posts, comments };
}

// 并行执行
async function goodPerformance() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);
  return { user, posts, comments };
}

// 按需并行
async function smartParallel() {
  const user = await fetchUser();

  const [posts, comments] = await Promise.all([
    fetchPosts(user.id),
    fetchComments(user.id)
  ]);

  return { user, posts, comments };
}
```

### 4. 代码组织

```javascript
// 使用类组织异步逻辑
class UserService {
  async getUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }

  async getUserWithDetails(userId) {
    const user = await this.getUser(userId);

    const [posts, comments] = await Promise.all([
      this.fetchPosts(userId),
      this.fetchComments(userId)
    ]);

    return { user, posts, comments };
  }

  async fetchPosts(userId) {
    const response = await fetch(`/api/users/${userId}/posts`);
    return response.json();
  }

  async fetchComments(userId) {
    const response = await fetch(`/api/users/${userId}/comments`);
    return response.json();
  }
}

// 使用
const userService = new UserService();
const userDetails = await userService.getUserWithDetails(1);
```

## 注意事项

1. **避免混合使用异步方式**：选择一种异步方式并在项目中保持一致。

2. **注意内存泄漏**：长时间运行的异步操作要正确清理资源。

3. **处理所有错误**：确保所有异步操作都有适当的错误处理。

4. **避免过度嵌套**：即使使用 Promise 或 async/await，也要保持代码扁平。

5. **考虑并发限制**：大量并发请求可能导致资源耗尽。

6. **使用超时控制**：为网络请求等异步操作设置合理的超时。

7. **测试异步代码**：异步代码需要专门的测试策略和工具。

8. **文档说明异步行为**：清晰标注函数的异步特性和返回值类型。

## 总结

JavaScript 异步编程已经从回调函数发展到 Promise，再到 async/await，每种方式都有其适用场景：

- **回调函数**：适合简单的异步操作和向后兼容
- **Promise**：适合处理多个异步操作和链式调用
- **async/await**：适合复杂的异步流程和错误处理

在现代 JavaScript 开发中，async/await 是首选的异步编程方式，它提供了最佳的可读性和错误处理能力。Promise 仍然是重要的基础，特别是在需要并行处理多个异步操作时。回调函数虽然老派，但在某些特定场景下仍然有用。

选择合适的异步方式，遵循最佳实践，能够编写出高质量、易维护的异步代码。记住，好的异步代码不仅要能正确工作，还要易于理解、调试和维护。