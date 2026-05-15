---
title: JavaScript 设计模式之代理
published: 2022-09-13
description: '代理模式的实现和应用的详细介绍和学习笔记'
image: ''
tags: ["设计模式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

代理模式(Proxy Pattern)是一种结构型设计模式，它为其他对象提供一种代理，以控制对这个对象的访问。代理对象在客户端和目标对象之间起到中介的作用，可以在不修改目标对象代码的情况下，扩展目标对象的功能。

在 JavaScript 中，ES6 引入了 Proxy 对象，为我们提供了一种更优雅和强大的方式来实现代理模式。

## 核心概念

代理模式主要包含以下几个核心概念：

1. **Subject（抽象主题角色）**：声明了真实主题和代理主题的共同接口
2. **RealSubject（真实主题角色）**：定义了代理所代表的真实对象
3. **Proxy（代理主题角色）**：持有对真实主题的引用，可以在调用真实主题前后添加额外操作

**代理模式的优点：**
- 职责清晰：真实对象只需关注业务逻辑，代理对象关注访问控制
- 扩展性强：可以在不修改真实对象的情况下扩展功能
- 控制访问：可以控制客户端对真实对象的访问权限
- 智能引用：可以智能地管理真实对象的生命周期

**代理模式的缺点：**
- 增加了系统的复杂度
- 可能会影响系统的响应速度
- 需要额外的代码维护

## 基本用法

### 1. 基础代理模式

```javascript
// 真实对象
class RealSubject {
  request() {
    console.log('处理真实对象的请求');
    return '真实对象的结果';
  }
}

// 代理对象
class ProxySubject {
  constructor(realSubject) {
    this.realSubject = realSubject;
  }

  request() {
    console.log('代理对象：在请求前进行预处理');
    const result = this.realSubject.request();
    console.log('代理对象：在请求后进行后处理');
    return result;
  }
}

// 使用示例
const realSubject = new RealSubject();
const proxy = new ProxySubject(realSubject);

proxy.request();
// 输出:
// 代理对象：在请求前进行预处理
// 处理真实对象的请求
// 代理对象：在请求后进行后处理
```

### 2. 使用 ES6 Proxy 实现代理模式

```javascript
// 目标对象
const target = {
  name: '张三',
  age: 25,
  sayHello() {
    console.log(`你好，我是 ${this.name}`);
  }
};

// 创建代理
const proxy = new Proxy(target, {
  // 拦截属性读取
  get(target, property, receiver) {
    console.log(`读取属性: ${property}`);
    return Reflect.get(target, property, receiver);
  },
  // 拦截属性设置
  set(target, property, value, receiver) {
    console.log(`设置属性: ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  },
  // 拦截函数调用
  apply(target, thisArg, args) {
    console.log('调用函数');
    return Reflect.apply(target, thisArg, args);
  }
});

// 使用示例
console.log(proxy.name); // 读取属性: name
proxy.age = 26; // 设置属性: age = 26
proxy.sayHello(); // 调用函数
```

### 3. 数据验证代理

```javascript
// 创建数据验证代理
function createValidationProxy(target, schema) {
  return new Proxy(target, {
    set(target, property, value) {
      const validator = schema[property];

      if (validator) {
        const error = validator(value);
        if (error) {
          throw new Error(`属性 ${property} 验证失败: ${error}`);
        }
      }

      target[property] = value;
      return true;
    }
  });
}

// 使用示例
const user = {};
const userSchema = {
  name: (value) => {
    if (typeof value !== 'string') return '必须是字符串';
    if (value.length < 2) return '长度不能小于2';
    return null;
  },
  age: (value) => {
    if (typeof value !== 'number') return '必须是数字';
    if (value < 0 || value > 150) return '年龄必须在0-150之间';
    return null;
  },
  email: (value) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) return '邮箱格式不正确';
    return null;
  }
};

const validatedUser = createValidationProxy(user, userSchema);

validatedUser.name = '李四'; // ✓
validatedUser.age = 30; // ✓
validatedUser.email = 'test@example.com'; // ✓

