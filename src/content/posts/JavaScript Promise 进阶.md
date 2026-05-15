---
title: JavaScript Promise 二
published: 2022-04-13
description: 'Promise.all、Promise.race 等方法的详细介绍和学习笔记'
image: ''
tags: ["JavaScript","异步"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

本文是 JavaScript Promise 进阶指南，在掌握 Promise 基础后，深入学习 Promise 的高级特性、并发控制、错误处理策略以及实际项目中的复杂异步场景处理。

## Promise 静态方法详解

### Promise.allSettled()

Promise.allSettled() 不会因为某个 Promise 失败而中断，它会等待所有 Promise 完成，返回每个 Promise 的状态和结果。

```javascript
// 基本用法
const promises = [
  Promise.resolve('成功1'),
  Promise.reject('失败1'),
  Promise.resolve('成功2'),
  Promise.reject('失败2')
];

Promise.allSettled(promises).then(results => {
  console.log(results);
  // [
  //   { status: 'fulfilled', value: '成功1' },
  //   { status: 'rejected', reason: '失败1' },
  //   { status: 'fulfilled', value: '成功2' },
  //   { status: 'rejected', reason: '失败2' }
  // ]
});

// 实际应用：批量上传文件
async function uploadFiles(files) {
  const uploadPromises = files.map(file => uploadFile(file));
  const results = await Promise.allSettled(uploadPromises);

  const successful = results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);

  const failed = results
    .filter(r => r.status === 'rejected')
    .map(r => r.reason);

  return {
    success: successful.length,
    failure: failed.length,
    successfulFiles: successful,
    failedFiles: failed
  };
}

// 使用示例
const files = ['file1.jpg', 'file2.jpg', 'file3.jpg'];
uploadFiles(files).then(result => {
  console.log(`成功: ${result.success}, 失败: ${result.failure}`);
});
```

### Promise.any()

Promise.any() 返回第一个成功的 Promise，只有当所有 Promise 都失败时才抛出 AggregateError。

```javascript
// 基本用法
const promises = [
  Promise.reject('错误1'),
  Promise.reject('错误2'),
  Promise.resolve('成功'),
  Promise.reject('错误3')
];

Promise.any(promises).then(result => {
  console.log(result); // '成功'
});

// 所有都失败的情况
Promise.all([
  Promise.reject('错误1'),
  Promise.reject('错误2')
]).catch(error => {
  console.log(error instanceof AggregateError); // true
  console.log(error.errors); // ['错误1', '错误2']
});

// 实际应用：多源数据获取
async function fetchFromMultipleSources(sources) {
  const promises = sources.map(source =>
    fetch(source).catch(() => null)
  );

  try {
    const response = await Promise.any(promises);
    if (response && response.ok) {
      return await response.json();
    }
    throw new Error('所有源都不可用');
  } catch (error) {
    throw new Error(`所有源都失败: ${error.message}`);
  }
}

// 使用示例
const dataSources = [
  'https://api1.example.com/data',
  'https://api2.example.com/data',
  'https://api3.example.com/data'
];

fetchFromMultipleSources(dataSources)
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

## Promise 并发控制

### 限制并发数量

当需要处理大量 Promise 时，需要限制并发数量以避免资源耗尽。

```javascript
class PromisePool {
  constructor(maxConcurrent = 5) {
    this.maxConcurrent = maxConcurrent;
    this.running = 0;
    this.queue = [];
  }

  add(promiseFactory) {
    return new Promise((resolve, reject) => {
      this.queue.push({
        promiseFactory,
        resolve,
        reject
      });
      this.runNext();
    });
  }

  runNext() {
    if (this.running >= this.maxConcurrent || this.queue.length === 0) {
      return;
    }

    const { promiseFactory, resolve, reject } = this.queue.shift();
    this.running++;

    promiseFactory()
      .then(resolve)
      .catch(reject)
      .finally(() => {
        this.running--;
        this.runNext();
      });
  }
}

// 使用示例
const pool = new PromisePool(3);

async function processItem(item) {
  console.log(`处理 ${item}`);
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log(`完成 ${item}`);
  return `处理结果: ${item}`;
}

const items = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

Promise.all(
  items.map(item => pool.add(() => processItem(item)))
).then(results => {
  console.log('所有任务完成:', results);
});

// 简化版本
async function limitConcurrency(tasks, limit = 3) {
  const results = [];
  const executing = [];

  for (const task of tasks) {
    const promise = task().then(result => {
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

// 使用
const tasks = [1, 2, 3, 4, 5].map(i => () =>
  fetch(`/api/data/${i}`).then(r => r.json())
);

limitConcurrency(tasks, 2).then(results => console.log(results));
```

### 分批处理

```javascript
// 分批处理数据
async function batchProcess(items, batchSize, processor) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await Promise.all(
      batch.map(item => processor(item))
    );
    results.push(...batchResults);

    // 批次间添加延迟，避免过载
    if (i + batchSize < items.length) {
      await new Promise(resolve => setTimeout(resolve, 100));
    }
  }

  return results;
}

// 使用示例
const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

batchProcess(data, 3, async item => {
  console.log(`处理 ${item}`);
  await new Promise(resolve => setTimeout(resolve, 500));
  return item * 2;
}).then(results => {
  console.log('所有结果:', results);
});
```

## Promise 高级模式

### Promise 队列

```javascript
class PromiseQueue {
  constructor() {
    this.queue = [];
    this.isProcessing = false;
  }

  add(promiseFactory) {
    return new Promise((resolve, reject) => {
      this.queue.push({ promiseFactory, resolve, reject });
      this.process();
    });
  }

  async process() {
    if (this.isProcessing || this.queue.length === 0) {
      return;
    }

    this.isProcessing = true;

    while (this.queue.length > 0) {
      const { promiseFactory, resolve, reject } = this.queue.shift();

      try {
        const result = await promiseFactory();
        resolve(result);
      } catch (error) {
        reject(error);
      }
    }

    this.isProcessing = false;
  }

  clear() {
    this.queue = [];
  }

  size() {
    return this.queue.length;
  }
}

// 使用示例
const queue = new PromiseQueue();

// 添加任务到队列
queue.add(() => fetch('/api/data1').then(r => r.json()))
  .then(data => console.log('数据1:', data));

queue.add(() => fetch('/api/data2').then(r => r.json()))
  .then(data => console.log('数据2:', data));

queue.add(() => fetch('/api/data3').then(r => r.json()))
  .then(data => console.log('数据3:', data));
```

### Promise 超时控制

```javascript
// 设置超时
function withTimeout(promise, timeout, errorMessage = '操作超时') {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error(errorMessage)), timeout);
  });

  return Promise.race([promise, timeoutPromise]);
}

