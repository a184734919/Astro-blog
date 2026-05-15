---
title: JavaScript 作用域与闭包
published: 2022-02-01
description: '作用域、作用域链及闭包的核心原理、实际应用与最佳实践详解'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 作用域与闭包

## 概述

JavaScript 作用域与闭包是语言中最核心也最易混淆的概念之一。作用域决定了变量和函数的可访问性，闭包则是基于作用域链形成的强大特性。理解这两个概念对于编写高质量的 JavaScript 代码至关重要，它们直接影响代码的执行顺序、变量访问权限、内存管理和模块化设计。

## 作用域详解

### 作用域类型

```javascript
// 1. 全局作用域
let globalVar = '全局变量';

function globalFunction() {
  console.log(globalVar); // 可以访问全局变量
}

globalFunction(); // '全局变量'

// 2. 函数作用域（ES5）
function functionScope() {
  var funcVar = '函数作用域变量';
  if (true) {
    var ifVar = 'if 块中的变量';
  }
  console.log(funcVar);  // '函数作用域变量'
  console.log(ifVar);    // 'if 块中的变量' - var 不受块级作用域限制
}

functionScope();

// 3. 块级作用域（ES6 let/const）
function blockScope() {
  if (true) {
    let blockLet = 'let 块级变量';
    const blockConst = 'const 块级变量';
    console.log(blockLet);   // 可访问
    console.log(blockConst); // 可访问
  }

  // console.log(blockLet);   // ReferenceError: blockLet is not defined
  // console.log(blockConst); // ReferenceError: blockConst is not defined
}

blockScope();

// 4. 模块作用域（ES6 模块）
// module.js
export const moduleVar = '模块变量';
const privateVar = '私有变量'; // 模块内可访问，外部不可见
```

### 作用域链

```javascript
// 作用域链的查找过程
let global = '全局';

function outer() {
  let outer = '外层函数';

  function inner() {
    let inner = '内层函数';

    console.log(inner);  // 查找顺序：inner -> outer -> global
    console.log(outer);  // 查找顺序：outer -> global
    console.log(global); // 查找顺序：global
  }

  inner();
}

outer();

// 模拟作用域链查找过程
function resolveVariable(scopeChain, variableName) {
  for (let scope of scopeChain) {
    if (variableName in scope) {
      return scope[variableName];
    }
  }
  return undefined; // 查找失败
}

// 实际作用域链示例
const scopeChain = [
  { inner: '内层函数' },
  { outer: '外层函数' },
  { global: '全局' }
];
```

### 作用域提升

```javascript
// 1. 变量提升（var）
function hoistingExample() {
  console.log(hoistedVar); // undefined（已声明但未赋值）
  var hoistedVar = '变量提升';

  console.log(hoistedVar); // '变量提升'
}

hoistingExample();

// 2. 函数提升
console.log(hoistedFunction()); // '函数提升'

function hoistedFunction() {
  return '函数提升';
}

// 3. 函数表达式不提升
// console.log(notHoisted()); // TypeError: notHoisted is not a function

var notHoisted = function() {
  return '不会被提升';
};

// 4. let/const 不存在变量提升（暂时性死区）
function temporalDeadZone() {
  // console.log(letVar); // ReferenceError: Cannot access 'letVar' before initialization

  let letVar = 'let 变量';
  console.log(letVar); // 'let 变量'
}

temporalDeadZone();
```

### 作用域最佳实践

```javascript
// 1. 最小化全局污染
// 不好的做法
var user = '张三';  // 全局变量
var age = 25;

// 更好的做法：使用 IIFE
(function() {
  var user = '张三';
  var age = 25;

  function getUser() {
    return user;
  }

  // 暴露必要的接口
  window.getUser = getUser;
})();

console.log(getUser());    // '张三'
// console.log(user);      // ReferenceError: user is not defined

// 2. 优先使用 let/const
function modernScope() {
  const constant = '常量';
  let variable = '变量';

  if (true) {
    const blockConstant = '块级常量';
    let blockVariable = '块级变量';
  }

  // console.log(blockConstant);  // 错误：块级作用域
}

// 3. 合理使用模块化
// userModule.js
const privateData = {
  apiKey: 'secret'
};

export function getUser(id) {
  // 使用私有数据
  console.log('Using API key:', privateData.apiKey);
  return { id, name: `User ${id}` };
}

export function setUser(user) {
  // 实现设置逻辑
}
```

## 闭包深度解析

### 闭包的定义和形成条件

