---
title: JavaScript async/await
published: 2022-04-19
description: '异步编程的最佳实践的详细介绍和学习笔记'
image: ''
tags: ["JS基础","异步"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 什么是 async/await

async/await 是 ES2017 引入的语法糖，让异步代码看起来像同步代码，使代码更加清晰易读。

## async 函数

```javascript
// async 函数总是返回 Promise
async function hello() {
  return 'Hello';
}

hello().then(result => {
  console.log(result); // 'Hello'
});

// async 函数中的错误
async function error() {
  throw new Error('出错了');
}

error().catch(err => {
  console.error(err);
});
```

## await 表达式

```javascript
// await 只能在 async 函数内使用
async function getData() {
  const response = await fetch('/api/data');
  const data = await response.json();
  return data;
}

// 错误处理
async function fetchUser() {
  try {
    const response = await fetch('/api/user');
    const user = await response.json();
    return user;
  } catch (error) {
    console.error('获取用户失败:', error);
    return null;
  }
}
```

## 并行执行

```javascript
// 串行执行（慢）
async function serial() {
  const a = await fetch('/api/a');
  const b = await fetch('/api/b');
  return [a, b];
}

// 并行执行（快）
async function parallel() {
  const [a, b] = await Promise.all([
    fetch('/api/a'),
    fetch('/api/b')
  ]);
  return [a, b];
}
```

## async/await 的工作原理

### async 函数的本质

```javascript
// async 函数总是返回 Promise
async function hello() {
  return 'Hello';
}

// 等价于
function hello() {
  return Promise.resolve('Hello');
}

console.log(hello() instanceof Promise); // true
```

### await 的行为

```javascript
// await 暂停函数执行，等待 Promise 解析
async function example() {
  console.log('1');

  await new Promise(resolve => setTimeout(resolve, 1000));

  console.log('2');
}

example();
console.log('3');

// 输出顺序: 1, 3, 2

// await 后面的值如果不是 Promise，会自动包装
async function example2() {
  const result = await 'Hello';
  console.log(result); // 'Hello'

  const num = await 42;
  console.log(num); // 42

  const obj = await { a: 1 };
  console.log(obj); // { a: 1 }
}

example2();

// thenable 对象
const thenable = {
  then: function(resolve) {
    resolve('thenable value');
  }
};

async function example3() {
  const result = await thenable;
  console.log(result); // 'thenable value'
}

example3();
```

## 错误处理

### try-catch 基础用法

```javascript
// 基本的错误处理
async function fetchUser(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);

    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    console.error('获取用户失败:', error);
    throw error; // 重新抛出错误
  }
}

// 使用
fetchUser(1)
  .then(user => console.log(user))
  .catch(error => console.error('最终错误:', error));
```

### 多层错误处理

```javascript
// 外层错误处理
async function processUserData(userId) {
  let user;
  try {
    user = await fetchUser(userId);
    const posts = await fetchUserPosts(userId);
    return { user, posts };
  } catch (error) {
    // 内层错误会冒泡到这里
    console.error('处理用户数据失败:', error);

    // 提供降级数据
    return {
      user: null,
      posts: [],
      error: error.message
    };
  }
}

// 分层错误处理
async function getFullUserData(userId) {
  try {
    const user = await withRetry(() => fetchUser(userId), 3);
    const [posts, comments] = await Promise.all([
      fetchUserPosts(userId).catch(() => []),
      fetchUserComments(userId).catch(() => [])
    ]);

    return { user, posts, comments };
  } catch (error) {
    console.error('严重错误:', error);
    throw new Error('无法获取用户数据');
  }
}
```

### 错误分类处理

```javascript
// 自定义错误类
class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'NetworkError';
    this.statusCode = statusCode;
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

// 错误分类处理
async function handleRequest(url, data) {
  try {
    const response = await fetch(url, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(data)
    });

    if (response.status === 400) {
      throw new ValidationError('数据验证失败', 'unknown');
    }

    if (!response.ok) {
      throw new NetworkError('网络请求失败', response.status);
    }

    return await response.json();
  } catch (error) {
    if (error instanceof NetworkError) {
      console.error('网络错误:', error.statusCode);
      // 重试或显示网络错误
    } else if (error instanceof ValidationError) {
      console.error('验证错误:', error.field);
      // 显示字段验证错误
    } else {
      console.error('未知错误:', error);
    }
    throw error;
  }
}
```

### 错误边界模式

```javascript
// 创建错误边界函数
async function errorBoundary(fn, errorHandler) {
  try {
    return await fn();
  } catch (error) {
    return errorHandler(error);
  }
}

// 使用
async function getUserData(userId) {
  return await errorBoundary(
    async () => {
      const user = await fetchUser(userId);
      const posts = await fetchUserPosts(userId);
      return { user, posts };
    },
    (error) => ({
      user: null,
      posts: [],
      error: error.message
    })
  );
}

// 多个错误处理
async function fetchWithFallbacks(urls) {
  for (const url of urls) {
    try {
      const response = await fetch(url);
      if (response.ok) {
        return await response.json();
      }
    } catch (error) {
      console.log(`${url} 失败，尝试下一个`);
    }
  }

  throw new Error('所有源都失败');
}
```

## 并行执行优化

### Promise.all 并行

```javascript
// 并行执行多个独立的异步操作
async function fetchDashboardData() {
  const [user, posts, comments, analytics] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments(),
    fetchAnalytics()
  ]);

  return {
    user,
    posts,
    comments,
    analytics
  };
}

// 带错误处理的并行执行
async function fetchWithPartialFailure() {
  const results = await Promise.allSettled([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);

  const data = {};
  const errors = [];

  results.forEach((result, index) => {
    if (result.status === 'fulfilled') {
      data[index] = result.value;
    } else {
      errors.push({ index, error: result.reason });
    }
  });

  return { data, errors };
}
```

### 分批并行执行

```javascript
// 分批执行以避免过载
async function batchFetch(items, batchSize = 5) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => fetchItem(item))
    );
    results.push(...batchResults);
  }

  return results;
}

// 并发控制
async function parallelWithConcurrency(items, limit = 3) {
  const results = [];
  const executing = [];

  for (const item of items) {
    const promise = fetchItem(item).then(result => {
      executing.splice(executing.indexOf(promise), 1);
      return result;
    });

    results.push(promise);
    executing.push(promise);

    if (executing.length >= limit) {
      await Promise.race(executing);
    }
  }

  return Promise.all(results);
}
```

### 竞态条件处理

```javascript
// 超时处理
async function fetchWithTimeout(url, timeout = 5000) {
  const controller = new AbortController();

  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      controller.abort();
      reject(new Error('请求超时'));
    }, timeout);
  });

  try {
    const response = await Promise.race([
      fetch(url, { signal: controller.signal }),
      timeoutPromise
    ]);

    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }

    return await response.json();
  } catch (error) {
    if (error.name === 'AbortError') {
      throw new Error('请求超时');
    }
    throw error;
  }
}

// 取消重复请求
class RequestCache {
  constructor() {
    this.cache = new Map();
  }

  async fetch(url) {
    if (this.cache.has(url)) {
      return this.cache.get(url);
    }

    const promise = fetch(url)
      .then(response => {
        if (!response.ok) {
          throw new Error(`HTTP 错误: ${response.status}`);
        }
        return response.json();
      })
      .finally(() => {
        this.cache.delete(url);
      });

    this.cache.set(url, promise);
    return promise;
  }
}

const requestCache = new RequestCache();

// 多次调用同一个 URL 只会发起一次请求
const data1 = await requestCache.fetch('/api/data');
const data2 = await requestCache.fetch('/api/data'); // 使用缓存
```

## 异步迭代

### for-await-of

```javascript
// 异步生成器
async function* generateSequence(start, end) {
  for (let i = start; i <= end; i++) {
    await new Promise(resolve => setTimeout(resolve, 100));
    yield i;
  }
}

// 使用 for-await-of 迭代
async function processSequence() {
  for await (const num of generateSequence(1, 5)) {
    console.log(num);
  }
}

// 迭代异步可迭代对象
async function processAsyncIterable() {
  const urls = [
    '/api/data1',
    '/api/data2',
    '/api/data3'
  ];

  const asyncIterable = {
    async *[Symbol.asyncIterator]() {
      for (const url of urls) {
        const response = await fetch(url);
        yield await response.json();
      }
    }
  };

  for await (const data of asyncIterable) {
    console.log(data);
  }
}
```

### 批量异步处理

```javascript
// 流式处理大数据
async function* streamLines(file) {
  const stream = file.stream();
  const reader = stream.getReader();
  const decoder = new TextDecoder();

  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();

    if (done) break;

    buffer += decoder.decode(value, { stream: true });

    const lines = buffer.split('\n');
    buffer = lines.pop();

    for (const line of lines) {
      if (line) yield line;
    }
  }

  if (buffer) yield buffer;
}

// 使用
async function processFile(file) {
  let count = 0;

  for await (const line of streamLines(file)) {
    console.log(line);
    count++;
  }

  console.log(`处理了 ${count} 行`);
}
```

## 实际应用场景

### 1. API 请求链

```javascript
// 串行 API 请求
async function getUserWithDetails(userId) {
  try {
    // 第一步：获取用户基本信息
    const user = await fetchUser(userId);

    // 第二步：基于用户信息获取更多数据
    const [posts, friends, profile] = await Promise.all([
      fetchUserPosts(userId),
      fetchUserFriends(userId),
      fetchUserProfile(userId)
    ]);

    return {
      user,
      posts,
      friends,
      profile,
      loaded: true
    };
  } catch (error) {
    console.error('加载用户详情失败:', error);
    return {
      user: null,
      loaded: false,
      error: error.message
    };
  }
}

// 使用
const userDetails = await getUserWithDetails(123);
```

### 2. 数据处理管道

```javascript
// 数据处理管道
class DataPipeline {
  constructor(steps = []) {
    this.steps = steps;
  }

  addStep(step) {
    this.steps.push(step);
    return this;
  }

  async process(data) {
    let result = data;

    for (const step of this.steps) {
      result = await step(result);
    }

    return result;
  }
}

// 使用
const pipeline = new DataPipeline()
  .addStep(async (data) => {
    console.log('验证数据...');
    await validateData(data);
    return data;
  })
  .addStep(async (data) => {
    console.log('清洗数据...');
    return cleanData(data);
  })
  .addStep(async (data) => {
    console.log('转换数据...');
    return transformData(data);
  })
  .addStep(async (data) => {
    console.log('保存数据...');
    await saveData(data);
    return data;
  });

const result = await pipeline.process(rawData);
```

### 3. 重试机制

```javascript
// 指数退避重试
async function retryWithBackoff(
  operation,
  { maxRetries = 3, baseDelay = 1000 } = {}
) {
  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      if (attempt === maxRetries) {
        throw error;
      }

      const delay = baseDelay * Math.pow(2, attempt);
      console.log(`第 ${attempt + 1} 次重试，等待 ${delay}ms`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}

// 使用重试装饰器
function withRetry(options) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function(...args) {
      return await retryWithBackoff(
        () => originalMethod.apply(this, args),
        options
      );
    };

    return descriptor;
  };
}

class ApiService {
  @withRetry({ maxRetries: 3, baseDelay: 1000 })
  async fetchData(url) {
    const response = await fetch(url);
    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }
    return await response.json();
  }
}
```

### 4. 缓存策略

```javascript
// 带缓存的异步函数
function memoizeAsync(fn, ttl = 60000) {
  const cache = new Map();

  return async function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      const { data, timestamp } = cache.get(key);

      if (Date.now() - timestamp < ttl) {
        console.log('从缓存返回:', key);
        return data;
      }
    }

    console.log('执行函数:', key);
    const data = await fn.apply(this, args);

    cache.set(key, { data, timestamp: Date.now() });

    // 自动清理过期缓存
    setTimeout(() => {
      cache.delete(key);
    }, ttl);

    return data;
  };
}

// 使用
const fetchUser = memoizeAsync(async (userId) => {
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
}, 60000); // 缓存 1 分钟

// 第一次调用发起请求
const user1 = await fetchUser(1);

// 第二次调用使用缓存
const user2 = await fetchUser(1);
```

### 5. 限流控制

```javascript
// 令牌桶限流
class RateLimiter {
  constructor(maxRequests, timeWindow) {
    this.maxRequests = maxRequests;
    this.timeWindow = timeWindow;
    this.tokens = maxRequests;
    this.lastRefill = Date.now();
  }

  async acquire() {
    const now = Date.now();
    const elapsed = now - this.lastRefill;

    // 补充令牌
    this.tokens = Math.min(
      this.maxRequests,
      this.tokens + (elapsed / this.timeWindow) * this.maxRequests
    );
    this.lastRefill = now;

    if (this.tokens >= 1) {
      this.tokens--;
      return true;
    }

    // 等待令牌
    const waitTime = ((1 - this.tokens) / this.maxRequests) * this.timeWindow;
    await new Promise(resolve => setTimeout(resolve, waitTime));

    return this.acquire();
  }
}

// 使用
const limiter = new RateLimiter(10, 1000); // 10 请求/秒

async function rateLimitedFetch(url) {
  await limiter.acquire();
  const response = await fetch(url);
  return response.json();
}

// 批量请求
const urls = Array.from({ length: 20 }, (_, i) => `/api/data/${i}`);

for (const url of urls) {
  rateLimitedFetch(url).then(data => console.log(data));
}
```

## 常见陷阱和最佳实践

### 陷阱 1: 遗漏 await

```javascript
// 错误：遗漏 await
async function badExample() {
  const result = fetch('/api/data');
  console.log(result); // Promise，不是数据
}

// 正确：使用 await
async function goodExample() {
  const result = await fetch('/api/data');
  console.log(result); // Response 对象
}
```

### 陷阱 2: 错误的并行执行

```javascript
// 错误：串行执行
async function badParallel() {
  const user = await fetchUser();
  const posts = await fetchPosts();
  const comments = await fetchComments();
  return { user, posts, comments };
}

// 正确：并行执行
async function goodParallel() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);

  return { user, posts, comments };
}
```

### 陷阱 3: 忽略错误处理

```javascript
// 错误：忽略错误
async function badErrorHandling() {
  const data = await fetchData();
  process(data); // 如果 fetchData 失败，会抛出未捕获的异常
}

// 正确：处理错误
async function goodErrorHandling() {
  try {
    const data = await fetchData();
    return process(data);
  } catch (error) {
    console.error('处理失败:', error);
    return null; // 或提供默认值
  }
}
```

### 陷阱 4: 在循环中滥用 await

```javascript
// 错误：串行处理
async function badLoop() {
  const items = [1, 2, 3, 4, 5];

  for (const item of items) {
    await processItem(item); // 串行执行
  }
}

// 正确：并行处理
async function goodLoop() {
  const items = [1, 2, 3, 4, 5];

  return Promise.all(items.map(item => processItem(item)));
}

// 需要顺序时使用 for...of
async function sequentialLoop() {
  const items = [1, 2, 3, 4, 5];

  for (const item of items) {
    await processItem(item); // 确实需要顺序执行
  }
}
```

### 陷阱 5: 混用 Promise 和 async/await

```javascript
// 不推荐：混用导致混乱
function badMixing() {
  return fetch('/api/data')
    .then(response => {
      return response.json();
    })
    .then(async data => {
      const processed = await processData(data);
      return processed;
    });
}

// 推荐：统一使用 async/await
async function goodMixing() {
  const response = await fetch('/api/data');
  const data = await response.json();
  return await processData(data);
}
```

## 性能考虑

### 1. 避免不必要的等待

```javascript
// 低效：串行等待
async function inefficient() {
  const user = await fetchUser(1);
  const user2 = await fetchUser(2);
  const user3 = await fetchUser(3);
  return [user, user2, user3];
}

// 高效：并行执行
async function efficient() {
  const [user, user2, user3] = await Promise.all([
    fetchUser(1),
    fetchUser(2),
    fetchUser(3)
  ]);

  return [user, user2, user3];
}
```

### 2. 合理使用 Promise.allSettled

```javascript
// 使用 Promise.allSettled 避免全部失败
async function batchOperation(items) {
  const results = await Promise.allSettled(
    items.map(item => processItem(item))
  );

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => r.reason);

  return { successful, failed };
}
```

### 3. 使用缓存减少重复请求

```javascript
// 使用 Map 缓存结果
const responseCache = new Map();

async function cachedFetch(url) {
  if (responseCache.has(url)) {
    return responseCache.get(url);
  }

  const response = await fetch(url);
  const data = await response.json();

  responseCache.set(url, data);

  // 5分钟后清除缓存
  setTimeout(() => {
    responseCache.delete(url);
  }, 5 * 60 * 1000);

  return data;
}
```

## 注意事项

1. **async 函数总是返回 Promise**：即使没有显式 return，也会返回 Promise.resolve(undefined)。

2. **await 只能在 async 函数内使用**：在顶层或普通函数中使用 await 会报错。

3. **错误处理要完整**：始终使用 try-catch 或 .catch() 处理可能的错误。

4. **合理使用并行执行**：独立的异步操作应该并行执行，提高性能。

5. **避免过深的嵌套**：虽然 async/await 解决了回调地狱，但仍要保持代码扁平。

6. **注意内存泄漏**：长时间运行的异步操作要注意清理，避免持有不必要的引用。

7. **超时控制很重要**：为网络请求等异步操作设置超时，避免无限等待。

8. **考虑使用 AbortController**：支持取消的异步操作，提供更好的用户体验。

## 总结

async/await 是处理异步操作的最佳方式，代码更简洁、更易读。通过掌握 async/await 的工作原理、错误处理、并行执行、异步迭代等高级特性，可以编写出高质量、高性能的异步代码。在实际开发中，要合理运用这些技巧，根据具体场景选择合适的方法，注意性能优化和错误处理。记住，好的异步代码不仅要能工作，还要易于理解、调试和维护。