---
title: Node.js 异步编程
published: 2023-02-21
description: '回调、Promise、async/await的详细介绍和学习笔记'
image: ''
tags: []
category: 'Node.js'
draft: false
lang: 'zh-CN'
---

## 概述

Node.js 的异步编程是其高性能的核心特性。由于 Node.js 是单线程的，异步编程使得它可以在不阻塞主线程的情况下处理大量并发操作。本文将详细介绍回调、Promise 和 async/await 三种异步编程模式。

## 核心概念

### 事件循环

Node.js 的事件循环是实现异步编程的机制，它允许 Node.js 执行非阻塞 I/O 操作。

### 回调函数

回调函数是作为参数传递给另一个函数的函数，在异步操作完成后被调用。

### Promise

Promise 是异步编程的现代化解决方案，提供了更优雅的方式来处理异步操作。

### async/await

async/await 是基于 Promise 的语法糖，让异步代码看起来像同步代码。

## 基本用法

### 回调函数

```javascript
// 基本回调函数
function fetchData(callback) {
  setTimeout(() => {
    const data = { name: '张三', age: 25 };
    callback(data);
  }, 1000);
}

fetchData(function(result) {
  console.log('收到数据:', result);
});

// 错误优先的回调模式
function readFile(filename, callback) {
  setTimeout(() => {
    if (filename === 'error.txt') {
      callback(new Error('文件不存在'));
    } else {
      callback(null, '文件内容');
    }
  }, 1000);
}

readFile('data.txt', (err, data) => {
  if (err) {
    console.error('读取失败:', err.message);
    return;
  }
  console.log('读取成功:', data);
});
```

### 回调地狱

```javascript
// 回调地狱的问题
fs.readFile('file1.txt', (err, data1) => {
  if (err) {
    console.error(err);
    return;
  }

  fs.readFile('file2.txt', (err, data2) => {
    if (err) {
      console.error(err);
      return;
    }

    fs.readFile('file3.txt', (err, data3) => {
      if (err) {
        console.error(err);
        return;
      }

      console.log('所有文件读取完成');
    });
  });
});
```

### Promise 基础

```javascript
// 创建 Promise
function fetchData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const success = true;
      if (success) {
        resolve({ name: '张三', age: 25 });
      } else {
        reject(new Error('获取数据失败'));
      }
    }, 1000);
  });
}

// 使用 Promise
fetchData()
  .then(data => {
    console.log('收到数据:', data);
  })
  .catch(error => {
    console.error('发生错误:', error.message);
  })
  .finally(() => {
    console.log('操作完成');
  });
```

### Promise 链式调用

```javascript
function readFile(filename) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(`文件 ${filename} 的内容`);
    }, 1000);
  });
}

// 链式调用
readFile('file1.txt')
  .then(content1 => {
    console.log('第一个文件:', content1);
    return readFile('file2.txt');
  })
  .then(content2 => {
    console.log('第二个文件:', content2);
    return readFile('file3.txt');
  })
  .then(content3 => {
    console.log('第三个文件:', content3);
  })
  .catch(error => {
    console.error('发生错误:', error);
  });
```

### Promise 并行操作

```javascript
// Promise.all - 所有 Promise 都成功
Promise.all([
  readFile('file1.txt'),
  readFile('file2.txt'),
  readFile('file3.txt')
])
  .then(results => {
    console.log('所有文件:', results);
  })
  .catch(error => {
    console.error('有文件读取失败:', error);
  });

// Promise.race - 最先完成的 Promise
Promise.race([
  readFile('file1.txt'),
  readFile('file2.txt'),
  readFile('file3.txt')
])
  .then(result => {
    console.log('最快的文件:', result);
  })
  .catch(error => {
    console.error('发生错误:', error);
  });

// Promise.allSettled - 所有 Promise 完成（无论成功失败）
Promise.allSettled([
  readFile('file1.txt'),
  Promise.reject(new Error('失败')),
  readFile('file3.txt')
])
  .then(results => {
    results.forEach((result, index) => {
      console.log(`文件 ${index + 1}:`, result.status);
      if (result.status === 'fulfilled') {
        console.log('  内容:', result.value);
      } else {
        console.log('  错误:', result.reason.message);
      }
    });
  });

// Promise.any - 任意一个成功的 Promise
Promise.any([
  Promise.reject(new Error('失败1')),
  Promise.resolve('成功'),
  Promise.reject(new Error('失败2'))
])
  .then(result => {
    console.log('第一个成功的结果:', result);
  })
  .catch(error => {
    console.error('所有 Promise 都失败:', error);
  });
```