```javascript
// 闭包形成的三个条件：
// 1. 函数嵌套
// 2. 内层函数引用外层函数的变量
// 3. 内层函数被外层函数以外的地方引用

function createClosure() {
  let count = 0; // 外层函数的局部变量

  // 内层函数引用了外层变量 count
  return function increment() {
    count++;
    return count;
  };
}

const counter = createClosure();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

### 闭包的实际应用

#### 1. 数据封装和私有变量

```javascript
function createCounter(initialValue = 0) {
  let count = initialValue;

  return {
    increment() {
      return ++count;
    },
    decrement() {
      return --count;
    },
    getCount() {
      return count;
    },
    reset() {
      count = initialValue;
      return count;
    }
  };
}

const counter = createCounter(10);
console.log(counter.increment()); // 11
console.log(counter.getCount());  // 11
console.log(counter.reset());     // 10
// counter.count 是无法直接访问的私有变量
```

#### 2. 函数柯里化

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

function add(a, b, c) {
  return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3));      // 6
console.log(curriedAdd(1, 2)(3));     // 6
console.log(curriedAdd(1)(2, 3));     // 6
```

#### 3. 单例模式

```javascript
function createSingleton(createFn) {
  let instance = null;

  return function() {
    if (!instance) {
      instance = createFn();
    }
    return instance;
  };
}

function DatabaseConnection() {
  this.connected = false;
  this.connect = function() {
    this.connected = true;
    console.log('Database connected');
  };
}

const getDatabase = createSingleton(() => new DatabaseConnection());

const db1 = getDatabase();
const db2 = getDatabase();

console.log(db1 === db2); // true - 相同的实例
db1.connect();            // 'Database connected'
console.log(db2.connected); // true
```

#### 4. 状态管理

```javascript
function createStore(reducer, initialState) {
  let state = initialState;
  let listeners = [];

  const getState = () => state;

  const dispatch = (action) => {
    state = reducer(state, action);
    listeners.forEach(listener => listener());
  };

  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      const index = listeners.indexOf(listener);
      listeners.splice(index, 1);
    };
  };

  return {
    getState,
    dispatch,
    subscribe
  };
}

// 使用示例
function counterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    default:
      return state;
  }
}

const store = createStore(counterReducer);

store.subscribe(() => {
  console.log('State updated:', store.getState());
});

store.dispatch({ type: 'INCREMENT' }); // State updated: { count: 1 }
store.dispatch({ type: 'INCREMENT' }); // State updated: { count: 2 }
```

### 闭包中的常见陷阱

#### 1. 循环中的闭包问题

```javascript
// 问题：所有闭包都引用同一个变量
function createClosures() {
  const closures = [];

  for (var i = 0; i < 5; i++) {
    closures.push(function() {
      console.log(i);
    });
  }

  return closures;
}

const closures = createClosures();
closures.forEach(fn => fn()); // 输出 5 次 5，而不是 0,1,2,3,4

// 解决方案 1：使用 let
function createClosuresWithLet() {
  const closures = [];

  for (let i = 0; i < 5; i++) {
    closures.push(function() {
      console.log(i);
    });
  }

  return closures;
}

const closuresLet = createClosuresWithLet();
closuresLet.forEach(fn => fn()); // 输出 0,1,2,3,4

// 解决方案 2：使用 IIFE
function createClosuresWithIIFE() {
  const closures = [];

  for (var i = 0; i < 5; i++) {
    closures.push((function(j) {
      return function() {
        console.log(j);
      };
    })(i));
  }

  return closures;
}

const closuresIIFE = createClosuresWithIIFE();
closuresIIFE.forEach(fn => fn()); // 输出 0,1,2,3,4

// 解决方案 3：使用 forEach
function createClosuresWithForEach() {
  const closures = [];
  [0, 1, 2, 3, 4].forEach(i => {
    closures.push(function() {
      console.log(i);
    });
  });
  return closures;
}
```

#### 2. 内存泄漏风险

```javascript
// 问题：大对象长期被闭包引用
function createMemoryLeak() {
  const largeData = new Array(1000000).fill('data');

  return function() {
    console.log('Function called');
    // largeData 一直被引用，无法被垃圾回收
  };
}

const leak = createMemoryLeak();

// 解决方案 1：只引用必要的数据
function createFixedClosure() {
  const largeData = new Array(1000000).fill('data');
  const necessaryInfo = { length: largeData.length };
  largeData = null; // 解除对大对象的引用

  return function() {
    console.log('Data length:', necessaryInfo.length);
  };
}

// 解决方案 2：手动清理
function createCleanableClosure() {
  const largeData = new Array(1000000).fill('data');

  const closure = function() {
    console.log('Function called');
  };

  // 提供清理方法
  closure.cleanup = function() {
    // 这里无法直接清理 largeData，
    // 需要在外部将 closure 设置为 null
  };

  return closure;
}

const cleanable = createCleanableClosure();
cleanable();
cleanable = null; // 手动解除引用
```

