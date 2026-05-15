---
title: JavaScript 流程控制语句
description: 深入解析 JavaScript 流程控制语句，包括条件判断、循环语句、异常处理等
pubDate: 2024-01-04
tags: ["JS基础","流程控制"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 流程控制语句

## 概述

JavaScript 流程控制语句是编程的核心基础，它们决定了代码的执行顺序和逻辑流向。通过合理使用流程控制语句，我们可以实现复杂的业务逻辑、条件判断、循环迭代等。掌握流程控制语句是编写高质量 JavaScript 代码的必备技能。

## 核心概念

### 流程控制的分类

JavaScript 的流程控制语句主要分为以下几类：

1. **条件语句**：根据条件决定执行哪段代码
   - if...else
   - switch...case
   - 三元运算符

2. **循环语句**：重复执行某段代码
   - for 循环
   - while 循环
   - do...while 循环
   - for...in 循环
   - for...of 循环

3. **跳转语句**：改变代码执行顺序
   - break
   - continue
   - return

4. **异常处理**：处理运行时错误
   - try...catch...finally
   - throw

## 条件语句

### 1. if...else 语句

#### 基础语法

```javascript
// 单条件判断
let score = 85;
if (score >= 60) {
  console.log('成绩合格'); // 条件成立时执行
} else {
  console.log('成绩不合格'); // 条件不成立时执行
}

// 多条件判断（if-else if-else）
if (score >= 90) {
  console.log('优秀');
} else if (score >= 80) {
  console.log('良好');
} else if (score >= 60) {
  console.log('合格');
} else {
  console.log('不合格');
}

// 嵌套条件判断
let age = 25;
let hasLicense = true;

if (age >= 18) {
  if (hasLicense) {
    console.log('可以开车');
  } else {
    console.log('需要先考驾照');
  }
} else {
  console.log('未到法定年龄');
}
```

#### 最佳实践

```javascript
// 使用布尔表达式简化条件
// 不好的写法
if (isValid === true) {
  // ...
}

// 好的写法
if (isValid) {
  // ...
}

// 提前返回减少嵌套
function processUser(user) {
  // 不好的写法：深层嵌套
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        // 处理用户
        return 'success';
      } else {
        return 'no permission';
      }
    } else {
      return 'user not active';
    }
  } else {
    return 'no user';
  }

  // 好的写法：提前返回
  if (!user) return 'no user';
  if (!user.isActive) return 'user not active';
  if (!user.hasPermission) return 'no permission';

  // 处理用户
  return 'success';
}

// 使用对象映射替代多个 if-else
function getGrade(score) {
  // 不好的写法
  if (score >= 90) return 'A';
  if (score >= 80) return 'B';
  if (score >= 70) return 'C';
  if (score >= 60) return 'D';
  return 'F';

  // 好的写法：使用对象和数组方法
  const gradeRanges = [
    { min: 90, grade: 'A' },
    { min: 80, grade: 'B' },
    { min: 70, grade: 'C' },
    { min: 60, grade: 'D' }
  ];

  const grade = gradeRanges.find(range => score >= range.min);
  return grade ? grade.grade : 'F';
}
```

### 2. switch...case 语句

#### 基础语法

```javascript
// 匹配不同的状态值
let status = 'success';

switch (status) {
  case 'success':
    console.log('操作成功');
    break; // 必须加break，否则会穿透到下一个case
  case 'error':
    console.log('操作失败');
    break;
  case 'loading':
    console.log('加载中...');
    break;
  default:
    console.log('未知状态'); // 所有case不匹配时执行
}

// 使用 case 穿透
let day = 3;
let dayType;

switch (day) {
  case 1:
  case 2:
  case 3:
  case 4:
  case 5:
    dayType = '工作日';
    break;
  case 6:
  case 7:
    dayType = '周末';
    break;
  default:
    dayType = '无效的日期';
}

console.log(dayType); // '工作日'
```

#### 高级用法

```javascript
// 在 case 中使用表达式
let action = 'save';

switch (action) {
  case 'save':
  case 'update':
  case 'delete':
    console.log('数据库操作');
    break;
  case 'read':
    console.log('查询操作');
    break;
  default:
    console.log('未知操作');
}

// 使用 switch 进行类型判断
function getType(value) {
  switch (typeof value) {
    case 'string':
      return '字符串';
    case 'number':
      return '数字';
    case 'boolean':
      return '布尔值';
    case 'object':
      return Array.isArray(value) ? '数组' : '对象';
    default:
      return '其他类型';
  }
}

console.log(getType('hello')); // '字符串'
console.log(getType([1, 2, 3])); // '数组'
```

#### 实际应用

```javascript
// 处理不同的 API 响应状态
function handleApiResponse(response) {
  switch (response.status) {
    case 200:
      console.log('请求成功');
      return response.data;
    case 201:
      console.log('创建成功');
      return response.data;
    case 400:
      console.error('请求错误:', response.message);
      throw new Error('Bad Request');
    case 401:
      console.error('未授权，请重新登录');
      // 跳转到登录页面
      window.location.href = '/login';
      break;
    case 403:
      console.error('无权访问');
      throw new Error('Forbidden');
    case 404:
      console.error('资源不存在');
      throw new Error('Not Found');
    case 500:
      console.error('服务器错误');
      throw new Error('Internal Server Error');
    default:
      console.error('未知状态:', response.status);
      throw new Error('Unknown Status');
  }
}

// 路由处理
function handleRoute(route) {
  switch (route.path) {
    case '/':
      return renderHomePage();
    case '/about':
      return renderAboutPage();
    case '/contact':
      return renderContactPage();
    case '/admin':
      if (route.user && route.user.isAdmin) {
        return renderAdminPage();
      } else {
        return renderAccessDeniedPage();
      }
    default:
      return renderNotFoundPage();
  }
}
```

### 3. 三元运算符

#### 基础语法

```javascript
// 基本用法
const age = 18;
const result = age >= 18 ? '成年人' : '未成年';
console.log(result); // '成年人'

// 嵌套三元运算符（不推荐，可读性差）
const score = 85;
const grade = score >= 90 ? 'A' : score >= 80 ? 'B' : score >= 70 ? 'C' : 'D';
console.log(grade); // 'B'
```

#### 最佳实践

```javascript
// 简单条件判断
const isLoggedIn = true;
const welcomeMessage = isLoggedIn ? '欢迎回来' : '请先登录';

// 提供默认值
const user = {
  name: 'John',
  age: 30
};

const userName = user.name || '匿名用户';
const userAge = user.age ?? 0; // 使用空值合并运算符

// 条件属性赋值
const config = {
  apiUrl: 'https://api.example.com',
  ...(process.env.NODE_ENV === 'production' ? {
    timeout: 5000,
    retries: 3
  } : {
    timeout: 10000,
    retries: 1
  })
};

// 函数参数默认值
function greet(name = 'Guest', message = 'Hello') {
  return `${message}, ${name}!`;
}

console.log(greet()); // 'Hello, Guest!'
console.log(greet('John')); // 'Hello, John!'
console.log(greet('John', 'Hi')); // 'Hi, John!'
```

## 循环语句

### 1. for 循环

#### 基础语法

```javascript
// 标准 for 循环
for (let i = 0; i < 10; i++) {
  console.log(i); // 输出 0-9
}

// 遍历数组
const fruits = ['苹果', '香蕉', '橙子'];
for (let i = 0; i < fruits.length; i++) {
  console.log(`索引 ${i}: ${fruits[i]}`);
}

// 倒序遍历
for (let i = fruits.length - 1; i >= 0; i--) {
  console.log(fruits[i]);
}

// 跳过某些元素
for (let i = 0; i < 10; i++) {
  if (i === 3) continue; // 跳过 3
  if (i === 7) break;    // 在 7 处停止
  console.log(i);
}
// 输出: 0, 1, 2, 4, 5, 6
```

#### 高级用法

```javascript
// 多变量初始化和更新
for (let i = 0, j = 10; i < j; i++, j--) {
  console.log(`i: ${i}, j: ${j}`);
}

// 复杂条件
const users = [
  { name: 'Alice', age: 25, active: true },
  { name: 'Bob', age: 30, active: false },
  { name: 'Charlie', age: 35, active: true }
];

for (let i = 0; i < users.length; i++) {
  const user = users[i];
  if (user.active && user.age > 30) {
    console.log('符合条件的用户:', user.name);
  }
}

// 遍历对象属性
const obj = { a: 1, b: 2, c: 3 };
const keys = Object.keys(obj);

for (let i = 0; i < keys.length; i++) {
  const key = keys[i];
  console.log(`${key}: ${obj[key]}`);
}
```

### 2. while 循环

#### 基础语法

```javascript
// while 循环：先判断后执行
let num = 1;
while (num <= 5) {
  console.log(num);
  num++; // 必须有自增/自减，否则会陷入死循环
}

// 计算阶乘
function factorial(n) {
  if (n < 0) return -1;
  if (n === 0) return 1;

  let result = 1;
  let i = 1;

  while (i <= n) {
    result *= i;
    i++;
  }

  return result;
}

console.log(factorial(5)); // 120
```

#### 实际应用

```javascript
// 等待某个条件满足
function waitForCondition(condition, maxAttempts = 10, interval = 100) {
  return new Promise((resolve, reject) => {
    let attempts = 0;

    const checkCondition = () => {
      attempts++;

      if (condition()) {
        resolve();
      } else if (attempts >= maxAttempts) {
        reject(new Error('条件超时'));
      } else {
        setTimeout(checkCondition, interval);
      }
    };

    checkCondition();
  });
}

// 使用示例
let dataLoaded = false;

setTimeout(() => { dataLoaded = true; }, 2000);

waitForCondition(() => dataLoaded)
  .then(() => console.log('数据已加载'))
  .catch(() => console.log('等待超时'));

// 查找元素
function findElement(arr, predicate) {
  let i = 0;
  while (i < arr.length) {
    if (predicate(arr[i])) {
      return arr[i];
    }
    i++;
  }
  return undefined;
}

const numbers = [1, 2, 3, 4, 5];
const found = findElement(numbers, num => num > 3);
console.log(found); // 4
```

### 3. do...while 循环

#### 基础语法

```javascript
// do...while 循环：先执行后判断
let x = 0;
do {
  console.log('执行次数:', x + 1);
  x++;
} while (x < 3);
// 输出: 执行次数: 1, 执行次数: 2, 执行次数: 3

// 用户输入验证
function getUserInput() {
  let input;
  do {
    input = prompt('请输入一个正数:');
    if (input === null) {
      console.log('用户取消输入');
      return null;
    }
  } while (isNaN(input) || Number(input) <= 0);

  return Number(input);
}
```

#### 实际应用

```javascript
// 游戏循环
function gameLoop() {
  let isPlaying = true;
  let score = 0;

  do {
    // 游戏逻辑
    console.log('游戏进行中...');
    score += 10;

    // 检查游戏是否结束
    isPlaying = score < 100;

    if (score >= 50) {
      console.log('达到里程碑！');
    }

  } while (isPlaying);

  console.log('游戏结束！最终得分:', score);
}

// 重试机制
function retryOperation(operation, maxRetries = 3) {
  let attempts = 0;
  let success = false;

  do {
    attempts++;
    try {
      const result = operation();
      success = true;
      return result;
    } catch (error) {
      console.log(`尝试 ${attempts} 失败:`, error.message);

      if (attempts >= maxRetries) {
        throw new Error(`操作失败，已重试 ${maxRetries} 次`);
      }
    }
  } while (!success && attempts < maxRetries);
}
```

### 4. for...in 循环

#### 基础语法

```javascript
// 遍历对象属性
const person = {
  name: 'John',
  age: 30,
  city: 'New York'
};

for (const key in person) {
  console.log(`${key}: ${person[key]}`);
}

// 遍历数组索引（不推荐）
const arr = ['a', 'b', 'c'];

for (const index in arr) {
  console.log(`索引: ${index}, 值: ${arr[index]}`);
}

// 检查是否是对象自身的属性
for (const key in person) {
  if (person.hasOwnProperty(key)) {
    console.log(`自身属性: ${key}`);
  }
}
```

#### 注意事项

```javascript
// for...in 遍历的是可枚举属性，包括继承的属性
const parent = { inheritedProp: 'parent' };
const child = Object.create(parent);
child.ownProp = 'child';

for (const key in child) {
  console.log(key); // ownProp, inheritedProp
}

// 只遍历自身属性
for (const key in child) {
  if (child.hasOwnProperty(key)) {
    console.log(key); // ownProp
  }
}

// 更好的方式：使用 Object.keys()
Object.keys(child).forEach(key => {
  console.log(key); // ownProp
});
```

### 5. for...of 循环

#### 基础语法

```javascript
// 遍历数组
const fruits = ['苹果', '香蕉', '橙子'];

for (const fruit of fruits) {
  console.log(fruit);
}

// 遍历字符串
const str = 'Hello';
for (const char of str) {
  console.log(char);
}

// 遍历 Map
const map = new Map([
  ['name', 'John'],
  ['age', 30]
]);

for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}

// 遍历 Set
const set = new Set([1, 2, 3, 3, 4]);
for (const value of set) {
  console.log(value); // 1, 2, 3, 4 (去重)
}

// 遍历 NodeList
const buttons = document.querySelectorAll('button');
for (const button of buttons) {
  button.addEventListener('click', handleClick);
}
```

#### 高级用法

```javascript
// 获取索引和值
const colors = ['red', 'green', 'blue'];

for (const [index, color] of colors.entries()) {
  console.log(`索引: ${index}, 颜色: ${color}`);
}

// 遍历对象的键值对
const obj = { a: 1, b: 2, c: 3 };

for (const [key, value] of Object.entries(obj)) {
  console.log(`${key}: ${value}`);
}

// 使用迭代器
function* generateSequence() {
  yield 1;
  yield 2;
  yield 3;
}

for (const value of generateSequence()) {
  console.log(value); // 1, 2, 3
}

// 异步迭代
async function processItems(items) {
  for (const item of items) {
    await processItem(item); // 等待每个项目处理完成
  }
}

async function processItem(item) {
  console.log('处理项目:', item);
  await new Promise(resolve => setTimeout(resolve, 100));
  console.log('项目完成:', item);
}
```

## 跳转语句

### 1. break 语句

```javascript
// 跳出循环
for (let i = 0; i < 10; i++) {
  if (i === 5) {
    break; // 跳出循环
  }
  console.log(i);
}
// 输出: 0, 1, 2, 3, 4

// 嵌套循环中的 break
for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break; // 只跳出内层循环
    }
    console.log(`i: ${i}, j: ${j}`);
  }
}

// 使用标签跳出多层循环
outerLoop: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (i === 1 && j === 1) {
      break outerLoop; // 跳出外层循环
    }
    console.log(`i: ${i}, j: ${j}`);
  }
}

// 在 switch 中使用 break
let day = 3;
switch (day) {
  case 1:
    console.log('星期一');
    break;
  case 2:
    console.log('星期二');
    break;
  case 3:
    console.log('星期三');
    break;
  default:
    console.log('其他');
}
```

### 2. continue 语句

```javascript
// 跳过当前迭代
for (let i = 0; i < 10; i++) {
  if (i % 2 === 0) {
    continue; // 跳过偶数
  }
  console.log(i);
}
// 输出: 1, 3, 5, 7, 9

// 过滤数据
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const oddNumbers = [];

for (const num of numbers) {
  if (num % 2 === 0) continue;
  oddNumbers.push(num);
}

console.log(oddNumbers); // [1, 3, 5, 7, 9]

// 在嵌套循环中使用 continue
outerLoop: for (let i = 0; i < 3; i++) {
  for (let j = 0; j < 3; j++) {
    if (j === 1) {
      continue outerLoop; // 跳到外层循环的下一次迭代
    }
    console.log(`i: ${i}, j: ${j}`);
  }
}
```

### 3. return 语句

```javascript
// 在函数中返回值
function add(a, b) {
  return a + b;
}

console.log(add(2, 3)); // 5

// 提前返回
function getUserRole(user) {
  if (!user) {
    return 'guest'; // 提前返回
  }

  if (user.isAdmin) {
    return 'admin';
  }

  return 'user';
}

// 返回多个值（使用对象或数组）
function getPersonInfo() {
  return {
    name: 'John',
    age: 30,
    city: 'New York'
  };
}

const { name, age } = getPersonInfo();
console.log(name, age); // John 30

// 在循环中返回
function findFirstEven(numbers) {
  for (const num of numbers) {
    if (num % 2 === 0) {
      return num; // 找到第一个偶数就返回
    }
  }
  return null; // 没找到返回 null
}

console.log(findFirstEven([1, 3, 5, 7, 8, 9])); // 8
```

## 异常处理

### 1. try...catch...finally

#### 基础语法

```javascript
// 基本的异常捕获
try {
  // 可能抛出异常的代码
  const result = 10 / 0;
  console.log(result); // Infinity
} catch (error) {
  // 处理异常
  console.error('发生错误:', error.message);
}

// 捕获特定类型的错误
try {
  const json = '{ "name": "John", "age": 30 }';
  const data = JSON.parse(json);
  console.log(data);
} catch (error) {
  if (error instanceof SyntaxError) {
    console.error('JSON 解析错误:', error.message);
  } else {
    console.error('其他错误:', error.message);
  }
}

// 使用 finally 块
function divide(a, b) {
  try {
    return a / b;
  } catch (error) {
    console.error('除法错误:', error.message);
    return 0;
  } finally {
    console.log('除法操作完成'); // 无论是否出错都会执行
  }
}

console.log(divide(10, 2)); // 5
console.log(divide(10, 0)); // 0
```

#### 高级用法

```javascript
// 链式异常处理
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);

    if (!response.ok) {
      throw new Error(`HTTP 错误: ${response.status}`);
    }

    const userData = await response.json();

    try {
      const userPosts = await fetch(`/api/users/${userId}/posts`);
      const postsData = await userPosts.json();

      return {
        user: userData,
        posts: postsData
      };
    } catch (error) {
      console.error('获取用户文章失败:', error);
      return {
        user: userData,
        posts: []
      };
    }
  } catch (error) {
    console.error('获取用户数据失败:', error);
    throw error;
  }
}

// 自定义错误类
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

class NetworkError extends Error {
  constructor(message, statusCode) {
    super(message);
    this.name = 'NetworkError';
    this.statusCode = statusCode;
  }
}

function validateUser(user) {
  if (!user.name) {
    throw new ValidationError('姓名不能为空', 'name');
  }

  if (!user.email || !user.email.includes('@')) {
    throw new ValidationError('邮箱格式不正确', 'email');
  }

  return true;
}

try {
  const user = { name: '', email: 'invalid' };
  validateUser(user);
} catch (error) {
  if (error instanceof ValidationError) {
    console.error(`验证错误 - ${error.field}: ${error.message}`);
  } else {
    console.error('未知错误:', error.message);
  }
}
```

### 2. throw 语句

```javascript
// 抛出基本错误
function divide(a, b) {
  if (b === 0) {
    throw new Error('除数不能为零');
  }
  return a / b;
}

try {
  console.log(divide(10, 0));
} catch (error) {
  console.error(error.message); // '除数不能为零'
}

// 抛出自定义错误
function validateAge(age) {
  if (typeof age !== 'number') {
    throw new TypeError('年龄必须是数字');
  }

  if (age < 0 || age > 150) {
    throw new RangeError('年龄必须在 0 到 150 之间');
  }

  return true;
}

// 抛出带有额外信息的错误
function processPayment(amount) {
  if (amount <= 0) {
    const error = new Error('支付金额必须大于零');
    error.code = 'INVALID_AMOUNT';
    error.details = { receivedAmount: amount };
    throw error;
  }

  if (amount > 10000) {
    const error = new Error('支付金额超过限额');
    error.code = 'AMOUNT_EXCEEDED';
    error.details = { limit: 10000, attemptedAmount: amount };
    throw error;
  }

  return { success: true, amount };
}

try {
  processPayment(15000);
} catch (error) {
  console.error('支付失败:', error.code);
  console.error('详细信息:', error.details);
}
```

## 实际应用

### 1. 表单验证

```javascript
function validateForm(formData) {
  const errors = {};

  // 用户名验证
  if (!formData.username) {
    errors.username = '用户名不能为空';
  } else if (formData.username.length < 3) {
    errors.username = '用户名长度至少为3个字符';
  } else if (!/^[a-zA-Z0-9_]+$/.test(formData.username)) {
    errors.username = '用户名只能包含字母、数字和下划线';
  }

  // 邮箱验证
  if (!formData.email) {
    errors.email = '邮箱不能为空';
  } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
    errors.email = '邮箱格式不正确';
  }

  // 密码验证
  if (!formData.password) {
    errors.password = '密码不能为空';
  } else if (formData.password.length < 8) {
    errors.password = '密码长度至少为8个字符';
  } else if (!/[A-Z]/.test(formData.password)) {
    errors.password = '密码必须包含大写字母';
  } else if (!/[a-z]/.test(formData.password)) {
    errors.password = '密码必须包含小写字母';
  } else if (!/[0-9]/.test(formData.password)) {
    errors.password = '密码必须包含数字';
  }

  // 确认密码
  if (formData.password !== formData.confirmPassword) {
    errors.confirmPassword = '两次输入的密码不一致';
  }

  // 手机号验证
  if (formData.phone && !/^1[3-9]\d{9}$/.test(formData.phone)) {
    errors.phone = '手机号格式不正确';
  }

  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}

// 使用示例
const formData = {
  username: 'john_doe',
  email: 'john@example.com',
  password: 'Password123',
  confirmPassword: 'Password123',
  phone: '13812345678'
};

const validation = validateForm(formData);

if (validation.isValid) {
  console.log('表单验证通过');
  // 提交表单
} else {
  console.log('表单验证失败:', validation.errors);
  // 显示错误信息
}
```

### 2. 数据处理管道

```javascript
function processDataPipeline(data) {
  // 第一步：数据清洗
  const cleanedData = data.filter(item => {
    return item != null &&
           typeof item === 'object' &&
           item.id &&
           item.value;
  });

  // 第二步：数据转换
  const transformedData = cleanedData.map(item => ({
    id: item.id,
    value: parseFloat(item.value) || 0,
    category: item.category || 'default',
    timestamp: item.timestamp ? new Date(item.timestamp) : new Date(),
    isValid: !isNaN(parseFloat(item.value))
  }));

  // 第三步：数据过滤
  const filteredData = transformedData.filter(item =>
    item.isValid &&
    item.value > 0 &&
    item.value < 10000
  );

  // 第四步：数据分组
  const groupedData = filteredData.reduce((acc, item) => {
    const category = item.category;
    if (!acc[category]) {
      acc[category] = {
        count: 0,
        total: 0,
        items: []
      };
    }
    acc[category].count++;
    acc[category].total += item.value;
    acc[category].items.push(item);
    return acc;
  }, {});

  // 第五步：计算统计信息
  const statistics = Object.entries(groupedData).map(([category, data]) => ({
    category,
    count: data.count,
    total: data.total,
    average: data.total / data.count,
    items: data.items
  }));

  return {
    originalCount: data.length,
    cleanedCount: cleanedData.length,
    transformedCount: transformedData.length,
    filteredCount: filteredData.length,
    statistics
  };
}

// 使用示例
const rawData = [
  { id: 1, value: '100', category: 'A', timestamp: '2024-01-01' },
  { id: 2, value: '200', category: 'B', timestamp: '2024-01-02' },
  null,
  { id: 3, value: 'invalid', category: 'A', timestamp: '2024-01-03' },
  { id: 4, value: '150', category: 'B', timestamp: '2024-01-04' },
  undefined
];

const result = processDataPipeline(rawData);
console.log(result);
```

### 3. 异步流程控制

```javascript
// 顺序执行异步操作
async function processItemsSequentially(items) {
  const results = [];

  for (const item of items) {
    try {
      const result = await processItem(item);
      results.push({ success: true, item, result });
    } catch (error) {
      results.push({ success: false, item, error: error.message });
    }
  }

  return results;
}

async function processItem(item) {
  console.log('处理项目:', item.id);
  // 模拟异步处理
  await new Promise(resolve => setTimeout(resolve, 100));

  if (item.shouldFail) {
    throw new Error(`项目 ${item.id} 处理失败`);
  }

  return { id: item.id, processed: true };
}

// 并行执行异步操作
async function processItemsInParallel(items) {
  const promises = items.map(async item => {
    try {
      const result = await processItem(item);
      return { success: true, item, result };
    } catch (error) {
      return { success: false, item, error: error.message };
    }
  });

  return await Promise.all(promises);
}

// 批量处理
async function processItemsInBatches(items, batchSize = 5) {
  const results = [];

  for (let i = 0; i < items.length; i += batchSize) {
    const batch = items.slice(i, i + batchSize);
    const batchResults = await processItemsInParallel(batch);
    results.push(...batchResults);
  }

  return results;
}

// 使用示例
const items = [
  { id: 1, shouldFail: false },
  { id: 2, shouldFail: true },
  { id: 3, shouldFail: false },
  { id: 4, shouldFail: false },
  { id: 5, shouldFail: true },
  { id: 6, shouldFail: false }
];

processItemsInBatches(items, 2).then(results => {
  console.log('处理结果:', results);
});
```

### 4. 游戏逻辑实现

```javascript
class Game {
  constructor() {
    this.isPlaying = false;
    this.score = 0;
    this.lives = 3;
    this.level = 1;
  }

  start() {
    if (this.isPlaying) {
      console.log('游戏已经在运行中');
      return;
    }

    this.isPlaying = true;
    this.score = 0;
    this.lives = 3;
    this.level = 1;

    console.log('游戏开始！');
    this.gameLoop();
  }

  gameLoop() {
    if (!this.isPlaying) {
      return;
    }

    // 游戏逻辑
    this.update();
    this.render();

    // 检查游戏状态
    if (this.lives <= 0) {
      this.gameOver();
      return;
    }

    // 继续游戏循环
    setTimeout(() => this.gameLoop(), 1000);
  }

  update() {
    // 更新游戏状态
    this.score += 10 * this.level;

    // 检查是否升级
    if (this.score >= this.level * 100) {
      this.levelUp();
    }

    // 随机事件
    if (Math.random() < 0.1) {
      this.randomEvent();
    }
  }

  render() {
    console.log(`分数: ${this.score}, 生命: ${this.lives}, 等级: ${this.level}`);
  }

  levelUp() {
    this.level++;
    console.log(`恭喜！升级到第 ${this.level} 级！`);
  }

  randomEvent() {
    const events = ['bonus', 'penalty', 'extraLife'];
    const event = events[Math.floor(Math.random() * events.length)];

    switch (event) {
      case 'bonus':
        this.score += 50;
        console.log('获得奖励：+50 分');
        break;
      case 'penalty':
        this.score = Math.max(0, this.score - 20);
        console.log('受到惩罚：-20 分');
        break;
      case 'extraLife':
        this.lives++;
        console.log('获得额外生命！');
        break;
    }
  }

  gameOver() {
    this.isPlaying = false;
    console.log('游戏结束！');
    console.log(`最终得分: ${this.score}`);
    console.log(`达到等级: ${this.level}`);
  }
}

// 使用示例
const game = new Game();
game.start();
```

## 性能优化

### 1. 循环优化

```javascript
// 避免在循环中重复计算
function processArrayBad(arr) {
  for (let i = 0; i < arr.length; i++) {
    // 每次循环都计算 arr.length
    console.log(arr[i]);
  }
}

function processArrayGood(arr) {
  const length = arr.length; // 缓存数组长度
  for (let i = 0; i < length; i++) {
    console.log(arr[i]);
  }
}

// 使用倒序循环（在某些情况下更快）
function processArrayReverse(arr) {
  for (let i = arr.length - 1; i >= 0; i--) {
    console.log(arr[i]);
  }
}

// 减少循环内的操作
function optimizeLoop(data) {
  // 不好的写法
  for (let i = 0; i < data.length; i++) {
    const item = data[i];
    const processed = expensiveOperation(item);
    const result = anotherOperation(processed);
    const final = yetAnotherOperation(result);
    data[i] = final;
  }

  // 好的写法：提取常用操作
  const processItem = (item) => {
    const processed = expensiveOperation(item);
    const result = anotherOperation(processed);
    return yetAnotherOperation(result);
  };

  for (let i = 0; i < data.length; i++) {
    data[i] = processItem(data[i]);
  }
}
```

### 2. 条件判断优化

```javascript
// 将最可能的条件放在前面
function checkCondition(value) {
  // 假设 value 大多数时候是 'type1'
  if (value === 'type1') {
    // 处理 type1
  } else if (value === 'type2') {
    // 处理 type2
  } else if (value === 'type3') {
    // 处理 type3
  }
}

// 使用查找表替代复杂的条件判断
function getActionByType(type) {
  const actions = {
    'create': () => console.log('创建操作'),
    'update': () => console.log('更新操作'),
    'delete': () => console.log('删除操作'),
    'read': () => console.log('读取操作')
  };

  const action = actions[type];
  return action ? action() : console.log('未知操作');
}

// 使用位运算优化
function hasPermission(userPermissions, requiredPermission) {
  // 假设权限用位掩码表示
  return (userPermissions & requiredPermission) === requiredPermission;
}

// 预先计算条件
function optimizeConditions(data) {
  // 不好的写法
  for (const item of data) {
    if (item.value > 10 && item.value < 20 && item.isActive && item.isValid) {
      // 处理项目
    }
  }

  // 好的写法
  for (const item of data) {
    const isValidRange = item.value > 10 && item.value < 20;
    const isValidStatus = item.isActive && item.isValid;

    if (isValidRange && isValidStatus) {
      // 处理项目
    }
  }
}
```

## 注意事项

### 1. 避免常见错误

```javascript
// 避免无限循环
// 错误示例
let i = 0;
while (i < 10) {
  console.log(i);
  // 忘记 i++
}

// 正确示例
let i = 0;
while (i < 10) {
  console.log(i);
  i++;
}

// 避免在循环中修改数组长度
// 错误示例
const arr = [1, 2, 3, 4, 5];
for (let i = 0; i < arr.length; i++) {
  if (arr[i] % 2 === 0) {
    arr.splice(i, 1); // 会导致跳过元素
  }
}

// 正确示例：倒序遍历
for (let i = arr.length - 1; i >= 0; i--) {
  if (arr[i] % 2 === 0) {
    arr.splice(i, 1);
  }
}

// 或使用 filter
const filtered = arr.filter(item => item % 2 !== 0);
```

### 2. 代码可读性

```javascript
// 避免过深的嵌套
// 不好的写法
function processUser(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        if (user.age > 18) {
          // 处理用户
        }
      }
    }
  }
}

// 好的写法：使用提前返回
function processUser(user) {
  if (!user) return;
  if (!user.isActive) return;
  if (!user.hasPermission) return;
  if (user.age <= 18) return;

  // 处理用户
}

// 给循环和条件添加有意义的变量名
// 不好的写法
for (let i = 0; i < arr.length; i++) {
  if (arr[i].x > 10) {
    console.log(arr[i]);
  }
}

// 好的写法
for (let index = 0; index < arr.length; index++) {
  const item = arr[index];
  if (item.value > 10) {
    console.log(item);
  }
}
```

### 3. 性能考虑

```javascript
// 对于大数组，考虑使用更高效的方法
const largeArray = Array.from({ length: 100000 }, (_, i) => i);

// 不好的写法：使用 forEach
largeArray.forEach(item => {
  // 处理逻辑
});

// 好的写法：使用 for 循环（性能更好）
for (let i = 0; i < largeArray.length; i++) {
  const item = largeArray[i];
  // 处理逻辑
}

// 避免在循环中进行 DOM 操作
// 不好的写法
const elements = document.querySelectorAll('.item');
elements.forEach(element => {
  element.style.display = 'none'; // 多次重排重绘
});

// 好的写法：批量操作
const elements = document.querySelectorAll('.item');
elements.forEach(element => {
  element.classList.add('hidden'); // 使用类名
});

// 或使用文档片段
const fragment = document.createDocumentFragment();
elements.forEach(element => {
  const clone = element.cloneNode(true);
  clone.style.display = 'none';
  fragment.appendChild(clone);
});
```

## 总结

JavaScript 流程控制语句是编程的基础，掌握它们对于编写高质量的代码至关重要。通过本文的学习，你应该能够：

1. **熟练使用条件语句**：if...else、switch...case、三元运算符
2. **掌握各种循环语句**：for、while、do...while、for...in、for...of
3. **理解跳转语句**：break、continue、return
4. **处理异常情况**：try...catch...finally、throw
5. **优化代码性能**：避免常见错误，提高代码效率

### 最佳实践总结：

1. **优先考虑可读性**：清晰的代码比微小的性能提升更重要
2. **避免深层嵌套**：使用提前返回等方式减少嵌套层次
3. **选择合适的数据结构**：根据场景选择最合适的数据处理方式
4. **注意边界情况**：处理空值、异常值等边界情况
5. **性能与可读性平衡**：在性能和可读性之间找到平衡点

持续练习和实践，你将能够熟练运用各种流程控制语句，编写出更加优雅、高效的 JavaScript 代码。