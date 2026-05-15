---
title: JavaScript 装饰器模式
published: 2022-10-02
description: '装饰器的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

装饰器模式（Decorator Pattern）是一种结构型设计模式，它允许在不改变对象结构的情况下，动态地给对象添加新的功能。在 JavaScript 中，装饰器模式可以通过函数组合、类装饰器或高阶函数来实现。

装饰器模式的核心思想是：**通过包装对象来扩展其功能，而不是通过继承**。

## 核心概念

### 装饰器模式的优点

1. **开放封闭原则**：对扩展开放，对修改封闭
2. **单一职责原则**：每个装饰器只关注一个功能
3. **避免类爆炸**：不需要创建大量子类来实现不同功能组合
4. **动态组合**：可以在运行时灵活组合不同功能

### JavaScript 中的装饰器

在 JavaScript 中，装饰器主要有以下几种实现方式：
- 函数装饰器
- 类装饰器
- 方法装饰器
- 属性装饰器

## 基本用法

### 函数装饰器

```javascript
// 基础函数装饰器
function withLogging(fn) {
  return function(...args) {
    console.log(`调用函数 ${fn.name}，参数:`, args);
    const result = fn.apply(this, args);
    console.log(`函数 ${fn.name} 返回结果:`, result);
    return result;
  };
}

// 使用装饰器
function add(a, b) {
  return a + b;
}

const loggedAdd = withLogging(add);
console.log(loggedAdd(3, 5));
// 输出：
// 调用函数 add，参数: [3, 5]
// 函数 add 返回结果: 8
// 8
```

### 类装饰器

```javascript
// 类装饰器
function withTimestamp(Class) {
  return class extends Class {
    constructor(...args) {
      super(...args);
      this.createdAt = new Date();
    }

    getCreationTime() {
      return this.createdAt.toISOString();
    }
  };
}

@withTimestamp
class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
}

const user = new User('Alice', 'alice@example.com');
console.log(user.getCreationTime());
```

### 方法装饰器

```javascript
// 方法装饰器 - 性能监控
function measurePerformance(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function(...args) {
    const start = performance.now();
    const result = originalMethod.apply(this, args);
    const end = performance.now();

    console.log(`${propertyKey} 执行时间: ${(end - start).toFixed(2)}ms`);
    return result;
  };

  return descriptor;
}

class Calculator {
  @measurePerformance
  factorial(n) {
    if (n <= 1) return 1;
    return n * this.factorial(n - 1);
  }

  @measurePerformance
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
}

const calc = new Calculator();
console.log(calc.factorial(10));
console.log(calc.fibonacci(30));
```

### 属性装饰器

```javascript
// 属性装饰器 - 只读属性
function readonly(target, propertyKey) {
  Object.defineProperty(target, propertyKey, {
    writable: false,
    configurable: false
  });
}

class Person {
  @readonly
  id = Math.random().toString(36).substr(2, 9);

  constructor(name) {
    this.name = name;
  }
}

const person = new Person('Alice');
person.id = 'new-id'; // TypeError: Cannot assign to read only property 'id'
```

## 实际应用

### API 请求装饰器

```javascript
// 请求缓存装饰器
function withCache(ttl = 60000) {
  const cache = new Map();

  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function(...args) {
      const cacheKey = JSON.stringify(args);

      if (cache.has(cacheKey)) {
        const { data, timestamp } = cache.get(cacheKey);
        if (Date.now() - timestamp < ttl) {
          console.log('从缓存获取数据');
          return data;
        }
      }

      const result = await originalMethod.apply(this, args);
      cache.set(cacheKey, {
        data: result,
        timestamp: Date.now()
      });

      return result;
    };

    return descriptor;
  };
}

// 错误重试装饰器
function withRetry(maxRetries = 3, delay = 1000) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = async function(...args) {
      let lastError;

      for (let i = 0; i < maxRetries; i++) {
        try {
          return await originalMethod.apply(this, args);
        } catch (error) {
          lastError = error;
          console.log(`尝试 ${i + 1}/${maxRetries} 失败，${delay}ms 后重试...`);
          await new Promise(resolve => setTimeout(resolve, delay));
        }
      }

      throw lastError;
    };

    return descriptor;
  };
}

class APIClient {
  @withCache()
  @withRetry(3, 1000)
  async fetchUser(id) {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error('请求失败');
    return response.json();
  }
}
```