#### 3. this 指向问题

```javascript
// 问题：闭包中 this 指向混乱
const obj = {
  value: 42,
  getValue: function() {
    return function() {
      console.log(this.value); // undefined，this 指向全局
    };
  }
};

obj.getValue()(); // undefined

// 解决方案 1：保存 this
const obj1 = {
  value: 42,
  getValue: function() {
    const self = this;
    return function() {
      console.log(self.value); // 42
    };
  }
};

// 解决方案 2：使用箭头函数
const obj2 = {
  value: 42,
  getValue: function() {
    return () => {
      console.log(this.value); // 42
    };
  }
};

// 解决方案 3：使用 bind
const obj3 = {
  value: 42,
  getValue: function() {
    return function() {
      console.log(this.value); // 42
    }.bind(this);
  }
};
```

### 箭头函数与闭包

```javascript
// 箭头函数没有自己的 this，arguments，super，new.target
// 它们会继承外层（非箭头）函数的这些值

class Counter {
  constructor() {
    this.count = 0;
    this.increment = this.increment.bind(this);
    this.incrementArrow = () => {
      this.count++;
      console.log(this.count);
    };
  }

  increment() {
    this.count++;
    console.log(this.count);
  }
}

const counter = new Counter();
const increment = counter.increment;
const incrementArrow = counter.incrementArrow;

// increment();    // TypeError: Cannot read property 'count' of undefined
incrementArrow();  // 1 - 箭头函数继承了 constructor 的 this

// setTimeout 中的箭头函数
class Timer {
  constructor() {
    this.seconds = 0;
    this.start();
  }

  start() {
    // 传统函数需要 bind
    // setInterval(function() {
    //   this.seconds++;
    //   console.log(this.seconds);
    // }.bind(this), 1000);

    // 箭头函数更简洁
    setInterval(() => {
      this.seconds++;
      console.log(this.seconds);
    }, 1000);
  }
}
```

## 高级应用

### 1. 模块化实现

```javascript
// 使用闭包创建私有模块
const UserModule = (function() {
  // 私有变量
  let users = [];
  let userIdCounter = 1;

  // 私有方法
  function generateId() {
    return userIdCounter++;
  }

  function validateUser(user) {
    return user && user.name && typeof user.name === 'string';
  }

  // 公共 API
  return {
    addUser(user) {
      if (!validateUser(user)) {
        throw new Error('Invalid user');
      }
      const newUser = {
        id: generateId(),
        name: user.name,
        createdAt: new Date()
      };
      users.push(newUser);
      return newUser;
    },

    getUser(id) {
      return users.find(user => user.id === id);
    },

    getAllUsers() {
      return [...users]; // 返回副本，保护内部数据
    },

    removeUser(id) {
      const index = users.findIndex(user => user.id === id);
      if (index !== -1) {
        users.splice(index, 1);
        return true;
      }
      return false;
    },

    userCount() {
      return users.length;
    }
  };
})();

// 使用模块
const user1 = UserModule.addUser({ name: '张三' });
const user2 = UserModule.addUser({ name: '李四' });

console.log(UserModule.getUser(user1.id)); // { id: 1, name: '张三', ... }
console.log(UserModule.userCount());       // 2

// UserModule.users 和 UserModule.userIdCounter 是无法访问的
```

### 2. 函数防抖和节流

```javascript
// 防抖：事件触发后延迟执行，如果在延迟时间内再次触发，重新计时
function debounce(fn, delay) {
  let timer = null;

  return function(...args) {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// 节流：限制函数执行频率，在指定时间内只执行一次
function throttle(fn, delay) {
  let lastTime = 0;
  let timer = null;

  return function(...args) {
    const now = Date.now();

    if (now - lastTime >= delay) {
      fn.apply(this, args);
      lastTime = now;
    } else {
      // 如果不是立即执行，可以设置定时器确保最后执行一次
      if (!timer) {
        timer = setTimeout(() => {
          fn.apply(this, args);
          lastTime = Date.now();
          timer = null;
        }, delay - (now - lastTime));
      }
    }
  };
}

// 使用示例
const searchInput = document.getElementById('search');

// 搜索输入防抖
searchInput.addEventListener('input', debounce(function(e) {
  console.log('搜索:', e.target.value);
  // 发送 AJAX 请求
}, 300));

// 窗口调整节流
window.addEventListener('resize', throttle(function() {
  console.log('窗口大小改变');
  // 重新计算布局
}, 200));
```

