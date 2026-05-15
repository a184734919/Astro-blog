---
title: JavaScript 错误处理机制
published: 2022-05-02
description: 'try-catch 和错误类型详解的详细介绍和学习笔记'
image: ''
tags: ["JS基础","错误处理机制"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 错误处理机制是前端开发中的重要内容。良好的错误处理可以让应用更加健壮，提供更好的用户体验，并帮助开发者快速定位和解决问题。本文将详细介绍 JavaScript 的错误类型、错误捕获方法以及最佳实践。

## 错误类型

### 内置错误类型

JavaScript 提供了多种内置错误类型，每种类型对应不同的错误情况：

```javascript
// 1. Error - 基础错误类型
const error = new Error('这是一个错误');
console.log(error.name);    // 'Error'
console.log(error.message); // '这是一个错误'
console.log(error.stack);   // 错误堆栈信息

// 2. TypeError - 类型错误
try {
  null.method(); // null 没有 method 方法
} catch (error) {
  console.log(error.name);    // 'TypeError'
  console.log(error.message); // 'null is not an object' 或类似信息
}

// 3. ReferenceError - 引用错误
try {
  console.log(undeclaredVariable); // 未声明的变量
} catch (error) {
  console.log(error.name);    // 'ReferenceError'
  console.log(error.message); // 'undeclaredVariable is not defined'
}

// 4. SyntaxError - 语法错误
try {
  eval('var x = ;'); // 不完整的表达式
} catch (error) {
  console.log(error.name);    // 'SyntaxError'
  console.log(error.message); // 'Unexpected end of input'
}

// 5. RangeError - 范围错误
try {
  new Array(-1); // 数组长度不能为负数
} catch (error) {
  console.log(error.name);    // 'RangeError'
  console.log(error.message); // 'Invalid array length'
}

// 6. URIError - URI 错误
try {
  decodeURIComponent('%'); // 无效的 URI 编码
} catch (error) {
  console.log(error.name);    // 'URIError'
  console.log(error.message); // 'URI malformed'
}

// 7. EvalError - eval 错误（已废弃，但仍存在）
try {
  throw new EvalError('eval 相关错误');
} catch (error) {
  console.log(error.name);    // 'EvalError'
  console.log(error.message); // 'eval 相关错误'
}
```

### 自定义错误类型

```javascript
// 基本自定义错误
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = 'CustomError';
  }
}

throw new CustomError('自定义错误');

// 带有额外信息的自定义错误
class ValidationError extends Error {
  constructor(message, field, value) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
    this.value = value;
    this.timestamp = new Date();
  }
}

try {
  throw new ValidationError('值无效', 'age', -5);
} catch (error) {
  if (error instanceof ValidationError) {
    console.log('字段:', error.field);
    console.log('值:', error.value);
    console.log('时间:', error.timestamp);
  }
}

// 网络错误
class NetworkError extends Error {
  constructor(message, statusCode, url) {
    super(message);
    this.name = 'NetworkError';
    this.statusCode = statusCode;
    this.url = url;
  }
}

// API 错误
class APIError extends Error {
  constructor(message, endpoint, statusCode, response) {
    super(message);
    this.name = 'APIError';
    this.endpoint = endpoint;
    this.statusCode = statusCode;
    this.response = response;
  }
}

// 业务逻辑错误
class BusinessError extends Error {
  constructor(message, code, details = {}) {
    super(message);
    this.name = 'BusinessError';
    this.code = code;
    this.details = details;
  }
}

// 使用示例
class UserService {
  async getUser(userId) {
    if (!userId) {
      throw new ValidationError('用户ID不能为空', 'userId', userId);
    }

    if (typeof userId !== 'number' || userId <= 0) {
      throw new BusinessError(
        '用户ID格式错误',
        'INVALID_USER_ID',
        { userId, expectedType: 'number' }
      );
    }

    // 模拟 API 调用
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new APIError(
        '获取用户失败',
        `/api/users/${userId}`,
        response.status,
        await response.json()
      );
    }

    return await response.json();
  }
}
```

## try-catch-finally

### 基本用法

```javascript
// 基本的 try-catch
try {
  // 可能出错的代码
  const result = 10 / 0; // Infinity，不是错误
  console.log(result);
} catch (error) {
  // 错误处理代码
  console.error('发生错误:', error.message);
}

// 带 finally 的 try-catch-finally
try {
  console.log('开始执行');
  throw new Error('测试错误');
} catch (error) {
  console.error('捕获错误:', error.message);
} finally {
  console.log('无论如何都会执行');
}

// 输出：
// 开始执行
// 捕获错误: 测试错误
// 无论如何都会执行
```

### 错误传播

```javascript
// 错误会向上传播
function level3() {
  throw new Error('level3 的错误');
}

function level2() {
  level3();
}

function level1() {
  try {
    level2();
  } catch (error) {
    console.error('在 level1 捕获:', error.message);
    throw error; // 重新抛出
  }
}

try {
  level1();
} catch (error) {
  console.error('在顶层捕获:', error.message);
}

// 输出：
// 在 level1 捕获: level3 的错误
// 在顶层捕获: level3 的错误
```

### 多个 catch 块

```javascript
try {
  const data = JSON.parse('invalid json');
} catch (error) {
  if (error instanceof SyntaxError) {
    console.error('语法错误:', error.message);
  } else if (error instanceof TypeError) {
    console.error('类型错误:', error.message);
  } else {
    console.error('未知错误:', error.message);
  }
}

// 使用条件判断
try {
  processUser({ name: '', age: -5 });
} catch (error) {
  switch (error.name) {
    case 'ValidationError':
      console.error('验证失败:', error.message);
      break;
    case 'BusinessError':
      console.error('业务错误:', error.message);
      break;
    case 'NetworkError':
      console.error('网络错误:', error.message);
      break;
    default:
      console.error('系统错误:', error.message);
  }
}
```

## throw 语句

### 抛出错误

```javascript
// 抛出字符串（不推荐）
throw '这是一个错误';

// 抛出数字（不推荐）
throw 404;

// 抛出对象（不推荐）
throw { message: '错误信息' };

// 推荐：抛出 Error 对象
throw new Error('这是一个错误');

// 抛出自定义错误
throw new ValidationError('验证失败', 'email', 'invalid');

// 条件抛出
function divide(a, b) {
  if (b === 0) {
    throw new Error('除数不能为零');
  }
  return a / b;
}

try {
  divide(10, 0);
} catch (error) {
  console.error(error.message); // '除数不能为零'
}

// 函数参数验证
function createUser(name, age) {
  if (!name || typeof name !== 'string') {
    throw new TypeError('name 必须是非空字符串');
  }

  if (typeof age !== 'number' || age <= 0) {
    throw new RangeError('age 必须是正数');
  }

  return { name, age };
}
```

### throw 表达式

```javascript
// ES2020+ 支持的 throw 表达式
function getValue(value) {
  return value ?? (() => {
    throw new Error('值不能为 null 或 undefined');
  })();

  // 等价于
  // return value ?? throw new Error('值不能为 null 或 undefined');
}

// 在模板字符串中使用
function formatMessage(message) {
  return message ?? throw new Error('消息不能为空');
}

// 在箭头函数中使用
const getUser = (id) => users.find(u => u.id === id) ?? throw new Error('用户不存在');
```

## 错误对象属性

### 标准属性

```javascript
const error = new Error('这是一个错误');

// name - 错误名称
console.log(error.name); // 'Error'

// message - 错误消息
console.log(error.message); // '这是一个错误'

// stack - 错误堆栈
console.log(error.stack);
// Error: 这是一个错误
//     at Object.<anonymous> (/path/to/file.js:1:13)
//     at Module._compile (internal/modules/cjs/loader.js:xxx:xx)
//     at ...

// 自定义属性
error.code = 'CUSTOM_ERROR';
error.statusCode = 500;
error.timestamp = new Date();

console.log(error.code);       // 'CUSTOM_ERROR'
console.log(error.statusCode); // 500
console.log(error.timestamp);  // Date 对象
```

### 堆栈追踪

```javascript
function first() {
  second();
}

function second() {
  third();
}

function third() {
  throw new Error('堆栈追踪示例');
}

try {
  first();
} catch (error) {
  console.log(error.stack);
}

// 输出示例：
// Error: 堆栈追踪示例
//     at third (/path/to/file.js:3:11)
//     at second (/path/to/file.js:2:3)
//     at first (/path/to/file.js:2:3)
//     at Object.<anonymous> (/path/to/file.js:6:3)
```

## 错误处理最佳实践

### 1. 全局错误处理

```javascript
// 全局错误捕获 - 同步代码
window.onerror = function(message, source, lineno, colno, error) {
  console.error('全局错误捕获:', {
    message,
    source,
    lineno,
    colno,
    error
  });

  // 发送到错误监控服务
  sendToErrorMonitoring({
    type: 'sync',
    message,
    source,
    lineno,
    colno,
    stack: error?.stack
  });

  // 返回 true 可以阻止默认错误处理
  return true;
};

// 全局未处理的 Promise 拒绝
window.addEventListener('unhandledrejection', (event) => {
  console.error('未处理的 Promise 拒绝:', event.reason);

  // 发送到错误监控
  sendToErrorMonitoring({
    type: 'promise',
    reason: event.reason,
    promise: event.promise
  });

  // 阻止默认的错误处理
  event.preventDefault();
});

// 处理已恢复的 Promise 拒绝
window.addEventListener('rejectionhandled', (event) => {
  console.log('Promise 拒绝已被处理:', event.reason);
});

// Node.js 中的全局错误处理
process.on('uncaughtException', (error) => {
  console.error('未捕获的异常:', error);
  // 不要在 uncaughtException 中继续运行应用
  // 应该优雅地关闭
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('未处理的 Promise 拒绝:', reason);
});
```

### 2. 异步错误处理

```javascript
// Promise 错误处理
fetch('/api/data')
  .then(response => {
    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }
    return response.json();
  })
  .then(data => {
    console.log(data);
  })
  .catch(error => {
    console.error('请求失败:', error);
    // 处理错误，提供降级方案
    return getDefaultData();
  });

// async/await 错误处理
async function fetchData() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }
    return await response.json();
  } catch (error) {
    console.error('获取数据失败:', error);
    throw error; // 重新抛出让上层处理
  }
}

async function main() {
  try {
    const data = await fetchData();
    renderData(data);
  } catch (error) {
    console.error('渲染失败:', error);
    showErrorMessage(error.message);
  }
}

// 事件处理中的错误
button.addEventListener('click', async () => {
  try {
    await handleButtonClick();
  } catch (error) {
    console.error('按钮点击处理失败:', error);
    showErrorToast(error.message);
  }
});

// 定时器中的错误
setTimeout(() => {
  try {
    riskyOperation();
  } catch (error) {
    console.error('定时器操作失败:', error);
  }
}, 1000);
```

### 3. 错误边界

```javascript
// 创建错误边界函数
function withErrorBoundary(fn, errorHandler) {
  return async function(...args) {
    try {
      return await fn.apply(this, args);
    } catch (error) {
      return errorHandler(error, ...args);
    }
  };
}

// 使用示例
const safeFetch = withErrorBoundary(
  async (url) => {
    const response = await fetch(url);
    return response.json();
  },
  (error, url) => {
    console.error(`请求 ${url} 失败:`, error);
    return null; // 提供默认值
  }
);

// 多层错误处理
class UserService {
  async getUser(userId) {
    try {
      return await this.fetchUser(userId);
    } catch (error) {
      if (error instanceof NetworkError) {
        // 尝试从缓存获取
        return this.getCachedUser(userId);
      }
      throw error;
    }
  }

  async fetchUser(userId) {
    const response = await fetch(`/api/users/${userId}`);
    if (!response.ok) {
      throw new NetworkError('网络请求失败', response.status);
    }
    return response.json();
  }

  async getCachedUser(userId) {
    const cached = localStorage.getItem(`user_${userId}`);
    return cached ? JSON.parse(cached) : null;
  }
}
```

### 4. 错误日志和监控

```javascript
// 错误日志工具
class ErrorLogger {
  constructor() {
    this.logs = [];
    this.maxLogs = 100;
  }

  log(error, context = {}) {
    const logEntry = {
      timestamp: new Date().toISOString(),
      error: {
        name: error.name,
        message: error.message,
        stack: error.stack
      },
      context,
      userAgent: navigator.userAgent,
      url: window.location.href
    };

    this.logs.push(logEntry);

    // 限制日志数量
    if (this.logs.length > this.maxLogs) {
      this.logs.shift();
    }

    // 发送到服务器
    this.sendToServer(logEntry);

    console.error('错误日志:', logEntry);
  }

  sendToServer(logEntry) {
    // 发送到错误监控服务
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(logEntry)
    }).catch(console.error);
  }

  getLogs() {
    return this.logs;
  }

  clearLogs() {
    this.logs = [];
  }
}

const errorLogger = new ErrorLogger();

// 使用错误日志
try {
  riskyOperation();
} catch (error) {
  errorLogger.log(error, {
    operation: 'riskyOperation',
    userId: currentUserId
  });
}

// React 错误边界示例
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    // 记录错误
    errorLogger.log(error, {
      componentStack: errorInfo.componentStack,
      errorBoundary: true
    });
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error-fallback">
          <h1>出错了</h1>
          <p>{this.state.error?.message}</p>
          <button onClick={() => window.location.reload()}>
            重新加载
          </button>
        </div>
      );
    }

    return this.props.children;
  }
}

// 使用 ErrorBoundary
<ErrorBoundary>
  <App />
</ErrorBoundary>
```

## 实际应用场景

### 1. API 请求错误处理

```javascript
class APIClient {
  constructor(baseURL) {
    this.baseURL = baseURL;
    this.errorLogger = new ErrorLogger();
  }

  async request(endpoint, options = {}) {
    const url = `${this.baseURL}${endpoint}`;

    try {
      const response = await fetch(url, {
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        },
        ...options
      });

      if (!response.ok) {
        const errorData = await response.json().catch(() => null);
        throw new APIError(
          errorData?.message || `HTTP 错误: ${response.status}`,
          endpoint,
          response.status,
          errorData
        );
      }

      return await response.json();

    } catch (error) {
      this.errorLogger.log(error, {
        endpoint,
        method: options.method || 'GET',
        url
      });

      throw error;
    }
  }

  async get(endpoint) {
    return this.request(endpoint, { method: 'GET' });
  }

  async post(endpoint, data) {
    return this.request(endpoint, {
      method: 'POST',
      body: JSON.stringify(data)
    });
  }

  async put(endpoint, data) {
    return this.request(endpoint, {
      method: 'PUT',
      body: JSON.stringify(data)
    });
  }

  async delete(endpoint) {
    return this.request(endpoint, { method: 'DELETE' });
  }
}

// 使用 API 客户端
const api = new APIClient('/api');

async function loadData() {
  try {
    const user = await api.get('/user');
    const posts = await api.get('/posts');
    return { user, posts };
  } catch (error) {
    if (error instanceof APIError) {
      if (error.statusCode === 401) {
        // 未授权，重定向到登录
        redirectToLogin();
      } else if (error.statusCode === 404) {
        // 资源不存在，显示友好提示
        showErrorMessage('请求的资源不存在');
      } else {
        // 其他错误
        showErrorMessage(`请求失败: ${error.message}`);
      }
    } else {
      showErrorMessage('网络连接失败，请稍后重试');
    }
    return null;
  }
}
```

### 2. 表单验证错误处理

```javascript
class FormValidator {
  constructor(rules) {
    this.rules = rules;
  }

  validate(data) {
    const errors = {};

    for (const field in this.rules) {
      const fieldRules = this.rules[field];
      const value = data[field];

      for (const rule of fieldRules) {
        try {
          rule(value, field, data);
        } catch (error) {
          errors[field] = error.message;
          break; // 第一个错误就停止
        }
      }
    }

    if (Object.keys(errors).length > 0) {
      throw new ValidationError('表单验证失败', null, errors);
    }

    return true;
  }
}

// 验证规则
const required = (value) => {
  if (!value || (typeof value === 'string' && value.trim() === '')) {
    throw new Error('此项为必填项');
  }
};

const email = (value) => {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (value && !emailRegex.test(value)) {
    throw new Error('请输入有效的邮箱地址');
  }
};

const minLength = (min) => (value) => {
  if (value && value.length < min) {
    throw new Error(`至少需要 ${min} 个字符`);
  }
};

const minLength6 = minLength(6);

const ageRange = (value) => {
  const age = parseInt(value);
  if (isNaN(age) || age < 0 || age > 150) {
    throw new Error('请输入有效的年龄');
  }
};

// 使用验证器
const userFormValidator = new FormValidator({
  name: [required, minLength(2)],
  email: [required, email],
  age: [required, ageRange],
  password: [required, minLength6]
});

async function handleFormSubmit(formData) {
  try {
    // 验证表单
    userFormValidator.validate(formData);

    // 提交数据
    const result = await api.post('/users', formData);
    showSuccessMessage('用户创建成功');
    return result;

  } catch (error) {
    if (error instanceof ValidationError) {
      // 显示表单验证错误
      displayFormErrors(error.value);
    } else {
      // 显示其他错误
      showErrorMessage(error.message);
    }
    return null;
  }
}
```

### 3. 资源清理错误处理

```javascript
// 确保资源正确释放
class ResourceManager {
  async useResource() {
    let resource;
    try {
      resource = await this.acquireResource();
      return await this.processResource(resource);
    } finally {
      if (resource) {
        await this.releaseResource(resource);
      }
    }
  }

  async acquireResource() {
    console.log('获取资源');
    return { id: 1, data: 'some data' };
  }

  async processResource(resource) {
    console.log('处理资源:', resource);
    // 模拟处理失败
    throw new Error('处理失败');
  }

  async releaseResource(resource) {
    console.log('释放资源:', resource.id);
    // 无论成功失败都会执行
  }
}

// 使用资源管理器
const manager = new ResourceManager();

try {
  await manager.useResource();
} catch (error) {
  console.error('操作失败:', error);
}
// 输出：
// 获取资源
// 处理资源: { id: 1, data: 'some data' }
// 释放资源: 1
// 操作失败: 处理失败
```

### 4. 重试机制错误处理

```javascript
class RetryHandler {
  constructor(options = {}) {
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 1000;
    this.backoffMultiplier = options.backoffMultiplier || 2;
  }

  async execute(fn, shouldRetry = () => true) {
    let lastError;
    let delay = this.retryDelay;

    for (let attempt = 0; attempt <= this.maxRetries; attempt++) {
      try {
        return await fn();
      } catch (error) {
        lastError = error;

        if (attempt === this.maxRetries || !shouldRetry(error)) {
          throw error;
        }

        console.log(`第 ${attempt + 1} 次失败，${delay}ms 后重试...`);

        // 指数退避
        await new Promise(resolve => setTimeout(resolve, delay));
        delay *= this.backoffMultiplier;
      }
    }

    throw lastError;
  }
}

// 使用重试处理器
const retryHandler = new RetryHandler({
  maxRetries: 3,
  retryDelay: 1000,
  backoffMultiplier: 2
});

async function fetchWithRetry(url) {
  return retryHandler.execute(
    async () => {
      const response = await fetch(url);
      if (!response.ok) {
        throw new Error(`HTTP 错误: ${response.status}`);
      }
      return response.json();
    },
    (error) => {
      // 只在网络错误或 5xx 错误时重试
      return error.name === 'TypeError' ||
             error.message.includes('5');
    }
  );
}

// 使用
fetchWithRetry('/api/data')
  .then(data => console.log(data))
  .catch(error => console.error('全部重试失败:', error));
```

## 注意事项

1. **不要捕获所有错误**：避免捕获所有异常，应该只捕获预期的错误。

2. **提供有意义的错误信息**：错误消息应该清晰、具体，帮助理解和解决问题。

3. **正确使用 throw**：始终抛出 Error 对象或其子类实例。

4. **处理异步错误**：异步代码中的错误需要特殊处理，使用 Promise.catch 或 try-catch。

5. **避免吞掉错误**：不要静默忽略错误，至少应该记录日志。

6. **合理使用全局错误处理**：全局错误处理应该用于监控，不应该作为主要的错误处理机制。

7. **考虑性能影响**：过度使用 try-catch 可能影响性能，只在需要的地方使用。

8. **区分业务错误和系统错误**：不同类型的错误需要不同的处理策略。

## 总结

JavaScript 错误处理机制是构建健壮应用的基础。通过理解错误类型、掌握 try-catch-finally、throw 等机制，结合最佳实践，可以有效地捕获和处理错误，提供更好的用户体验。

关键要点：
- 使用适当的错误类型区分不同错误
- 合理使用 try-catch-finally 进行错误捕获和资源清理
- 提供有意义的错误信息，帮助调试和用户理解
- 实现全局错误处理，捕获未处理的异常
- 对于异步操作，正确处理 Promise 和 async/await 的错误
- 建立错误日志和监控系统，及时发现和解决问题
- 根据不同的业务场景，选择合适的错误处理策略

良好的错误处理不仅能防止应用崩溃，还能提供更好的用户体验，帮助开发者快速定位和解决问题。