### 表单验证装饰器

```javascript
// 验证装饰器工厂
function validate(validator, errorMessage) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args) {
      if (!validator(...args)) {
        throw new Error(errorMessage);
      }
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class FormHandler {
  @validate(
    email => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email),
    '邮箱格式不正确'
  )
  @validate(
    email => email.length <= 50,
    '邮箱长度不能超过 50 个字符'
  )
  setEmail(email) {
    this.email = email;
    console.log('邮箱设置成功:', email);
  }

  @validate(
    password => password.length >= 8,
    '密码长度不能少于 8 个字符'
  )
  @validate(
    password => /[A-Z]/.test(password),
    '密码必须包含大写字母'
  )
  setPassword(password) {
    this.password = password;
    console.log('密码设置成功');
  }
}

const form = new FormHandler();
try {
  form.setEmail('valid@email.com');
  form.setPassword('Password123');
} catch (error) {
  console.error(error.message);
}
```

### 权限控制装饰器

```javascript
// 权限装饰器
function requirePermission(permission) {
  return function(target, propertyKey, descriptor) {
    const originalMethod = descriptor.value;

    descriptor.value = function(...args) {
      if (!this.permissions || !this.permissions.includes(permission)) {
        throw new Error(`缺少权限: ${permission}`);
      }
      return originalMethod.apply(this, args);
    };

    return descriptor;
  };
}

class User {
  constructor(permissions) {
    this.permissions = permissions;
  }

  @requirePermission('read')
  readData() {
    return '数据内容';
  }

  @requirePermission('write')
  writeData(data) {
    console.log('写入数据:', data);
  }

  @requirePermission('admin')
  deleteUser(id) {
    console.log('删除用户:', id);
  }
}

const admin = new User(['read', 'write', 'admin']);
const user = new User(['read']);

admin.readData();      // 成功
admin.writeData('test'); // 成功
admin.deleteUser(1);    // 成功

user.readData();       // 成功
user.writeData('test'); // 抛出错误
```

### 日志记录装饰器

```javascript
// 日志装饰器
function log(target, propertyKey, descriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = async function(...args) {
    const startTime = Date.now();
    console.log(`[${new Date().toISOString()}] 开始执行 ${propertyKey}`);

    try {
      const result = await originalMethod.apply(this, args);
      const duration = Date.now() - startTime;
      console.log(`[${new Date().toISOString()}] ${propertyKey} 执行成功，耗时 ${duration}ms`);
      return result;
    } catch (error) {
      const duration = Date.now() - startTime;
      console.error(`[${new Date().toISOString()}] ${propertyKey} 执行失败，耗时 ${duration}ms，错误:`, error.message);
      throw error;
    }
  };

  return descriptor;
}

class UserService {
  @log
  async getUser(id) {
    // 模拟 API 调用
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id, name: 'Alice' };
  }

  @log
  async updateUser(id, data) {
    // 模拟更新操作
    await new Promise(resolve => setTimeout(resolve, 50));
    return { ...data, id };
  }
}
```

## 注意事项

1. **装饰器顺序**：装饰器的执行顺序从下到上（靠近方法的是第一个执行）
2. **性能影响**：每个装饰器都会创建新的函数，注意不要过度使用
3. **this 绑定**：在装饰器中要注意 this 的正确绑定
4. **错误处理**：装饰器中要妥善处理异常，避免吞掉错误
5. **调试难度**：过多的装饰器会增加调试难度，保持装饰器简单明了

## 总结

装饰器模式是 JavaScript 中实现横切关注点的优雅方案，它能够：

- **关注点分离**：将日志、验证、缓存等横切关注点从业务逻辑中分离
- **代码复用**：通过组合装饰器实现功能复用
- **灵活扩展**：动态地添加或移除功能，无需修改原有代码
- **声明式编程**：使用装饰器使代码更加声明式和可读

合理使用装饰器模式，可以让你的代码更加清晰、可维护，同时保持高度的灵活性。