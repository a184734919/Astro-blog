---
title: JavaScript 数据类型深入
published: 2022-01-07
description: '深入理解 JavaScript 中的基本类型、引用类型以及它们之间的区别与应用场景'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 数据类型深入

## 概述

JavaScript 是一种动态类型语言，数据类型是其最基础也是最重要的知识点之一。无论是变量声明、函数参数传递，还是对象操作、异步编程，都离不开对数据类型的深入理解。

掌握数据类型不仅有助于编写正确的代码，还能提升性能、避免常见的陷阱，并在面试中游刃有余地回答相关问题。

## JavaScript 数据类型分类

### 基本类型（Primitive Types）

JavaScript 有 7 种基本类型：

```javascript
// 1. String - 字符串
let name = "张三";
let greeting = 'Hello';
let template = `Welcome ${name}`;

// 2. Number - 数字（包括整数和浮点数）
let age = 25;
let price = 19.99;
let infinity = Infinity;
let notANumber = NaN;

// 3. Boolean - 布尔值
let isActive = true;
let isDisabled = false;

// 4. Undefined - 未定义
let undefinedVar;
console.log(undefinedVar); // undefined

// 5. Null - 空值
let nullVar = null;

// 6. Symbol - 符号（ES6新增，用于创建唯一标识）
const symbol1 = Symbol('description');
const symbol2 = Symbol('description');
console.log(symbol1 === symbol2); // false

// 7. BigInt - 大整数（ES2020新增）
const bigNumber = 9007199254740991n; // 超过Number.MAX_SAFE_INTEGER
const anotherBigInt = BigInt("123456789012345678901234567890");
```

### 引用类型（Reference Types）

引用类型主要包括：

```javascript
// 1. Object - 对象
let person = {
  name: '张三',
  age: 25,
  greet() {
    console.log('Hello!');
  }
};

// 2. Array - 数组
let fruits = ['apple', 'banana', 'orange'];

// 3. Function - 函数
function add(a, b) {
  return a + b;
}

// 4. Date - 日期
let now = new Date();

// 5. RegExp - 正则表达式
let pattern = /\d+/;

// 6. Map - 键值对集合
let map = new Map();
map.set('key1', 'value1');

// 7. Set - 唯一值集合
let set = new Set([1, 2, 3, 3, 4]);
console.log(set); // Set {1, 2, 3, 4}
```

## 核心概念

### 基本类型的特性

```javascript
// 基本类型的值是不可变的
let str = "Hello";
str = str + " World"; // 创建新的字符串，而不是修改原字符串

// 基本类型按值传递
function modifyPrimitive(x) {
  x = 10; // 修改的是函数内部的副本
}

let num = 5;
modifyPrimitive(num);
console.log(num); // 仍然是 5
```

### 引用类型的特性

```javascript
// 引用类型的值是可变的
let obj = { name: '张三' };
obj.name = '李四'; // 直接修改原对象

// 引用类型按引用传递
function modifyObject(obj) {
  obj.name = '修改后的名字'; // 修改的是原对象
}

let person = { name: '原始名字' };
modifyObject(person);
console.log(person.name); // '修改后的名字'
```

### 深浅拷贝详解

#### 浅拷贝

```javascript
// 浅拷贝只复制第一层
const original = {
  name: '张三',
  details: {
    age: 25,
    city: '北京'
  }
};

// 方法1：Object.assign()
const copy1 = Object.assign({}, original);

// 方法2：展开运算符
const copy2 = { ...original };

// 方法3：Array.from()（如果是数组）
const originalArray = [1, 2, { a: 3 }];
const copyArray = Array.from(originalArray);

// 修改嵌套对象
copy1.details.city = '上海';
console.log(original.details.city); // '上海' - 原对象也被修改！
```

#### 深拷贝