// 使用示例
async function fetchWithTimeout(url, timeout = 5000) {
  const response = await withTimeout(
    fetch(url),
    timeout,
    `请求 ${url} 超时 (${timeout}ms)`
  );

  if (!response.ok) {
    throw new Error(`HTTP 错误: ${response.status}`);
  }

  return response.json();
}

// 进阶：可重试的超时
function withTimeoutAndRetry(
  promiseFactory,
  { timeout, retries = 3, retryDelay = 1000 } = {}
) {
  return new Promise((resolve, reject) => {
    let attempt = 0;

    function attemptOperation() {
      const timeoutPromise = new Promise((_, reject) => {
        setTimeout(() => reject(new Error('操作超时')), timeout);
      });

      Promise.race([promiseFactory(), timeoutPromise])
        .then(resolve)
        .catch(error => {
          attempt++;
          if (attempt < retries) {
            console.log(`重试 ${attempt}/${retries}...`);
            setTimeout(attemptOperation, retryDelay);
          } else {
            reject(error);
          }
        });
    }

    attemptOperation();
  });
}

// 使用
withTimeoutAndRetry(
  () => fetch('/api/slow-endpoint'),
  { timeout: 2000, retries: 3 }
)
  .then(response => response.json())
  .catch(error => console.error(error));
```

### Promise 重试机制

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
      const jitter = Math.random() * baseDelay; // 添加随机性
      const totalDelay = delay + jitter;

      console.log(`第 ${attempt + 1} 次重试，等待 ${Math.round(totalDelay)}ms`);
      await new Promise(resolve => setTimeout(resolve, totalDelay));
    }
  }
}

// 使用示例
async function fetchData() {
  const response = await fetch('/api/unstable');
  if (!response.ok) {
    throw new Error(`HTTP 错误: ${response.status}`);
  }
  return response.json();
}

retryWithBackoff(fetchData, { maxRetries: 5, baseDelay: 1000 })
  .then(data => console.log(data))
  .catch(error => console.error('全部重试失败:', error));

// 条件重试：只在特定错误时重试
async function conditionalRetry(
  operation,
  shouldRetry,
  { maxRetries = 3, delay = 1000 } = {}
) {
  let lastError;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await operation();
    } catch (error) {
      lastError = error;

      if (!shouldRetry(error) || attempt === maxRetries) {
        throw error;
      }

      console.log(`错误可重试，${delay}ms 后重试...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// 使用：只在网络错误时重试
