---
title: JavaScript 字符串处理
published: 2022-03-12
description: '常用字符串方法和模板字符串的详细介绍和学习笔记'
image: ''
tags: ["JS基础","字符串"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 字符串处理是前端开发中的重要内容，本文将详细介绍字符串的创建、常用方法、模板字符串以及实际应用场景。

## 字符串基础

### 创建字符串

JavaScript 提供了多种创建字符串的方式：

```javascript
// 使用单引号
let str1 = 'Hello World';

// 使用双引号
let str2 = "Hello World";

// 使用反引号（模板字符串）
let str3 = `Hello World`;

// 使用 String 构造函数
let str4 = String('Hello World');

// 从字符数组创建
let str5 = String.fromCharCode(72, 101, 108, 108, 111); // "Hello"
```

### 字符串的不可变性

JavaScript 字符串是不可变的，一旦创建就不能修改：

```javascript
let str = 'Hello';
str[0] = 'h'; // 不会生效
console.log(str); // "Hello"

// 必须创建新字符串
str = str.toLowerCase();
console.log(str); // "hello"
```

## 常用字符串方法

### 1. 字符串查找和检查

#### includes() - 检查是否包含子字符串

```javascript
let text = 'Hello, JavaScript!';

console.log(text.includes('Java')); // true
console.log(text.includes('Python')); // false
console.log(text.includes('hello')); // false (区分大小写)

// 从指定位置开始查找
console.log(text.includes('JavaScript', 10)); // false
```

#### startsWith() 和 endsWith()

```javascript
let filename = 'document.pdf';

console.log(filename.startsWith('doc')); // true
console.log(filename.endsWith('.pdf')); // true
console.log(filename.endsWith('.txt')); // false

// 从指定位置开始检查
console.log(filename.startsWith('ument', 3)); // true
```

#### indexOf() 和 lastIndexOf()

```javascript
let text = 'Hello World, Hello JavaScript';

console.log(text.indexOf('Hello')); // 0
console.log(text.indexOf('Hello', 6)); // 13
console.log(text.lastIndexOf('Hello')); // 13
console.log(text.indexOf('Python')); // -1 (未找到)
```

### 2. 字符串提取

#### slice() - 提取子字符串

```javascript
let text = 'JavaScript is awesome';

console.log(text.slice(0, 10)); // "JavaScript"
console.log(text.slice(4)); // "Script is awesome"
console.log(text.slice(-7)); // "awesome"
console.log(text.slice(-10, -7)); // "awe"

// 如果索引超出范围，返回空字符串
console.log(text.slice(100)); // ""
```

#### substring() - 提取子字符串

```javascript
let text = 'JavaScript';

console.log(text.substring(0, 4)); // "Java"
console.log(text.substring(4)); // "Script"

// 自动交换参数
console.log(text.substring(10, 4)); // "Script"

// 不支持负数索引
console.log(text.substring(-3)); // "JavaScript"
```

#### substr() - 提取指定长度的子字符串

```javascript
let text = 'JavaScript';

console.log(text.substr(0, 4)); // "Java"
console.log(text.substr(4, 6)); // "Script"

// 从末尾开始计数
console.log(text.substr(-6)); // "Script"

// 注意：substr 已被标记为过时，建议使用 slice 或 substring
```

#### charAt() 和 charCodeAt()

```javascript
let text = 'Hello';

console.log(text.charAt(0)); // "H"
console.log(text.charAt(10)); // "" (超出范围返回空字符串)
console.log(text.charCodeAt(0)); // 72

// 使用方括号语法
console.log(text[0]); // "H"
console.log(text[10]); // undefined (超出范围返回 undefined)
```

### 3. 字符串转换

#### toUpperCase() 和 toLowerCase()

```javascript
let text = 'JavaScript';

console.log(text.toUpperCase()); // "JAVASCRIPT"
console.log(text.toLowerCase()); // "javascript"

// 本地化大小写转换
console.log(text.toLocaleUpperCase('tr-TR'));
```

#### trim() - 去除空白字符

```javascript
let text = '  Hello World  ';

console.log(text.trim()); // "Hello World"
console.log(text.trimStart()); // "Hello World  "
console.log(text.trimEnd()); // "  Hello World"

// 自定义空白字符
let custom = '---Hello---';
console.log(custom.replace(/^-+|-+$/g, '')); // "Hello"
```

### 4. 字符串替换

#### replace() 和 replaceAll()

```javascript
let text = 'Hello World, Hello JavaScript';

// 替换第一个匹配项
console.log(text.replace('Hello', 'Hi')); // "Hi World, Hello JavaScript"

// 使用正则表达式替换所有匹配项
console.log(text.replace(/Hello/g, 'Hi')); // "Hi World, Hi JavaScript"

// replaceAll() 替换所有匹配项（ES2021）
console.log(text.replaceAll('Hello', 'Hi')); // "Hi World, Hi JavaScript"

// 使用回调函数
let result = text.replace(/Hello/g, (match, offset) => {
  return match.toUpperCase();
});
console.log(result); // "HELLO World, HELLO JavaScript"
```

### 5. 字符串分割和连接

#### split() - 分割字符串为数组

```javascript
let text = 'JavaScript,Python,Java,C++';

// 使用逗号分割
console.log(text.split(',')); // ["JavaScript", "Python", "Java", "C++"]

// 限制分割次数
console.log(text.split(',', 2)); // ["JavaScript", "Python"]

// 使用正则表达式
let sentence = 'Hello  World  JavaScript';
console.log(sentence.split(/\s+/)); // ["Hello", "World", "JavaScript"]

// 转换为字符数组
console.log(text.split('')); // ["J", "a", "v", "a", "S", "c", "r", "i", "p", "t", ",", "P", ...]
```

#### join() - 连接数组为字符串

```javascript
let arr = ['JavaScript', 'Python', 'Java'];

console.log(arr.join(', ')); // "JavaScript, Python, Java"
console.log(arr.join(' - ')); // "JavaScript - Python - Java"
console.log(arr.join()); // "JavaScript,Python,Java"
```

### 6. 字符串重复和填充

#### repeat() - 重复字符串

```javascript
let star = '*';
console.log(star.repeat(5)); // "*****"
console.log('Hello'.repeat(3)); // "HelloHelloHello"
console.log('abc'.repeat(0)); // ""

// 创建分隔线
console.log('-'.repeat(20)); // "--------------------"
```

#### padStart() 和 padEnd()

```javascript
// 字符串填充
let num = '42';
console.log(num.padStart(5, '0')); // "00042"
console.log(num.padEnd(5, '0')); // "42000"

// 格式化输出
console.log('123'.padStart(6, '*')); // "***123"
console.log('test'.padEnd(10, '-')); // "test------"

// 实际应用：对齐文本
console.log('Alice'.padEnd(10) + ' 25');   // "Alice       25"
console.log('Bob'.padEnd(10) + ' 30');     // "Bob         30"
```

## 模板字符串

模板字符串（Template Literals）使用反引号（`）创建，提供更强大的字符串处理能力。

### 基本用法

```javascript
let name = 'JavaScript';
let version = 'ES6';

// 多行字符串
let message = `Hello, ${name}!
Welcome to ${version}.
Enjoy coding!`;

console.log(message);
// Hello, JavaScript!
// Welcome to ES6.
// Enjoy coding!
```

### 表达式插值

```javascript
let price = 99.99;
let quantity = 5;
let total = price * quantity;

console.log(`总价: ${total.toFixed(2)} 元`); // "总价: 499.95 元"

// 调用函数
function greet(name) {
  return `Hello, ${name}!`;
}
console.log(`欢迎: ${greet('World')}`); // "欢迎: Hello, World!"

// 使用表达式
console.log(`10 + 20 = ${10 + 20}`); // "10 + 20 = 30"
console.log(`是否成年: ${18 >= 18 ? '是' : '否'}`); // "是否成年: 是"
```

### 标签模板

```javascript
// 自定义标签函数
function highlight(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] !== undefined ? `<strong>${values[i]}</strong>` : '';
    return result + string + value;
  }, '');
}