```javascript
// 方法1：JSON.stringify() + JSON.parse()
const deepCopy1 = JSON.parse(JSON.stringify(original));

// 缺点：无法处理函数、undefined、Symbol、循环引用
const hasFunction = {
  name: 'Test',
  fn: () => console.log('Hello')
};
const badCopy = JSON.parse(JSON.stringify(hasFunction));
console.log(badCopy.fn); // undefined - 函数丢失

// 方法2：Lodash 的 _.cloneDeep()
const _ = require('lodash');
const deepCopy2 = _.cloneDeep(original);

// 方法3：自定义深拷贝函数
function deepClone(obj, hash = new WeakMap()) {
  // 处理基本类型和 null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }

  // 处理循环引用
  if (hash.has(obj)) {
    return hash.get(obj);
  }

  // 处理日期对象
  if (obj instanceof Date) {
    return new Date(obj);
  }

  // 处理正则表达式
  if (obj instanceof RegExp) {
    return new RegExp(obj);
  }

  // 处理数组
  if (Array.isArray(obj)) {
    const copy = [];
    hash.set(obj, copy);
    return obj.map(item => deepClone(item, hash));
  }

  // 处理普通对象
  const copy = {};
  hash.set(obj, copy);
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      copy[key] = deepClone(obj[key], hash);
    }
  }
  return copy;
}

// 使用示例
const circularObj = { name: 'Circular' };
circularObj.self = circularObj;

const clonedCircular = deepClone(circularObj);
console.log(clonedCircular.self === clonedCircular); // true
```

## 类型检测

### typeof 操作符

```javascript
// 基本类型检测
typeof 'hello'           // 'string'
typeof 123               // 'number'
typeof true              // 'boolean'
typeof undefined         // 'undefined'
typeof Symbol()          // 'symbol'
typeof 123n              // 'bigint'

// 特殊情况
typeof null              // 'object' - 历史遗留问题
typeof []                // 'object'
typeof {}                // 'object'
typeof function() {}     // 'function'
```

### instanceof 操作符

```javascript
// 检测对象是否为某个类的实例
const arr = [1, 2, 3];
arr instanceof Array        // true
arr instanceof Object       // true

function Person(name) {
  this.name = name;
}

const person = new Person('张三');
person instanceof Person     // true
person instanceof Object     // true

// instanceof 的工作原理
function myInstanceof(left, right) {
  let proto = Object.getPrototypeOf(left);
  let prototype = right.prototype;

  while (true) {
    if (proto === null) return false;
    if (proto === prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }
}
```

### Object.prototype.toString

```javascript
// 最准确的类型检测方法
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

getType('hello')           // 'String'
getType(123)               // 'Number'
getType(true)              // 'Boolean'
getType(undefined)         // 'Undefined'
getType(null)              // 'Null'
getType([1, 2, 3])         // 'Array'
getType({})                // 'Object'
getType(new Date())        // 'Date'
getType(/\d+/)             // 'RegExp'
getType(function() {})     // 'Function'
getType(new Map())         // 'Map'
getType(new Set())         // 'Set'
```

### Array.isArray

```javascript
// 检测数组的最可靠方法
Array.isArray([1, 2, 3])   // true
Array.isArray({})          // false

// polyfill
if (!Array.isArray) {
  Array.isArray = function(arg) {
    return Object.prototype.toString.call(arg) === '[object Array]';
  };
}
```

## 类型转换

### 显式类型转换

```javascript
// 转为字符串
String(123)                    // '123'
String(true)                   // 'true'
String(null)                   // 'null'
String(undefined)              // 'undefined'
String({})                     // '[object Object]'
String([1, 2, 3])              // '1,2,3'

// 转为数字
Number('123')                  // 123
Number('123abc')               // NaN
Number('')                     // 0
Number(true)                   // 1
Number(false)                  // 0
Number(null)                   // 0
Number(undefined)              // NaN

parseInt('123px')              // 123
parseFloat('123.45abc')        // 123.45
parseInt('0xFF', 16)           // 255（指定进制）

// 转为布尔值
Boolean('')                    // false
Boolean(0)                     // false
Boolean(NaN)                   // false
Boolean(null)                  // false
Boolean(undefined)             // false
Boolean('false')               // true（非空字符串都是true）
```

### 隐式类型转换

```javascript
// 字符串拼接
1 + '2'                        // '12'
true + '1'                     // 'true1'
null + '1'                     // 'null1'

// 数学运算
'123' - 1                      // 122
'123' * 2                      // 246
'123abc' - 1                   // NaN
true + 1                       // 2
false + 1                      // 1

// 比较运算
'123' == 123                   // true
null == undefined              // true
null === undefined             // false
[] == false                    // true
[] == 0                        // true

// 逻辑运算
'text' && 123                  // 123
'' && 123                      // ''
'text' || 123                  // 'text'
'' || 123                      // 123

// 一元运算符
+'123'                         // 123
-'123'                         // -123
!true                          // false
!!true                         // true
```