conditionalRetry(
  () => fetch('/api/data'),
  error => error.name === 'TypeError' || error.code === 'ECONNRESET',
  { maxRetries: 3, delay: 2000 }
)
  .then(response => response.json())
  .catch(error => console.error(error));
```

### Promise 取消

```javascript
// 使用 AbortController 取消 fetch 请求
class CancellablePromise {
  constructor(promise, abortController) {
    this.promise = promise;
    this.abortController = abortController;
  }

  then(onFulfilled, onRejected) {
    return new CancellablePromise(
      this.promise.then(onFulfilled, onRejected),
      this.abortController
    );
  }

  catch(onRejected) {
    return this.then(null, onRejected);
  }

  finally(onFinally) {
    return new CancellablePromise(
      this.promise.finally(onFinally),
      this.abortController
    );
  }

  cancel(reason = 'Operation cancelled') {
    this.abortController.abort(reason);
    return this;
  }
}

function cancellableFetch(url, options = {}) {
  const abortController = new AbortController();
  const promise = fetch(url, {
    ...options,
    signal: abortController.signal
  });

  return new CancellablePromise(promise, abortController);
}

// 使用示例
const request = cancellableFetch('/api/data');

request
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => {
    if (error.name === 'AbortError') {
      console.log('请求被取消');
    } else {
      console.error('请求失败:', error);
    }
  });

// 取消请求
// setTimeout(() => request.cancel(), 1000);

// 通用取消模式
function makeCancellable(promise) {
  let cancelled = false;
  let resolveCancel = () => {};

  const wrappedPromise = new Promise((resolve, reject) => {
    promise
      .then(result => {
        if (!cancelled) resolve(result);
      })
      .catch(error => {
        if (!cancelled) reject(error);
      });

    resolveCancel = () => {
      cancelled = true;
    };
  });

  wrappedPromise.cancel = resolveCancel;
  return wrappedPromise;
}

// 使用
const task = makeCancellable(
  new Promise(resolve => setTimeout(() => resolve('完成'), 2000))
);

task.then(result => console.log(result));

// 取消任务
// task.cancel();
```

## Promise 缓存和去重

### 结果缓存

```javascript
class PromiseCache {
  constructor() {
    this.cache = new Map();
  }

  get(key) {
    return this.cache.get(key);
  }

  set(key, promise) {
    this.cache.set(key, promise);

    // 缓存失败时自动清理
    promise.catch(() => {
      this.cache.delete(key);
    });

    return promise;
  }

  delete(key) {
    this.cache.delete(key);
  }

  clear() {
    this.cache.clear();
  }

  size() {
    return this.cache.size;
  }
}

// 带缓存的函数装饰器
function memoize(fn, keyGenerator = (...args) => JSON.stringify(args)) {
  const cache = new Map();

  return async function(...args) {
    const key = keyGenerator(...args);

    if (cache.has(key)) {
      console.log('从缓存返回:', key);
      return cache.get(key);
    }

    const promise = fn.apply(this, args)
      .finally(() => {
        // 5分钟后清除缓存
        setTimeout(() => {
          cache.delete(key);
        }, 5 * 60 * 1000);
      });

    cache.set(key, promise);
    return promise;
  };
}

// 使用示例
const fetchUser = memoize(async (userId) => {
  console.log('获取用户数据:', userId);
  const response = await fetch(`/api/users/${userId}`);
  return response.json();
});

// 第一次调用会发起请求
fetchUser(1).then(user => console.log(user));

// 第二次调用会从缓存返回
fetchUser(1).then(user => console.log(user));
```

### 请求去重

```javascript
class PromiseDeduplicator {
  constructor() {
    this.pending = new Map();
  }

  deduplicate(key, promiseFactory) {
    // 如果有相同的请求正在进行，返回同一个 Promise
    if (this.pending.has(key)) {
      console.log('使用正在进行的请求:', key);
      return this.pending.get(key);
    }

    const promise = promiseFactory()
      .then(result => {
        this.pending.delete(key);
        return result;
      })
      .catch(error => {
        this.pending.delete(key);
        throw error;
      });

    this.pending.set(key, promise);
    return promise;
  }

  size() {
    return this.pending.size;
  }
}

const deduplicator = new PromiseDeduplicator();

async function fetchWithDeduplication(url) {
  return deduplicator.deduplicate(url, () =>
    fetch(url).then(response => {
      if (!response.ok) {
        throw new Error(`HTTP 错误: ${response.status}`);
      }
      return response.json();
    })
  );
}