### 3. 异步流程控制

```javascript
// 使用闭包实现 Promise
function MyPromise(executor) {
  let state = 'pending';
  let value = undefined;
  let reason = undefined;
  let onFulfilledCallbacks = [];
  let onRejectedCallbacks = [];

  function resolve(value) {
    if (state === 'pending') {
      state = 'fulfilled';
      value = value;
      onFulfilledCallbacks.forEach(fn => fn(value));
    }
  }

  function reject(reason) {
    if (state === 'pending') {
      state = 'rejected';
      reason = reason;
      onRejectedCallbacks.forEach(fn => fn(reason));
    }
  }

  this.then = function(onFulfilled, onRejected) {
    return new MyPromise((resolve, reject) => {
      function handleFulfilled(value) {
        try {
          const result = onFulfilled ? onFulfilled(value) : value;
          resolve(result);
        } catch (error) {
          reject(error);
        }
      }

      function handleRejected(reason) {
        try {
          const result = onRejected ? onRejected(reason) : reason;
          reject(result);
        } catch (error) {
          reject(error);
        }
      }

      if (state === 'fulfilled') {
        setTimeout(() => handleFulfilled(value), 0);
      } else if (state === 'rejected') {
        setTimeout(() => handleRejected(reason), 0);
      } else {
        onFulfilledCallbacks.push(handleFulfilled);
        onRejectedCallbacks.push(handleRejected);
      }
    });
  };

  try {
    executor(resolve, reject);
  } catch (error) {
    reject(error);
  }
}

// 使用自定义 Promise
const promise = new MyPromise((resolve, reject) => {
  setTimeout(() => {
    resolve('Hello from custom Promise!');
  }, 1000);
});

promise
  .then(value => console.log(value))
  .catch(error => console.error(error));
```

### 4. 惰性函数

```javascript
// 惰性函数：函数只在第一次调用时执行复杂计算，后续调用返回缓存结果
function createLazyFunction(fn) {
  let result = null;
  let executed = false;

  return function(...args) {
    if (!executed) {
      result = fn.apply(this, args);
      executed = true;
    }
    return result;
  };
}

// 使用示例
function expensiveCalculation(x) {
  console.log('执行复杂计算...');
  return x * 2;
}

const lazyCalculate = createLazyFunction(expensiveCalculation);

console.log(lazyCalculate(5)); // 执行复杂计算... 10
console.log(lazyCalculate(5)); // 10（直接返回缓存结果）
console.log(lazyCalculate(5)); // 10（直接返回缓存结果）
```

## 性能优化

### 1. 避免闭包过重

```javascript
// 不好：闭包引用大量数据
function createHeavyClosure() {
  const largeData = new Array(10000).fill('data');
  const moreData = new Array(10000).fill('more data');

  return function() {
    console.log('Function called');
  };
}

// 更好：只引用必要数据
function createLightClosure() {
  const largeData = new Array(10000).fill('data');
  const summary = {
    length: largeData.length,
    first: largeData[0]
  };
  largeData = null; // 解除引用

  return function() {
    console.log('Summary:', summary);
  };
}
```

### 2. 及时清理闭包

```javascript
// 使用完成后手动清理
function createCleanableResource() {
  let resource = null;
  let isDisposed = false;

  function useResource() {
    if (isDisposed) {
      throw new Error('Resource has been disposed');
    }
    if (!resource) {
      resource = acquireResource();
    }
    return resource;
  }

  function dispose() {
    if (resource) {
      releaseResource(resource);
      resource = null;
    }
    isDisposed = true;
  }

  return {
    use: useResource,
    dispose: dispose
  };
}

// 使用示例
const resource = createCleanableResource();
try {
  const data = resource.use();
  // 处理数据
} finally {
  resource.dispose(); // 确保资源被清理
}
```

### 3. 避免不必要的闭包创建

```javascript
// 不好：每次调用都创建闭包
function processArrayBad(arr) {
  return arr.map(function(item) {
    return item * 2;
  });
}

// 更好：使用箭头函数（闭包更轻量）
function processArrayGood(arr) {
  return arr.map(item => item * 2);
}

// 或者预先定义函数
const multiplyBy2 = item => item * 2;
function processArrayBetter(arr) {
  return arr.map(multiplyBy2);
}
```

## 调试技巧

### 1. 使用 Chrome DevTools 调试闭包

