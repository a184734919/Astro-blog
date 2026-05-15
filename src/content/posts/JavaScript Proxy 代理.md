---
title: JavaScript Proxy 代理
published: 2022-06-09
description: 'Proxy 的基本用法和应用场景的详细介绍和学习笔记'
image: ''
tags: ["JS","代理"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

Proxy（代理）是 ES6 引入的一种新特性，用于定义基本操作的自定义行为（如属性查找、赋值、枚举、函数调用等）。Proxy 可以理解为在目标对象之前架设一层"拦截"，外界对该对象的访问，都必须先通过这层拦截，因此提供了一种机制，可以对外界的访问进行过滤和改写。

## Proxy 基本概念

### 创建 Proxy

Proxy 对象由两个部分组成：
- **target**：要代理的目标对象
- **handler**：一个对象，定义拦截行为

```javascript
const proxy = new Proxy(target, handler);
```

## 基本用法

### 简单示例

```javascript
const target = {
  name: '张三',
  age: 25
};

const handler = {
  get(target, property, receiver) {
    console.log(`访问属性: ${property}`);
    return target[property];
  },
  set(target, property, value, receiver) {
    console.log(`设置属性 ${property} = ${value}`);
    target[property] = value;
    return true;
  }
};

const proxy = new Proxy(target, handler);

proxy.name; // 访问属性: name
proxy.age = 26; // 设置属性 age = 26
```

## Proxy 拦截器方法

### 1. get(target, property, receiver)

拦截对象属性的读取操作。

```javascript
const handler = {
  get(target, property) {
    if (property in target) {
      return target[property];
    } else {
      throw new ReferenceError(`属性 "${property}" 不存在`);
    }
  }
};

const user = new Proxy(
  { name: '张三', age: 25 },
  handler
);

console.log(user.name); // '张三'
console.log(user.address); // 抛出 ReferenceError

// 实现负索引访问数组
function createArray(elements) {
  const handler = {
    get(target, property, receiver) {
      const index = Number(property);

      if (index < 0) {
        property = String(target.length + index);
      }

      return Reflect.get(target, property, receiver);
    }
  };

  return new Proxy(elements, handler);
}

const array = createArray([1, 2, 3]);
console.log(array[-1]); // 3
console.log(array[-2]); // 2
```

### 2. set(target, property, value, receiver)

拦截对象属性的赋值操作。

```javascript
const handler = {
  set(target, property, value) {
    if (property === 'age' && typeof value !== 'number') {
      throw new TypeError('年龄必须是数字');
    }

    if (property === 'age' && (value < 0 || value > 120)) {
      throw new RangeError('年龄必须在 0-120 之间');
    }

    target[property] = value;
    return true;
  }
};

const user = new Proxy({}, handler);

user.name = '李四';
user.age = 30;

// user.age = -5; // 抛出 RangeError
// user.age = '三十'; // 抛出 TypeError

// 实现数据验证
function createValidator(target, schema) {
  return new Proxy(target, {
    set(target, property, value) {
      const validator = schema[property];

      if (validator && !validator.validate(value)) {
        throw new Error(validator.message);
      }

      target[property] = value;
      return true;
    }
  });
}

const person = createValidator({}, {
  name: {
    validate: (val) => typeof val === 'string' && val.length >= 2,
    message: '姓名必须是至少 2 个字符的字符串'
  },
  age: {
    validate: (val) => typeof val === 'number' && val >= 0,
    message: '年龄必须是正数'
  }
});

person.name = '王';
person.age = 25;
```

### 3. has(target, property)

拦截 `in` 操作符的判断。

```javascript
const handler = {
  has(target, property) {
    if (property.startsWith('_')) {
      return false; // 隐藏私有属性
    }
    return property in target;
  }
};

const user = new Proxy(
  { name: '张三', _password: '123456' },
  handler
);

console.log('name' in user); // true
console.log('_password' in user); // false
```

### 4. deleteProperty(target, property)

拦截 `delete` 操作符的删除操作。

```javascript
const handler = {
  deleteProperty(target, property) {
    if (property.startsWith('_')) {
      throw new Error('不能删除私有属性');
    }
    delete target[property];
    return true;
  }
};

const user = new Proxy(
  { name: '张三', _id: 123 },
  handler
);

delete user.name; // 成功
// delete user._id; // 抛出错误
```

### 5. ownKeys(target)

拦截 `Object.getOwnPropertyNames()`、`Object.getOwnPropertySymbols()`、`Object.keys()` 等操作。

```javascript
const handler = {
  ownKeys(target) {
    return Object.keys(target).filter(key => !key.startsWith('_'));
  }
};

const user = new Proxy(
  { name: '张三', _password: '123456', age: 25 },
  handler
);

console.log(Object.keys(user)); // ['name', 'age']
console.log(Object.getOwnPropertyNames(user)); // ['name', 'age']
```

### 6. getOwnPropertyDescriptor(target, property)

拦截 `Object.getOwnPropertyDescriptor()` 操作。

```javascript
const handler = {
  getOwnPropertyDescriptor(target, property) {
    const descriptor = Object.getOwnPropertyDescriptor(target, property);

    if (descriptor && property.startsWith('_')) {
      descriptor.enumerable = false; // 隐藏私有属性
    }

    return descriptor;
  }
};

const user = new Proxy(
  { name: '张三', _password: '123456' },
  handler
);

console.log(Object.getOwnPropertyDescriptor(user, 'name'));
console.log(Object.getOwnPropertyDescriptor(user, '_password'));
```

### 7. defineProperty(target, property, descriptor)

拦截 `Object.defineProperty()` 操作。

```javascript
const handler = {
  defineProperty(target, property, descriptor) {
    if (property.startsWith('_')) {
      throw new Error('不能定义私有属性');
    }
    Object.defineProperty(target, property, descriptor);
    return true;
  }
};

const user = new Proxy({}, handler);

Object.defineProperty(user, 'name', { value: '张三' });
// Object.defineProperty(user, '_password', { value: '123456' }); // 抛出错误
```

### 8. getPrototypeOf(target)

拦截 `Object.getPrototypeOf()` 操作。

```javascript
const handler = {
  getPrototypeOf(target) {
    console.log('获取原型');
    return Object.getPrototypeOf(target);
  }
};

const user = new Proxy({}, handler);
Object.getPrototypeOf(user); // 输出: 获取原型
```

### 9. setPrototypeOf(target, prototype)

拦截 `Object.setPrototypeOf()` 操作。

```javascript
const handler = {
  setPrototypeOf(target, prototype) {
    console.log('设置原型');
    return Object.setPrototypeOf(target, prototype);
  }
};

const user = new Proxy({}, handler);
Object.setPrototypeOf(user, null); // 输出: 设置原型
```

### 10. apply(target, thisArg, argumentsList)

拦截函数调用操作。

```javascript
function sum(a, b) {
  return a + b;
}

const handler = {
  apply(target, thisArg, argumentsList) {
    console.log(`调用函数 ${target.name}`);
    console.log(`参数: ${argumentsList.join(', ')}`);

    const result = Reflect.apply(target, thisArg, argumentsList);

    console.log(`结果: ${result}`);
    return result;
  }
};

const proxySum = new Proxy(sum, handler);
proxySum(2, 3);
// 调用函数 sum
// 参数: 2, 3
// 结果: 5

// 实现函数调用次数统计
function countCalls(fn) {
  let count = 0;

  return new Proxy(fn, {
    apply(target, thisArg, args) {
      count++;
      console.log(`函数 ${target.name} 已被调用 ${count} 次`);
      return Reflect.apply(target, thisArg, args);
    }
  });
}

const myFunction = countCalls(function(x) {
  return x * 2;
});

myFunction(5); // 函数 myFunction 已被调用 1 次
myFunction(10); // 函数 myFunction 已被调用 2 次
```

### 11. construct(target, argumentsList, newTarget)

拦截 `new` 操作符的调用。

```javascript
function User(name, age) {
  this.name = name;
  this.age = age;
}

const handler = {
  construct(target, args, newTarget) {
    console.log(`创建 ${target.name} 实例`);

    // 可以修改构造函数行为
    if (args.length < 2) {
      throw new Error('User 需要 name 和 age 参数');
    }

    return Reflect.construct(target, args, newTarget);
  }
};

const UserProxy = new Proxy(User, handler);

const user1 = new UserProxy('张三', 25);
console.log(user1); // User { name: '张三', age: 25 }

// const user2 = new UserProxy('李四'); // 抛出错误

// 实现单例模式
function createSingleton(Class) {
  let instance;

  return new Proxy(Class, {
    construct(target, args, newTarget) {
      if (!instance) {
        instance = Reflect.construct(target, args, newTarget);
      }
      return instance;
    }
  });
}

class Database {
  constructor() {
    console.log('创建数据库连接');
  }
}

const SingletonDatabase = createSingleton(Database);

const db1 = new SingletonDatabase(); // 创建数据库连接
const db2 = new SingletonDatabase(); // 不再创建
console.log(db1 === db2); // true
```

## 实际应用场景

### 1. 数据绑定和响应式

```javascript
// 简单的响应式系统
function reactive(obj, callback) {
  return new Proxy(obj, {
    set(target, property, value) {
      const oldValue = target[property];
      target[property] = value;

      if (oldValue !== value) {
        callback(property, value, oldValue);
      }

      return true;
    }
  });
}

const state = reactive(
  { count: 0, name: '张三' },
  (property, newValue, oldValue) => {
    console.log(`${property} 从 ${oldValue} 变为 ${newValue}`);
  }
);

state.count = 1; // count 从 0 变为 1
state.name = '李四'; // name 从 张三 变为 李四
```

### 2. 防抖和节流

```javascript
// 防抖函数
function debounce(fn, delay) {
  let timer;

  return new Proxy(fn, {
    apply(target, thisArg, args) {
      clearTimeout(timer);
      timer = setTimeout(() => {
        Reflect.apply(target, thisArg, args);
      }, delay);
    }
  });
}

const debouncedSearch = debounce(function(query) {
  console.log('搜索:', query);
}, 300);

debouncedSearch('j');
debouncedSearch('ja');
debouncedSearch('jav'); // 只有这个会执行

// 节流函数
function throttle(fn, interval) {
  let lastTime = 0;

  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const now = Date.now();

      if (now - lastTime >= interval) {
        lastTime = now;
        Reflect.apply(target, thisArg, args);
      }
    }
  });
}

const throttledScroll = throttle(function() {
  console.log('滚动事件触发');
}, 100);

// 模拟多次滚动
for (let i = 0; i < 10; i++) {
  throttledScroll();
}
```

### 3. 属性访问日志

```javascript
function createLogger(target, name = 'Object') {
  return new Proxy(target, {
    get(target, property) {
      console.log(`[${name}] 读取属性: ${property}`);
      return target[property];
    },
    set(target, property, value) {
      console.log(`[${name}] 设置属性 ${property} = ${value}`);
      target[property] = value;
      return true;
    }
  });
}

const user = createLogger({ name: '张三', age: 25 }, 'User');

user.name; // [User] 读取属性: name
user.age = 26; // [User] 设置属性 age = 26
```

### 4. 私有属性模拟

```javascript
function createPrivateProps(target) {
  const privateProps = new WeakMap();

  return new Proxy(target, {
    get(target, property) {
      const privateValue = privateProps.get(target)?.[property];
      if (privateValue !== undefined) {
        return privateValue;
      }
      return target[property];
    },
    set(target, property, value) {
      if (property.startsWith('_')) {
        let props = privateProps.get(target);
        if (!props) {
          props = {};
          privateProps.set(target, props);
        }
        props[property] = value;
        return true;
      }
      target[property] = value;
      return true;
    }
  });
}

class User {
  constructor(name) {
    this.name = name;
  }

  getName() {
    return this.name;
  }
}

const PrivateUser = createPrivateProps(User.prototype);

const user = new PrivateUser('张三');
user._password = '123456';

console.log(user.name); // '张三'
console.log(user._password); // '123456'
console.log(Object.keys(user)); // ['name'] - 私有属性不可枚举
```

### 5. API 请求拦截器

```javascript
// 创建 API 客户端
function createAPIClient(baseURL) {
  const client = {
    baseURL,
    token: null
  };

  return new Proxy(client, {
    get(target, property) {
      if (property in target) {
        return target[property];
      }

      // 动态创建 API 方法
      return async function(...args) {
        const url = `${target.baseURL}/${property}`;
        const options = args[0] || {};

        // 添加认证 token
        if (target.token) {
          options.headers = {
            ...options.headers,
            'Authorization': `Bearer ${target.token}`
          };
        }

        console.log(`请求 ${options.method || 'GET'} ${url}`);

        const response = await fetch(url, options);

        if (!response.ok) {
          throw new Error(`HTTP 错误: ${response.status}`);
        }

        return response.json();
      };
    }
  });
}

const api = createAPIClient('https://api.example.com');

api.token = 'your-token-here';

// 动态调用 API
api.users(); // GET https://api.example.com/users
api.posts({ method: 'POST', body: JSON.stringify({ title: 'Hello' }) });
// POST https://api.example.com/posts
```

### 6. 缓存代理

```javascript
function createCache(target) {
  const cache = new Map();

  return new Proxy(target, {
    get(target, property) {
      if (typeof target[property] === 'function') {
        return function(...args) {
          const key = `${property}:${JSON.stringify(args)}`;

          if (cache.has(key)) {
            console.log(`从缓存返回: ${key}`);
            return cache.get(key);
          }

          console.log(`执行函数: ${key}`);
          const result = target[property].apply(target, args);
          cache.set(key, result);
          return result;
        };
      }

      return target[property];
    }
  });
}

const fibonacci = {
  calculate(n) {
    if (n <= 1) return n;
    return this.calculate(n - 1) + this.calculate(n - 2);
  }
};

const cachedFibonacci = createCache(fibonacci);

console.log(cachedFibonacci.calculate(40)); // 第一次计算
console.log(cachedFibonacci.calculate(40)); // 从缓存返回
```

### 7. 表单验证

```javascript
function createFormValidator(schema) {
  return new Proxy({}, {
    set(target, property, value) {
      const validator = schema[property];

      if (validator) {
        const { validate, message } = validator;

        if (!validate(value)) {
          throw new Error(message);
        }
      }

      target[property] = value;
      return true;
    }
  });
}

const formSchema = {
  username: {
    validate: (val) => /^[a-zA-Z0-9]{4,16}$/.test(val),
    message: '用户名必须是 4-16 位的字母和数字'
  },
  email: {
    validate: (val) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(val),
    message: '请输入有效的邮箱地址'
  },
  password: {
    validate: (val) => val.length >= 8,
    message: '密码至少需要 8 个字符'
  }
};

const formData = createFormValidator(formSchema);

try {
  formData.username = 'user123';
  formData.email = 'user@example.com';
  formData.password = 'password123';

  console.log('表单验证通过');
} catch (error) {
  console.error('验证失败:', error.message);
}
```

### 8. 数组操作增强

```javascript
function createSmartArray(target) {
  return new Proxy(target, {
    get(target, property) {
      // 支持负索引
      if (typeof property === 'string' && /^-\d+$/.test(property)) {
        const index = parseInt(property);
        return target[target.length + index];
      }

      // 支持方法调用
      if (property in target) {
        return target[property].bind(target);
      }

      return target[property];
    },

    set(target, property, value) {
      // 自动扩展数组
      const index = parseInt(property);

      if (!isNaN(index) && index >= target.length) {
        target.length = index + 1;
      }

      target[property] = value;
      return true;
    }
  });
}

const smartArray = createSmartArray([1, 2, 3]);

console.log(smartArray[-1]); // 3
console.log(smartArray[-2]); // 2

smartArray[10] = 11;
console.log(smartArray.length); // 11
console.log(smartArray[10]); // 11
```

## Reflect 对象

Reflect 是一个内置对象，提供了拦截 JavaScript 操作的方法。这些方法与 Proxy 的拦截器方法一一对应。

```javascript
const obj = { name: '张三', age: 25 };

// Reflect 方法
Reflect.get(obj, 'name'); // '张三'
Reflect.set(obj, 'age', 26); // true
Reflect.has(obj, 'name'); // true
Reflect.deleteProperty(obj, 'age'); // true
Reflect.ownKeys(obj); // ['name', 'age']

// 函数调用
function greet(name) {
  return `Hello, ${name}!`;
}

Reflect.apply(greet, null, ['World']); // 'Hello, World!'

// 构造函数调用
function User(name) {
  this.name = name;
}

Reflect.construct(User, ['张三']); // User { name: '张三' }

// 使用 Reflect 的好处
const handler = {
  get(target, property, receiver) {
    // 1. 可以返回默认值
    if (property in target) {
      return Reflect.get(target, property, receiver);
    }
    return `Default ${property}`;

    // 2. 正确处理 this 指向
    // receiver 确保正确的 this 绑定
  }
};

const proxy = new Proxy(obj, handler);
```

## 注意事项

1. **性能考虑**：Proxy 会带来一定的性能开销，在性能敏感的场景要谨慎使用。

2. **this 指向问题**：某些操作中 Proxy 可能改变 this 的指向，使用 receiver 保持正确绑定。

3. **不可撤销的 Proxy**：一旦创建 Proxy，就无法断开与目标对象的连接。

4. **内置对象限制**：某些内置对象不能被代理，或者代理行为有限制。

5. **不可变性**：目标对象仍然可以直接访问，Proxy 不提供真正的封装。

6. **对象原型**：Proxy 代理的是目标对象本身，不会影响原型链。

7. **Symbol 属性**：Symbol 类型的属性也可以被代理和拦截。

8. **嵌套 Proxy**：可以创建嵌套的 Proxy，但要注意性能影响。

## 最佳实践

1. **合理使用**：只在真正需要拦截或增强对象行为时使用 Proxy。

2. **与 Reflect 配合**：使用 Reflect 方法可以确保操作的正确性。

3. **提供默认值**：在 get 拦截器中提供合理的默认值。

4. **错误处理**：在拦截器中添加适当的错误处理和验证。

5. **文档说明**：为使用 Proxy 的代码添加清晰的文档说明。

6. **性能优化**：对于频繁访问的属性，考虑使用缓存机制。

7. **测试覆盖**：确保 Proxy 的行为有充分的测试覆盖。

8. **渐进增强**：在不支持 Proxy 的环境中提供降级方案。

## 总结

通过本文的学习，相信你已经对 JavaScript Proxy 代理有了更深入的理解。Proxy 是一个强大的元编程工具，可以拦截和自定义 JavaScript 的各种操作。

Proxy 的主要优势：
- 提供了对对象行为的全面拦截能力
- 支持函数和构造函数的拦截
- 可以实现数据绑定、验证、缓存等多种模式
- 与 Reflect 配合使用更加安全和一致

在实际开发中，合理使用 Proxy 可以实现许多优雅的功能，但要注意性能影响和兼容性问题。记住，Proxy 是一个强大的工具，但不是所有问题都需要用 Proxy 来解决，要根据具体场景选择合适的方案。