### async/await 基础

```javascript
// 基本用法
async function getData() {
  try {
    const data = await fetchData();
    console.log('收到数据:', data);
  } catch (error) {
    console.error('发生错误:', error.message);
  }
}

getData();

// 箭头函数形式
const getData2 = async () => {
  const data = await fetchData();
  return data;
};
```

### async/await 顺序执行

```javascript
async function readFiles() {
  try {
    const content1 = await readFile('file1.txt');
    console.log('第一个文件:', content1);

    const content2 = await readFile('file2.txt');
    console.log('第二个文件:', content2);

    const content3 = await readFile('file3.txt');
    console.log('第三个文件:', content3);

    return '所有文件读取完成';
  } catch (error) {
    console.error('读取失败:', error);
    throw error;
  }
}

readFiles()
  .then(message => console.log(message))
  .catch(error => console.error('最终错误:', error));
```

### async/await 并行执行

```javascript
async function readFilesParallel() {
  try {
    const [content1, content2, content3] = await Promise.all([
      readFile('file1.txt'),
      readFile('file2.txt'),
      readFile('file3.txt')
    ]);

    return {
      file1: content1,
      file2: content2,
      file3: content3
    };
  } catch (error) {
    console.error('读取失败:', error);
    throw error;
  }
}

readFilesParallel()
  .then(contents => console.log('所有文件:', contents))
  .catch(error => console.error('最终错误:', error));
```

## 实际应用

### HTTP 请求

```javascript
const https = require('https');

// 回调方式
function fetchUrl(url, callback) {
  https.get(url, (res) => {
    let data = '';

    res.on('data', (chunk) => {
      data += chunk;
    });

    res.on('end', () => {
      callback(null, data);
    });
  }).on('error', (error) => {
    callback(error);
  });
}

// Promise 方式
function fetchUrlPromise(url) {
  return new Promise((resolve, reject) => {
    https.get(url, (res) => {
      let data = '';

      res.on('data', (chunk) => {
        data += chunk;
      });

      res.on('end', () => {
        resolve(data);
      });
    }).on('error', (error) => {
      reject(error);
    });
  });
}

// async/await 方式
async function fetchUrlAsync(url) {
  try {
    const data = await fetchUrlPromise(url);
    return JSON.parse(data);
  } catch (error) {
    console.error('获取失败:', error);
    throw error;
  }
}

// 使用示例
(async () => {
  try {
    const data = await fetchUrlAsync('https://api.example.com/data');
    console.log('数据:', data);
  } catch (error) {
    console.error('请求失败:', error);
  }
})();
```

### 数据库操作

```javascript
// 模拟数据库
const db = {
  users: [
    { id: 1, name: '张三', email: 'zhangsan@example.com' },
    { id: 2, name: '李四', email: 'lisi@example.com' }
  ]
};

// 查询用户
async function getUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const user = db.users.find(u => u.id === id);
      if (user) {
        resolve(user);
      } else {
        reject(new Error('用户不存在'));
      }
    }, 100);
  });
}

// 创建用户
async function createUser(userData) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const newUser = {
        id: db.users.length + 1,
        ...userData
      };
      db.users.push(newUser);
      resolve(newUser);
    }, 100);
  });
}

// 更新用户
async function updateUser(id, userData) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const index = db.users.findIndex(u => u.id === id);
      if (index !== -1) {
        db.users[index] = { ...db.users[index], ...userData };
        resolve(db.users[index]);
      } else {
        reject(new Error('用户不存在'));
      }
    }, 100);
  });
}

// 删除用户
async function deleteUser(id) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const index = db.users.findIndex(u => u.id === id);
      if (index !== -1) {
        db.users.splice(index, 1);
        resolve({ id });
      } else {
        reject(new Error('用户不存在'));
      }
    }, 100);
  });
}

// 使用示例
(async () => {
  try {
    // 查询用户
    const user = await getUser(1);
    console.log('查询结果:', user);

    // 创建用户
    const newUser = await createUser({
      name: '王五',
      email: 'wangwu@example.com'
    });
    console.log('创建成功:', newUser);

    // 更新用户
    const updated = await updateUser(1, { name: '张三丰' });
    console.log('更新成功:', updated);

    // 删除用户
    await deleteUser(2);
    console.log('删除成功');
  } catch (error) {
    console.error('操作失败:', error.message);
  }
})();
```

