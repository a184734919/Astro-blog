---
title: JavaScript 正则表达式基础
published: 2022-06-28
description: '正则表达式语法和常用方法的详细介绍和学习笔记'
image: ''
tags: ["JS","正则"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

正则表达式（Regular Expression，简称 Regex）是一种强大的文本模式匹配工具，用于在字符串中搜索、替换、验证特定的模式。JavaScript 通过 RegExp 对象支持正则表达式，广泛应用于表单验证、数据提取、文本处理等场景。

## 创建正则表达式

### 字面量方式

```javascript
// 使用斜杠 / 包裹正则表达式
const regex1 = /pattern/;

// 带标志的正则表达式
const regex2 = /pattern/gi;

// 示例：匹配数字
const numberPattern = /\d+/;
console.log(numberPattern.test('123')); // true
console.log(numberPattern.test('abc')); // false
```

### 构造函数方式

```javascript
// 使用 RegExp 构造函数
const regex1 = new RegExp('pattern');

// 带标志
const regex2 = new RegExp('pattern', 'gi');

// 动态创建正则表达式
const keyword = 'hello';
const dynamicRegex = new RegExp(keyword, 'i');
console.log(dynamicRegex.test('Hello World')); // true
```

## 正则表达式标志

### 常用标志

```javascript
// g - 全局匹配（global）
const globalRegex = /a/g;
console.log('aaa'.match(globalRegex)); // ['a', 'a', 'a']

// i - 忽略大小写（ignore case）
const caseInsensitiveRegex = /hello/i;
console.log(caseInsensitiveRegex.test('HELLO')); // true

// m - 多行模式（multiline）
const multilineRegex = /^hello/m;
const text = 'world\nhello';
console.log(multilineRegex.test(text)); // true

// u - Unicode 模式
const unicodeRegex = /\u{1F600}/u;
console.log(unicodeRegex.test('😀')); // true

// y - 粘附模式（sticky）
const stickyRegex = /\d+/y;
console.log(stickyRegex.test('123abc')); // true

// s - dotAll 模式（ES2018）
const dotAllRegex = /hello.world/s;
console.log(dotAllRegex.test('hello\nworld')); // true
```

## 字符类

### 基本字符类

```javascript
// \d - 匹配数字
const digitPattern = /\d/;
console.log(digitPattern.test('a1b')); // true

// \D - 匹配非数字
const nonDigitPattern = /\D/;
console.log(nonDigitPattern.test('a1b')); // true

// \w - 匹配字母、数字、下划线
const wordCharPattern = /\w/;
console.log(wordCharPattern.test('a1_')); // true

// \W - 匹配非字母、数字、下划线
const nonWordCharPattern = /\W/;
console.log(nonWordCharPattern.test('@#')); // true

// \s - 匹配空白字符（空格、制表符、换行等）
const whitespacePattern = /\s/;
console.log(whitespacePattern.test('a b')); // true

// \S - 匹配非空白字符
const nonWhitespacePattern = /\S/;
console.log(nonWhitespacePattern.test('a b')); // true

// . - 匹配任意字符（除换行符外）
const anyCharPattern = /./;
console.log(anyCharPattern.test('a')); // true
console.log(anyCharPattern.test('\n')); // false
```

### 自定义字符类

```javascript
// [abc] - 匹配 a、b 或 c
const abcPattern = /[abc]/;
console.log(abcPattern.test('a')); // true
console.log(abcPattern.test('d')); // false

// [^abc] - 匹配除 a、b、c 之外的字符
const notAbcPattern = /[^abc]/;
console.log(notAbcPattern.test('d')); // true
console.log(notAbcPattern.test('a')); // false

// [a-z] - 匹配小写字母
const lowercasePattern = /[a-z]/;
console.log(lowercasePattern.test('a')); // true
console.log(lowercasePattern.test('A')); // false

// [A-Z] - 匹配大写字母
const uppercasePattern = /[A-Z]/;
console.log(uppercasePattern.test('A')); // true

// [0-9] - 匹配数字
const numberPattern2 = /[0-9]/;
console.log(numberPattern2.test('5')); // true

// [a-zA-Z0-9] - 匹配字母和数字
const alnumPattern = /[a-zA-Z0-9]/;
console.log(alnumPattern.test('A1')); // true
```

## 量词

### 基本量词

```javascript
// * - 0 次或多次
const starPattern = /a*/;
console.log('aaa'.match(starPattern)); // ['aaa']

// + - 1 次或多次
const plusPattern = /a+/;
console.log('aaa'.match(plusPattern)); // ['aaa']

// ? - 0 次或 1 次
const questionPattern = /a?/;
console.log('aa'.match(questionPattern)); // ['a']

// {n} - 恰好 n 次
const exactlyPattern = /a{3}/;
console.log(exactlyPattern.test('aaa')); // true
console.log(exactlyPattern.test('aa')); // false

// {n,} - 至少 n 次
const atLeastPattern = /a{2,}/;
console.log(atLeastPattern.test('aaa')); // true
console.log(atLeastPattern.test('a')); // false

// {n,m} - n 到 m 次
const rangePattern = /a{2,4}/;
console.log(rangePattern.test('aaa')); // true
console.log(rangePattern.test('a')); // false
console.log(rangePattern.test('aaaaa')); // false
```

### 贪婪与非贪婪

```javascript
// 默认贪婪匹配
const greedyPattern = /a.*b/;
console.log('aabb aab'.match(greedyPattern)); // ['aabb aab']

// 非贪婪（懒惰）匹配
const lazyPattern = /a.*?b/;
console.log('aabb aab'.match(lazyPattern)); // ['aab']
```

## 锚点

### 位置锚点

```javascript
// ^ - 匹配字符串开头
const startPattern = /^hello/;
console.log(startPattern.test('hello world')); // true
console.log(startPattern.test('world hello')); // false

// $ - 匹配字符串结尾
const endPattern = /world$/;
console.log(endPattern.test('hello world')); // true
console.log(endPattern.test('world hello')); // false

// \b - 匹配单词边界
const wordBoundaryPattern = /\bhello\b/;
console.log(wordBoundaryPattern.test('hello world')); // true
console.log(wordBoundaryPattern.test('helloworld')); // false

// \B - 匹配非单词边界
const nonWordBoundaryPattern = /\Bhello\B/;
console.log(nonWordBoundaryPattern.test('helloworld')); // true
```

## 正则表达式方法

### test() 方法

```javascript
// 检查字符串是否匹配模式
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailPattern.test('test@example.com')); // true
console.log(emailPattern.test('invalid.email')); // false
```

### exec() 方法

```javascript
// 执行搜索，返回匹配信息
const pattern = /(\d+)-(\w+)/;
const text = '123-abc 456-def';

let match;
while ((match = pattern.exec(text)) !== null) {
  console.log(match[0]);   // 完整匹配
  console.log(match[1]);   // 第一个捕获组
  console.log(match[2]);   // 第二个捕获组
  console.log(match.index); // 匹配位置
}
```

### String.prototype.match()

```javascript
// 不带 g 标志：返回第一个匹配
const pattern1 = /(\d+)-(\w+)/;
const text1 = '123-abc 456-def';

const match1 = text1.match(pattern1);
console.log(match1);
// ['123-abc', '123', 'abc', index: 0, input: '123-abc 456-def']

// 带 g 标志：返回所有匹配
const pattern2 = /(\d+)-(\w+)/g;
const text2 = '123-abc 456-def';

const match2 = text2.match(pattern2);
console.log(match2); // ['123-abc', '456-def']
```

### String.prototype.matchAll()

```javascript
// 返回所有匹配的迭代器
const pattern = /(\d+)-(\w+)/g;
const text = '123-abc 456-def';

const matches = [...text.matchAll(pattern)];
console.log(matches);
// [
//   ['123-abc', '123', 'abc', index: 0, input: '123-abc 456-def'],
//   ['456-def', '456', 'def', index: 7, input: '123-abc 456-def']
// ]
```

### String.prototype.replace()

```javascript
// 替换匹配的字符串
const pattern = /apple/g;
const text = 'I like apple and apple';

const replaced = text.replace(pattern, 'orange');
console.log(replaced); // 'I like orange and orange'

// 使用函数替换
const numbers = '1 2 3 4 5';
const doubled = numbers.replace(/\d+/g, match => {
  return Number(match) * 2;
});
console.log(doubled); // '2 4 6 8 10'

// 使用捕获组
const date = '2023-12-25';
const formatted = date.replace(/(\d{4})-(\d{2})-(\d{2})/, '$2/$3/$1');
console.log(formatted); // '12/25/2023'
```

### String.prototype.search()

```javascript
// 查找第一个匹配的位置
const pattern = /world/;
const text = 'hello world';

const index = text.search(pattern);
console.log(index); // 6
```

### String.prototype.split()

```javascript
// 使用正则表达式分割字符串
const text = 'a1b2c3d4e';
const parts = text.split(/\d/);
console.log(parts); // ['a', 'b', 'c', 'd', 'e']

// 捕获组保留分隔符
const text2 = 'a1b2c3';
const parts2 = text2.split(/(\d)/);
console.log(parts2); // ['a', '1', 'b', '2', 'c', '3']
```

## 捕获组

### 基本捕获组

```javascript
// ( ) - 捕获组
const pattern = /(\d{4})-(\d{2})-(\d{2})/;
const date = '2023-12-25';

const match = date.match(pattern);
console.log(match[1]); // '2023'
console.log(match[2]); // '12'
console.log(match[3]); // '25'
```

### 非捕获组

```javascript
// (?: ) - 非捕获组
const pattern = /(?:\d{4})-(\d{2})-(\d{2})/;
const date = '2023-12-25';

const match = date.match(pattern);
console.log(match[1]); // '12'
console.log(match[2]); // '25'
console.log(match[0]); // '2023-12-25'
```

### 命名捕获组

```javascript
// (?<name> ) - 命名捕获组
const pattern = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const date = '2023-12-25';

const match = date.match(pattern);
console.log(match.groups.year); // '2023'
console.log(match.groups.month); // '12'
console.log(match.groups.day); // '25'
```

## 转义字符

```javascript
// 转义特殊字符
const specialChars = /\\[\^\$\.\|\?\*\+\(\)\[\]\{\}\\]/g;
const text = '\\^\\$\\.\\|\\?\\*\\+\\(\\)\\[\\]\\{\\}';

console.log(text.replace(specialChars, '\\$&')); // \^\$\.\|\?\*\+\(\)\[\]\{\}

// 常用转义
console.log(/\./.test('a.b')); // true (匹配点号)
console.log(/\$/.test('$100')); // true (匹配美元符号)
console.log(/\*/.test('a*b')); // true (匹配星号)
```

## 常用正则表达式

### 邮箱验证

```javascript
const emailPattern = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;

console.log(emailPattern.test('test@example.com')); // true
console.log(emailPattern.test('test.example.com')); // false
console.log(emailPattern.test('test@example')); // false
```

### 手机号验证

```javascript
// 中国手机号
const phonePattern = /^1[3-9]\d{9}$/;

console.log(phonePattern.test('13800138000')); // true
console.log(phonePattern.test('10800138000')); // false
console.log(phonePattern.test('1380013800')); // false
```

### 身份证号验证

```javascript
// 18位身份证号
const idCardPattern = /^[1-9]\d{5}(18|19|20)\d{2}(0[1-9]|1[0-2])(0[1-9]|[12]\d|3[01])\d{3}[\dXx]$/;

console.log(idCardPattern.test('11010519491231002X')); // true
console.log(idCardPattern.test('110105194912310021')); // false
```

### URL 验证

```javascript
const urlPattern = /^https?:\/\/[\w\-]+(\.[\w\-]+)+[/#?]?.*$/;

console.log(urlPattern.test('https://www.example.com')); // true
console.log(urlPattern.test('http://example.com/path')); // true
console.log(urlPattern.test('ftp://example.com')); // false
```

### IPv4 地址验证

```javascript
const ipv4Pattern = /^(\d{1,3}\.){3}\d{1,3}$/;

console.log(ipv4Pattern.test('192.168.1.1')); // true
console.log(ipv4Pattern.test('256.168.1.1')); // true (需要额外验证范围)
console.log(ipv4Pattern.test('192.168.1')); // false
```

### 密码强度验证

```javascript
// 至少8位，包含大小写字母、数字
const passwordPattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/;

console.log(passwordPattern.test('Password123')); // true
console.log(passwordPattern.test('password')); // false
console.log(passwordPattern.test('PASSWORD')); // false
console.log(passwordPattern.test('Password')); // false
```

## 实际应用

### 表单验证

```javascript
function validateForm(formData) {
  const errors = {};

  // 邮箱验证
  if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(formData.email)) {
    errors.email = '请输入有效的邮箱地址';
  }

  // 手机号验证
  if (!/^1[3-9]\d{9}$/.test(formData.phone)) {
    errors.phone = '请输入有效的手机号码';
  }

  // 密码验证
  if (!/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d).{8,}$/.test(formData.password)) {
    errors.password = '密码至少8位，包含大小写字母和数字';
  }

  return {
    isValid: Object.keys(errors).length === 0,
    errors
  };
}

// 使用
const formData = {
  email: 'test@example.com',
  phone: '13800138000',
  password: 'Password123'
};

const result = validateForm(formData);
console.log(result.isValid); // true
```

### 数据提取

```javascript
// 提取URL中的参数
function getUrlParams(url) {
  const pattern = /[?&]([^=#]+)=([^&#]*)/g;
  const params = {};
  let match;

  while ((match = pattern.exec(url)) !== null) {
    params[decodeURIComponent(match[1])] = decodeURIComponent(match[2]);
  }

  return params;
}

const url = 'https://example.com?name=John&age=30';
const params = getUrlParams(url);
console.log(params); // { name: 'John', age: '30' }
```

### 文本清理

```javascript
// 去除HTML标签
function stripHtml(html) {
  return html.replace(/<[^>]*>/g, '');
}

const html = '<div><p>Hello</p></div>';
console.log(stripHtml(html)); // 'Hello'

// 去除多余空格
function trimSpaces(text) {
  return text.replace(/\s+/g, ' ').trim();
}

const text = 'Hello    world   !';
console.log(trimSpaces(text)); // 'Hello world !'
```

### 高亮关键词

```javascript
function highlightKeywords(text, keywords) {
  const pattern = new RegExp(`(${keywords.join('|')})`, 'gi');
  return text.replace(pattern, '<mark>$1</mark>');
}

const text = 'JavaScript is a powerful language. JavaScript is used everywhere.';
const keywords = ['JavaScript', 'powerful'];

const highlighted = highlightKeywords(text, keywords);
console.log(highlighted);
// '<mark>JavaScript</mark> is a <mark>powerful</mark> language. <mark>JavaScript</mark> is used everywhere.'
```

## 注意事项

1. **性能考虑**：复杂的正则表达式可能影响性能，避免过度使用
2. **转义特殊字符**：需要匹配特殊字符时记得转义
3. **贪婪 vs 非贪婪**：根据需要选择贪婪或非贪婪匹配
4. **锚点使用**：合理使用锚点可以避免不必要的匹配
5. **正则表达式测试**：使用在线工具测试和调试正则表达式
6. **可读性**：复杂的正则表达式可以拆分成多个步骤
7. **边界情况**：考虑空字符串、特殊字符等边界情况
8. **全局标志**：使用全局标志时注意 exec 方法的使用

## 最佳实践

1. **使用字面量**：静态正则表达式优先使用字面量方式
2. **添加注释**：复杂的正则表达式添加注释说明
3. **测试充分**：对正则表达式进行充分的测试
4. **性能优化**：避免使用回溯，优先使用简单模式
5. **工具辅助**：使用在线工具帮助编写和测试正则表达式
6. **代码复用**：将常用的正则表达式封装成函数
7. **错误处理**：处理正则表达式可能出现的错误
8. **文档说明**：为正则表达式添加说明文档

## 总结

通过本文的学习，相信你已经对 JavaScript 正则表达式基础有了更深入的理解。正则表达式是处理字符串的强大工具，掌握基本的语法和方法，可以帮助你更高效地完成字符串匹配、验证、提取等任务。

记住，编写好的正则表达式需要：
- 明确的匹配目标
- 合理的字符选择
- 适当的量词使用
- 正确的锚点定位
- 充分的测试验证

持续练习和实践，你将能够熟练运用正则表达式解决各种字符串处理问题。