```javascript
// 在 Chrome DevTools 中，可以查看闭包的变量
function createDebugger() {
  let secret = '秘密数据';
  let count = 0;

  return function() {
    count++;
    console.log(`Called ${count} times`);
    return secret;
  };
}

const debuggerFn = createDebugger();

// 在 DevTools 中：
// 1. 在 return 语句前设置断点
// 2. 调用 debuggerFn()
// 3. 在 Scope 面板中可以看到 Closure 变量
```

### 2. 使用 console.log 调试

```javascript
function createDebugClosure() {
  let value = 0;

  return {
    increment: function() {
      console.log('Before increment:', value);
      value++;
      console.log('After increment:', value);
      return value;
    },
    getValue: function() {
      console.log('Current value:', value);
      return value;
    }
  };
}
```

### 3. 使用 WeakMap 追踪闭包

```javascript
const closureTracker = new WeakMap();

function trackClosure(createFn) {
  const closure = createFn();

  if (closureTracker.has(createFn)) {
    console.log('闭包被重复创建');
  } else {
    closureTracker.set(createFn, {
      createdAt: Date.now(),
      callCount: 0
    });
  }

  return closure;
}

const myClosure = trackClosure(() => {
  let count = 0;
  return () => ++count;
});
```

## 常见面试题

### 1. 什么是闭包？有什么应用场景？

```javascript
// 闭包是函数和其词法环境的组合
// 应用场景：
// 1. 数据封装
const counter = (function() {
  let count = 0;
  return {
    increment: () => ++count,
    get: () => count
  };
})();

// 2. 函数柯里化
const add = a => b => a + b;

// 3. 模块模式
const Module = (function() {
  let private = 'private';
  return {
    getPrivate: () => private
  };
})();
```

### 2. 闭包会导致内存泄漏吗？如何避免？

```javascript
// 闭包本身不会导致内存泄漏，但不当使用可能导致问题

// 潜在问题
function createLeak() {
  const largeData = new Array(1000000).fill('data');
  return function() {
    console.log('hello');
  };
}

// 避免方法：
// 1. 只引用必要数据
// 2. 及时清理
// 3. 使用 WeakMap/WeakSet
// 4. 避免在循环中创建大量闭包
```

### 3. var、let、const 的作用域区别？

```javascript
// var: 函数作用域，存在变量提升
var functionVar = '函数作用域';

// let: 块级作用域，不存在变量提升，有暂时性死区
let blockLet = '块级作用域';

// const: 块级作用域，必须初始化，值不可重新赋值
const blockConst = '块级常量';

// 实际例子
function testScope() {
  for (var i = 0; i < 3; i++) {
    setTimeout(() => console.log('var:', i), 100);
  }

  for (let j = 0; j < 3; j++) {
    setTimeout(() => console.log('let:', j), 100);
  }
}

// 输出：var: 3, var: 3, var: 3, let: 0, let: 1, let: 2
```

### 4. 如何在循环中正确使用闭包？

```javascript
// 解决方案 1：let
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}

// 解决方案 2：IIFE
for (var i = 0; i < 3; i++) {
  (function(j) {
    setTimeout(() => console.log(j), 100);
  })(i);
}

// 解决方案 3：forEach
[0, 1, 2].forEach(i => {
  setTimeout(() => console.log(i), 100);
});
```

### 5. 箭头函数与普通函数在闭包中的区别？

```javascript
// 1. this 继承
const obj = {
  value: 42,
  regular: function() {
    return function() { console.log(this.value); };
  },
  arrow: function() {
    return () => console.log(this.value);
  }
};

obj.regular()(); // undefined
obj.arrow()();   // 42

// 2. 没有 arguments
function regular() {
  return function() { console.log(arguments); };
}

const arrowFunc = () => {
  return () => console.log(arguments); // ReferenceError
};

// 3. 不能作为构造函数
const regularFn = function() {};
const arrowFn = () => {};

new regularFn(); // 正常
new arrowFn();   // TypeError
```

## 总结

JavaScript 作用域与闭包是语言的精妙设计，理解它们对于编写高质量代码至关重要。关键要点：

1. **作用域类型**：理解全局、函数、块级、模块作用域的区别和使用场景
2. **作用域链**：掌握变量查找机制和性能影响
3. **闭包原理**：理解闭包的形成条件和内存管理
4. **实际应用**：掌握闭包在数据封装、模块化、异步处理中的应用
5. **避免陷阱**：了解循环闭包、内存泄漏、this 指向等常见问题
6. **性能优化**：避免不必要的闭包创建，及时清理资源
7. **现代开发**：结合箭头函数、模块化等现代特性使用

深入理解作用域与闭包，不仅能帮助我们编写更优雅的代码，还能提升代码的性能和可维护性。在实际开发中，要根据具体需求选择合适的作用域策略，合理使用闭包的强大功能。