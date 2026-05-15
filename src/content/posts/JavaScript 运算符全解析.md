---
title: JavaScript 运算符全解析
description: 深入理解 JavaScript 中的算术、比较、逻辑、赋值与位运算符，掌握运算符优先级和最佳实践
published: 2024-01-05
tags: ["JS基础","运算符"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 运算符全解析

## 概述

运算符是 JavaScript 中最基础也是最核心的语法之一。无论是变量计算、条件判断，还是复杂逻辑控制，都离不开各种运算符的配合使用。本文将系统讲解 JavaScript 中的各类运算符，帮助你彻底理解运算符体系，避免常见陷阱，写出更优雅的代码。

## 核心概念

### 运算符分类

JavaScript 运算符可以分为以下几大类：

1. **算术运算符**：执行数学运算（+、-、*、/、%、++、--）
2. **比较运算符**：比较两个值（==、===、!=、!==、>、<、>=、<=）
3. **逻辑运算符**：执行逻辑运算（&&、||、!）
4. **赋值运算符**：给变量赋值（=、+=、-=、*=、/=等）
5. **位运算符**：按位操作（&、|、^、~、<<、>>、>>>）
6. **其他运算符**：三元运算符、逗号运算符、delete、typeof、void 等

### 运算符优先级

理解运算符优先级对于写出正确的表达式至关重要。以下是 JavaScript 运算符的优先级（从高到低）：

```javascript
// 优先级示例
1. () // 分组
2. . [] // 成员访问
3. new // 创建对象
4. ++ -- ++ -- // 前置递增递减
5. ! ~ + - typeof void delete // 一元运算符
6. ** // 幂运算
7. * / % // 乘法、除法、取余
8. + - // 加法、减法
9. << >> >>> // 位移
10. < <= > >= instanceof in // 比较
11. == != === !== // 相等
12. & // 按位与
13. ^ // 按位异或
14. | // 按位或
15. && // 逻辑与
16. || // 逻辑或
17. ? : // 三元运算符
18. = += -= *= /= %= <<= >>= >>>= &= ^= |= // 赋值
19. , // 逗号
```

## 算术运算符

### 1. 加法 (+)

```javascript
// 数字加法
const sum = 10 + 20; // 30

// 字符串拼接
const greeting = 'Hello' + ' ' + 'World'; // "Hello World"

// 类型转换
console.log(1 + '2'); // "12" (字符串拼接)
console.log(1 + 2 + '3'); // "33" (先计算 1+2=3，然后拼接 "3")
console.log('1' + 2 + 3); // "123" (全部拼接)
console.log(1 + true); // 2 (true 转换为 1)
console.log(1 + false); // 1 (false 转换为 0)
console.log(1 + null); // 1 (null 转换为 0)
console.log(1 + undefined); // NaN

// 实际应用：字符串模板拼接
function buildUrl(base, path, params) {
  let url = base + path;
  const queryString = Object.entries(params)
    .map(([key, value]) => `${key}=${value}`)
    .join('&');
  return queryString ? url + '?' + queryString : url;
}

console.log(buildUrl('https://api.example.com', '/users', { page: 1, limit: 10 }));
// "https://api.example.com/users?page=1&limit=10"
```

### 2. 减法 (-)

```javascript
// 数字减法
const difference = 20 - 10; // 10

// 类型转换
console.log(10 - '5'); // 5 (字符串转换为数字)
console.log(10 - 'abc'); // NaN
console.log(10 - null); // 10
console.log(10 - undefined); // NaN

// 实际应用：计算差值
function calculateAge(birthYear) {
  const currentYear = new Date().getFullYear();
  return currentYear - birthYear;
}

console.log(calculateAge(1990)); // 34 (假设当前年份是 2024)
```

### 3. 乘法 (*)

```javascript
// 数字乘法
const product = 5 * 2; // 10

// 类型转换
console.log(5 * '2'); // 10
console.log(5 * 'abc'); // NaN
console.log(5 * null); // 0
console.log(5 * undefined); // NaN

// 实际应用：计算总价
function calculateTotal(price, quantity, discount = 1) {
  return price * quantity * discount;
}

console.log(calculateTotal(100, 3, 0.9)); // 270 (100*3*0.9)
```

### 4. 除法 (/)

```javascript
// 数字除法
const quotient = 10 / 2; // 5

// 除以零
console.log(10 / 0); // Infinity
console.log(-10 / 0); // -Infinity
console.log(0 / 0); // NaN

// 类型转换
console.log(10 / '2'); // 5
console.log(10 / 'abc'); // NaN

// 实际应用：计算平均值
function calculateAverage(numbers) {
  if (numbers.length === 0) return 0;
  const sum = numbers.reduce((acc, num) => acc + num, 0);
  return sum / numbers.length;
}

console.log(calculateAverage([10, 20, 30, 40, 50])); // 30
```

### 5. 取余 (%)

```javascript
// 基础取余
console.log(10 % 3); // 1
console.log(15 % 4); // 3

// 处理负数
console.log(-10 % 3); // -1
console.log(10 % -3); // 1
console.log(-10 % -3); // -1

// 实际应用：判断奇偶
function isEven(number) {
  return number % 2 === 0;
}

function isOdd(number) {
  return number % 2 !== 0;
}

console.log(isEven(4)); // true
console.log(isOdd(5)); // true

// 分页计算
function calculatePagination(totalItems, itemsPerPage) {
  return {
    totalPages: Math.ceil(totalItems / itemsPerPage),
    hasNextPage: totalItems % itemsPerPage !== 0
  };
}

console.log(calculatePagination(23, 10));
// { totalPages: 3, hasNextPage: true }

// 循环轮询
function getNextItem(index, arrayLength) {
  return (index + 1) % arrayLength;
}

const items = ['A', 'B', 'C', 'D'];
console.log(items[getNextItem(0, items.length)]); // 'B'
console.log(items[getNextItem(3, items.length)]); // 'A'
```

### 6. 幂运算 (**)

```javascript
// 基础幂运算
console.log(2 ** 3); // 8
console.log(3 ** 2); // 9

// 负数幂
console.log(2 ** -2); // 0.25
console.log((-2) ** 2); // 4

// 实际应用：计算平方和立方
function square(x) { return x ** 2; }
function cube(x) { return x ** 3; }

console.log(square(5)); // 25
console.log(cube(3)); // 27

// 指数增长计算
function compoundInterest(principal, rate, time) {
  return principal * (1 + rate) ** time;
}

console.log(compoundInterest(1000, 0.05, 10)); // 1628.89
```

### 7. 自增 (++) 和自减 (--)

```javascript
// 前置自增/自减
let a = 1;
console.log(++a); // 2 (先自增，再返回)
console.log(a);   // 2

let b = 1;
console.log(--b); // 0 (先自减，再返回)
console.log(b);   // 0

// 后置自增/自减
let c = 1;
console.log(c++); // 1 (先返回，再自增)
console.log(c);   // 2

let d = 1;
console.log(d--); // 1 (先返回，再自减)
console.log(d);   // 0

// 实际应用：计数器
class Counter {
  constructor(initial = 0) {
    this.count = initial;
  }

  increment() {
    return ++this.count;
  }

  decrement() {
    return --this.count;
  }

  getValue() {
    return this.count;
  }
}

const counter = new Counter(0);
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.decrement()); // 1

// 数组索引遍历
function iterateArray(array) {
  let index = 0;
  while (index < array.length) {
    console.log(`Index ${index}: ${array[index]}`);
    index++;
  }
}

iterateArray(['apple', 'banana', 'orange']);
```

## 比较运算符

### 1. 相等运算符 (==)

```javascript
// 松散相等（会进行类型转换）
console.log(1 == '1'); // true
console.log(0 == false); // true
console.log(null == undefined); // true
console.log('' == 0); // true
console.log('' == false); // true

// 对象比较
console.log([] == false); // true
console.log([1] == 1); // true
console.log({} == '[object Object]'); // false

// 注意事项：避免使用 ==
if (x == null) {
  // 这个条件在 x 为 null 或 undefined 时都为 true
}
```

### 2. 严格相等运算符 (===)

```javascript
// 严格相等（不进行类型转换）
console.log(1 === '1'); // false
console.log(0 === false); // false
console.log(null === undefined); // false
console.log('' === 0); // false

// 对象比较
console.log({} === {}); // false (不同引用)
console.log([] === []); // false (不同引用)

const obj1 = {};
const obj2 = obj1;
console.log(obj1 === obj2); // true (相同引用)

// 最佳实践：始终使用 ===
function isGreaterThanZero(value) {
  return typeof value === 'number' && value > 0;
}

console.log(isGreaterThanZero(5)); // true
console.log(isGreaterThanZero('5')); // false
```

### 3. 不等运算符 (!= 和 !==)

```javascript
// 松散不等
console.log(1 != '2'); // true
console.log(0 != false); // false

// 严格不等
console.log(1 !== '1'); // true
console.log(0 !== false); // true

// 实际应用：参数验证
function processValue(value) {
  if (value === null || value === undefined) {
    throw new Error('Value cannot be null or undefined');
  }
  // 处理值
}

// 更简洁的写法
function processValue(value) {
  if (value == null) { // 同时检查 null 和 undefined
    throw new Error('Value cannot be null or undefined');
  }
  // 处理值
}
```

### 4. 关系运算符 (<, >, <=, >=)

```javascript
// 数字比较
console.log(10 > 5); // true
console.log(3 < 1); // false
console.log(5 >= 5); // true
console.log(2 <= 1); // false

// 字符串比较（按字母顺序）
console.log('apple' < 'banana'); // true
console.log('Z' < 'a'); // true (大写字母的 ASCII 码小于小写字母)

// 类型转换
console.log('10' > 5); // true
console.log('abc' > 5); // false

// NaN 比较
console.log(NaN > 5); // false
console.log(NaN < 5); // false
console.log(NaN === NaN); // false

// 实际应用：范围检查
function isInRange(value, min, max) {
  return value >= min && value <= max;
}

console.log(isInRange(5, 1, 10)); // true
console.log(isInRange(15, 1, 10)); // false

// 版本比较
function compareVersions(version1, version2) {
  const v1Parts = version1.split('.').map(Number);
  const v2Parts = version2.split('.').map(Number);

  for (let i = 0; i < Math.max(v1Parts.length, v2Parts.length); i++) {
    const v1Part = v1Parts[i] || 0;
    const v2Part = v2Parts[i] || 0;

    if (v1Part > v2Part) return 1;
    if (v1Part < v2Part) return -1;
  }

  return 0;
}

console.log(compareVersions('1.2.3', '1.2.4')); // -1
console.log(compareVersions('2.0.0', '1.9.9')); // 1
console.log(compareVersions('1.0.0', '1.0.0')); // 0
```

## 逻辑运算符

### 1. 逻辑与 (&&)

```javascript
// 基础用法
console.log(true && true); // true
console.log(true && false); // false
console.log(false && true); // false
console.log(false && false); // false

// 短路求值
console.log(true && 'hello'); // 'hello'
console.log(false && 'hello'); // false
console.log(1 && 2 && 3); // 3
console.log(0 && 2 && 3); // 0

// 实际应用：条件执行
function greet(name) {
  name && console.log('Hello, ' + name);
}

greet('Alice'); // "Hello, Alice"
greet(''); // 不输出

// 防御性编程
const user = {
  name: 'Alice',
  address: {
    city: 'New York'
  }
};

// 安全访问嵌套属性
const city = user && user.address && user.address.city;
console.log(city); // 'New York'

// 使用可选链操作符（ES2020）
const city2 = user?.address?.city;
console.log(city2); // 'New York'

// 条件赋值
const isLoggedIn = true;
const welcomeMessage = isLoggedIn && 'Welcome back!';
console.log(welcomeMessage); // 'Welcome back!'

// 函数调用保护
const logger = {
  log: function(message) {
    console.log(message);
  }
};

logger && logger.log && logger.log('Logging message');
```

### 2. 逻辑或 (||)

```javascript
// 基础用法
console.log(true || false); // true
console.log(false || true); // true
console.log(true || true); // true
console.log(false || false); // false

// 短路求值
console.log(true || 'hello'); // true
console.log(false || 'hello'); // 'hello'
console.log(0 || 1); // 1
console.log(1 || 0); // 1

// 默认值设置
const name = userName || 'Guest';
console.log(name); // 如果 userName 为 falsy，则使用 'Guest'

// 实际应用：配置默认值
function createOptions(options) {
  return {
    timeout: options.timeout || 5000,
    retries: options.retries || 3,
    debug: options.debug || false
  };
}

const userOptions = { timeout: 10000 };
const finalOptions = createOptions(userOptions);
console.log(finalOptions);
// { timeout: 10000, retries: 3, debug: false }

// 提供备用函数
const saveData = saveToDatabase || saveToFile;

// 多重备选方案
const theme = userTheme || systemTheme || defaultTheme;

// 注意：空字符串和 0 会被视为 falsy
const count = inputCount || 10; // 如果 inputCount 为 0，会使用 10
```

### 3. 逻辑非 (!)

```javascript
// 基础用法
console.log(!true); // false
console.log(!false); // true

// 双重取反（转换为布尔值）
console.log(!!1); // true
console.log(!!0); // false
console.log(!!'hello'); // true
console.log(!!''); // false
console.log(!!null); // false
console.log(!!undefined); // false
console.log(!!{}); // true
console.log(!![]); // true

// 实际应用：布尔转换
function toBoolean(value) {
  return !!value;
}

console.log(toBoolean(1)); // true
console.log(toBoolean(0)); // false

// 条件反转
function isNotLoggedIn(user) {
  return !user.isLoggedIn;
}

// 空值检查
function hasValue(value) {
  return value != null; // 同时检查 null 和 undefined
}

// 函数参数验证
function processOptions(options) {
  if (!options) {
    options = {};
  }
  // 处理选项
}

// 更简洁的写法
function processOptions(options = {}) {
  // 处理选项
}
```

### 4. 空值合并运算符 (??) (ES2020)

```javascript
// 只在值为 null 或 undefined 时使用默认值
const name = null ?? 'Guest';
console.log(name); // 'Guest'

const age = undefined ?? 18;
console.log(age); // 18

const city = '' ?? 'Unknown';
console.log(city); // '' (空字符串不是 null 或 undefined)

const count = 0 ?? 10;
console.log(count); // 0 (0 不是 null 或 undefined)

// 与 || 的区别
console.log(0 || 10); // 10 (0 是 falsy)
console.log(0 ?? 10); // 0 (0 不是 null 或 undefined)

console.log('' || 'default'); // 'default'
console.log('' ?? 'default'); // ''

console.log(false || 'default'); // 'default'
console.log(false ?? 'default'); // false

// 实际应用：处理可能为 null/undefined 的值
function getUserName(user) {
  return user.name ?? 'Anonymous';
}

const user1 = { name: 'Alice' };
const user2 = { name: null };
const user3 = {};

console.log(getUserName(user1)); // 'Alice'
console.log(getUserName(user2)); // 'Anonymous'
console.log(getUserName(user3)); // 'Anonymous'

// 链式空值合并
const config = {
  timeout: null,
  retries: undefined,
  debug: false
};

const finalTimeout = config.timeout ?? 5000;
const finalRetries = config.retries ?? 3;
const finalDebug = config.debug ?? true;

console.log(finalTimeout); // 5000
console.log(finalRetries); // 3
console.log(finalDebug); // false (false 不是 null 或 undefined)
```

### 5. 可选链运算符 (?.) (ES2020)

```javascript
// 安全访问嵌套属性
const user = {
  name: 'Alice',
  address: {
    city: 'New York',
    zip: '10001'
  }
};

console.log(user.address?.city); // 'New York'
console.log(user.profile?.age); // undefined
console.log(user.contact?.email); // undefined

// 可选链调用
const result = user.getAddress?.();
console.log(result); // undefined (如果 getAddress 方法不存在)

// 数组访问
const items = [{ id: 1, name: 'Item 1' }];
console.log(items[0]?.name); // 'Item 1'
console.log(items[1]?.name); // undefined

// 实际应用：安全访问 API 响应
function getUserCity(response) {
  return response?.data?.user?.address?.city || 'Unknown';
}

const apiResponse = {
  data: {
    user: {
      name: 'Alice',
      address: {
        city: 'New York'
      }
    }
  }
};

console.log(getUserCity(apiResponse)); // 'New York'
console.log(getUserCity({})); // 'Unknown'

// 与空值合并运算符结合使用
const city = user?.address?.city ?? 'Unknown';

// 多层可选链
const street = user?.address?.details?.street?.name ?? 'No street info';
```

## 赋值运算符

### 1. 基础赋值 (=)

```javascript
// 变量声明和赋值
let x = 10;
const y = 20;

// 链式赋值
let a, b, c;
a = b = c = 5;
console.log(a, b, c); // 5, 5, 5

// 对象属性赋值
const obj = {};
obj.name = 'Alice';
obj.age = 30;

// 解构赋值
const { name, age } = obj;
console.log(name, age); // 'Alice', 30
```

### 2. 算术赋值运算符

```javascript
let x = 10;

// 加法赋值
x += 5; // 相当于 x = x + 5
console.log(x); // 15

// 减法赋值
x -= 3; // 相当于 x = x - 3
console.log(x); // 12

// 乘法赋值
x *= 2; // 相当于 x = x * 2
console.log(x); // 24

// 除法赋值
x /= 4; // 相当于 x = x / 4
console.log(x); // 6

// 取余赋值
x %= 4; // 相当于 x = x % 4
console.log(x); // 2

// 幂赋值
x **= 3; // 相当于 x = x ** 3
console.log(x); // 8

// 实际应用：累加器
let sum = 0;
const numbers = [1, 2, 3, 4, 5];
numbers.forEach(num => sum += num);
console.log(sum); // 15

// 字符串拼接
let message = 'Hello';
message += ' World';
console.log(message); // 'Hello World'
```

### 3. 位运算赋值运算符

```javascript
let x = 5; // 二进制: 101

// 按位与赋值
x &= 3; // 相当于 x = x & 3
console.log(x); // 1 (101 & 011 = 001)

// 按位或赋值
x |= 2; // 相当于 x = x | 2
console.log(x); // 3 (001 | 010 = 011)

// 按位异或赋值
x ^= 1; // 相当于 x = x ^ 1
console.log(x); // 2 (011 ^ 001 = 010)

// 左移赋值
x <<= 1; // 相当于 x = x << 1
console.log(x); // 4 (010 << 1 = 100)

// 右移赋值
x >>= 1; // 相当于 x = x >> 1
console.log(x); // 2 (100 >> 1 = 010)
```

## 三元运算符

### 基础用法

```javascript
// 语法：condition ? value1 : value2
const age = 18;
const status = age >= 18 ? '成年人' : '未成年';
console.log(status); // '成年人'

// 嵌套三元运算符
const score = 85;
const grade = score >= 90 ? 'A' : score >= 80 ? 'B' : score >= 70 ? 'C' : 'D';
console.log(grade); // 'B'

// 注意：避免过度嵌套，影响可读性
```

### 实际应用

```javascript
// 条件赋值
const isAuthenticated = true;
const welcomeMessage = isAuthenticated ? 'Welcome back!' : 'Please log in';

// 函数参数默认值
function greet(name = 'Guest') {
  const prefix = name === 'Guest' ? 'Hello' : 'Welcome back';
  return `${prefix}, ${name}!`;
}

console.log(greet()); // 'Hello, Guest!'
console.log(greet('Alice')); // 'Welcome back, Alice!'

// 动态样式
function getButtonStyle(disabled) {
  return {
    backgroundColor: disabled ? '#ccc' : '#007bff',
    color: disabled ? '#666' : '#fff',
    cursor: disabled ? 'not-allowed' : 'pointer'
  };
}

console.log(getButtonStyle(true));
console.log(getButtonStyle(false));

// 安全访问和默认值
function getUsername(user) {
  return user ? user.name : 'Anonymous';
}

const user = { name: 'Alice' };
console.log(getUsername(user)); // 'Alice'
console.log(getUsername(null)); // 'Anonymous'

// 复杂条件判断
function calculateDiscount(price, memberLevel) {
  return memberLevel === 'gold' ? price * 0.8 :
         memberLevel === 'silver' ? price * 0.9 :
         memberLevel === 'bronze' ? price * 0.95 :
         price; // 普通会员不打折
}

console.log(calculateDiscount(100, 'gold')); // 80
console.log(calculateDiscount(100, 'silver')); // 90
```

## 其他运算符

### 1. 展开运算符 (...)

```javascript
// 数组展开
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 对象展开
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 函数参数展开
function sum(...numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}

console.log(sum(1, 2, 3, 4)); // 10

// 数组复制
const original = [1, 2, 3];
const copy = [...original];
copy[0] = 10;
console.log(original); // [1, 2, 3] (原数组不变)
console.log(copy); // [10, 2, 3]
```

### 2. 逗号运算符

```javascript
// 逗号运算符：从左到右计算表达式，返回最后一个表达式的值
let a = (1, 2, 3);
console.log(a); // 3

// 在 for 循环中使用
for (let i = 0, j = 10; i < 5; i++, j--) {
  console.log(i, j); // 0 10, 1 9, 2 8, 3 7, 4 6
}

// 实际应用：同时执行多个操作
function processUser(user) {
  return (
    console.log('Processing user:', user.name),
    console.log('User ID:', user.id),
    user.age >= 18 ? 'Adult' : 'Minor'
  );
}

console.log(processUser({ name: 'Alice', id: 1, age: 25 }));
```

### 3. typeof 运算符

```javascript
// 基础用法
console.log(typeof 42); // 'number'
console.log(typeof 'hello'); // 'string'
console.log(typeof true); // 'boolean'
console.log(typeof undefined); // 'undefined'
console.log(typeof null); // 'object' (这是一个已知的 bug)
console.log(typeof {}); // 'object'
console.log(typeof []); // 'object'
console.log(typeof function() {}); // 'function'
console.log(typeof Symbol()); // 'symbol'

// 实际应用：类型检查
function isNumber(value) {
  return typeof value === 'number' && !isNaN(value);
}

function isString(value) {
  return typeof value === 'string';
}

function isFunction(value) {
  return typeof value === 'function';
}

console.log(isNumber(42)); // true
console.log(isNumber('42')); // false
console.log(isString('hello')); // true
console.log(isFunction(() => {})); // true

// 注意：typeof null 返回 'object'
function isNull(value) {
  return value === null;
}

function isObject(value) {
  return typeof value === 'object' && value !== null;
}
```

### 4. instanceof 运算符

```javascript
// 基础用法
console.log([] instanceof Array); // true
console.log({} instanceof Object); // true
console.log(function() {} instanceof Function); // true

// 自定义类
class Person {}
const person = new Person();
console.log(person instanceof Person); // true

// 实际应用：类型检查
function isArray(value) {
  return value instanceof Array;
}

function isDate(value) {
  return value instanceof Date;
}

console.log(isArray([1, 2, 3])); // true
console.log(isArray({ 0: 1, 1: 2 })); // false

// 跨 iframe 问题
const iframe = document.createElement('iframe');
document.body.appendChild(iframe);
const iframeArray = iframe.contentWindow.Array;
console.log([] instanceof iframeArray); // false

// 更好的数组检查方法
console.log(Array.isArray([1, 2, 3])); // true
console.log(Array.isArray({ 0: 1, 1: 2 })); // false
```

### 5. in 运算符

```javascript
// 检查属性是否存在
const obj = { name: 'Alice', age: 30 };

console.log('name' in obj); // true
console.log('gender' in obj); // false

// 检查数组索引
const arr = [1, 2, 3];
console.log(0 in arr); // true
console.log(3 in arr); // false

// 检查原型链上的属性
console.log('toString' in obj); // true (来自 Object.prototype)

// 实际应用：安全的属性检查
function hasOwnProperty(obj, prop) {
  return Object.prototype.hasOwnProperty.call(obj, prop);
}

console.log(hasOwnProperty(obj, 'name')); // true
console.log(hasOwnProperty(obj, 'toString')); // false

// 结合可选链使用
function getProperty(obj, prop) {
  return prop in obj ? obj[prop] : undefined;
}
```

### 6. delete 运算符

```javascript
// 删除对象属性
const obj = { name: 'Alice', age: 30, city: 'New York' };
delete obj.age;
console.log(obj); // { name: 'Alice', city: 'New York' }

// 删除数组元素（会留下空洞）
const arr = [1, 2, 3, 4, 5];
delete arr[2];
console.log(arr); // [1, 2, empty, 4, 5]
console.log(arr.length); // 5 (长度不变)

// 删除不存在的属性
delete obj.nonExistent; // 不会报错，返回 true

// 实际应用：清理对象
function cleanObject(obj, propertiesToRemove) {
  const cleaned = { ...obj };
  propertiesToRemove.forEach(prop => delete cleaned[prop]);
  return cleaned;
}

const user = {
  id: 1,
  name: 'Alice',
  password: 'secret',
  tempData: 'temporary'
};

const safeUser = cleanObject(user, ['password', 'tempData']);
console.log(safeUser); // { id: 1, name: 'Alice' }

// 注意：不能删除 var 声明的变量
var x = 10;
delete x; // false (无法删除)

// 可以删除对象的属性，但不能删除继承的属性
```

## 运算符优先级详解

### 1. 优先级规则

```javascript
// 基础示例
console.log(1 + 2 * 3); // 7 (先乘后加)
console.log((1 + 2) * 3); // 9 (括号改变优先级)

// 复杂表达式
const result = 1 + 2 * 3 ** 2 / 4 % 3;
console.log(result); // 1

// 计算步骤：
// 1. 3 ** 2 = 9
// 2. 2 * 9 = 18
// 3. 18 / 4 = 4.5
// 4. 4.5 % 3 = 1.5
// 5. 1 + 1.5 = 2.5

// 逻辑运算符优先级
console.log(true || false && true); // true (&& 优先级高于 ||)
console.log((true || false) && true); // true
```

### 2. 实际应用示例

```javascript
// 复杂条件判断
function validateUser(user) {
  return user &&
         user.age >= 18 &&
         (user.hasLicense || user.hasPassport) &&
         user.documents.every(doc => doc.isValid);
}

// 数学计算
function calculatePrice(basePrice, discount, tax, shipping) {
  return (basePrice - discount) * (1 + tax) + shipping;
}

console.log(calculatePrice(100, 10, 0.1, 5)); // 99.5

// 位运算和数学运算混合
const flags = 0b1010;
const mask = 0b1100;
const result = (flags & mask) << 1;
console.log(result.toString(2)); // '10000'
```

### 3. 最佳实践

```javascript
// 使用括号提高可读性
const result1 = (a + b) * (c - d); // 清晰
const result2 = a + b * c - d; // 容易混淆

// 复杂逻辑拆分
function complexCondition(a, b, c, d) {
  const condition1 = a > b && c < d;
  const condition2 = b > 0 || d < 0;
  const condition3 = c !== d;

  return condition1 && condition2 && condition3;
}

// 避免依赖记忆优先级
// 好的做法
const area = (width * height) / 2;

// 不好的做法
const area = width * height / 2; // 虽然结果相同，但不清晰
```

## 常见陷阱和解决方案

### 1. 浮点数精度问题

```javascript
// 问题：浮点数计算不精确
console.log(0.1 + 0.2); // 0.30000000000000004
console.log(0.1 * 0.2); // 0.020000000000000004

// 解决方案1：使用整数计算
function add(a, b) {
  const factor = Math.pow(10, Math.max(getDecimalPlaces(a), getDecimalPlaces(b)));
  return (a * factor + b * factor) / factor;
}

function getDecimalPlaces(num) {
  const match = num.toString().match(/\.(\d+)/);
  return match ? match[1].length : 0;
}

console.log(add(0.1, 0.2)); // 0.3

// 解决方案2：使用 toFixed
console.log((0.1 + 0.2).toFixed(2)); // '0.30'
console.log(parseFloat((0.1 + 0.2).toFixed(2))); // 0.3

// 解决方案3：使用专业库
// import Decimal from 'decimal.js';
// const result = new Decimal(0.1).plus(0.2).toNumber();
```

### 2. 类型转换陷阱

```javascript
// 相等运算符的陷阱
console.log('0' == 0); // true
console.log('' == 0); // true
console.log('' == false); // true
console.log(null == undefined); // true

// 解决方案：始终使用严格相等
console.log('0' === 0); // false
console.log('' === 0); // false
console.log(null === undefined); // false

// 加法运算符的陷阱
console.log('5' + 3); // '53' (字符串拼接)
console.log(5 + '3'); // '53' (字符串拼接)
console.log('5' - 3); // 2 (字符串转换为数字)

// 解决方案：明确类型转换
console.log(Number('5') + 3); // 8
console.log(String(5) + 3); // '53'
```

### 3. 位运算陷阱

```javascript
// JavaScript 位运算在 32 位整数上操作
console.log(2147483647 << 1); // -2 (溢出)
console.log(Math.pow(2, 31) >>> 0); // 2147483648

// 解决方案：使用 BigInt
console.log(2147483647n << 1n); // 4294967294n

// 位运算的符号问题
console.log(-1 >>> 0); // 4294967295 (无符号右移)
console.log(-1 >> 0); // -1 (有符号右移)
```

## 性能优化

### 1. 运算符性能比较

```javascript
// 位运算比数学运算更快
// 奇数判断
const number = 123;

// 方法1：取余运算
const isOdd1 = number % 2 !== 0;

// 方法2：位运算（更快）
const isOdd2 = (number & 1) === 1;

// 乘法优化
const x = 4;

// 方法1：普通乘法
const result1 = x * 2;

// 方法2：左移运算（更快）
const result2 = x << 1;

// 整数除法优化
const y = 10;

// 方法1：普通除法
const result3 = y / 2;

// 方法2：右移运算（更快，仅适用于正整数）
const result4 = y >> 1;
```

### 2. 避免不必要的运算

```javascript
// 短路求值优化
function getName(user) {
  // 好的做法：利用短路求值
  return user && user.name && user.name.trim() || 'Unknown';

  // 不好的做法：多次检查
  if (user) {
    if (user.name) {
      if (user.name.trim()) {
        return user.name.trim();
      }
    }
  }
  return 'Unknown';
}

// 条件优化
function checkCondition(a, b, c) {
  // 好的做法：利用短路求值
  return a && (b || c);

  // 不好的做法：不必要的计算
  return a && b || a && c;
}
```

## 实际应用案例

### 1. 表单验证

```javascript
function validateForm(formData) {
  const errors = {};

  // 必填字段验证
  if (!formData.username?.trim()) {
    errors.username = '用户名不能为空';
  }

  if (!formData.email?.includes('@')) {
    errors.email = '请输入有效的邮箱地址';
  }

  // 密码强度验证
  if (formData.password) {
    const hasUpperCase = /[A-Z]/.test(formData.password);
    const hasLowerCase = /[a-z]/.test(formData.password);
    const hasNumber = /\d/.test(formData.password);
    const hasSpecial = /[!@#$%^&*]/.test(formData.password);

    if (!hasUpperCase || !hasLowerCase || !hasNumber || !hasSpecial) {
      errors.password = '密码必须包含大小写字母、数字和特殊字符';
    }
  }

  // 年龄范围验证
  if (formData.age && (formData.age < 18 || formData.age > 120)) {
    errors.age = '年龄必须在 18-120 之间';
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
  password: 'SecurePass123',
  age: 25
};

const validation = validateForm(formData);
console.log(validation);
```

### 2. 数据处理管道

```javascript
function processData(rawData) {
  return rawData
    .filter(item => item.active && item.value > 0) // 筛选有效数据
    .map(item => ({
      ...item,
      processedValue: item.value * 1.1, // 应用折扣
      category: categorize(item.value), // 分类
      isValid: validateItem(item) // 验证
    }))
    .filter(item => item.isValid) // 筛选验证通过的数据
    .sort((a, b) => b.processedValue - a.processedValue) // 排序
    .slice(0, 10); // 只取前10个
}

function categorize(value) {
  return value > 100 ? 'high' : value > 50 ? 'medium' : 'low';
}

function validateItem(item) {
  return item.name && item.value > 0 && item.value < 1000;
}
```

### 3. 权限管理系统

```javascript
class PermissionManager {
  constructor() {
    this.permissions = new Map();
  }

  // 添加权限
  addPermission(role, permission) {
    if (!this.permissions.has(role)) {
      this.permissions.set(role, new Set());
    }
    this.permissions.get(role).add(permission);
  }

  // 检查权限
  hasPermission(userRole, requiredPermission) {
    const userPermissions = this.permissions.get(userRole);
    return userPermissions ? userPermissions.has(requiredPermission) : false;
  }

  // 检查多个权限（所有权限都必须满足）
  hasAllPermissions(userRole, permissions) {
    return permissions.every(perm => this.hasPermission(userRole, perm));
  }

  // 检查多个权限（至少满足一个权限）
  hasAnyPermission(userRole, permissions) {
    return permissions.some(perm => this.hasPermission(userRole, perm));
  }
}

// 使用示例
const pm = new PermissionManager();

pm.addPermission('admin', 'read');
pm.addPermission('admin', 'write');
pm.addPermission('admin', 'delete');
pm.addPermission('user', 'read');

console.log(pm.hasPermission('admin', 'write')); // true
console.log(pm.hasPermission('user', 'write')); // false
console.log(pm.hasAllPermissions('admin', ['read', 'write'])); // true
console.log(pm.hasAnyPermission('user', ['read', 'write'])); // true
```

## 总结

JavaScript 运算符是编程中的基础工具，掌握它们对于编写高效、正确的代码至关重要。通过本文的学习，你应该能够：

1. **理解各类运算符**：清楚每种运算符的作用和用法
2. **避免常见陷阱**：特别是类型转换、浮点数精度等问题
3. **优化代码性能**：合理使用运算符，提高代码效率
4. **编写清晰代码**：使用括号、注释等提高可读性
5. **解决实际问题**：运用运算符解决各种编程问题

### 最佳实践总结：

1. **使用严格相等**：始终使用 `===` 而不是 `==`
2. **使用空值合并**：使用 `??` 而不是 `||` 来处理 null/undefined
3. **使用可选链**：使用 `?.` 安全访问嵌套属性
4. **注意优先级**：不确定时使用括号明确优先级
5. **避免类型转换**：显式转换类型，避免隐式转换的陷阱

持续练习和实践，你将能够熟练运用 JavaScript 运算符，编写出更加优雅、高效的代码。