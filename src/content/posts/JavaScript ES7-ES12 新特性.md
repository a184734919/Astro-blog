---
title: JavaScript ES7-ES16 新特性
published: 2026-05-10
description: '近年来的新增特性介绍的详细介绍和学习笔记'
image: ''
tags: ["JS","ES7+"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 自 ES6（2015年）之后，每年都会发布新版本。本文将详细介绍 ES7 到 ES16 的主要新特性，这些特性大大提升了 JavaScript 的开发效率和代码质量。

## ES7 (2016) 新特性

### Array.prototype.includes()

`includes()` 方法用于判断数组是否包含某个元素。

```javascript
// 基本用法
const numbers = [1, 2, 3, 4, 5];

console.log(numbers.includes(3)); // true
console.log(numbers.includes(6)); // false

// 支持第二个参数：从指定位置开始搜索
console.log(numbers.includes(3, 3)); // false
console.log(numbers.includes(4, 3)); // true

// 处理 NaN
console.log([NaN].includes(NaN)); // true
// 相比 indexOf：
console.log([NaN].indexOf(NaN)); // -1

// 实际应用：检查元素是否存在
function hasPermission(permissions, permission) {
  return permissions.includes(permission);
}

const userPermissions = ['read', 'write', 'delete'];
console.log(hasPermission(userPermissions, 'write')); // true
console.log(hasPermission(userPermissions, 'admin')); // false
```

### 指数运算符

指数运算符 `**` 用于计算幂运算，等价于 `Math.pow()`。

```javascript
// 基本用法
console.log(2 ** 3); // 8 (2的3次方)
console.log(3 ** 2); // 9

// 等价于 Math.pow
console.log(2 ** 3 === Math.pow(2, 3)); // true

// 复合赋值
let x = 2;
x **= 3;
console.log(x); // 8

// 实际应用：计算面积
function calculateCircleArea(radius) {
  return Math.PI * radius ** 2;
}

console.log(calculateCircleArea(5)); // 78.53981633974483

// 计算阶乘
function factorial(n) {
  return n === 0 ? 1 : n * factorial(n - 1);
}

console.log(factorial(5)); // 120
```

## ES8 (2017) 新特性

### Object.values()

`Object.values()` 方法返回一个包含对象所有可枚举属性值的数组。

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 获取所有属性值
const values = Object.values(user);
console.log(values); // ['张三', 25, 'zhangsan@example.com']

// 与 Object.keys 对比
console.log(Object.keys(user)); // ['name', 'age', 'email']

// 处理类数组对象
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
console.log(Object.values(arrayLike)); // ['a', 'b', 'c']

// 实际应用：获取对象的所有值进行操作
function sumObjectValues(obj) {
  return Object.values(obj).reduce((sum, value) => {
    return typeof value === 'number' ? sum + value : sum;
  }, 0);
}

const data = { a: 1, b: 2, c: 3, name: 'test' };
console.log(sumObjectValues(data)); // 6
```

### Object.entries()

`Object.entries()` 方法返回一个包含对象所有可枚举属性的键值对数组。

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 获取键值对
const entries = Object.entries(user);
console.log(entries);
// [['name', '张三'], ['age', 25], ['email', 'zhangsan@example.com']]

// 遍历对象
Object.entries(user).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// 转换为 Map
const userMap = new Map(Object.entries(user));
console.log(userMap.get('name')); // '张三'

// 反向：从键值对数组创建对象
const entries2 = [['a', 1], ['b', 2], ['c', 3]];
const obj = Object.fromEntries(entries2);
console.log(obj); // { a: 1, b: 2, c: 3 }

// 实际应用：过滤对象属性
function filterObject(obj, predicate) {
  return Object.fromEntries(
    Object.entries(obj).filter(([key, value]) => predicate(key, value))
  );
}

const data = {
  id: 1,
  name: '张三',
  age: 25,
  __proto__: null
};

const filteredData = filterObject(data, (key) => !key.startsWith('__'));
console.log(filteredData); // { id: 1, name: '张三', age: 25 }
```

### Object.getOwnPropertyDescriptors()

`Object.getOwnPropertyDescriptors()` 方法返回指定对象所有自有属性（非继承属性）的属性描述符。

```javascript
const user = {
  name: '张三',
  get fullName() {
    return `${this.name} 先生`;
  }
};

// 获取所有属性描述符
const descriptors = Object.getOwnPropertyDescriptors(user);
console.log(descriptors);
/*
{
  name: {
    value: '张三',
    writable: true,
    enumerable: true,
    configurable: true
  },
  fullName: {
    get: [Function: get fullName],
    set: undefined,
    enumerable: true,
    configurable: true
  }
}
*/

// 实际应用：精确复制对象
function shallowCopy(obj) {
  return Object.create(
    Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj)
  );
}

const copy = shallowCopy(user);
console.log(copy.fullName); // '张三 先生'

// 与 Object.assign 的区别
const assignCopy = Object.assign({}, user);
// Object.assign 不会复制 getter/setter，只会复制值
```

### 字符串填充方法

`padStart()` 和 `padEnd()` 方法用于字符串填充。

```javascript
// padStart - 在开头填充
'5'.padStart(2, '0'); // '05'
'123'.padStart(5, '0'); // '00123'
'abc'.padStart(5, 'x'); // 'xxabc'

// padEnd - 在末尾填充
'5'.padEnd(2, '0'); // '50'
'123'.padEnd(5, '0'); // '12300'
'abc'.padEnd(5, 'x'); // 'abcxx'

// 实际应用：格式化数字
function formatNumber(num, length = 2) {
  return String(num).padStart(length, '0');
}

console.log(formatNumber(1)); // '01'
console.log(formatNumber(15)); // '15'
console.log(formatNumber(5, 4)); // '0005'

// 格式化时间
function formatTime(hours, minutes, seconds) {
  return `${formatNumber(hours)}:${formatNumber(minutes)}:${formatNumber(seconds)}`;
}

console.log(formatTime(9, 5, 3)); // '09:05:03'

// 文本对齐
function alignText(text, length, char = ' ', align = 'left') {
  if (align === 'left') {
    return text.padEnd(length, char);
  } else if (align === 'right') {
    return text.padStart(length, char);
  } else {
    const padding = length - text.length;
    const leftPadding = Math.floor(padding / 2);
    const rightPadding = padding - leftPadding;
    return ''.padStart(leftPadding, char) + text + ''.padEnd(rightPadding, char);
  }
}

console.log(alignText('hello', 10, '-', 'left')); // 'hello-----'
console.log(alignText('hello', 10, '-', 'right')); // '-----hello'
console.log(alignText('hello', 10, '-', 'center')); // '--hello---'
```

### 函数参数末尾逗号

允许在函数定义和调用时在最后一个参数后面添加逗号。

```javascript
// 函数定义
function example(a, b, c,) {
  console.log(a, b, c);
}

example(1, 2, 3,);

// 多行参数更易维护
const options = {
  baseUrl: 'https://api.example.com',
  timeout: 5000,
  headers: {
    'Content-Type': 'application/json',
  }, // 末尾逗号
};

// 数组解构
const [
  first,
  second,
  third,
] = [1, 2, 3];

// 对象解构
const {
  name,
  age,
  email,
} = user;
```

### async/await

详见《JavaScript async-await》文章。

## ES9 (2018) 新特性

### 异步迭代器

为异步操作添加迭代器支持。

```javascript
// 异步生成器
async function* asyncGenerator() {
  yield Promise.resolve(1);
  yield Promise.resolve(2);
  yield Promise.resolve(3);
}

// 使用 for-await-of
(async () => {
  for await (const value of asyncGenerator()) {
    console.log(value);
  }
})();

// 实际应用：分页获取数据
async function* fetchPaginatedData(url, maxPages = 5) {
  let page = 1;
  let hasMore = true;

  while (hasMore && page <= maxPages) {
    const response = await fetch(`${url}?page=${page}`);
    const data = await response.json();

    yield data;

    hasMore = data.hasMore;
    page++;
  }
}

// 使用
(async () => {
  for await (const pageData of fetchPaginatedData('/api/posts')) {
    console.log('页面数据:', pageData);
  }
})();
```

### Promise.prototype.finally()

`finally()` 方法无论 Promise 最终状态如何都会执行。

```javascript
fetch('/api/data')
  .then(response => response.json())
  .then(data => console.log(data))
  .catch(error => console.error(error))
  .finally(() => {
    console.log('请求完成');
    // 清理工作，如关闭 loading
    hideLoading();
  });

// 实际应用：资源管理
async function fetchDataWithCleanup(url) {
  const controller = new AbortController();

  try {
    const response = await fetch(url, {
      signal: controller.signal
    });
    return await response.json();
  } catch (error) {
    console.error('请求失败:', error);
    throw error;
  } finally {
    // 无论成功失败都清理资源
    controller.abort();
    console.log('清理完成');
  }
}

// finally 不接收参数，无法知道是成功还是失败
fetch('/api/data')
  .finally(() => {
    console.log('这里不知道 Promise 的状态');
  })
  .then(result => console.log('成功:', result))
  .catch(error => console.log('失败:', error));
```

### 对象展开运算符

在对象字面量中使用展开运算符。

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// 合并对象
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 覆盖属性
const overwritten = { ...obj1, a: 10, ...obj2 };
console.log(overwritten); // { a: 10, b: 2, c: 3, d: 4 }

// 实际应用：状态更新
const initialState = {
  user: {
    name: '张三',
    age: 25
  },
  theme: 'light'
};

const updatedState = {
  ...initialState,
  user: {
    ...initialState.user,
    age: 26
  }
};

console.log(updatedState);
// { user: { name: '张三', age: 26 }, theme: 'light' }

// 创建新对象添加属性
const newUser = { ...obj1, e: 5 };
console.log(newUser); // { a: 1, b: 2, e: 5 }
```

### 对象剩余属性

在对象解构中使用剩余属性收集。

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  phone: '13800138000',
  address: '北京'
};

// 提取部分属性，剩余属性收集到新对象
const { name, age, ...rest } = user;

console.log(name); // '张三'
console.log(age); // 25
console.log(rest); // { email: '...', phone: '...', address: '...' }

// 实际应用：更新表单数据
function updateFormData(originalData, updates) {
  return {
    ...originalData,
    ...updates
  };
}

const formData = {
  username: 'zhangsan',
  password: '123456',
  email: 'zhangsan@example.com'
};

const updates = {
  email: 'newemail@example.com',
  phone: '13800138000'
};

const newFormData = updateFormData(formData, updates);
console.log(newFormData);
// { username: 'zhangsan', password: '123456', email: 'newemail@...', phone: '...' }
```

## ES10 (2019) 新特性

### Array.prototype.flat()

`flat()` 方法将嵌套数组拉平。

```javascript
// 基本用法
const nested = [1, [2, [3, [4]]]];

console.log(nested.flat()); // [1, 2, [3, [4]]] (默认拉平一层)
console.log(nested.flat(2)); // [1, 2, 3, [4]] (拉平两层)
console.log(nested.flat(Infinity)); // [1, 2, 3, 4] (拉平所有层级)

// 实际应用：处理嵌套数据
const apiResponse = {
  users: [
    {
      id: 1,
      posts: [
        { id: 101, comments: [1, 2, 3] },
        { id: 102, comments: [4, 5] }
      ]
    }
  ]
};

// 提取所有评论 ID
const allCommentIds = apiResponse.users
  .flatMap(user => user.posts)
  .flatMap(post => post.comments);

console.log(allCommentIds); // [1, 2, 3, 4, 5]
```

### Array.prototype.flatMap()

`flatMap()` 方法先映射再拉平，等价于 `map()` 后调用 `flat(1)`。

```javascript
// 基本用法
const numbers = [1, 2, 3, 4];

const doubled = numbers.flatMap(x => [x * 2]);
console.log(doubled); // [2, 4, 6, 8]

// 与 map 的对比
const mapped = numbers.map(x => [x * 2]);
console.log(mapped); // [[2], [4], [6], [8]]

// 实际应用：拆分句子为单词
const sentences = [
  'Hello world',
  'JavaScript is awesome',
  'ES10 features'
];

const words = sentences.flatMap(sentence => sentence.split(' '));
console.log(words); // ['Hello', 'world', 'JavaScript', 'is', 'awesome', 'ES10', 'features']

// 展开并过滤
const filtered = numbers.flatMap(x => x > 2 ? [x] : []);
console.log(filtered); // [3, 4]
```

### Object.fromEntries()

`Object.fromEntries()` 方法将键值对数组转换为对象。

```javascript
// 基本用法
const entries = [
  ['name', '张三'],
  ['age', 25],
  ['email', 'zhangsan@example.com']
];

const obj = Object.fromEntries(entries);
console.log(obj); // { name: '张三', age: 25, email: '...' }

// 转换 Map 为对象
const map = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3]
]);

const mapObj = Object.fromEntries(map);
console.log(mapObj); // { a: 1, b: 2, c: 3 }

// 与 Object.entries 配合使用
const original = { name: '李四', age: 30 };
const newObj = Object.fromEntries(
  Object.entries(original)
    .filter(([key]) => key !== 'age')
);

console.log(newObj); // { name: '李四' }

// 实际应用：查询参数处理
const searchParams = new URLSearchParams('?name=张三&age=25&city=北京');
const params = Object.fromEntries(searchParams);
console.log(params); // { name: '张三', age: '25', city: '北京' }
```

### String.prototype.trimStart() 和 trimEnd()

分别去除字符串开头和结尾的空白字符。

```javascript
const str = '  Hello World  ';

// trimStart - 去除开头空白
console.log(str.trimStart()); // 'Hello World  '

// trimEnd - 去除结尾空白
console.log(str.trimEnd()); // '  Hello World'

// trim - 去除两端空白
console.log(str.trim()); // 'Hello World'

// 实际应用：清理用户输入
function cleanInput(input) {
  return input.trim().replace(/\s+/g, ' ');
}

console.log(cleanInput('  hello    world  ')); // 'hello world'
```

### 可选的 catch 绑定

允许在 catch 块中省略错误参数。

```javascript
// 之前必须写错误参数
try {
  JSON.parse('invalid json');
} catch (error) {
  console.log('解析失败'); // error 未使用
}

// 现在可以省略
try {
  JSON.parse('invalid json');
} catch {
  console.log('解析失败');
}

// 实际应用：清理资源
async function processData() {
  let connection;

  try {
    connection = await connectToDatabase();
    await connection.query('SELECT * FROM users');
  } catch {
    // 不关心具体错误，只需清理资源
    console.log('操作失败，清理资源');
  } finally {
    if (connection) {
      await connection.close();
    }
  }
}
```

### Function.prototype.toString() 修订

现在 `toString()` 返回原始代码，包括注释和空格。

```javascript
function example() {
  // 这是一个注释
  return 'Hello';
}

console.log(example.toString());
/*
function example() {
  // 这是一个注释
  return 'Hello';
}
*/

// 箭头函数
const arrow = () => 'World';
console.log(arrow.toString()); // "() => 'World'"
```

## ES11 (2020) 新特性

### BigInt

`BigInt` 是一种新的数字类型，可以表示任意大的整数。

```javascript
// 创建 BigInt
const bigNumber = 9007199254740991n; // 在数字后面加 n
const bigNumber2 = BigInt('9007199254740992'); // 使用 BigInt() 函数

console.log(bigNumber); // 9007199254740991n
console.log(typeof bigNumber); // 'bigint'

// BigInt 运算
const a = 100n;
const b = 25n;

console.log(a + b); // 125n
console.log(a - b); // 75n
console.log(a * b); // 2500n
console.log(a / b); // 4n (整数除法)
console.log(a % b); // 0n

// BigInt 与普通数字混用会报错
// console.log(100n + 50); // TypeError

// 需要显式转换
console.log(100n + BigInt(50)); // 150n
console.log(Number(100n) + 50); // 150

// 实际应用：大整数计算
function factorial(n) {
  if (n <= 1n) return 1n;
  return n * factorial(n - 1n);
}

console.log(factorial(20n)); // 2432902008176640000n
console.log(factorial(50n)); // 30414093201713378043612608166064768844377641568960512000000000000n

// 处理大数 ID
const bigId = 9007199254740991n;
console.log(bigId + 1n); // 9007199254740992n (不会丢失精度)
```

### 可选链操作符 (?.)

可选链操作符允许安全地访问嵌套对象属性。

```javascript
const user = {
  name: '张三',
  address: {
    city: '北京',
    street: '朝阳区'
  }
};

// 基本用法
console.log(user?.name); // '张三'
console.log(user.address?.city); // '北京'

// 安全访问深层嵌套属性
console.log(user?.profile?.age); // undefined
// 之前需要多次判断
console.log(user && user.profile && user.profile.age); // undefined

// 可选链调用方法
const obj = {
  fn: () => console.log('Called')
};

obj?.fn(); // 'Called'
obj2?.fn(); // 不会报错，返回 undefined

// 可选链访问数组元素
const arr = [1, 2, 3];
console.log(arr?.[0]); // 1
console.log(arr2?.[0]); // undefined

// 实际应用：API 响应处理
function getUserCity(user) {
  return user?.address?.city ?? '未知';
}

console.log(getUserCity(user)); // '北京'
console.log(getUserCity({})); // '未知'
```

### 空值合并运算符 (??)

空值合并运算符用于为 null 或 undefined 提供默认值。

```javascript
// 基本用法
const value1 = null ?? 'default';
console.log(value1); // 'default'

const value2 = undefined ?? 'default';
console.log(value2); // 'default'

const value3 = '' ?? 'default';
console.log(value3); // '' (不是 null/undefined)
const value4 = 0 ?? 'default';
console.log(value4); // 0 (不是 null/undefined)
const value5 = false ?? 'default';
console.log(value5); // false (不是 null/undefined)

// 与 || 的区别
console.log('' || 'default'); // 'default'
console.log('' ?? 'default'); // ''

console.log(0 || 'default'); // 'default'
console.log(0 ?? 'default'); // 0

console.log(false || 'default'); // 'default'
console.log(false ?? 'default'); // false

// 实际应用：函数参数默认值
function greet(name = 'Guest') {
  // 使用 ?? 提供默认值
  const userName = name ?? 'Guest';
  return `Hello, ${userName}!`;
}

console.log(greet()); // 'Hello, Guest!'
console.log(greet('张三')); // 'Hello, 张三!'

// 配置对象默认值
const config = {
  theme: null,
  language: '',
  debug: false
};

const finalConfig = {
  theme: config.theme ?? 'light',
  language: config.language ?? 'zh-CN',
  debug: config.debug ?? false
};

console.log(finalConfig);
// { theme: 'light', language: '', debug: false }
```

### Promise.allSettled()

`Promise.allSettled()` 返回所有 Promise 的结果，无论成功失败。

```javascript
const promises = [
  Promise.resolve('成功1'),
  Promise.reject('失败1'),
  Promise.resolve('成功2')
];

// Promise.allSettled 等待所有完成
Promise.allSettled(promises).then(results => {
  console.log(results);
  /*
  [
    { status: 'fulfilled', value: '成功1' },
    { status: 'rejected', reason: '失败1' },
    { status: 'fulfilled', value: '成功2' }
  ]
  */
});

// 实际应用：批量操作
async function uploadFiles(files) {
  const uploadPromises = files.map(file => uploadFile(file));
  const results = await Promise.allSettled(uploadPromises);

  const successful = results.filter(r => r.status === 'fulfilled');
  const failed = results.filter(r => r.status === 'rejected');

  console.log(`成功: ${successful.length}, 失败: ${failed.length}`);

  return {
    successful: successful.map(r => r.value),
    failed: failed.map(r => r.reason)
  };
}

// 使用
uploadFiles(['file1.jpg', 'file2.jpg', 'file3.jpg'])
  .then(results => console.log(results));
```

### String.prototype.matchAll()

`matchAll()` 方法返回所有匹配正则表达式的迭代器。

```javascript
const str = 'test1 test2 test3';

const regex = /test(\d)/g;

// 使用 matchAll
const matches = str.matchAll(regex);

for (const match of matches) {
  console.log(match);
  // ['test1', '1', index: 0, input: 'test1 test2 test3']
  // ['test2', '2', index: 6, input: 'test1 test2 test3']
  // ['test3', '3', index: 12, input: 'test1 test2 test3']
}

// 转换为数组
const matchArray = [...str.matchAll(regex)];
console.log(matchArray);

// 实际应用：提取所有邮箱
const text = '联系邮箱: test1@example.com, test2@example.com';
const emailRegex = /(\w+@\w+\.\w+)/g;

const emails = [...text.matchAll(emailRegex)].map(match => match[1]);
console.log(emails); // ['test1@example.com', 'test2@example.com']
```

### import.meta

`import.meta` 提供当前模块的元数据。

```javascript
// 获取模块 URL
console.log(import.meta.url); // 当前模块的 URL

// 实际应用：动态加载资源
const loadResource = async (path) => {
  const baseUrl = new URL(import.meta.url);
  const resourceUrl = new URL(path, baseUrl);
  const response = await fetch(resourceUrl);
  return response.json();
};

// 使用
const data = await loadResource('./data.json');

// 条件加载
if (import.meta.env?.NODE_ENV === 'development') {
  console.log('开发模式');
}
```

### globalThis

`globalThis` 提供统一的全局对象访问方式。

```javascript
// 在浏览器中
console.log(globalThis === window); // true

// 在 Node.js 中
console.log(globalThis === global); // true

// 在 Web Worker 中
console.log(globalThis === self); // true

// 实际应用：编写跨环境代码
function getGlobal() {
  return globalThis;
}

// 设置全局变量
globalThis.myGlobal = 'Hello';

// 访问全局变量
console.log(globalThis.myGlobal); // 'Hello'
```

## ES12 (2021) 新特性

### String.prototype.replaceAll()

`replaceAll()` 方法替换所有匹配的字符串。

```javascript
// 基本用法
const str = 'hello world, hello universe';

// 使用 replaceAll 替换所有匹配
const result = str.replaceAll('hello', 'hi');
console.log(result); // 'hi world, hi universe'

// 与 replace 的对比
console.log(str.replace('hello', 'hi')); // 'hi world, hello universe' (只替换第一个)

// 使用正则表达式
console.log(str.replaceAll(/hello/g, 'hi')); // 'hi world, hi universe'

// 实际应用：模板替换
const template = 'Hello {{name}}, welcome to {{place}}!';

const data = { name: '张三', place: '北京' };

const replaced = template.replaceAll(/\{\{(\w+)\}\}/g, (match, key) => {
  return data[key] || match;
});

console.log(replaced); // 'Hello 张三, welcome to 北京!'
```

### Promise.any()

`Promise.any()` 返回第一个成功的 Promise。

```javascript
const promises = [
  Promise.reject('错误1'),
  Promise.reject('错误2'),
  Promise.resolve('成功'),
  Promise.reject('错误3')
];

// 返回第一个成功的 Promise
Promise.any(promises).then(result => {
  console.log(result); // '成功'
});

// 全部失败时抛出 AggregateError
Promise.any([
  Promise.reject('错误1'),
  Promise.reject('错误2')
]).catch(error => {
  console.log(error instanceof AggregateError); // true
  console.log(error.errors); // ['错误1', '错误2']
});

// 实际应用：多源数据获取
async function fetchFromMultipleSources(sources) {
  const promises = sources.map(source =>
    fetch(source).catch(() => null)
  );

  try {
    const response = await Promise.any(promises);
    if (response && response.ok) {
      return await response.json();
    }
    throw new Error('所有源都不可用');
  } catch (error) {
    throw new Error(`所有源都失败: ${error.message}`);
  }
}

// 使用
const sources = [
  'https://api1.example.com/data',
  'https://api2.example.com/data',
  'https://api3.example.com/data'
];

fetchFromMultipleSources(sources)
  .then(data => console.log(data))
  .catch(error => console.error(error));
```

### 逻辑赋值运算符

`??=`、`||=`、`&&=` 等逻辑赋值运算符。

```javascript
// ??= - 空值合并赋值
let a = null;
a ??= 'default';
console.log(a); // 'default'

a ??= 'new default';
console.log(a); // 'default' (不会覆盖)

// ||= - 逻辑或赋值
let b = '';
b ||= 'default';
console.log(b); // 'default' (空字符串是 falsy)

// &&= - 逻辑与赋值
let c = 'hello';
c &&= 'world';
console.log(c); // 'world'

c &&= 'new world';
console.log(c); // 'new world'

// 实际应用：配置默认值
const config = {
  theme: null,
  language: '',
  debug: false
};

config.theme ??= 'light';
config.language ||= 'en';
config.debug &&= true;

console.log(config);
// { theme: 'light', language: 'en', debug: true }
```

### 数字分隔符

使用下划线分隔数字，提高可读性。

```javascript
// 基本用法
const billion = 1_000_000_000;
console.log(billion); // 1000000000

const phone = 138_0013_8000;
console.log(phone); // 13800138000

// 与其他进制配合
const binary = 0b1010_0001_1000_0101;
const hex = 0xA0_8A_05;

// 实际应用：配置常量
const CONFIG = {
  MAX_SIZE: 100_000,
  TIMEOUT: 30_000,
  PORT: 8_080,
  VERSION: 1_2_3
};

// 位操作更清晰
const bitmask = 0b1111_1111;
const rgba = 0xFF_FF_FF_FF;
```

### 顶层 await

允许在模块顶层使用 await。

```javascript
// 顶层使用 await
const data = await fetch('/api/config').then(r => r.json());
console.log('配置加载完成:', data);

// 动态导入
const module = await import('./module.js');
module.doSomething();

// 条件导入
let module;
if (condition) {
  module = await import('./module1.js');
} else {
  module = await import('./module2.js');
}

// 实际应用：配置文件
// config.js
export const config = await fetch('/api/config')
  .then(response => response.json())
  .catch(() => ({
    defaultTheme: 'light',
    language: 'zh-CN'
  }));

// 其他模块可以直接导入配置
import { config } from './config.js';
console.log(config);
```

## 实际应用示例

### 现代化 JavaScript 代码示例

```javascript
// ES7-ES12 特性综合使用
class UserService {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.cache = new Map();
  }

  // 使用可选链和空值合并
  async getUser(userId) {
    const cacheKey = `user_${userId}`;

    // 检查缓存
    const cached = this.cache.get(cacheKey);
    if (cached) {
      return cached;
    }

    try {
      // 使用顶层 await 和 fetch
      const response = await fetch(`${this.baseUrl}/users/${userId}`);

      if (!response.ok) {
        throw new Error(`HTTP 错误: ${response.status}`);
      }

      const user = await response.json();

      // 缓存结果
      this.cache.set(cacheKey, user);

      return user;
    } catch (error) {
      console.error('获取用户失败:', error);
      throw error;
    }
  }

  // 使用 Promise.allSettled 批量获取
  async getUsersBatch(userIds) {
    const promises = userIds.map(id => this.getUser(id));

    const results = await Promise.allSettled(promises);

    return results.map((result, index) => ({
      userId: userIds[index],
      success: result.status === 'fulfilled',
      data: result.status === 'fulfilled' ? result.value : null,
      error: result.status === 'rejected' ? result.reason.message : null
    }));
  }

  // 使用 matchAll 提取数据
  extractEmails(text) {
    const emailRegex = /(\w+@\w+\.\w+)/g;
    return [...text.matchAll(emailRegex)].map(match => match[1]);
  }

  // 使用replaceAll 处理模板
  fillTemplate(template, data) {
    return template.replaceAll(/\{\{(\w+)\}\}/g, (match, key) => {
      return data[key] ?? match;
    });
  }

  // 使用 BigInt 处理大数 ID
  async getLargeIdEntity(bigId) {
    const id = BigInt(bigId);
    const response = await fetch(`${this.baseUrl}/entities/${id}`);
    return response.json();
  }
}

// 使用示例
const userService = new UserService('https://api.example.com');

async function example() {
  // 获取单个用户
  const user = await userService.getUser(1);
  console.log('用户:', user);

  // 批量获取用户
  const results = await userService.getUsersBatch([1, 2, 3]);
  console.log('批量结果:', results);

  // 提取邮箱
  const text = '联系: test1@example.com, test2@example.com';
  const emails = userService.extractEmails(text);
  console.log('邮箱:', emails);

  // 填充模板
  const template = 'Hello {{name}}, your balance is {{balance}}';
  const filled = userService.fillTemplate(template, { name: '张三', balance: '1000' });
  console.log('填充结果:', filled);
}

// 数字分隔符示例
const config = {
  MAX_USERS: 10_000,
  TIMEOUT_MS: 30_000,
  PORT: 3_000,
  LARGE_NUMBER: 9_007_199_254_740_991n
};

console.log('配置:', config);
```

## ES13 (2022) 新特性

### Array.prototype.at()

`at()` 方法支持正负索引，更简洁地访问数组元素。

```javascript
// 基本用法
const arr = [10, 20, 30, 40, 50];

// 正索引（从 0 开始）
console.log(arr.at(0)); // 10
console.log(arr.at(2)); // 30

// 负索引（从 -1 开始）
console.log(arr.at(-1)); // 50 (最后一个元素)
console.log(arr.at(-2)); // 40 (倒数第二个)

// 与传统索引的对比
console.log(arr[arr.length - 1]); // 50 (传统方式)
console.log(arr.at(-1)); // 50 (更简洁)

// 实际应用：访问链式方法的最后一个元素
function getLastElement(arr) {
  return arr.at(-1);
}

console.log(getLastElement([1, 2, 3])); // 3

// 字符串也支持 at()
const str = 'Hello';
console.log(str.at(-1)); // 'o'
```

### Object.hasOwn()

`Object.hasOwn()` 是 `Object.prototype.hasOwnProperty()` 的安全替代方法。

```javascript
// 基本用法
const obj = { name: '张三', age: 25 };

console.log(Object.hasOwn(obj, 'name')); // true
console.log(Object.hasOwn(obj, 'age')); // true
console.log(Object.hasOwn(obj, 'toString')); // false (继承属性)

// 与 hasOwnProperty 的对比
const proto = { inherited: 'value' };
const obj2 = Object.create(proto);
obj2.own = 'own value';

console.log(Object.hasOwn(obj2, 'inherited')); // false
console.log(Object.prototype.hasOwnProperty.call(obj2, 'inherited')); // false

console.log(obj2.hasOwnProperty('own')); // true
console.log(Object.hasOwn(obj2, 'own')); // true

// 安全性：不会受到原型链污染影响
const obj3 = Object.create(null);
obj3.toString = 'custom';
console.log(Object.hasOwn(obj3, 'toString')); // true

// 实际应用：检查对象自有属性
function printOwnProperties(obj) {
  Object.keys(obj).forEach(key => {
    if (Object.hasOwn(obj, key)) {
      console.log(`${key}: ${obj[key]}`);
    }
  });
}

printOwnProperties({ name: '李四', age: 30 });
```

### 类字段声明

支持在类中直接声明公有和私有字段。

```javascript
// 类字段声明
class User {
  // 公有字段
  name = 'Unknown';
  age = 0;

  // 私有字段（# 前缀）
  #password = '';
  #id = Math.random();

  constructor(name, age, password) {
    this.name = name;
    this.age = age;
    this.#password = password;
  }

  // 私有方法
  #encryptPassword() {
    return this.#password.split('').reverse().join('');
  }

  // 公有方法
  getInfo() {
    return {
      name: this.name,
      age: this.age,
      id: this.#id
    };
  }

  // 静态私有字段
  static #count = 0;
  static #maxUsers = 100;

  static createUser(name, age, password) {
    if (User.#count >= User.#maxUsers) {
      throw new Error('达到最大用户数');
    }
    User.#count++;
    return new User(name, age, password);
  }
}

// 使用
const user = User.createUser('张三', 25, 'password123');
console.log(user.getInfo());
// user.#password; // SyntaxError (私有字段外部无法访问)

console.log(User.#count); // SyntaxError (静态私有字段外部无法访问)
```

### await 顶层支持

在模块顶层直接使用 await（已在 ES12 中提到，但 ES13 做了标准化）。

```javascript
// config.js
export const config = await fetch('/api/config')
  .then(res => res.json());

export const database = await connectDatabase();

// main.js
import { config, database } from './config.js';
console.log(config);
```

### 正则表达式匹配索引

正则表达式的 `d` 标志提供匹配索引信息。

```javascript
// 基本用法
const regex = /a(b)c/d;
const match = regex.exec('abc');

console.log(match);
// [
//   'abc',
//   'b',
//   index: 0,
//   input: 'abc',
//   groups: undefined,
//   indices: [
//     [0, 3],    // 整个匹配的位置
//     [1, 2]     // 捕获组的位置
//   ]
// ]

// 实际应用：高亮匹配文本
function highlightMatches(text, regex) {
  const match = regex.exec(text);
  if (!match) return text;

  const indices = match.indices[0];
  const before = text.slice(0, indices[0]);
  const matched = text.slice(indices[0], indices[1]);
  const after = text.slice(indices[1]);

  return `${before}<mark>${matched}</mark>${after}`;
}

const text = 'Hello, JavaScript!';
const pattern = /JavaScript/d;
console.log(highlightMatches(text, pattern));
// 'Hello, <mark>JavaScript</mark>!'
```

### String.prototype.at()

字符串也支持 `at()` 方法（与数组相同）。

```javascript
const str = 'Hello World';

console.log(str.at(0)); // 'H'
console.log(str.at(-1)); // 'd'
console.log(str.at(6)); // 'W'
```

## ES14 (2023) 新特性

### 数组从尾部查找

新增从数组尾部开始查找的方法。

```javascript
// findLast - 从后往前查找满足条件的元素
const numbers = [1, 2, 3, 4, 5, 4, 3, 2, 1];

const lastEven = numbers.findLast(num => num % 2 === 0);
console.log(lastEven); // 2 (最后一个偶数)

// findLastIndex - 从后往前查找满足条件的元素的索引
const lastEvenIndex = numbers.findLastIndex(num => num % 2 === 0);
console.log(lastEvenIndex); // 7

// 与 find/findIndex 对比
const firstEven = numbers.find(num => num % 2 === 0);
const firstEvenIndex = numbers.findIndex(num => num % 2 === 0);

console.log(firstEven); // 2
console.log(firstEvenIndex); // 1

// 实际应用：查找最近的错误日志
const logs = [
  { time: '10:00', level: 'info', message: 'Started' },
  { time: '10:01', level: 'warning', message: 'Slow response' },
  { time: '10:02', level: 'error', message: 'Failed to connect' },
  { time: '10:03', level: 'info', message: 'Retrying' },
  { time: '10:04', level: 'error', message: 'Connection timeout' },
  { time: '10:05', level: 'info', message: 'Success' }
];

const lastError = logs.findLast(log => log.level === 'error');
console.log(lastError);
// { time: '10:04', level: 'error', message: 'Connection timeout' }
```

### Hashbang 语法

支持 JavaScript 文件的 Hashbang 语法（shebang）。

```javascript
#!/usr/bin/env node

// script.js
console.log('Hello from Hashbang script!');

// 可以直接执行
// chmod +x script.js
// ./script.js
```

### Symbol 作为 WeakMap 键

`WeakMap` 现在可以使用 Symbol 作为键。

```javascript
// ES14 之前：WeakMap 只能用对象作为键
// ES14：Symbol 也可以作为键
const weakMap = new WeakMap();

const sym1 = Symbol('key1');
const sym2 = Symbol('key2');

weakMap.set(sym1, 'value1');
weakMap.set(sym2, 'value2');

console.log(weakMap.get(sym1)); // 'value1'
console.log(weakMap.get(sym2)); // 'value2'

// 实际应用：使用 Symbol 标识符作为 WeakMap 键
const createCache = () => {
  const cache = new WeakMap();
  const key = Symbol('cache');

  return {
    set(value) {
      cache.set(key, value);
    },
    get() {
      return cache.get(key);
    }
  };
};

const myCache = createCache();
myCache.set({ data: 'test' });
console.log(myCache.get());
```

### 通过复制更改数组

新增 `toReversed()`、`toSorted()`、`with()`、`toSpliced()` 方法，返回新数组而不修改原数组。

```javascript
// toReversed() - 反转数组，返回新数组
const arr = [1, 2, 3, 4, 5];
const reversed = arr.toReversed();

console.log(arr); // [1, 2, 3, 4, 5] (原数组不变)
console.log(reversed); // [5, 4, 3, 2, 1]

// toSorted() - 排序数组，返回新数组
const numbers = [3, 1, 4, 1, 5];
const sorted = numbers.toSorted();

console.log(numbers); // [3, 1, 4, 1, 5] (原数组不变)
console.log(sorted); // [1, 1, 3, 4, 5]

// 支持比较函数
const users = [
  { name: '张三', age: 25 },
  { name: '李四', age: 30 },
  { name: '王五', age: 20 }
];

const sortedUsers = users.toSorted((a, b) => a.age - b.age);
console.log(sortedUsers);
// [{ name: '王五', age: 20 }, { name: '张三', age: 25 }, { name: '李四', age: 30 }]

// with() - 返回指定索引被替换的新数组
const arr2 = [1, 2, 3, 4, 5];
const arr3 = arr2.with(2, 99);

console.log(arr2); // [1, 2, 3, 4, 5] (原数组不变)
console.log(arr3); // [1, 2, 99, 4, 5]

// 支持负索引
const arr4 = arr2.with(-1, 100);
console.log(arr4); // [1, 2, 3, 4, 100]

// toSpliced() - 返回删除/插入元素后的新数组
const arr5 = [1, 2, 3, 4, 5];
const arr6 = arr5.toSpliced(1, 2, 99, 100);

console.log(arr5); // [1, 2, 3, 4, 5] (原数组不变)
console.log(arr6); // [1, 99, 100, 4, 5]

// 实际应用：不可变数据操作
const state = {
  items: [1, 2, 3],
  users: [
    { id: 1, name: '张三' },
    { id: 2, name: '李四' }
  ]
};

// 创建新的状态而不修改原状态
const newState = {
  ...state,
  items: state.items.toReversed(),
  users: state.users.toSorted((a, b) => a.name.localeCompare(b.name))
};

console.log(newState);
```

## ES15 (2024) 新特性

### Promise.withResolvers()

更简洁地创建 Promise 及其解析和拒绝函数。

```javascript
// 基本用法
const { promise, resolve, reject } = Promise.withResolvers();

// 在异步操作中使用
setTimeout(() => {
  resolve('操作完成');
}, 1000);

promise.then(result => console.log(result));

// 对比传统方式
function createPromise() {
  let resolve, reject;
  const promise = new Promise((res, rej) => {
    resolve = res;
    reject = rej;
  });

  return { promise, resolve, reject };
}

// 实际应用：事件处理器
function waitForEvent(element, eventType) {
  const { promise, resolve } = Promise.withResolvers();

  element.addEventListener(eventType, resolve, { once: true });

  return promise;
}

// 使用
const button = document.querySelector('button');
waitForEvent(button, 'click').then(() => {
  console.log('按钮被点击');
});

// 异步迭代器
async function* asyncGenerator() {
  for (let i = 0; i < 3; i++) {
    yield new Promise(resolve => {
      setTimeout(() => resolve(i), 1000);
    });
  }
}

const { promise: donePromise, resolve: done } = Promise.withResolvers();

(async () => {
  for await (const value of asyncGenerator()) {
    console.log(value);
  }
  done();
})();

await donePromise;
console.log('迭代完成');
```

### Object.groupBy()

按照条件对对象进行分组。

```javascript
// 基本用法
const people = [
  { name: '张三', age: 25 },
  { name: '李四', age: 30 },
  { name: '王五', age: 25 },
  { name: '赵六', age: 30 }
];

// 按年龄分组
const groupedByAge = Object.groupBy(people, person => person.age);

console.log(groupedByAge);
/*
{
  25: [
    { name: '张三', age: 25 },
    { name: '王五', age: 25 }
  ],
  30: [
    { name: '李四', age: 30 },
    { name: '赵六', age: 30 }
  ]
}
*/

// 按名字长度分组
const groupedByNameLength = Object.groupBy(
  people,
  person => person.name.length
);

console.log(groupedByNameLength);
// { 2: [...], 3: [...] }

// 实际应用：数据分类
const products = [
  { name: 'iPhone', price: 999, category: 'electronics' },
  { name: 'MacBook', price: 1999, category: 'electronics' },
  { name: 'T-shirt', price: 29, category: 'clothing' },
  { name: 'Jeans', price: 79, category: 'clothing' }
];

const productsByCategory = Object.groupBy(
  products,
  product => product.category
);

console.log(productsByCategory);
```

### Map.groupBy()

Map 版本的 groupBy 方法。

```javascript
// 基本用法
const data = [
  { id: 1, type: 'A' },
  { id: 2, type: 'B' },
  { id: 3, type: 'A' },
  { id: 4, type: 'C' }
];

const grouped = Map.groupBy(data, item => item.type);

console.log(grouped);
// Map {
//   'A' => [{ id: 1, type: 'A' }, { id: 3, type: 'A' }],
//   'B' => [{ id: 2, type: 'B' }],
//   'C' => [{ id: 4, type: 'C' }]
// }

// 访问分组
console.log(grouped.get('A')); // [{ id: 1, type: 'A' }, { id: 3, type: 'A' }]

// 实际应用：动态键分组
const users = [
  { name: 'Alice', role: 'admin' },
  { name: 'Bob', role: 'user' },
  { name: 'Charlie', role: 'admin' },
  { name: 'David', role: 'guest' }
];

const groupedByRole = Map.groupBy(users, user => user.role);

// 遍历分组
for (const [role, usersInRole] of groupedByRole) {
  console.log(`${role}:`, usersInRole.map(u => u.name).join(', '));
}
// admin: Alice, Charlie
// user: Bob
// guest: David
```

###Temporal API（提案中）

日期时间处理的新 API（仍在提案中）。

```javascript
// 注意：这是提案中的 API，可能还没有完全标准化

// 创建 Temporal 对象
const now = Temporal.Now.plainDateTimeISO();
const date = Temporal.PlainDate.from('2024-05-14');
const time = Temporal.PlainTime.from('14:30:00');

// 日期计算
const futureDate = date.add({ days: 7, months: 1 });
console.log(futureDate.toString()); // '2024-06-21'

// 时间比较
const date1 = Temporal.PlainDate.from('2024-05-14');
const date2 = Temporal.PlainDate.from('2024-05-20');

console.log(date1.until(date2).days); // 6

// 时区处理
const zonedDateTime = Temporal.Now.zonedDateTimeISO('Asia/Shanghai');
console.log(zonedDateTime.toString());

// 实际应用：日期范围计算
function getDaysInMonth(year, month) {
  const date = Temporal.PlainDate.from({ year, month });
  return date.daysInMonth;
}

console.log(getDaysInMonth(2024, 2)); // 29 (闰年)
console.log(getDaysInMonth(2023, 2)); // 28
```

## ES16 (2025) 新特性

### Iterator Helper Methods

为迭代器添加辅助方法，如 map、filter、take 等。

```javascript
// 基本用法
const array = [1, 2, 3, 4, 5];

// 创建迭代器
const iterator = array.values();

// 使用辅助方法
const mapped = iterator.map(x => x * 2);
console.log([...mapped]); // [2, 4, 6, 8, 10]

// 过滤
const filtered = array.values().filter(x => x % 2 === 0);
console.log([...filtered]); // [2, 4]

// 取前 N 个元素
const taken = array.values().take(3);
console.log([...taken]); // [1, 2, 3]

// 跳过前 N 个元素
const dropped = array.values().drop(2);
console.log([...dropped]); // [3, 4, 5]

// 扁平化
const nested = [[1, 2], [3, 4], [5, 6]];
const flattened = nested.values().flat();
console.log([...flattened]); // [1, 2, 3, 4, 5, 6]

// 链式调用
const result = array.values()
  .filter(x => x % 2 === 0)
  .map(x => x * 2)
  .take(2);

console.log([...result]); // [4, 8]

// 实际应用：处理大型数据集
function* generateNumbers() {
  let i = 1;
  while (true) {
    yield i++;
  }
}

// 处理无限数据流
const firstTenPrimes = generateNumbers()
  .filter(isPrime)
  .take(10);

console.log([...firstTenPrimes]);

function isPrime(n) {
  if (n < 2) return false;
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}
```

### Set 方法更新

新增 Set 的交集、并集、差集等操作。

```javascript
// 基本用法
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// 交集
const intersection = setA.intersection(setB);
console.log(intersection); // Set {3, 4}

// 并集
const union = setA.union(setB);
console.log(union); // Set {1, 2, 3, 4, 5, 6}

// 差集
const difference = setA.difference(setB);
console.log(difference); // Set {1, 2}

// 对称差集
const symmetricDifference = setA.symmetricDifference(setB);
console.log(symmetricDifference); // Set {1, 2, 5, 6}

// 判断子集
const isSubset = new Set([1, 2]).isSubsetOf(setA);
console.log(isSubset); // true

// 判断超集
const isSuperset = setA.isSupersetOf(new Set([1, 2]));
console.log(isSuperset); // true

// 判断不相交
const isDisjoint = new Set([7, 8]).isDisjointFrom(setA);
console.log(isDisjoint); // true

// 实际应用：权限管理
const adminPermissions = new Set(['read', 'write', 'delete', 'admin']);
const userPermissions = new Set(['read', 'write']);

// 检查用户是否具有所有必需权限
const requiredPermissions = new Set(['read', 'write']);
const hasAllRequired = requiredPermissions.isSubsetOf(userPermissions);
console.log(hasAllRequired); // true

// 获取用户缺少的权限
const missingPermissions = requiredPermissions.difference(userPermissions);
console.log([...missingPermissions]); // []

// 获取用户拥有的额外权限
const extraPermissions = userPermissions.difference(requiredPermissions);
console.log([...extraPermissions]); // []
```

### RegExp 修饰符 v

新的正则表达式修饰符，提供更好的 Unicode 支持。

```javascript
// 使用 v 修饰符
const regex = /p{Script=Hiragana}/v;

// 更好的字符类匹配
const regex2 = /[\p{Emoji}\p{Emoji_Presentation}]/v;

// 匹配表情符号
const text = 'Hello 👋 World 🌍';
const emojis = [...text.matchAll(/[\p{Emoji}\p{Emoji_Presentation}]/gv)];
console.log(emojis.map(m => m[0])); // ['👋', '🌍']

// 更精确的字符属性
const regex3 = /\p{Script_Extensions=Han}/v;
const chineseText = 'Hello 你好 World';
const chineseChars = [...chineseText.matchAll(regex3)];
console.log(chineseChars.map(m => m[0])); // ['你', '好']

// 实际应用：文本处理
function removeEmojis(text) {
  return text.replace(/[\p{Emoji}\p{Emoji_Presentation}]/gv, '');
}

const message = 'Great job! 🎉 Well done! 👏';
const cleanMessage = removeEmojis(message);
console.log(cleanMessage); // 'Great job!  Well done! '
```

### String.prototype.isWellFormed()

检查字符串是否是格式良好的 UTF-16。

```javascript
// 基本用法
const validString = 'Hello 世界';
const invalidString = 'Hello \uD800'; // 孤立的代理对

console.log(validString.isWellFormed()); // true
console.log(invalidString.isWellFormed()); // false

// toWellFormed() - 修复格式错误的字符串
const fixedString = invalidString.toWellFormed();
console.log(fixedString); // 'Hello ' (替换了错误的字符)

// 实际应用：输入验证
function validateInput(input) {
  if (!input.isWellFormed()) {
    throw new Error('输入包含无效的 Unicode 字符');
  }
  return input;
}

try {
  validateInput('正常文本'); // 通过
  validateInput('错误文本\uD800'); // 抛出错误
} catch (error) {
  console.error(error.message);
}
```

## 新特性总结对比表

| 版本 | 年份 | 主要新特性 |
|------|------|-----------|
| ES7 | 2016 | Array.includes(), 指数运算符 ** |
| ES8 | 2017 | Object.values/entries, async/await, 字符串填充 |
| ES9 | 2018 | 异步迭代器, Promise.finally, 对象展开 |
| ES10 | 2019 | flat/flatMap, Object.fromEntries, trimStart/End |
| ES11 | 2020 | BigInt, 可选链 ?, 空值合并 ??, globalThis |
| ES12 | 2021 | replaceAll, Promise.any, 逻辑赋值, 顶层 await |
| ES13 | 2022 | at(), Object.hasOwn, 私有类字段, 正则索引 |
| ES14 | 2023 | findLast/LastIndex, Hashbang, Symbol WeakMap, 不可变数组 |
| ES15 | 2024 | Promise.withResolvers, Object/Map.groupBy, Temporal API |
| ES16 | 2025 | Iterator Helpers, Set 方法, RegExp v, isWellFormed |

## 注意事项

1. **浏览器兼容性**：ES13-ES16 的特性较新，在某些浏览器中可能需要 Polyfill 或转译。

2. **渐进采用**：不要一次性使用所有新特性，应根据项目需求逐步采用。

3. **性能考虑**：某些新特性（如 Iterator Helpers）可能有性能开销，需要权衡。

4. **团队协作**：确保团队成员都熟悉使用的新特性，避免代码理解困难。

5. **工具链支持**：确保构建工具和 linter 支持新语法。

6. **TypeScript 兼容**：注意 TypeScript 版本对新特性的支持。

7. **API 稳定性**：对于仍在提案阶段的特性（如 Temporal API），要谨慎使用。

## 最佳实践

1. **持续学习**：JavaScript 每年都在发展，保持学习新特性。

2. **特性选择**：根据项目需求和目标环境选择合适的特性。

3. **代码一致性**：在项目中保持一致的代码风格和特性使用。

4. **性能监控**：关注新特性对性能的影响，必要时进行优化。

5. **错误处理**：新特性可能引入新的错误类型，要相应调整错误处理。

6. **文档更新**：及时更新项目文档，说明使用的新特性。

7. **测试覆盖**：确保新特性的使用有充分的测试覆盖。

8. **社区反馈**：关注社区对新特性的反馈和使用经验。

## 总结

JavaScript 从 ES7 到 ES16 的演进展示了语言的持续发展：

- **ES7-ES12**：奠定了现代 JavaScript 的基础，引入了 async/await、可选链、空值合并等核心特性。
- **ES13-ES14**：完善了语言细节，如数组查找、类字段、不可变操作等。
- **ES15-ES16**：增强了开发体验，提供了更好的工具和方法。

掌握这些新特性不仅能提高开发效率，还能编写出更优雅、更易维护的代码。在实际开发中，要根据项目需求和团队情况，合理选择和使用这些特性。