---
title: JavaScript Promise 一
published: 2022-04-06
description: 'Promise 的基本概念和用法的详细介绍和学习笔记'
image: ''
tags: ["JS基础","异步"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 什么是 Promise

Promise 是 JavaScript 中处理异步操作的一种方案，它代表一个异步操作的最终完成或失败。

## Promise 的三种状态

- **Pending（进行中）**：初始状态
- **Fulfilled（已成功）**：操作成功完成
- **Rejected（已失败）**：操作失败

```javascript
const promise = new Promise((resolve, reject) => {
  // 异步操作
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('操作成功');
    } else {
      reject('操作失败');
    }
  }, 1000);
});
```

## Promise 的基本用法

```javascript
// 创建 Promise
const p = new Promise((resolve, reject) => {
  resolve('成功');
});

// 使用 then 处理成功
p.then(result => {
  console.log(result); // '成功'
});

// 使用 catch 处理失败
p.catch(error => {
  console.error(error);
});

// 使用 finally 无论成功失败都执行
p.finally(() => {
  console.log('完成');
});
```

## Promise 链式调用

```javascript
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => console.log(posts))
  .catch(error => console.error(error));
```

## Promise.all 和 Promise.race

```javascript
// Promise.all: 所有 Promise 都成功才成功
Promise.all([
  fetch('/api/user'),
  fetch('/api/posts')
]).then(([user, posts]) => {
  console.log(user, posts);
});

// Promise.race: 返回最快的那个 Promise
Promise.race([
  fetch('/api/1'),
  fetch('/api/2')
]).then(result => {
  console.log(result);
});
```

## Promise 的工作原理

### 状态转换

Promise 的状态一旦改变，就不会再变：

```javascript
const promise = new Promise((resolve, reject) => {
  resolve('成功');
  reject('失败'); // 不会生效，状态已经改变
});

console.log(promise); // Promise {<fulfilled>: '成功'}

// 反向转换也不可能
const promise2 = new Promise((resolve, reject) => {
  reject('失败');
  resolve('成功'); // 不会生效
});

console.log(promise2); // Promise {<rejected>: '失败'}
```

### 执行时机

Promise 创建后立即执行，但回调函数会在微任务队列中等待：

```javascript
console.log('1');

const promise = new Promise((resolve) => {
  console.log('2');
  resolve('3');
});

console.log('4');

promise.then(result => {
  console.log(result);
});

console.log('5');

// 输出顺序: 1, 2, 4, 5, 3
```

## Promise 的详细用法

### then 方法

```javascript
const promise = new Promise((resolve) => {
  resolve('成功');
});

// 基本用法
promise.then(result => {
  console.log(result); // '成功'
});

// 链式调用
promise
  .then(result => {
    console.log(result); // '成功'
    return result + '1';
  })
  .then(result => {
    console.log(result); // '成功1'
    return result + '2';
  })
  .then(result => {
    console.log(result); // '成功12'
  });

// 返回 Promise
promise
  .then(result => {
    console.log(result); // '成功'
    return new Promise(resolve => {
      setTimeout(() => {
        resolve('延迟成功');
      }, 1000);
    });
  })
  .then(result => {
    console.log(result); // '延迟成功'
  });

// 两个回调参数
promise.then(
  result => {
    console.log('成功:', result);
  },
  error => {
    console.log('失败:', error);
  }
);
```

### catch 方法

```javascript
const promise = new Promise((resolve, reject) => {
  reject('失败');
});

// 基本错误捕获
promise.catch(error => {
  console.log('错误:', error); // '错误: 失败'
});

// 链式错误处理
promise
  .then(result => {
    console.log(result);
    return result;
  })
  .catch(error => {
    console.log('捕获错误:', error);
    return '默认值'; // 错误处理后继续执行
  })
  .then(result => {
    console.log('继续执行:', result); // '继续执行: 默认值'
  });

// then 中的错误也会被 catch 捕获
new Promise((resolve) => {
  resolve('成功');
})
  .then(result => {
    throw new Error('then 中的错误');
  })
  .catch(error => {
    console.log('捕获错误:', error.message); // '捕获错误: then 中的错误'
  });
```

### finally 方法

```javascript
const promise = new Promise((resolve) => {
  setTimeout(() => {
    resolve('成功');
  }, 1000);
});

promise
  .then(result => {
    console.log('结果:', result);
    return result;
  })
  .catch(error => {
    console.log('错误:', error);
    throw error;
  })
  .finally(() => {
    console.log('无论如何都会执行');
    // finally 不接收参数，无法知道是成功还是失败
    // finally 的返回值会被忽略
  });

// finally 中抛出的错误会被传递
promise
  .finally(() => {
    console.log('清理工作');
    throw new Error('finally 中的错误');
  })
  .catch(error => {
    console.log('捕获 finally 错误:', error.message);
  });
```

## Promise 的静态方法

### Promise.resolve()

```javascript
// 创建已成功的 Promise
const p1 = Promise.resolve('成功');
console.log(p1); // Promise {<fulfilled>: '成功'}

// 包装非 Promise 值
const p2 = Promise.resolve({ id: 1, name: '张三' });
p2.then(data => console.log(data)); // { id: 1, name: '张三' }

// 传递 Promise 会返回原 Promise
const original = Promise.resolve('原始');
const returned = Promise.resolve(original);
console.log(original === returned); // true

// thenable 对象
const thenable = {
  then: function(resolve) {
    resolve('thenable 对象');
  }
};
Promise.resolve(thenable).then(result => {
  console.log(result); // 'thenable 对象'
});
```

### Promise.reject()

```javascript
// 创建已失败的 Promise
const p1 = Promise.reject('失败');
console.log(p1); // Promise {<rejected>: '失败'}

// 包装错误对象
const p2 = Promise.reject(new Error('自定义错误'));
p2.catch(error => {
  console.log(error.message); // '自定义错误'
});

// 传递 Promise 会创建新的失败 Promise
const original = Promise.reject('原始错误');
const returned = Promise.reject(original);
console.log(original === returned); // false
```

### Promise.all()

```javascript
// 所有 Promise 成功才成功
const p1 = Promise.resolve('p1');
const p2 = Promise.resolve('p2');
const p3 = Promise.resolve('p3');

Promise.all([p1, p2, p3])
  .then(results => {
    console.log(results); // ['p1', 'p2', 'p3']
  });

// 只要有一个失败就失败
const p4 = Promise.reject('p4 失败');
const p5 = Promise.resolve('p5');

Promise.all([p4, p5])
  .then(results => {
    console.log(results);
  })
  .catch(error => {
    console.log('错误:', error); // '错误: p4 失败'
  });

// 实际应用：并行请求
function getUser(id) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id, name: `用户${id}` });
    }, 100);
  });
}

Promise.all([
  getUser(1),
  getUser(2),
  getUser(3)
]).then(users => {
  console.log('所有用户:', users);
});
```

### Promise.race()

```javascript
// 返回最先完成的 Promise
const p1 = new Promise(resolve => {
  setTimeout(() => resolve('p1 成功'), 300);
});

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 成功'), 100);
});

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 成功'), 200);
});

Promise.race([p1, p2, p3])
  .then(result => {
    console.log(result); // 'p2 成功' (最快完成的)
  });

// 竞态条件处理
function fetchWithTimeout(url, timeout) {
  const fetchPromise = fetch(url);
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(new Error('请求超时')), timeout);
  });

  return Promise.race([fetchPromise, timeoutPromise]);
}

// 使用
// fetchWithTimeout('/api/data', 5000)
//   .then(response => response.json())
//   .catch(error => console.error(error));
```

### Promise.allSettled()

```javascript
// 返回所有 Promise 的结果，无论成功失败
const p1 = Promise.resolve('p1');
const p2 = Promise.reject('p2 失败');
const p3 = Promise.resolve('p3');

Promise.allSettled([p1, p2, p3])
  .then(results => {
    console.log(results);
    // [
    //   { status: 'fulfilled', value: 'p1' },
    //   { status: 'rejected', reason: 'p2 失败' },
    //   { status: 'fulfilled', value: 'p3' }
    // ]
  });

// 实际应用：批量操作不希望全部失败
async function batchUpload(files) {
  const uploadPromises = files.map(file => uploadFile(file));
  const results = await Promise.allSettled(uploadPromises);

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`成功: ${successful.length}, 失败: ${failed.length}`);
  return { successful, failed };
}
```

### Promise.any()

```javascript
// 返回第一个成功的 Promise
const p1 = Promise.reject('p1 失败');
const p2 = Promise.resolve('p2 成功');
const p3 = Promise.reject('p3 失败');

Promise.any([p1, p2, p3])
  .then(result => {
    console.log(result); // 'p2 成功'
  });

// 全部失败时返回 AggregateError
Promise.any([
  Promise.reject('错误1'),
  Promise.reject('错误2')
]).catch(error => {
  console.log(error instanceof AggregateError); // true
  console.log(error.errors); // ['错误1', '错误2']
});

// 实际应用：多源数据获取
function fetchData(sources) {
  const promises = sources.map(source => fetch(source));

  return Promise.any(promises);
}

// 只要有任何一个源可用就能获取到数据
```

## 实际应用场景

### 1. 封装异步操作

```javascript
// 封装 setTimeout
function delay(ms) {
  return new Promise(resolve => {
    setTimeout(resolve, ms);
  });
}

// 使用
delay(1000).then(() => {
  console.log('1秒后执行');
});

// 封装 XMLHttpRequest
function getJSON(url) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    xhr.open('GET', url);
    xhr.responseType = 'json';

    xhr.onload = () => {
      if (xhr.status === 200) {
        resolve(xhr.response);
      } else {
        reject(new Error(`HTTP 错误: ${xhr.status}`));
      }
    };

    xhr.onerror = () => reject(new Error('网络错误'));
    xhr.send();
  });
}

// 封装事件监听
function waitForEvent(element, eventType) {
  return new Promise(resolve => {
    element.addEventListener(eventType, resolve, { once: true });
  });
}

// 使用
// waitForEvent(button, 'click').then(() => {
//   console.log('按钮被点击');
// });
```

### 2. 串行异步操作

```javascript
// 顺序执行
async function sequentialOperations() {
  const result1 = await operation1();
  const result2 = await operation2(result1);
  const result3 = await operation3(result2);
  return result3;
}

// 使用 Promise.then 链式调用
operation1()
  .then(result1 => {
    console.log('操作1完成:', result1);
    return operation2(result1);
  })
  .then(result2 => {
    console.log('操作2完成:', result2);
    return operation3(result2);
  })
  .then(result3 => {
    console.log('操作3完成:', result3);
  })
  .catch(error => {
    console.error('出错:', error);
  });
```

### 3. 并行异步操作

```javascript
// 并行执行多个独立的异步操作
async function parallelOperations() {
  const [user, posts, comments] = await Promise.all([
    fetchUser(),
    fetchPosts(),
    fetchComments()
  ]);

  return { user, posts, comments };
}

// 批量处理
async function batchProcess(items) {
  const results = await Promise.all(
    items.map(item => processItem(item))
  );

  return results;
}

// 分批并行处理
async function batchProcessInChunks(items, chunkSize = 10) {
  const results = [];

  for (let i = 0; i < items.length; i += chunkSize) {
    const chunk = items.slice(i, i + chunkSize);
    const chunkResults = await Promise.all(
      chunk.map(item => processItem(item))
    );
    results.push(...chunkResults);
  }

  return results;
}
```

### 4. 错误处理和重试

```javascript
// 带重试的异步操作
function retry(fn, times = 3) {
  return new Promise((resolve, reject) => {
    function attempt(remainingAttempts) {
      fn()
        .then(resolve)
        .catch(error => {
          if (remainingAttempts <= 1) {
            reject(error);
          } else {
            console.log(`重试中... 剩余次数: ${remainingAttempts - 1}`);
            setTimeout(() => {
              attempt(remainingAttempts - 1);
            }, 1000);
          }
        });
    }

    attempt(times);
  });
}

// 使用
retry(() => fetch('/api/data'))
  .then(response => response.json())
  .catch(error => console.error('全部失败:', error));

// 指数退避重试
async function exponentialBackoff(fn, maxAttempts = 3) {
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      if (attempt === maxAttempts) {
        throw error;
      }

      const delay = Math.pow(2, attempt) * 1000;
      console.log(`等待 ${delay}ms 后重试...`);
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
}
```

### 5. 缓存和去重

```javascript
// Promise 缓存
const promiseCache = new Map();

function cachedFetch(url) {
  if (promiseCache.has(url)) {
    return promiseCache.get(url);
  }

  const promise = fetch(url)
    .then(response => {
      if (!response.ok) {
        throw new Error(`HTTP 错误: ${response.status}`);
      }
      return response.json();
    })
    .catch(error => {
      promiseCache.delete(url); // 失败时移除缓存
      throw error;
    });

  promiseCache.set(url, promise);
  return promise;
}

// 使用
// cachedFetch('/api/user').then(user => console.log(user));
// cachedFetch('/api/user').then(user => console.log(user)); // 使用缓存

// 去重相同的 Promise 请求
function deduplicatedFetch(url) {
  const pendingRequests = new Map();

  return function(url) {
    if (pendingRequests.has(url)) {
      return pendingRequests.get(url);
    }

    const promise = fetch(url)
      .then(response => response.json())
      .finally(() => {
        pendingRequests.delete(url);
      });

    pendingRequests.set(url, promise);
    return promise;
  };
}

const uniqueFetch = deduplicatedFetch();
```

### 6. 超时处理

```javascript
// 设置 Promise 超时
function withTimeout(promise, timeout, error = new Error('操作超时')) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => reject(error), timeout);
  });

  return Promise.race([promise, timeoutPromise]);
}

// 使用
withTimeout(
  fetch('/api/data'),
  5000,
  new Error('请求超时，请检查网络连接')
)
  .then(response => response.json())
  .catch(error => console.error(error));

// 可取消的 Promise
function makeCancellable(promise) {
  let cancelled = false;

  const wrappedPromise = new Promise((resolve, reject) => {
    promise
      .then(result => {
        if (!cancelled) resolve(result);
      })
      .catch(error => {
        if (!cancelled) reject(error);
      });
  });

  return {
    promise: wrappedPromise,
    cancel() {
      cancelled = true;
    }
  };
}

// 使用
const { promise, cancel } = makeCancellable(
  fetch('/api/data').then(res => res.json())
);

// cancel(); // 取消请求
```

## 常见陷阱和最佳实践

### 1. 忘记返回 Promise

```javascript
// 错误示例
function badExample() {
  fetch('/api/data')
    .then(response => {
      response.json(); // 忘记返回
    })
    .then(data => {
      console.log(data); // undefined
    });
}

// 正确示例
function goodExample() {
  return fetch('/api/data')
    .then(response => {
      return response.json(); // 返回 Promise
    })
    .then(data => {
      console.log(data);
    });
}
```

### 2. 在 forEach 中使用 Promise

```javascript
// 错误示例
[1, 2, 3].forEach(async num => {
  await processData(num);
});
// 无法等待所有 Promise 完成

// 正确示例
await Promise.all(
  [1, 2, 3].map(num => processData(num))
);

// 或使用 for...of
for (const num of [1, 2, 3]) {
  await processData(num);
}
```

### 3. 忽略错误处理

```javascript
// 错误示例
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data));
// 错误未被捕获

// 正确示例
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error('错误:', error));
```

### 4. 过度嵌套

```javascript
// 错误示例
fetch('/api/user')
  .then(response => {
    return response.json();
  })
  .then(user => {
    return fetch(`/api/posts/${user.id}`)
      .then(response => {
        return response.json();
      })
      .then(posts => {
        return fetch(`/api/comments/${posts[0].id}`)
          .then(response => {
            return response.json();
          })
          .then(comments => {
            console.log(comments);
          });
      });
  });

// 正确示例
fetch('/api/user')
  .then(response => response.json())
  .then(user => fetch(`/api/posts/${user.id}`))
  .then(response => response.json())
  .then(posts => fetch(`/api/comments/${posts[0].id}`))
  .then(response => response.json())
  .then(comments => console.log(comments))
  .catch(error => console.error(error));
```

### 5. 混用 Promise 和 async/await

```javascript
// 好的实践：async 函数中返回 Promise
async function fetchData() {
  const response = await fetch('/api/data');
  return response.json(); // 返回 Promise
}

// 调用
fetchData().then(data => console.log(data));

// 好的实践：Promise 链中处理异步
fetch('/api/data')
  .then(response => response.json())
  .then(async data => {
    const processed = await processData(data);
    return processed;
  })
  .then(result => console.log(result));
```

## 注意事项

1. **Promise 一旦创建就会立即执行**：即使没有调用 then，executor 函数也会立即执行。

2. **状态一旦改变就不会再变**：Promise 只能从 pending 变为 fulfilled 或 rejected，不能逆向转换。

3. **then 方法返回的是新的 Promise**：链式调用中每个 then 都返回新的 Promise 实例。

4. **错误处理很重要**：始终使用 catch 或在 then 的第二个回调中处理错误，避免未捕获的 Promise 拒绝。

5. **避免内存泄漏**：长时间运行的 Promise 要注意清理，避免持有不必要的引用。

6. **合理使用 Promise.all**：确保传入的 Promise 数组不是过大，避免同时发起过多请求。

7. **注意微任务和宏任务**：Promise 的回调在微任务队列中执行，执行时机与 setTimeout 不同。

8. **避免全局异常**：未捕获的 Promise 拒绝可能导致全局异常，可以使用 `unhandledrejection` 事件处理。

## 总结

Promise 让异步代码更加优雅和易于维护，是现代 JavaScript 异步编程的基石。掌握 Promise 的基础用法、静态方法、错误处理和最佳实践，对于编写高质量的异步代码至关重要。在实际开发中，合理运用 Promise 及其相关方法，可以大大提高代码的可读性和可维护性。记住要始终处理错误，避免常见的陷阱，根据具体场景选择合适的方法来处理异步操作。