try {
  validatedUser.name = 'A'; // ✗
} catch (error) {
  console.log(error.message); // 属性 name 验证失败: 长度不能小于2
}
```

## 实际应用

### 1. 虚拟代理（延迟加载）

```javascript
// 图片加载器
class ImageLoader {
  constructor(url) {
    this.url = url;
    this.image = null;
  }

  load() {
    if (!this.image) {
      console.log(`正在加载图片: ${this.url}`);
      this.image = new Image();
      this.image.src = this.url;
    }
    return this.image;
  }

  show() {
    if (this.image && this.image.complete) {
      console.log('显示图片');
      document.body.appendChild(this.image);
    } else {
      console.log('图片尚未加载完成');
    }
  }
}

// 图片代理（虚拟代理）
class ImageProxy {
  constructor(url) {
    this.url = url;
    this.loader = null;
  }

  load() {
    if (!this.loader) {
      this.loader = new ImageLoader(this.url);
    }
    return this.loader.load();
  }

  show() {
    this.load();
    // 延迟显示，模拟网络延迟
    setTimeout(() => {
      this.loader.show();
    }, 1000);
  }
}

// 使用示例
const imageProxy = new ImageProxy('https://example.com/image.jpg');
imageProxy.show(); // 图片会在1秒后显示
```

### 2. 缓存代理

```javascript
// 计算函数（耗时的计算）
function calculateFibonacci(n) {
  console.log(`计算 Fibonacci(${n})`);
  if (n <= 1) return n;
  return calculateFibonacci(n - 1) + calculateFibonacci(n - 2);
}

// 缓存代理
function createCacheProxy(fn) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log(`从缓存中获取: ${args}`);
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// 使用示例
const cachedFibonacci = createCacheProxy(calculateFibonacci);

console.log(cachedFibonacci(10)); // 执行计算
console.log(cachedFibonacci(10)); // 从缓存获取
console.log(cachedFibonacci(8)); // 执行计算
console.log(cachedFibonacci(10)); // 从缓存获取
```

### 3. 访问控制代理

```javascript
// 权限管理
class PermissionManager {
  constructor() {
    this.permissions = new Set();
  }

  hasPermission(permission) {
    return this.permissions.has(permission);
  }

  grantPermission(permission) {
    this.permissions.add(permission);
  }

  revokePermission(permission) {
    this.permissions.delete(permission);
  }
}

// 用户系统
class UserSystem {
  constructor() {
    this.users = new Map();
  }

  addUser(id, userData) {
    this.users.set(id, userData);
    console.log(`添加用户: ${id}`);
  }

  deleteUser(id) {
    this.users.delete(id);
    console.log(`删除用户: ${id}`);
  }

  getUser(id) {
    return this.users.get(id);
  }
}

// 访问控制代理
class AccessControlProxy {
  constructor(userSystem, permissionManager) {
    this.userSystem = userSystem;
    this.permissionManager = permissionManager;
  }

  addUser(id, userData) {
    if (this.permissionManager.hasPermission('user:write')) {
      this.userSystem.addUser(id, userData);
    } else {
      throw new Error('没有添加用户的权限');
    }
  }

  deleteUser(id) {
    if (this.permissionManager.hasPermission('user:delete')) {
      this.userSystem.deleteUser(id);
    } else {
      throw new Error('没有删除用户的权限');
    }
  }

  getUser(id) {
    if (this.permissionManager.hasPermission('user:read')) {
      return this.userSystem.getUser(id);
    } else {
      throw new Error('没有查看用户的权限');
    }
  }
}

// 使用示例
const permissionManager = new PermissionManager();
permissionManager.grantPermission('user:read');
permissionManager.grantPermission('user:write');

const userSystem = new UserSystem();
const proxy = new AccessControlProxy(userSystem, permissionManager);

proxy.addUser('1', { name: '张三', age: 25 }); // ✓
console.log(proxy.getUser('1')); // { name: '张三', age: 25 }