### 类型转换陷阱

```javascript
// 1. + 运算符的歧义
const a = '10';
const b = '20';
a + b                          // '1020'（字符串拼接）
Number(a) + Number(b)          // 30（数字相加）

// 2. == vs ===
0 == ''                        // true
0 === ''                       // false
false == '0'                   // true
false === '0'                  // false

// 3. NaN 的特殊性
NaN === NaN                    // false
isNaN(NaN)                     // true
Number.isNaN(NaN)              // true（更可靠）

// 4. 对象转换
const obj = {
  valueOf() { return 10; },
  toString() { return '20'; }
};
+obj                           // 10（优先使用valueOf）
'' + obj                       // '20'（字符串拼接优先使用toString）

// 5. 数组转换
[] + []                        // ''（空数组转空字符串）
[] + {}                        // '[object Object]'
{} + []                        // 0（{}被解析为代码块）
({}) + []                      // '[object Object]'
```

## 实际应用场景

### 1. 表单数据处理

```javascript
function processFormData(formData) {
  // 类型验证和转换
  const age = Number(formData.age);
  const salary = parseFloat(formData.salary);
  const isActive = formData.isActive === 'true' || formData.isActive === true;

  // 验证
  if (isNaN(age) || age < 0 || age > 120) {
    throw new Error('Invalid age');
  }

  if (isNaN(salary) || salary < 0) {
    throw new Error('Invalid salary');
  }

  return {
    age,
    salary,
    isActive
  };
}

// 使用
try {
  const result = processFormData({
    age: '25',
    salary: '5000.50',
    isActive: 'true'
  });
  console.log(result);
  // { age: 25, salary: 5000.5, isActive: true }
} catch (error) {
  console.error('表单数据错误:', error.message);
}
```

### 2. 数据存储和传输

```javascript
// 使用 localStorage 存储对象
function saveToStorage(key, value) {
  try {
    // 序列化对象
    const serialized = JSON.stringify(value);
    localStorage.setItem(key, serialized);
    return true;
  } catch (error) {
    console.error('存储失败:', error);
    return false;
  }
}

function loadFromStorage(key, defaultValue = null) {
  try {
    const serialized = localStorage.getItem(key);
    if (serialized === null) return defaultValue;

    // 反序列化对象
    return JSON.parse(serialized);
  } catch (error) {
    console.error('读取失败:', error);
    return defaultValue;
  }
}

// 使用
const user = {
  id: 1,
  name: '张三',
  preferences: {
    theme: 'dark',
    language: 'zh-CN'
  }
};

saveToStorage('user', user);
const loadedUser = loadFromStorage('user', {});
console.log(loadedUser);
```

### 3. API 响应处理

```javascript
async function fetchUser(userId) {
  const response = await fetch(`/api/users/${userId}`);

  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }

  const data = await response.json();

  // 验证数据结构
  if (!data || typeof data !== 'object') {
    throw new Error('Invalid response format');
  }

  // 类型转换和默认值
  return {
    id: Number(data.id) || 0,
    name: String(data.name || ''),
    email: data.email || '',
    age: Number(data.age) || 0,
    isActive: Boolean(data.isActive),
    createdAt: new Date(data.createdAt)
  };
}

// 使用错误边界处理
async function safeFetchUser(userId) {
  try {
    const user = await fetchUser(userId);
    return { success: true, data: user };
  } catch (error) {
    console.error('获取用户失败:', error);
    return { success: false, error: error.message };
  }
}
```

### 4. 状态管理中的不可变更新

```javascript
// 在 React/Vue 中的状态更新
function updateUserState(state, updates) {
  // 深拷贝避免修改原状态
  const newState = deepClone(state);

  // 应用更新
  Object.assign(newState, updates);

  return newState;
}

// 或者使用展开运算符（浅拷贝）
function updateUser(user, updates) {
  return {
    ...user,
    ...updates,
    // 嵌套对象需要单独处理
    address: updates.address ? {
      ...user.address,
      ...updates.address
    } : user.address
  };
}

// 使用示例
const currentUser = {
  id: 1,
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    street: '朝阳区'
  }
};

const updatedUser = updateUser(currentUser, {
  age: 26,
  address: {
    city: '上海'
  }
});

console.log(currentUser.age);        // 25（原状态未变）
console.log(updatedUser.age);         // 26（新状态已更新）
```