### 文件操作

```javascript
const fs = require('fs').promises;
const path = require('path');

// 读取文件
async function readConfigFile() {
  try {
    const filePath = path.join(__dirname, 'config.json');
    const content = await fs.readFile(filePath, 'utf8');
    return JSON.parse(content);
  } catch (error) {
    if (error.code === 'ENOENT') {
      console.log('配置文件不存在，使用默认配置');
      return { default: true };
    }
    throw error;
  }
}

// 写入文件
async function writeConfigFile(config) {
  try {
    const filePath = path.join(__dirname, 'config.json');
    const content = JSON.stringify(config, null, 2);
    await fs.writeFile(filePath, content);
    console.log('配置文件已保存');
  } catch (error) {
    console.error('保存配置失败:', error);
    throw error;
  }
}

// 批量处理文件
async function processFiles(filePaths) {
  const results = [];

  for (const filePath of filePaths) {
    try {
      const content = await fs.readFile(filePath, 'utf8');
      const processed = content.toUpperCase();
      await fs.writeFile(filePath, processed);
      results.push({ file: filePath, status: 'success' });
    } catch (error) {
      results.push({ file: filePath, status: 'error', error: error.message });
    }
  }

  return results;
}

// 使用示例
(async () => {
  try {
    // 读取配置
    const config = await readConfigFile();
    console.log('配置:', config);

    // 修改配置
    config.version = '2.0.0';
    await writeConfigFile(config);

    // 批量处理
    const files = ['file1.txt', 'file2.txt', 'file3.txt'];
    const results = await processFiles(files);
    console.log('处理结果:', results);
  } catch (error) {
    console.error('操作失败:', error);
  }
})();
```

### 并发控制

```javascript
class ConcurrencyControl {
  constructor(maxConcurrent) {
    this.maxConcurrent = maxConcurrent;
    this.running = 0;
    this.queue = [];
  }

  async run(fn) {
    // 如果达到最大并发数，等待
    while (this.running >= this.maxConcurrent) {
      await new Promise(resolve => this.queue.push(resolve));
    }

    this.running++;

    try {
      const result = await fn();
      return result;
    } finally {
      this.running--;
      // 通知等待的任务
      if (this.queue.length > 0) {
        const resolve = this.queue.shift();
        resolve();
      }
    }
  }
}

// 使用示例
async function processUrl(url) {
  console.log('处理:', url);
  await new Promise(resolve => setTimeout(resolve, 1000));
  console.log('完成:', url);
  return `处理完成: ${url}`;
}

(async () => {
  const concurrencyControl = new ConcurrencyControl(3);

  const urls = [
    'https://example.com/1',
    'https://example.com/2',
    'https://example.com/3',
    'https://example.com/4',
    'https://example.com/5'
  ];

  const tasks = urls.map(url =>
    concurrencyControl.run(() => processUrl(url))
  );

  const results = await Promise.all(tasks);
  console.log('所有任务完成:', results);
})();
```

### 重试机制