// 使用示例：同时发起多个相同请求
const url = '/api/user/123';

Promise.all([
  fetchWithDeduplication(url),
  fetchWithDeduplication(url),
  fetchWithDeduplication(url)
]).then(results => {
  console.log('所有请求完成，实际只发起了一次请求');
});
```

## 错误处理最佳实践

### 全局错误处理

```javascript
// 处理未捕获的 Promise 拒绝
window.addEventListener('unhandledrejection', (event) => {
  console.error('未处理的 Promise 拒绝:', event.reason);

  // 可以在这里发送错误日志到服务器
  // logErrorToServer(event.reason);

  // 阻止默认的控制台错误输出
  // event.preventDefault();
});

// 处理已处理的 Promise 拒绝
window.addEventListener('rejectionhandled', (event) => {
  console.log('Promise 拒绝已被处理:', event.reason);
});

// 创建带有默认错误处理的 Promise
class SafePromise {
  static create(executor) {
    return new Promise((resolve, reject) => {
      executor(
        value => {
          resolve(value);
        },
        error => {
          console.error('Promise 错误:', error);
          reject(error);
        }
      );
    });
  }
}

// 使用
SafePromise.create((resolve, reject) => {
  throw new Error('测试错误');
}).catch(error => {
  console.log('捕获到错误:', error.message);
});
```

### 错误分类处理

```javascript
// 错误类型定义
class NetworkError extends Error {
  constructor(message, originalError) {
    super(message);
    this.name = 'NetworkError';
    this.originalError = originalError;
  }
}

class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class TimeoutError extends Error {
  constructor(message, timeout) {
    super(message);
    this.name = 'TimeoutError';
    this.timeout = timeout;
  }
}

// 错误处理器
class ErrorHandler {
  static handle(error) {
    if (error instanceof NetworkError) {
      console.error('网络错误:', error.message);
      // 显示网络错误提示
      return '网络连接失败，请检查网络';
    } else if (error instanceof ValidationError) {
      console.error('验证错误:', error.message);
      // 显示字段验证错误
      return `字段 ${error.field} 验证失败`;
    } else if (error instanceof TimeoutError) {
      console.error('超时错误:', error.message);
      // 显示超时提示
      return '请求超时，请重试';
    } else {
      console.error('未知错误:', error);
      // 显示通用错误提示
      return '发生未知错误，请稍后重试';
    }
  }
}

// 使用示例
async function fetchWithBetterErrorHandling(url) {
  try {
    const response = await fetch(url);
    if (!response.ok) {
      throw new NetworkError(
        `HTTP 错误: ${response.status}`,
        new Error(response.statusText)
      );
    }
    return await response.json();
  } catch (error) {
    if (error.name === 'TypeError') {
      throw new NetworkError('网络连接失败', error);
    }
    throw error;
  }
}

fetchWithBetterErrorHandling('/api/data')
  .then(data => console.log(data))
  .catch(error => {
    const errorMessage = ErrorHandler.handle(error);
    alert(errorMessage);
  });
```

### 错误恢复策略

```javascript
// 回退策略
async function fetchWithFallback(primaryUrl, fallbackUrls) {
  const urls = [primaryUrl, ...fallbackUrls];

  for (const [index, url] of urls.entries()) {
    try {
      const response = await fetch(url);
      if (response.ok) {
        return await response.json();
      }
    } catch (error) {
      console.log(`尝试 ${index + 1}/${urls.length} 失败:`, error.message);

      if (index < urls.length - 1) {
        console.log(`尝试备用源: ${urls[index + 1]}`);
      }
    }
  }

  throw new Error('所有数据源都失败');
}

// 使用示例
fetchWithFallback(
  'https://api1.example.com/data',
  ['https://api2.example.com/data', 'https://api3.example.com/data']
)
  .then(data => console.log(data))
  .catch(error => console.error(error));

// 降级策略
async function fetchWithDegradation(url) {
  try {
    // 尝试获取完整数据
    const response = await fetch(url);
    if (response.ok) {
      return {
        status: 'full',
        data: await response.json()
      };
    }
  } catch (error) {
    console.log('完整数据获取失败，尝试降级');
  }

  try {
    // 尝试获取缓存数据
    const cachedData = await getCachedData(url);
    return {
      status: 'cached',
      data: cachedData
    };
  } catch (error) {
    console.log('缓存数据获取失败，返回默认数据');
  }

  // 返回默认数据
  return {
    status: 'default',
    data: getDefaultData()
  };
}
```

## 性能优化

### Promise 批处理

```javascript
// 批量处理函数调用
class BatchProcessor {
  constructor(batchSize = 10, batchTimeout = 100) {
    this.batchSize = batchSize;
    this.batchTimeout = batchTimeout;
    this.batch = [];
    this.timer = null;
  }