## 性能优化

### 1. 减少不必要的类型转换

```javascript
// 不好：频繁类型转换
function sumNumbers(arr) {
  return arr.reduce((sum, item) => sum + Number(item), 0);
}

// 更好：确保输入类型正确
function sumNumbersOptimized(arr) {
  let sum = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i]; // 假设已经是数字类型
  }
  return sum;
}
```

### 2. 使用 TypedArray 处理数值数据

```javascript
// 普通数组
const normalArray = [1, 2, 3, 4, 5];

// TypedArray（更高效，内存占用更少）
const int8Array = new Int8Array([1, 2, 3, 4, 5]);
const float64Array = new Float64Array([1.1, 2.2, 3.3]);

// 性能对比
function processNormalArray(arr) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    result.push(arr[i] * 2);
  }
  return result;
}

function processTypedArray(typedArr) {
  const result = new Float64Array(typedArr.length);
  for (let i = 0; i < typedArr.length; i++) {
    result[i] = typedArr[i] * 2;
  }
  return result;
}

// TypedArray 在处理大量数值时性能更好
```

### 3. 避免大对象的深拷贝

```javascript
// 不好：深拷贝大对象
function updateLargeObject(largeObj, update) {
  const copy = JSON.parse(JSON.stringify(largeObj)); // 慢
  Object.assign(copy, update);
  return copy;
}

// 更好：使用不可变更新策略
function updateLargeObjectOptimized(largeObj, update) {
  return {
    ...largeObj,
    ...update
  };
}

// 或者使用专门的对象更新库
const { produce } = require('immer');

function updateWithImmer(baseState, update) {
  return produce(baseState, draft => {
    Object.assign(draft, update);
  });
}
```

## 常见面试题

### 1. 基本类型和引用类型的区别？

```javascript
// 基本类型：存储在栈中，按值传递
let a = 10;
let b = a;
b = 20;
console.log(a); // 10

// 引用类型：存储在堆中，按引用传递
let obj1 = { name: 'Tom' };
let obj2 = obj1;
obj2.name = 'Jerry';
console.log(obj1.name); // 'Jerry'
```

### 2. typeof null 返回 'object' 的原因？

这是 JavaScript 的一个历史遗留问题。在 JavaScript 早期版本中，使用 32 位系统存储类型信息，前 3 位用于表示类型：

- 000: 对象
- 001: 整数
- 010: 浮点数
- 100: 字符串
- 110: 布尔值

`null` 的二进制表示全是 0，因此被错误地识别为对象。

### 3. 如何判断一个变量是数组？

```javascript
// 方法1：Array.isArray（推荐）
Array.isArray(arr)

// 方法2：Object.prototype.toString
Object.prototype.toString.call(arr) === '[object Array]'

// 方法3：instanceof
arr instanceof Array

// 方法4：constructor（不可靠）
arr.constructor === Array
```

### 4. == 和 === 的区别？

```javascript
// == 进行类型转换后比较
'1' == 1     // true
null == undefined  // true
0 == ''      // true

// === 严格比较，不进行类型转换
'1' === 1    // false
null === undefined  // false
0 === ''     // false
```

### 5. 如何实现深拷贝？

```javascript
// 方法1：JSON 方法（简单但不完美）
const copy1 = JSON.parse(JSON.stringify(obj));

// 方法2：递归深拷贝
function deepClone(obj, hash = new WeakMap()) {
  if (obj === null || typeof obj !== 'object') return obj;

  if (hash.has(obj)) return hash.get(obj);

  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof RegExp) return new RegExp(obj);

  const clone = Array.isArray(obj) ? [] : {};
  hash.set(obj, clone);

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      clone[key] = deepClone(obj[key], hash);
    }
  }
  return clone;
}

// 方法3：使用 Lodash
const copy3 = _.cloneDeep(obj);
```

## 最佳实践

### 1. 使用 const 声明变量

```javascript
// 推荐
const CONFIG = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

// 只在需要重新赋值时使用 let
let counter = 0;

// 避免使用 var
// var 有函数作用域，容易导致变量提升和全局污染
```

### 2. 显式类型转换

```javascript
// 不好：依赖隐式转换
if (userInput) {
  // 可能把空字符串、0、false都当作true
}

// 更好：明确检查
if (typeof userInput === 'string' && userInput.trim() !== '') {
  // 确保是非空字符串
}

// 数字转换
const count = parseInt(input, 10);  // 明确指定进制
const total = Number(input) || 0;   // 提供默认值
```

