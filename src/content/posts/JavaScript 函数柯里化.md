---
title: JavaScript 函数柯里化
published: 2022-07-30
description: '柯里化的概念和实现的详细介绍和学习笔记'
image: ''
tags: ["JS","函数式"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

柯里化(Currying)是一种函数式编程技术,它将接受多个参数的函数转换为一系列接受单个参数的函数。这种技术可以提高代码的复用性、可读性和可维护性。

## 核心概念

### 什么是柯里化

柯里化是将一个多参数函数转换为一系列单参数函数的过程:

```javascript
// 普通函数
function add(a, b, c) {
  return a + b + c;
}

// 柯里化后的函数
function curriedAdd(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}

// 使用
console.log(add(1, 2, 3));           // 6
console.log(curriedAdd(1)(2)(3));    // 6
```

### 箭头函数写法

```javascript
const curriedAdd = a => b => c => a + b + c;

// 使用
const add1 = curriedAdd(1);
const add1And2 = add1(2);
const result = add1And2(3); // 6

// 或链式调用
console.log(curriedAdd(1)(2)(3)); // 6
```

## 柯里化实现

### 基础实现

```javascript
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    } else {
      return function(...moreArgs) {
        return curried.apply(this, args.concat(moreArgs));
      };
    }
  };
}

// 使用
function multiply(a, b, c) {
  return a * b * c;
}

const curriedMultiply = curry(multiply);
console.log(curriedMultiply(2)(3)(4));    // 24
console.log(curriedMultiply(2, 3)(4));    // 24
console.log(curriedMultiply(2)(3, 4));    // 24
console.log(curriedMultiply(2, 3, 4));    // 24
```

### 支持占位符的柯里化

```javascript
const _ = Symbol('placeholder');

function curry(fn, placeholder = _) {
  return function curried(...args) {
    // 过滤掉占位符
    const filteredArgs = args.filter(arg => arg !== placeholder);
    
    // 检查是否所有必需参数都已提供
    if (filteredArgs.length >= fn.length) {
      // 填充占位符
      const filledArgs = fillPlaceholders(args, fn.length);
      return fn.apply(this, filledArgs);
    }
    
    return function(...moreArgs) {
      // 合并参数
      const newArgs = args.map(arg => {
        if (arg === placeholder && moreArgs.length > 0) {
          return moreArgs.shift();
        }
        return arg;
      });
      
      return curried.apply(this, [...newArgs, ...moreArgs]);
    };
  };
}

function fillPlaceholders(args, requiredLength) {
  const result = [];
  let argIndex = 0;
  
  for (let i = 0; i < requiredLength; i++) {
    if (argIndex < args.length && args[argIndex] !== _) {
      result.push(args[argIndex]);
      argIndex++;
    } else {
      // 需要从后面填充,这里简化处理
      result.push(null);
    }
  }
  
  return result;
}

// 使用
function createMessage(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const curriedMessage = curry(createMessage);

console.log(curriedMessage('Hello')('World')('!')); // Hello, World!
console.log(curriedMessage(_, 'World')('!')('Hello')); // Hello, World!
```

## 实际应用

### 1. 创建可复用的函数

```javascript
// 创建特定格式的日期格式化函数
function formatDate(date, format) {
  // 简化的格式化逻辑
  const d = new Date(date);
  if (format === 'YYYY-MM-DD') {
    return d.toISOString().split('T')[0];
  }
  return d.toLocaleDateString();
}

const curriedFormatDate = curry(formatDate);

const formatDateAsISO = curriedFormatDate(_, 'YYYY-MM-DD');
const formatDateAsLocale = curriedFormatDate(_, 'locale');

console.log(formatDateAsISO(new Date())); // 2024-01-01
console.log(formatDateAsLocale(new Date())); // 2024/1/1
```

### 2. 数据处理管道

```javascript
// 创建数据验证管道
function validate(data, rules) {
  const errors = [];
  
  for (const rule of rules) {
    if (!rule.validate(data)) {
      errors.push(rule.message);
    }
  }
  
  return {
    valid: errors.length === 0,
    errors
  };
}

const curriedValidate = curry(validate);

// 定义验证规则
const rules = [
  { validate: data => data.length >= 6, message: '密码长度至少6位' },
  { validate: data => /[A-Z]/.test(data), message: '必须包含大写字母' },
  { validate: data => /[0-9]/.test(data), message: '必须包含数字' }
];

// 创建特定验证器
const validatePassword = curriedValidate(_, rules);

console.log(validatePassword('abc')); // { valid: false, errors: [...] }
console.log(validatePassword('Abc123')); // { valid: true, errors: [] }
```

### 3. API 请求封装

```javascript
function apiRequest(method, url, data) {
  return fetch(url, {
    method,
    headers: { 'Content-Type': 'application/json' },
    body: data ? JSON.stringify(data) : null
  }).then(res => res.json());
}

const curriedApiRequest = curry(apiRequest);

// 创建特定方法的请求函数
const get = curriedApiRequest('GET');
const post = curriedApiRequest('POST');
const put = curriedApiRequest('PUT');
const del = curriedApiRequest('DELETE');

// 创建特定端点的请求函数
const getUser = get('/api/users/');
const createUser = post('/api/users');
const updateUser = put('/api/users/');
const deleteUser = del('/api/users/');

// 使用
getUser(1).then(data => console.log(data));
createUser({ name: 'Alice' }).then(data => console.log(data));
updateUser(1, { name: 'Bob' }).then(data => console.log(data));
deleteUser(1).then(data => console.log(data));
```

### 4. 事件处理

```javascript
function createElement(tag, className, content) {
  const el = document.createElement(tag);
  if (className) el.className = className;
  if (content) el.textContent = content;
  return el;
}

const curriedCreateElement = curry(createElement);

// 创建特定元素的函数
const createDiv = curriedCreateElement('div');
const createButton = curriedCreateElement('button');
const createSpan = curriedCreateElement('span');

// 创建特定样式的元素
const createPrimaryButton = createButton('btn-primary');
const createSuccessDiv = createDiv('alert-success');

// 使用
const button = createPrimaryButton('Click Me');
const alert = createSuccessDiv('Success!');
```

### 5. 函数组合

```javascript
function compose(...fns) {
  return function(x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}

// 基础函数
const add = a => b => a + b;
const multiply = a => b => a * b;
const divide = a => b => b / a;

// 组合函数
const add5 = add(5);
const multiplyBy2 = multiply(2);
const divideBy3 = divide(3);

const calculate = compose(
  divideBy3,
  multiplyBy2,
  add5
);

console.log(calculate(10)); // ((10 + 5) * 2) / 3 = 10
```

## 性能优化

### 记忆化

```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// 使用记忆化优化柯里化函数
const expensiveOperation = memoize(curry((a, b, c) => {
  // 模拟耗时操作
  console.log('Computing...');
  return a * b * c;
}));

console.log(expensiveOperation(2)(3)(4)); // Computing... 24
console.log(expensiveOperation(2)(3)(4)); // 24 (从缓存中获取)
```

### 惰性求值

```javascript
function lazy(fn) {
  let cached = null;
  let computed = false;
  
  return function(...args) {
    if (!computed) {
      cached = fn.apply(this, args);
      computed = true;
    }
    return cached;
  };
}

const curriedLazy = lazy(curry((a, b) => {
  console.log('Heavy computation');
  return a + b;
}));

const add10 = curriedLazy(10);
console.log(add10(20)); // Heavy computation 30
console.log(add10(30)); // 30 (使用缓存结果)
```

## 注意事项

### 1. 性能考虑

```javascript
// 柯里化会增加函数调用开销
// 对于性能敏感的场景,谨慎使用

// 不好: 频繁调用的地方使用柯里化
function processItems(items) {
  return items.map(item => {
    const process = curriedProcessor(item.id);
    return process(item.data);
  });
}

// 更好: 直接使用普通函数
function processItems(items) {
  return items.map(item => process(item.id, item.data));
}
```

### 2. 调试难度

```javascript
// 柯里化函数的调用栈可能难以理解
const result = curriedAdd(1)(2)(3);

// 可以添加日志帮助调试
function debugCurry(fn, name) {
  return function(...args) {
    console.log(`${name} called with:`, args);
    const result = fn.apply(this, args);
    console.log(`${name} returned:`, result);
    return result;
  };
}

const debugCurriedAdd = debugCurry(curry((a, b, c) => a + b + c), 'add');
```

### 3. 过度使用

```javascript
// 不好的做法: 简单操作也使用柯里化
const curriedLog = curry((msg) => console.log(msg));
curriedLog('Hello'); // 过度复杂

// 更好的做法: 直接使用
console.log('Hello');
```

### 4. 类型推断

```javascript
// 柯里化可能影响 TypeScript 的类型推断
// 需要明确类型注解

type Curried<A, B, C, R> = 
  | ((a: A, b: B, c: C) => R)
  | ((a: A) => Curried<B, C, R>)
  | ((a: A, b: B) => Curried<C, R>);

function typedCurry<A, B, C, R>(
  fn: (a: A, b: B, c: C) => R
): Curried<A, B, C, R> {
  return curry(fn) as any;
}
```

## 与偏函数应用的区别

### 柯里化 vs 偏函数

```javascript
// 柯里化: 每次只接受一个参数
const curriedAdd = a => b => c => a + b + c;
const add1 = curriedAdd(1);
const add1And2 = add1(2);
const result = add1And2(3); // 6

// 偏函数: 固定部分参数
function add(a, b, c) {
  return a + b + c;
}

// 使用 bind
const add5and6 = add.bind(null, 5, 6);
console.log(add5and6(7)); // 18

// 使用箭头函数
const add5 = (b, c) => add(5, b, c);
console.log(add5(6, 7)); // 18
```

### 偏函数应用函数

```javascript
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn.apply(this, [...presetArgs, ...laterArgs]);
  };
}

// 使用
function multiply(a, b, c) {
  return a * b * c;
}

const multiplyBy2 = partial(multiply, 2);
const multiplyBy2And3 = partial(multiply, 2, 3);

console.log(multiplyBy2(3, 4)); // 24
console.log(multiplyBy2And3(4)); // 24
```

## 实际项目应用

### React 组件示例

```javascript
// 创建可复用的事件处理器
const createEventHandler = curry((eventName, handler, component) => {
  component.addEventListener(eventName, handler);
  return {
    remove: () => component.removeEventListener(eventName, handler)
  };
});

// 使用
const clickHandler = (e) => console.log('Clicked', e.target);
const addClickListener = createEventHandler('click');

const button = document.getElementById('button');
const listener = addClickListener(clickHandler, button);

// 移除监听器
listener.remove();
```

### 表单验证

```javascript
function createValidator(field, rules) {
  return {
    validate: (value) => {
      for (const rule of rules) {
        if (!rule.test(value)) {
          return { valid: false, error: rule.message };
        }
      }
      return { valid: true, error: null };
    },
    field
  };
}

const curriedValidator = curry(createValidator);

// 创建字段验证器
const emailValidator = curriedValidator('email')([
  { test: v => v.includes('@'), message: '必须包含 @' },
  { test: v => v.includes('.'), message: '必须包含 .' }
]);

const passwordValidator = curriedValidator('password')([
  { test: v => v.length >= 8, message: '至少8位' },
  { test: v => /[A-Z]/.test(v), message: '必须包含大写字母' }
]);

// 使用
console.log(emailValidator.validate('test@email.com'));
console.log(passwordValidator.validate('Password123'));
```

## 总结

柯里化的核心要点:

1. **基本概念**: 将多参数函数转换为单参数函数链
2. **实现方法**: 递归实现、支持占位符、记忆化优化
3. **实际应用**: API封装、数据处理、事件处理、函数组合
4. **性能考虑**: 注意调用开销,合理使用缓存
5. **注意事项**: 避免过度使用,注意调试难度
6. **区别理解**: 柯里化vs偏函数,明确使用场景
7. **最佳实践**: 在合适的场景使用,提升代码可读性和复用性

柯里化是函数式编程的重要技术,合理使用可以提升代码质量和开发效率。