try {
  proxy.deleteUser('1'); // ✗
} catch (error) {
  console.log(error.message); // 没有删除用户的权限
}
```

### 4. 防抖和节流代理

```javascript
// 防抖代理
function createDebounceProxy(fn, delay = 300) {
  let timer = null;

  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// 节流代理
function createThrottleProxy(fn, delay = 300) {
  let lastCall = 0;

  return function(...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}

// 使用示例
const searchHandler = (query) => {
  console.log(`搜索: ${query}`);
};

const debouncedSearch = createDebounceProxy(searchHandler, 500);
const throttledScroll = createThrottleProxy(() => {
  console.log('滚动事件处理');
}, 100);

// 模拟快速输入
debouncedSearch('a');
debouncedSearch('ab');
debouncedSearch('abc');
debouncedSearch('abcd');
// 只会执行最后一次 'abcd' 的搜索
```

### 5. 日志记录代理

```javascript
// 日志记录代理
function createLoggingProxy(target, logName = 'Proxy') {
  const logger = {
    log(...args) {
      console.log(`[${logName}]`, ...args);
    },
    error(...args) {
      console.error(`[${logName}]`, ...args);
    }
  };

  return new Proxy(target, {
    get(target, property) {
      const value = target[property];

      if (typeof value === 'function') {
        return function(...args) {
          logger.log(`调用方法: ${property}`, '参数:', args);
          const startTime = Date.now();

          try {
            const result = value.apply(target, args);
            const endTime = Date.now();
            logger.log(`方法 ${property} 执行成功`, '耗时:', endTime - startTime + 'ms');
            return result;
          } catch (error) {
            logger.error(`方法 ${property} 执行失败:`, error.message);
            throw error;
          }
        };
      }

      logger.log(`读取属性: ${property}`);
      return value;
    },

    set(target, property, value) {
      logger.log(`设置属性: ${property} =`, value);
      target[property] = value;
      return true;
    }
  });
}

// 使用示例
const api = {
  async getUser(id) {
    await new Promise(resolve => setTimeout(resolve, 100));
    return { id, name: '张三' };
  },

  async updateUser(id, data) {
    await new Promise(resolve => setTimeout(resolve, 150));
    return { ...data, id };
  }
};

const loggedApi = createLoggingProxy(api, 'API');

(async () => {
  await loggedApi.getUser(1);
  await loggedApi.updateUser(1, { name: '李四' });
})();
```

## 注意事项

1. **代理透明性**：代理对象应该尽可能保持对客户端的透明性，不要改变真实对象的接口
2. **性能开销**：代理模式会引入额外的对象和方法调用，需要考虑性能开销
3. **内存管理**：代理对象可能持有对真实对象的引用，需要注意内存泄漏问题
4. **过度代理**：不要为所有对象都创建代理，根据实际需求选择合适的场景
5. **错误处理**：代理对象需要妥善处理真实对象可能抛出的异常
6. **线程安全**：在多线程环境下，需要注意代理对象的线程安全问题
7. **调试难度**：代理模式可能会增加调试的难度，需要提供足够的日志信息

## 总结

代理模式是一种非常实用的设计模式，它通过引入代理对象来控制对真实对象的访问，从而在不修改真实对象的情况下扩展其功能。

**ES6 Proxy 的强大之处：**
- 支持多种拦截操作：get、set、apply、has、deleteProperty 等
- 可以拦截对象的各种操作，提供细粒度的控制
- 结合 Reflect API，可以更灵活地操作目标对象
- 支持撤销代理：Proxy.revocable()

**代理模式的最佳实践：**
- 根据业务需求选择合适的代理类型（虚拟代理、缓存代理、保护代理等）
- 保持代理对象的简单性，避免在代理中包含过多的业务逻辑
- 使用 Proxy 时要考虑浏览器兼容性问题
- 对于简单的拦截场景，可以考虑使用 getter/setter 替代 Proxy
- 代理对象应该保持与真实对象相同的接口

通过合理使用代理模式，我们可以在不修改原始代码的情况下，为系统添加诸如权限控制、缓存、日志记录、延迟加载等功能，大大提高了代码的可维护性和扩展性。