### 3. 合理的类型检查

```javascript
// 检查对象是否为空
function isEmpty(obj) {
  if (obj == null) return true;
  if (Array.isArray(obj)) return obj.length === 0;
  if (typeof obj === 'object') return Object.keys(obj).length === 0;
  return false;
}

// 检查是否为纯对象
function isPlainObject(value) {
  if (typeof value !== 'object' || value === null) return false;
  const proto = Object.getPrototypeOf(value);
  return proto === null || proto === Object.prototype;
}

// 安全的属性访问
function safeGet(obj, path, defaultValue) {
  return path.split('.').reduce((acc, key) => {
    return acc && acc[key] !== undefined ? acc[key] : defaultValue;
  }, obj);
}

const user = {
  profile: {
    settings: {
      theme: 'dark'
    }
  }
};

console.log(safeGet(user, 'profile.settings.theme', 'light')); // 'dark'
console.log(safeGet(user, 'profile.unknown', 'default'));      // 'default'
```

### 4. 使用 TypeScript 增强类型安全

```javascript
// TypeScript 类型定义
interface User {
  id: number;
  name: string;
  email: string;
  age?: number;  // 可选属性
  createdAt: Date;
}

function processUser(user: User): string {
  // TypeScript 提供类型检查
  return `User ${user.name} (${user.email})`;
}

// 使用类型断言
const data = JSON.parse('{"name": "张三"}') as Partial<User>;
console.log(data.name);  // 类型安全
```

## 注意事项

### 1. 避免修改原型

```javascript
// 危险：修改内置对象原型
Array.prototype.first = function() {
  return this[0];
};

// 问题：可能与其他库冲突，影响整个应用
// 更好：使用工具函数
function getFirst(arr) {
  return arr && arr[0];
}
```

### 2. 注意 NaN 的特殊性质

```javascript
// NaN 不等于自身
if (isNaN(value)) {
  // 检查 NaN
}

// 更好
if (Number.isNaN(value)) {
  // 不会强制类型转换
}

// 检查有限数字
function isFiniteNumber(value) {
  return typeof value === 'number' &&
         !Number.isNaN(value) &&
         value !== Infinity &&
         value !== -Infinity;
}
```

### 3. 避免使用 Object 的 __proto__ 属性

```javascript
// 不推荐
const obj = {};
obj.__proto__ = {};  // 已废弃

// 推荐
const obj2 = Object.create({});  // 使用 Object.create
const obj3 = {};
Object.setPrototypeOf(obj3, {});  // 使用 Object.setPrototypeOf
```

### 4. 注意内存泄漏

```javascript
// 问题：闭包导致大对象无法回收
function createClosure() {
  const largeData = new Array(1000000).fill('data');
  return function() {
    console.log('Hello');
  };
}

const closure = createClosure();  // largeData 仍被引用

// 解决：手动清理或避免引用大对象
function createClosureFixed() {
  const largeData = new Array(1000000).fill('data');
  const result = processData(largeData);  // 只保存处理结果
  largeData = null;  // 解除引用
  return function() {
    console.log(result);
  };
}
```

## 总结

JavaScript 数据类型是前端开发的基础核心，掌握它们对于编写高质量代码至关重要。本文涵盖了：

1. **7 种基本类型和多种引用类型**：理解它们的特点和使用场景
2. **深浅拷贝的区别和实现**：根据需求选择合适的拷贝策略
3. **类型检测方法**：typeof、instanceof、Object.prototype.toString 等
4. **类型转换规则**：避免隐式转换带来的陷阱
5. **实际应用场景**：表单处理、API 响应、状态管理等
6. **性能优化技巧**：减少不必要的类型转换、使用 TypedArray 等
7. **最佳实践**：使用 const、显式类型转换、安全属性访问等

深入理解数据类型，不仅能避免常见的编程错误，还能提升代码的性能和可维护性。在实际开发中，要特别注意类型转换的边界情况，选择合适的拷贝策略，并养成良好的编码习惯。

后续可以继续学习：
- ES6+ 新增的数据结构（Map、Set、WeakMap、WeakSet）
- TypeScript 类型系统
- 性能分析和优化
- 内存管理和垃圾回收机制