let name = 'JavaScript';
let version = 'ES6';
let result = highlight`欢迎学习 ${name} ${version}!`;
console.log(result); // "欢迎学习 <strong>JavaScript</strong> <strong>ES6</strong>!"

// HTML 转义示例
function safeHtml(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] !== undefined
      ? String(values[i]).replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;')
      : '';
    return result + string + value;
  }, '');
}

let userInput = '<script>alert("XSS")</script>';
let safe = safeHtml`用户输入: ${userInput}`;
console.log(safe); // "用户输入: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;"
```

### 原始字符串

```javascript
// 使用 String.raw 获取原始字符串
let raw = String.raw`Hello\nWorld`;
console.log(raw); // "Hello\nWorld" (不转义)

let normal = `Hello\nWorld`;
console.log(normal); // "Hello\nWorld" (转义换行)
```

## 实际应用

### 1. 字符串格式化

```javascript
// 格式化金额
function formatMoney(amount) {
  return `¥${amount.toFixed(2).replace(/\d(?=(\d{3})+\.)/g, '$&,')}`;
}
console.log(formatMoney(1234567.89)); // "¥1,234,567.89"

// 格式化电话号码
function formatPhone(phone) {
  return phone.replace(/(\d{3})(\d{4})(\d{4})/, '$1-$2-$3');
}
console.log(formatPhone('13812345678')); // "138-1234-5678"

// 格式化日期
function formatDate(date) {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
}
console.log(formatDate(new Date())); // "2025-01-14"
```

### 2. 字符串验证

```javascript
// 验证邮箱
function isValidEmail(email) {
  const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return regex.test(email);
}

console.log(isValidEmail('test@example.com')); // true
console.log(isValidEmail('invalid.email')); // false

// 验证手机号
function isValidPhone(phone) {
  const regex = /^1[3-9]\d{9}$/;
  return regex.test(phone);
}

console.log(isValidPhone('13812345678')); // true
console.log(isValidPhone('12345678901')); // false