```javascript
async function retry(fn, options = {}) {
  const {
    maxRetries = 3,
    delay = 1000,
    backoff = 2,
    onRetry = null
  } = options;

  let lastError;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      if (attempt < maxRetries) {
        const waitTime = delay * Math.pow(backoff, attempt - 1);
        console.log(`重试 ${attempt}/${maxRetries}，等待 ${waitTime}ms`);

        if (onRetry) {
          onRetry(attempt, error);
        }

        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }

  throw lastError;
}

// 使用示例
async function fetchWithRetry(url) {
  return retry(
    () => fetchUrlPromise(url),
    {
      maxRetries: 5,
      delay: 1000,
      backoff: 2,
      onRetry: (attempt, error) => {
        console.log(`第 ${attempt} 次重试，错误:`, error.message);
      }
    }
  );
}

(async () => {
  try {
    const data = await fetchWithRetry('https://api.example.com/data');
    console.log('数据:', data);
  } catch (error) {
    console.error('最终失败:', error);
  }
})();
```

## 注意事项

### 1. 错误处理

```javascript
// ❌ 错误：不处理错误
fetchData().then(data => console.log(data));

// ✅ 正确：处理错误
fetchData()
  .then(data => console.log(data))
  .catch(error => console.error(error));

// ❌ 错误：async/await 不处理错误
async function getData() {
  const data = await fetchData();
  return data;
}

// ✅ 正确：使用 try/catch
async function getData() {
  try {
    const data = await fetchData();
    return data;
  } catch (error) {
    console.error(error);
    throw error;
  }
}
```

### 2. 避免在循环中串行执行

```javascript
// ❌ 错误：串行执行，效率低
async function processItems(items) {
  const results = [];
  for (const item of items) {
    const result = await processItem(item);
    results.push(result);
  }
  return results;
}

// ✅ 正确：并行执行
async function processItems(items) {
  const results = await Promise.all(
    items.map(item => processItem(item))
  );
  return results;
}

// ✅ 正确：限制并发数的并行执行
async function processItems(items, concurrency = 10) {
  const chunks = [];
  for (let i = 0; i < items.length; i += concurrency) {
    chunks.push(items.slice(i, i + concurrency));
  }

  const results = [];
  for (const chunk of chunks) {
    const chunkResults = await Promise.all(
      chunk.map(item => processItem(item))
    );
    results.push(...chunkResults);
  }

  return results;
}
```

### 3. 混合使用回调和 Promise

```javascript
// ❌ 错误：不要混合使用
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  Promise.resolve(data).then(content => {
    console.log(content);
  });
});

// ✅ 正确：统一使用 Promise
const fs = require('fs').promises;
fs.readFile('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));

// ✅ 正确：统一使用回调
fs.readFile('file.txt', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

### 4. Promise 链的正确使用

```javascript
// ❌ 错误：嵌套 then
promise1
  .then(result1 => {
    return promise2
      .then(result2 => {
        return promise3.then(result3 => {
          console.log(result1, result2, result3);
        });
      });
  });

// ✅ 正确：扁平化
promise1
  .then(result1 => {
    return Promise.all([
      result1,
      promise2,
      promise3
    ]);
  })
  .then(([result1, result2, result3]) => {
    console.log(result1, result2, result3);
  });

// ✅ 正确：使用 async/await
async function process() {
  const result1 = await promise1;
  const result2 = await promise2;
  const result3 = await promise3;
  console.log(result1, result2, result3);
}
```

### 5. 避免忘记 await

```javascript
// ❌ 错误：忘记 await
async function getData() {
  const data = fetchData(); // 返回 Promise 对象，不是数据
  console.log(data); // Promise { <pending> }
}

// ✅ 正确：使用 await
async function getData() {
  const data = await fetchData();
  console.log(data);
}

// ✅ 正确：如果要并行执行，显式说明
async function getData() {
  const dataPromise = fetchData(); // 不等待
  const otherPromise = doSomethingElse(); // 不等待

  const data = await dataPromise;
  const other = await otherPromise;

  return { data, other };
}
```

## 总结

异步编程是 Node.js 的核心特性，掌握三种异步编程模式对于编写高效的 Node.js 应用至关重要：

- **回调函数**: 最基础的异步模式，但容易产生回调地狱
- **Promise**: 更优雅的异步解决方案，支持链式调用和并行操作
- **async/await**: 基于 Promise 的语法糖，让异步代码更易读易维护

选择合适的异步编程模式，遵循最佳实践，可以帮助我们编写出更清晰、更可靠的异步代码。