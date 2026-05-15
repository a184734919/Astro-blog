---
title: JavaScript 高阶函数应用
description: 深入理解高阶函数的原理、应用场景和最佳实践，掌握函数式编程核心概念
published: 2024-01-03
tags: ["函数式编程","高阶函数"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 高阶函数应用

## 概述

高阶函数（Higher-Order Function）是函数式编程的核心概念，也是 JavaScript 最强大的特性之一。简单来说，高阶函数是指：

1. **接收函数作为参数**：如 `map`、`filter`、`reduce` 等数组方法
2. **返回函数作为结果**：如函数柯里化、高阶组件等

JavaScript 的函数是第一类公民，这意味着函数可以像其他数据类型一样被传递、赋值和操作。这为高阶函数的应用提供了天然的基础。

## 核心概念

### 什么是高阶函数

```javascript
// 接收函数作为参数的高阶函数
function operate(a, b, operation) {
  return operation(a, b);
}

const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

console.log(operate(5, 3, add));      // 8
console.log(operate(5, 3, multiply)); // 15

// 返回函数的高阶函数
function createMultiplier(multiplier) {
  return function(x) {
    return x * multiplier;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

### 高阶函数的价值

1. **代码复用**：通过函数参数实现通用逻辑
2. **抽象能力**：将通用模式抽象为可重用函数
3. **声明式编程**：描述"做什么"而不是"怎么做"
4. **组合性**：小函数可以组合成复杂功能
5. **可测试性**：纯函数易于单元测试

## 基本用法

### 1. 数组高阶函数

#### map - 转换数组

```javascript
const numbers = [1, 2, 3, 4, 5];

// 基础用法
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 转换对象数组
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

const names = users.map(user => user.name);
console.log(names); // ['Alice', 'Bob']

// 高级用法：提取多个属性
const userDetails = users.map(({ id, name }) => ({
  userId: id,
  displayName: name
}));
```

#### filter - 过滤数组

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 基础过滤
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// 复杂过滤条件
const products = [
  { id: 1, name: 'Laptop', price: 1000, stock: 5 },
  { id: 2, name: 'Phone', price: 500, stock: 10 },
  { id: 3, name: 'Tablet', price: 300, stock: 0 }
];

const inStock = products.filter(item => item.stock > 0);
const affordable = products.filter(item => item.price < 600);
const premiumInStock = products.filter(item =>
  item.price > 800 && item.stock > 0
);
```

#### reduce - 归约数组

```javascript
// 数组求和
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

// 数组转对象
const users = [
  { id: 1, name: 'Alice' },
  { id: 2, name: 'Bob' }
];

const userMap = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log(userMap);
// { 1: { id: 1, name: 'Alice' }, 2: { id: 2, name: 'Bob' } }

// 统计词频
const words = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const wordCount = words.reduce((acc, word) => {
  acc[word] = (acc[word] || 0) + 1;
  return acc;
}, {});

console.log(wordCount); // { apple: 3, banana: 2, orange: 1 }
```

### 2. 自定义高阶函数

#### 函数组合

```javascript
// 组合多个函数
function compose(...functions) {
  return function(x) {
    return functions.reduceRight((acc, fn) => fn(acc), x);
  };
}

const toUpper = str => str.toUpperCase();
const reverse = str => str.split('').reverse().join('');
const exclaim = str => str + '!';

const transform = compose(exclaim, reverse, toUpper);
console.log(transform('hello')); // 'OLLEH!'

// 从右到左执行：toUpper -> reverse -> exclaim
// 'hello' -> 'HELLO' -> 'OLLEH' -> 'OLLEH!'

// 管道函数（从左到右）
function pipe(...functions) {
  return function(x) {
    return functions.reduce((acc, fn) => fn(acc), x);
  };
}

const transformPipe = pipe(toUpper, reverse, exclaim);
console.log(transformPipe('hello')); // '!OLLEH'
```

#### 柯里化

```javascript
// 创建柯里化函数
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6

// 实际应用：创建专用函数
const multiply = (a, b, c) => a * b * c;
const curriedMultiply = curry(multiply);

const double = curriedMultiply(2);
const triple = curriedMultiply(3);

console.log(double(5)(10)); // 100
console.log(triple(5)(10)); // 150

// 更实用的例子：HTTP 请求
const request = (method, url, data) => {
  // 模拟 HTTP 请求
  return { method, url, data };
};

const curriedRequest = curry(request);
const get = curriedRequest('GET');
const post = curriedRequest('POST');

const getUser = get('/api/users');
const updateUser = post('/api/users/update');

console.log(getUser({ id: 1 }));
// { method: 'GET', url: '/api/users', data: { id: 1 } }

console.log(updateUser({ id: 1, name: 'John' }));
// { method: 'POST', url: '/api/users/update', data: { id: 1, name: 'John' } }
```

#### 防抖与节流

```javascript
// 防抖函数
function debounce(func, delay) {
  let timeoutId;
  return function(...args) {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => {
      func.apply(this, args);
    }, delay);
  };
}

// 节流函数
function throttle(func, delay) {
  let lastCall = 0;
  return function(...args) {
    const now = new Date().getTime();
    if (now - lastCall < delay) return;
    lastCall = now;
    func.apply(this, args);
  };
}

// 使用示例
const searchInput = document.getElementById('search');
const debouncedSearch = debounce(event => {
  console.log('Searching for:', event.target.value);
}, 300);

searchInput.addEventListener('input', debouncedSearch);

// 滚动事件节流
const throttledScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100);

window.addEventListener('scroll', throttledScroll);
```

#### 记忆化

```javascript
// 记忆化函数
function memoize(fn, keyGenerator = (...args) => args.join('_')) {
  const cache = new Map();

  return function(...args) {
    const key = keyGenerator(...args);

    if (cache.has(key)) {
      console.log('Cache hit:', key);
      return cache.get(key);
    }

    console.log('Cache miss:', key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// 计算斐波那契数列
const fibonacci = memoize(function(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

console.log(fibonacci(10)); // 55
console.log(fibonacci(10)); // 从缓存中获取

// 缓存 API 请求
const fetchData = memoize(async function(url) {
  const response = await fetch(url);
  return response.json();
});

// 第一次调用会发送请求
fetchData('/api/users').then(data => console.log(data));

// 第二次调用从缓存获取
fetchData('/api/users').then(data => console.log(data));
```

## 实际应用

### 1. API 数据处理

```javascript
async function fetchUsers() {
  const response = await fetch('/api/users');
  const users = await response.json();

  // 使用高阶函数链式处理数据
  return users
    .filter(user => user.isActive) // 筛选活跃用户
    .map(user => ({ // 转换数据格式
      id: user.id,
      name: user.name,
      email: user.email
    }))
    .sort((a, b) => a.name.localeCompare(b.name)); // 按名称排序
}

// 复杂数据处理管道
function processApiResponse(apiResponse) {
  return apiResponse
    .filter(item => item.status === 'active')
    .map(item => ({
      ...item,
      processedValue: item.value * 1.1,
      category: categorize(item.value),
      timestamp: new Date(item.created_at).toISOString()
    }))
    .reduce((acc, item) => {
      // 按类别分组统计
      if (!acc[item.category]) {
        acc[item.category] = {
          count: 0,
          total: 0,
          items: []
        };
      }
      acc[item.category].count++;
      acc[item.category].total += item.processedValue;
      acc[item.category].items.push(item);
      return acc;
    }, {});
}
```

### 2. 表单验证

```javascript
// 创建验证器
function createValidator(rules) {
  return function(value) {
    const errors = [];
    let isValid = true;

    for (const rule of rules) {
      if (!rule.validator(value)) {
        errors.push(rule.error);
        isValid = false;
      }
    }

    return { isValid, errors };
  };
}

// 邮箱验证器
const emailValidator = createValidator([
  { validator: v => v.includes('@'), error: '必须包含 @ 符号' },
  { validator: v => v.includes('.'), error: '必须包含 . 符号' },
  { validator: v => v.length > 5, error: '长度必须大于 5' }
]);

console.log(emailValidator('test@email.com'));
// { isValid: true, errors: [] }

console.log(emailValidator('invalid'));
// { isValid: false, errors: ['必须包含 @ 符号', '必须包含 . 符号', '长度必须大于 5'] }

// 密码验证器
const passwordValidator = createValidator([
  { validator: v => v.length >= 8, error: '密码长度至少8位' },
  { validator: v => /[A-Z]/.test(v), error: '必须包含大写字母' },
  { validator: v => /[a-z]/.test(v), error: '必须包含小写字母' },
  { validator: v => /[0-9]/.test(v), error: '必须包含数字' }
]);

console.log(passwordValidator('Password123'));
// { isValid: true, errors: [] }

// 综合表单验证
function validateForm(formData, validators) {
  const errors = {};
  let isValid = true;

  for (const [field, validator] of Object.entries(validators)) {
    const result = validator(formData[field]);
    if (!result.isValid) {
      errors[field] = result.errors;
      isValid = false;
    }
  }

  return { isValid, errors };
}

const formValidators = {
  email: emailValidator,
  password: passwordValidator,
  username: createValidator([
    { validator: v => v.length >= 3, error: '用户名长度至少3位' },
    { validator: v => /^[a-zA-Z0-9_]+$/.test(v), error: '只能包含字母、数字和下划线' }
  ])
};

const formData = {
  email: 'test@example.com',
  password: 'Password123',
  username: 'john_doe'
};

const validationResult = validateForm(formData, formValidators);
console.log(validationResult);
```

### 3. 事件处理

```javascript
// 创建事件处理器工厂
function createEventHandler(handler, options = {}) {
  return function(event) {
    if (options.preventDefault) {
      event.preventDefault();
    }

    if (options.stopPropagation) {
      event.stopPropagation();
    }

    if (options.validate && !options.validate()) {
      return;
    }

    handler(event);
  };
}

// 表单提交处理器
const formSubmitHandler = createEventHandler(
  (event) => {
    const formData = new FormData(event.target);
    const data = Object.fromEntries(formData);
    console.log('Form submitted:', data);
    // 提交数据到服务器
  },
  {
    preventDefault: true,
    validate: () => {
      const form = event.target;
      return form.checkValidity();
    }
  }
);

document.getElementById('myForm').addEventListener('submit', formSubmitHandler);

// 按钮点击处理器
const buttonClickHandler = createEventHandler(
  (event) => {
    console.log('Button clicked:', event.target.id);
    // 执行按钮点击逻辑
  },
  {
    stopPropagation: true
  }
);

document.getElementById('myButton').addEventListener('click', buttonClickHandler);
```

### 4. 数据转换和格式化

```javascript
// 创建数据转换器
function createTransformer(transformers) {
  return function(data) {
    return transformers.reduce((result, transformer) => {
      return transformer(result);
    }, data);
  };
}

// API 数据转换
const apiDataTransformer = createTransformer([
  // 去除空值
  data => data.filter(item => item != null),
  // 格式化日期
  data => data.map(item => ({
    ...item,
    createdAt: item.createdAt ? new Date(item.createdAt).toISOString() : null
  })),
  // 转换数值类型
  data => data.map(item => ({
    ...item,
    price: parseFloat(item.price) || 0,
    quantity: parseInt(item.quantity) || 0
  })),
  // 添加计算字段
  data => data.map(item => ({
    ...item,
    total: (item.price * item.quantity).toFixed(2)
  }))
]);

// 使用示例
const rawApiData = [
  { id: 1, name: 'Product A', price: '99.99', quantity: '2', createdAt: '2024-01-01' },
  { id: 2, name: 'Product B', price: '149.99', quantity: '1', createdAt: null }
];

const transformedData = apiDataTransformer(rawApiData);
console.log(transformedData);
```

### 5. 权限控制

```javascript
// 创建权限检查函数
function createPermissionChecker(userPermissions) {
  return function(requiredPermission) {
    return userPermissions.includes(requiredPermission) ||
           userPermissions.includes('admin');
  };
}

// 创建受保护的路由处理器
function createProtectedHandler(handler, permissionChecker) {
  return function(...args) {
    if (!permissionChecker('access')) {
      throw new Error('Access denied');
    }
    return handler(...args);
  };
}

// 使用示例
const userPermissions = ['read', 'write'];
const checkPermission = createPermissionChecker(userPermissions);

const protectedHandler = createProtectedHandler(
  (data) => {
    console.log('Processing data:', data);
    return { success: true, data };
  },
  checkPermission
);

try {
  const result = protectedHandler({ message: 'Hello' });
  console.log(result);
} catch (error) {
  console.error('Error:', error.message);
}
```

### 6. 日志和监控

```javascript
// 创建日志记录器
function createLogger(options = {}) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args) {
      const startTime = performance.now();

      if (options.logArgs) {
        console.log(`[${propertyKey}] Args:`, args);
      }

      try {
        const result = originalMethod.apply(this, args);

        if (options.logResult) {
          console.log(`[${propertyKey}] Result:`, result);
        }

        const endTime = performance.now();
        if (options.logTime) {
          console.log(`[${propertyKey}] Time: ${(endTime - startTime).toFixed(2)}ms`);
        }

        return result;
      } catch (error) {
        console.error(`[${propertyKey}] Error:`, error);
        throw error;
      }
    };

    return descriptor;
  };
}

// 创建性能监控函数
function createPerformanceMonitor(threshold = 1000) {
  return function(fn) {
    return function(...args) {
      const startTime = performance.now();
      const result = fn.apply(this, args);
      const endTime = performance.now();
      const duration = endTime - startTime;

      if (duration > threshold) {
        console.warn(`Performance warning: ${fn.name} took ${duration.toFixed(2)}ms`);
      }

      return result;
    };
  };
}

// 使用示例
const monitoredFunction = createPerformanceMonitor(500)(function slowFunction() {
  // 模拟耗时操作
  let sum = 0;
  for (let i = 0; i < 10000000; i++) {
    sum += i;
  }
  return sum;
});

monitoredFunction(); // 如果执行时间超过500ms，会输出警告
```

## 高级应用场景

### 1. 响应式数据流

```javascript
// 创建可观察对象
function createObservable(initialValue) {
  let value = initialValue;
  const subscribers = new Set();

  return {
    get value() {
      return value;
    },
    set value(newValue) {
      if (value !== newValue) {
        value = newValue;
        subscribers.forEach(subscriber => subscriber(value));
      }
    },
    subscribe(subscriber) {
      subscribers.add(subscriber);
      return () => subscribers.delete(subscriber);
    }
  };
}

// 创建计算属性
function createComputed(getter, dependencies) {
  let cachedValue = getter();
  let isDirty = false;

  dependencies.forEach(dep => {
    dep.subscribe(() => {
      isDirty = true;
    });
  });

  return {
    get value() {
      if (isDirty) {
        cachedValue = getter();
        isDirty = false;
      }
      return cachedValue;
    }
  };
}

// 使用示例
const firstName = createObservable('John');
const lastName = createObservable('Doe');

const fullName = createComputed(
  () => `${firstName.value} ${lastName.value}`,
  [firstName, lastName]
);

console.log(fullName.value); // 'John Doe'

firstName.value = 'Jane';
console.log(fullName.value); // 'Jane Doe'
```

### 2. 中间件模式

```javascript
// 创建中间件管道
function createPipeline(...middlewares) {
  return function(context) {
    const dispatch = (i) => {
      if (i >= middlewares.length) {
        return Promise.resolve();
      }

      return middlewares[i](context, () => dispatch(i + 1));
    };

    return dispatch(0);
  };
}

// 日志中间件
const loggerMiddleware = (context, next) => {
  console.log('Request:', context.request);
  return next().then(() => {
    console.log('Response:', context.response);
  });
};

// 认证中间件
const authMiddleware = (context, next) => {
  if (!context.user) {
    throw new Error('Unauthorized');
  }
  return next();
};

// 错误处理中间件
const errorHandlerMiddleware = (context, next) => {
  return next().catch(error => {
    context.error = error;
    console.error('Error:', error);
  });
};

// 使用示例
const pipeline = createPipeline(
  loggerMiddleware,
  authMiddleware,
  errorHandlerMiddleware
);

const context = {
  request: { url: '/api/users' },
  user: { id: 1, name: 'John' }
};

pipeline(context).then(() => {
  console.log('Pipeline completed');
});
```

### 3. 状态管理

```javascript
// 创建 Redux-like store
function createStore(reducer, initialState) {
  let state = initialState;
  const listeners = [];

  return {
    getState: () => state,
    dispatch: (action) => {
      state = reducer(state, action);
      listeners.forEach(listener => listener());
    },
    subscribe: (listener) => {
      listeners.push(listener);
      return () => {
        const index = listeners.indexOf(listener);
        listeners.splice(index, 1);
      };
    }
  };
}

// 创建 action creators
function createActionCreator(type) {
  return (payload) => ({ type, payload });
}

const increment = createActionCreator('INCREMENT');
const decrement = createActionCreator('DECREMENT');
const setValue = createActionCreator('SET_VALUE');

// 创建 reducer
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'SET_VALUE':
      return { count: action.payload };
    default:
      return state;
  }
}

// 使用示例
const store = createStore(counterReducer, { count: 0 });

store.subscribe(() => {
  console.log('Current state:', store.getState());
});

store.dispatch(increment());
store.dispatch(increment());
store.dispatch(setValue(10));
```

### 4. 异步流程控制

```javascript
// 创建并行执行器
function parallel(tasks) {
  return Promise.all(tasks.map(task => task()));
}

// 创建串行执行器
function series(tasks) {
  return tasks.reduce((promise, task) => {
    return promise.then(result => {
      return task().then(taskResult => [...result, taskResult]);
    });
  }, Promise.resolve([]));
}

// 创建重试函数
function retry(fn, options = {}) {
  const {
    maxAttempts = 3,
    delay = 1000,
    backoff = 2
  } = options;

  return async function(...args) {
    let lastError;

    for (let attempt = 1; attempt <= maxAttempts; attempt++) {
      try {
        return await fn(...args);
      } catch (error) {
        lastError = error;
        if (attempt < maxAttempts) {
          const waitTime = delay * Math.pow(backoff, attempt - 1);
          await new Promise(resolve => setTimeout(resolve, waitTime));
        }
      }
    }

    throw lastError;
  };
}

// 创建超时函数
function withTimeout(fn, timeout) {
  return function(...args) {
    return Promise.race([
      fn(...args),
      new Promise((_, reject) =>
        setTimeout(() => reject(new Error('Timeout')), timeout)
      )
    ]);
  };
}

// 使用示例
const tasks = [
  () => new Promise(resolve => setTimeout(() => resolve('Task 1'), 1000)),
  () => new Promise(resolve => setTimeout(() => resolve('Task 2'), 500)),
  () => new Promise(resolve => setTimeout(() => resolve('Task 3'), 1500))
];

// 并行执行
parallel(tasks).then(results => {
  console.log('Parallel results:', results);
});

// 串行执行
series(tasks).then(results => {
  console.log('Series results:', results);
});

// 带重试的异步操作
const fetchDataWithRetry = retry(
  async () => {
    const response = await fetch('/api/data');
    if (!response.ok) throw new Error('Network error');
    return response.json();
  },
  { maxAttempts: 3, delay: 1000 }
);

// 带超时的异步操作
const fetchDataWithTimeout = withTimeout(
  async () => {
    const response = await fetch('/api/data');
    return response.json();
  },
  5000
);
```

## 性能优化

### 1. 函数缓存

```javascript
// 创建缓存装饰器
function cache(fn, options = {}) {
  const cache = new Map();
  const { keyGenerator = (...args) => JSON.stringify(args), ttl } = options;

  return function(...args) {
    const key = keyGenerator(...args);
    const cached = cache.get(key);

    if (cached) {
      if (ttl && Date.now() - cached.timestamp > ttl) {
        cache.delete(key);
      } else {
        return cached.value;
      }
    }

    const result = fn.apply(this, args);
    cache.set(key, { value: result, timestamp: Date.now() });
    return result;
  };
}

// 使用示例
const expensiveCalculation = cache(
  (n) => {
    console.log('Calculating...');
    let sum = 0;
    for (let i = 0; i < n; i++) {
      sum += Math.sqrt(i);
    }
    return sum;
  },
  { ttl: 60000 } // 缓存1分钟
);

console.log(expensiveCalculation(1000)); // 计算并缓存
console.log(expensiveCalculation(1000)); // 从缓存获取
```

### 2. 惰性求值

```javascript
// 创建惰性求值函数
function lazy(fn) {
  let cachedValue = null;
  let isComputed = false;

  return function() {
    if (!isComputed) {
      cachedValue = fn();
      isComputed = true;
    }
    return cachedValue;
  };
}

// 使用示例
const heavyComputation = lazy(() => {
  console.log('Performing heavy computation...');
  let result = 0;
  for (let i = 0; i < 1000000; i++) {
    result += Math.random();
  }
  return result;
});

// 只有在调用时才会执行
console.log(heavyComputation()); // 第一次调用执行计算
console.log(heavyComputation()); // 后续调用返回缓存值
```

### 3. 批处理

```javascript
// 创建批处理器
function createBatchProcessor(processor, options = {}) {
  const {
    batchSize = 10,
    delay = 100
  } = options;

  let batch = [];
  let timeoutId = null;

  function flush() {
    if (batch.length === 0) return;

    const currentBatch = batch;
    batch = [];

    return processor(currentBatch);
  }

  return function(item) {
    batch.push(item);

    if (batch.length >= batchSize) {
      clearTimeout(timeoutId);
      return flush();
    }

    clearTimeout(timeoutId);
    timeoutId = setTimeout(flush, delay);
  };
}

// 使用示例
const saveToDatabase = async (items) => {
  console.log('Saving batch to database:', items.length, 'items');
  // 模拟数据库保存
  await new Promise(resolve => setTimeout(resolve, 100));
  return { success: true };
};

const batchSaver = createBatchProcessor(saveToDatabase, {
  batchSize: 5,
  delay: 1000
});

// 模拟添加数据
for (let i = 0; i < 15; i++) {
  batchSaver({ id: i, data: `Item ${i}` });
}
```

## 注意事项

### 1. 性能考虑

- **避免过度嵌套**：过深的函数嵌套会影响性能和可读性
- **内存泄漏**：注意闭包中引用的变量，避免意外的内存占用
- **this 绑定**：箭头函数不会绑定自己的 this，在类方法中使用时要注意

```javascript
// 性能不佳的嵌套
const bad = data
  .filter(item => item.active)
  .map(item => item.value)
  .filter(value => value > 10)
  .map(value => value * 2);

// 性能更好的写法
const good = data
  .filter(item => item.active && item.value > 10)
  .map(item => item.value * 2);
```

### 2. 错误处理

```javascript
// 创建安全的函数组合
function safeCompose(...functions) {
  return function(x) {
    try {
      return functions.reduceRight((acc, fn) => fn(acc), x);
    } catch (error) {
      console.error('Composition error:', error);
      return null;
    }
  };
}

// 创建带错误处理的包装器
function withErrorHandler(fn, errorHandler) {
  return function(...args) {
    try {
      return fn(...args);
    } catch (error) {
      return errorHandler(error, ...args);
    }
  };
}

// 使用示例
const safeDivide = withErrorHandler(
  (a, b) => a / b,
  (error) => {
    console.error('Division error:', error.message);
    return 0;
  }
);

console.log(safeDivide(10, 2)); // 5
console.log(safeDivide(10, 0)); // 0 (错误被捕获)
```

### 3. 类型安全

```javascript
// 创建类型检查装饰器
function typeCheck(fn, types) {
  return function(...args) {
    args.forEach((arg, index) => {
      const expectedType = types[index];
      if (expectedType && typeof arg !== expectedType) {
        throw new TypeError(
          `Argument ${index} should be ${expectedType}, got ${typeof arg}`
        );
      }
    });

    return fn(...args);
  };
}

// 使用示例
const addNumbers = typeCheck(
  (a, b) => a + b,
  ['number', 'number']
);

console.log(addNumbers(2, 3)); // 5
console.log(addNumbers('2', '3')); // TypeError
```

## 总结

高阶函数是 JavaScript 函数式编程的强大工具，能够帮助我们：

1. **编写更简洁的代码**：用声明式的方式描述业务逻辑
2. **提高代码可重用性**：通过函数参数实现通用逻辑
3. **增强代码可维护性**：纯函数和不可变性让代码更易于理解和修改
4. **实现复杂逻辑**：通过函数组合构建复杂的数据处理流程

### 学习路径建议：

1. **基础掌握**：理解高阶函数的概念和基本用法
2. **练习组合**：掌握函数组合、柯里化等技巧
3. **实际应用**：在真实项目中应用高阶函数解决实际问题
4. **性能优化**：了解高阶函数的性能特性和优化方法
5. **深入学习**：探索函数式编程的更多概念和模式

### 常用高阶函数速查：

| 函数类型 | 常用函数 | 用途 |
|---------|---------|------|
| 数组方法 | map, filter, reduce | 数据转换和处理 |
| 函数组合 | compose, pipe | 函数组合和复用 |
| 函数增强 | debounce, throttle | 性能优化 |
| 函数缓存 | memoize, cache | 性能优化 |
| 异步控制 | retry, withTimeout | 异步流程控制 |

通过持续练习和实践，你将能够熟练运用高阶函数，编写出更加优雅、高效的 JavaScript 代码。高阶函数不仅是技术工具，更是一种思维方式，它能帮助你更好地组织和表达复杂的业务逻辑。