// 验证密码强度
function checkPassword(password) {
  const hasUpper = /[A-Z]/.test(password);
  const hasLower = /[a-z]/.test(password);
  const hasNumber = /\d/.test(password);
  const hasSpecial = /[!@#$%^&*]/.test(password);

  return {
    score: [hasUpper, hasLower, hasNumber, hasSpecial].filter(Boolean).length,
    requirements: {
      uppercase: hasUpper,
      lowercase: hasLower,
      number: hasNumber,
      special: hasSpecial,
      length: password.length >= 8
    }
  };
}

console.log(checkPassword('abc123')); // { score: 2, requirements: {...} }
```

### 3. 字符串搜索

```javascript
// 查找所有匹配项
function findAllMatches(text, pattern) {
  const matches = [];
  let match;
  const regex = new RegExp(pattern, 'g');

  while ((match = regex.exec(text)) !== null) {
    matches.push({
      text: match[0],
      index: match.index,
      groups: match.slice(1)
    });
  }

  return matches;
}

const text = 'JavaScript is JavaScript, not Java';
const results = findAllMatches(text, 'Java');
console.log(results);
// [
//   { text: 'Java', index: 0, groups: [] },
//   { text: 'Java', index: 14, groups: [] },
//   { text: 'Java', index: 28, groups: [] }
// ]

// 模糊搜索
function fuzzySearch(query, text) {
  const queryLower = query.toLowerCase();
  const textLower = text.toLowerCase();

  let queryIndex = 0;
  for (let i = 0; i < textLower.length && queryIndex < queryLower.length; i++) {
    if (textLower[i] === queryLower[queryIndex]) {
      queryIndex++;
    }
  }

  return queryIndex === queryLower.length;
}

console.log(fuzzySearch('js', 'JavaScript')); // true
console.log(fuzzySearch('abc', 'JavaScript')); // false
```

### 4. 字符串处理工具

```javascript
// 驼峰命名转换
function toCamelCase(str) {
  return str
    .replace(/[-_\s]+(.)?/g, (_, char) => char ? char.toUpperCase() : '')
    .replace(/^(.)/, (_, char) => char.toLowerCase());
}

console.log(toCamelCase('hello-world')); // "helloWorld"
console.log(toCamelCase('hello_world')); // "helloWorld"
console.log(toCamelCase('hello world')); // "helloWorld"

// 蛇形命名转换
function toSnakeCase(str) {
  return str
    .replace(/([A-Z])/g, '_$1')
    .toLowerCase()
    .replace(/^_/, '');
}

console.log(toSnakeCase('helloWorld')); // "hello_world"
console.log(toSnakeCase('HelloWorld')); // "hello_world"

// 首字母大写
function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

console.log(capitalize('hello world')); // "Hello world"

// 截断文本
function truncate(str, length, suffix = '...') {
  if (str.length <= length) return str;
  return str.slice(0, length - suffix.length) + suffix;
}

console.log(truncate('Hello, JavaScript!', 10)); // "Hello, ..."
```

## 性能优化

### 1. 避免频繁的字符串拼接

```javascript
// 低效的方式
let result = '';
for (let i = 0; i < 10000; i++) {
  result += i; // 每次都创建新字符串
}

// 高效的方式 - 使用数组
let parts = [];
for (let i = 0; i < 10000; i++) {
  parts.push(i);
}
let result = parts.join('');

// 高效的方式 - 使用模板字符串
let result = '';
for (let i = 0; i < 10000; i++) {
  result = `${result}${i}`;
}
```

### 2. 正则表达式优化

```javascript
// 缓存正则表达式
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

function validateEmail(email) {
  return EMAIL_REGEX.test(email);
}

// 使用非贪婪量词
const lazyRegex = /<.*?>/; // 匹配第一个标签
const greedyRegex = /<.*>/; // 匹配到最后一个标签
```

### 3. 选择合适的方法

```javascript
// 简单查找 - 使用 includes
if (text.includes('keyword')) { }

// 需要索引 - 使用 indexOf
if (text.indexOf('keyword') !== -1) { }

// 复杂模式 - 使用正则表达式
if (text.match(/pattern/i)) { }

// 替换所有匹配 - 使用 replaceAll
text = text.replaceAll('old', 'new');
```

## 注意事项

1. **区分大小写**：大多数字符串方法默认区分大小写，如需忽略大小写可先转换或使用正则表达式。

2. **索引越界**：访问超出范围的索引不会抛出错误，而是返回空字符串或 undefined。

3. **正则表达式**：复杂字符串操作时，合理使用正则表达式可以大大简化代码。

4. **编码问题**：处理多语言文本时，注意使用 `codePointAt()` 和 `String.fromCodePoint()` 处理 Unicode 字符。

5. **性能考虑**：避免在循环中进行大量的字符串操作，优先使用数组拼接。

6. **模板注入**：使用模板字符串插值时，注意防止 XSS 攻击，对用户输入进行转义。

## 总结

通过本文的学习，相信你已经对 JavaScript 字符串处理有了更深入的理解。掌握这些字符串方法和技巧，将帮助你更高效地处理文本数据，提升代码质量和开发效率。在实际开发中，根据具体需求选择合适的方法，并注意性能优化和安全性问题。