  add(item) {
    return new Promise((resolve, reject) => {
      this.batch.push({ item, resolve, reject });

      if (this.batch.length >= this.batchSize) {
        this.process();
      } else if (!this.timer) {
        this.timer = setTimeout(() => this.process(), this.batchTimeout);
      }
    });
  }

  async process() {
    if (this.timer) {
      clearTimeout(this.timer);
      this.timer = null;
    }

    const currentBatch = this.batch.splice(0, this.batch.length);

    try {
      const results = await this.batchProcess(currentBatch.map(b => b.item));
      currentBatch.forEach((item, index) => {
        item.resolve(results[index]);
      });
    } catch (error) {
      currentBatch.forEach(item => item.reject(error));
    }
  }

  async batchProcess(items) {
    // 子类实现具体的批处理逻辑
    throw new Error('子类必须实现 batchProcess 方法');
  }
}

// 批量 API 请求
class BatchAPI extends BatchProcessor {
  constructor(apiEndpoint, batchSize = 10, batchTimeout = 100) {
    super(batchSize, batchTimeout);
    this.apiEndpoint = apiEndpoint;
  }

  async batchProcess(items) {
    const response = await fetch(this.apiEndpoint, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({ ids: items })
    });

    if (!response.ok) {
      throw new Error(`批处理请求失败: ${response.status}`);
    }

    return await response.json();
  }
}

// 使用示例
const batchAPI = new BatchAPI('/api/batch-users');

async function getUser(userId) {
  return batchAPI.add(userId);
}

// 调用
Promise.all([
  getUser(1),
  getUser(2),
  getUser(3),
  getUser(4),
  getUser(5)
]).then(users => {
  console.log('用户数据:', users);
});
```

### Promise 优先级队列

```javascript
class PriorityQueue {
  constructor() {
    this.highPriority = [];
    this.lowPriority = [];
    this.isProcessing = false;
  }

  add(priority, promiseFactory) {
    return new Promise((resolve, reject) => {
      const task = { promiseFactory, resolve, reject };

      if (priority === 'high') {
        this.highPriority.push(task);
      } else {
        this.lowPriority.push(task);
      }

      this.process();
    });
  }

  async process() {
    if (this.isProcessing) {
      return;
    }

    this.isProcessing = true;

    while (this.highPriority.length > 0 || this.lowPriority.length > 0) {
      const task = this.highPriority.shift() || this.lowPriority.shift();

      try {
        const result = await task.promiseFactory();
        task.resolve(result);
      } catch (error) {
        task.reject(error);
      }
    }

    this.isProcessing = false;
  }
}

// 使用示例
const queue = new PriorityQueue();

// 高优先级任务
queue.add('high', () => fetch('/api/critical-data').then(r => r.json()))
  .then(data => console.log('关键数据:', data));

// 低优先级任务
queue.add('low', () => fetch('/api/analytics').then(r => r.json()))
  .then(data => console.log('分析数据:', data));

// 高优先级任务
queue.add('high', () => fetch('/api/user-data').then(r => r.json()))
  .then(data => console.log('用户数据:', data));
```

## 注意事项

1. **内存泄漏风险**：长时间运行的 Promise 要注意清理，避免持有不必要的引用。

2. **并发控制**：大量并发 Promise 可能导致资源耗尽，需要合理控制并发数量。

3. **错误处理**：始终处理 Promise 拒绝，避免未捕获的异常导致应用崩溃。

4. **超时设置**：为长时间运行的异步操作设置超时，避免无限等待。

5. **缓存策略**：合理使用缓存，但要注意缓存失效和内存占用。

6. **取消操作**：对于可取消的异步操作，提供取消机制以释放资源。

7. **性能监控**：监控 Promise 执行时间和成功率，及时发现性能问题。

8. **错误恢复**：设计合理的错误恢复策略，提高系统健壮性。

## 总结

Promise 进阶技巧让复杂的异步编程变得更加优雅和可控。通过掌握并发控制、错误处理、缓存策略等高级模式，可以构建出高性能、高可靠性的异步应用。在实际项目中，要根据具体需求选择合适的策略，平衡性能和可维护性。记住，好的异步代码不仅要能工作，还要易于理解、调试和维护。