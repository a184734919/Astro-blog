---
title: JavaScript 正则表达式进阶
published: 2022-07-05
description: '高级正则表达式技巧的详细介绍和学习笔记'
image: ''
tags: ["JS","正则"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 正则表达式进阶技巧是在掌握基础知识后，学习更复杂的匹配模式、性能优化和实际应用场景。本文将详细介绍高级正则表达式技巧，帮助你在实际项目中高效地处理复杂的文本匹配需求。

## 捕获组和非捕获组

### 普通捕获组

```javascript
// 基本捕获组
const regex = /(\d{4})-(\d{2})-(\d{2})/;
const date = '2023-12-25';
const match = date.match(regex);

console.log(match[0]); // '2023-12-25' (完整匹配)
console.log(match[1]); // '2023' (第一个捕获组)
console.log(match[2]); // '12' (第二个捕获组)
console.log(match[3]); // '25' (第三个捕获组)

// 使用捕获组进行替换
const formatted = date.replace(regex, '$2/$3/$1');
console.log(formatted); // '12/25/2023'

// 命名捕获组（ES2018）
const namedRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const namedMatch = date.match(namedRegex);
console.log(namedMatch.groups.year); // '2023'
console.log(namedMatch.groups.month); // '12'
console.log(namedMatch.groups.day); // '25'
```

### 非捕获组

```javascript
// 非捕获组 (?:)
const regex1 = /(?:https?|ftp):\/\/([^\s\/]+)/;
const url = 'https://example.com/path';
const match1 = url.match(regex1);
console.log(match1[1]); // 'example.com' (只有需要的部分被捕获)

// 对比：普通捕获组
const regex2 = /(https?|ftp):\/\/([^\s\/]+)/;
const match2 = url.match(regex2);
console.log(match2[1]); // 'https'
console.log(match2[2]); // 'example.com'
```

### 嵌套捕获组

```javascript
// 嵌套捕获组
const htmlRegex = /<([a-z]+)(?:\s+[^>]*)?>([^<]*)<\/\1>/;
const html = '<div>Hello</div>';
const htmlMatch = html.match(htmlRegex);

console.log(htmlMatch[1]); // 'div' (标签名)
console.log(htmlMatch[2]); // 'Hello' (内容)

// 复杂的嵌套示例
const complexRegex = /((?:\d{3}-)?\d{3}-\d{4})/;
const phone = '(123)456-7890';
const phoneMatch = phone.match(complexRegex);
console.log(phoneMatch[1]); // '(123)456-7890'
```

## 零宽断言

### 正向先行断言 (?=)

```javascript
// 匹配后面跟着特定文本的内容
const regex1 = /\d+(?= yuan)/;
const text1 = '100 yuan 50 dollars';
console.log(text1.match(regex1)[0]); // '100'

// 匹配单词结尾是 'ing' 的单词
const regex2 = /\w+(?=ing\b)/g;
const text2 = 'running jumping swimming';
console.log(text2.match(regex2)); // ['runn', 'jump', 'swimm']
```

### 负向先行断言 (?!)

```javascript
// 匹配后面不跟着特定文本的内容
const regex1 = /\d+(?! yuan)/;
const text1 = '100 yuan 50 dollars';
console.log(text1.match(regex1)[0]); // '50'

// 匹配不以 'ing' 结尾的单词
const regex2 = /\b\w+(?!ing\b)\b/g;
const text2 = 'running jumping swim walk';
console.log(text2.match(regex2)); // ['swim', 'walk']
```

### 正向后行断言 (?<=) （ES2018）

```javascript
// 匹配前面有特定文本的内容
const regex1 = /(?<=\$)\d+/;
const text1 = 'Price: $100';
console.log(text1.match(regex1)[0]); // '100'

// 匹配 'apple:' 后面的单词
const regex2 = /(?<=apple:)\s*\w+/g;
const text2 = 'apple: red banana: yellow';
console.log(text2.match(regex2)); // [' red', ' yellow']
```

### 负向后行断言 (?<!) （ES2018）

```javascript
// 匹配前面没有特定文本的内容
const regex1 = /(?<!\$)\d+/;
const text1 = '$100 200';
console.log(text1.match(regex1)[0]); // '200'

// 匹配不是 'apple:' 后面的单词
const regex2 = /(?<!apple:)\s*\w+/g;
const text2 = 'apple: red banana: yellow';
console.log(text2.match(regex2)); // [' yellow']
```

## 反向引用

### 基本反向引用

```javascript
// 匹配重复单词
const regex = /\b(\w+)\s+\1\b/g;
const text = 'hello hello world world test';
console.log(text.match(regex)); // ['hello hello', 'world world']

// 在替换中使用反向引用
const replaced = text.replace(regex, '$1 (repeated)');
console.log(replaced); // 'hello (repeated) world (repeated) test'
```

### 命名反向引用

```javascript
// 使用命名捕获组进行反向引用
const regex = /(?<word>\w+)\s+(?<duplicate>\k<word>)/g;
const text = 'test test demo demo';
console.log(text.match(regex)); // ['test test', 'demo demo']

// 在替换中使用命名反向引用
const replaced = text.replace(regex, '$<word> (duplicate: $<duplicate>)');
console.log(replaced); // 'test (duplicate: test) demo (duplicate: demo)'
```

### 多级反向引用

```javascript
// 匹配 HTML 标签对
const htmlRegex = /<(\w+)([^>]*)>.*?<\/\1>/g;
const html = '<div id="main">Content</div><p>Text</p>';
console.log(html.match(htmlRegex)); // ['<div id="main">Content</div>', '<p>Text</p>']

// 验证成对括号
const parenRegex = /\(([^()]|(?1))*\)/g; // 使用递归（某些正则引擎支持）
```

## 贪婪和非贪婪匹配

### 贪婪匹配（默认）

```javascript
// 贪婪匹配：尽可能多匹配
const greedyRegex = /a.*b/;
const text = 'aab aabab';
console.log(text.match(greedyRegex)[0]); // 'aab aabab' (尽可能匹配到最后一个 b)

const htmlRegex = /<div>.*<\/div>/;
const html = '<div>First</div><div>Second</div>';
console.log(html.match(htmlRegex)[0]); // 匹配整个字符串
```

### 非贪婪匹配

```javascript
// 非贪婪匹配：尽可能少匹配
const lazyRegex = /a.*?b/;
const text = 'aab aabab';
console.log(text.match(lazyRegex)[0]); // 'aab' (匹配到第一个 b 就停止)

const htmlRegex = /<div>.*?<\/div>/;
const html = '<div>First</div><div>Second</div>';
const match = html.match(htmlRegex);
console.log(match[0]); // '<div>First</div>'
```

### 贪婪量词示例对比

```javascript
// 贪婪 vs 非贪婪
const text = 'data: "value1", data: "value2", data: "value3"';

// 贪婪匹配所有
const greedy = /"(.*)"/g;
console.log(text.match(greedy)[0]); // '"value1", data: "value2", data: "value3"'

// 非贪婪匹配单个
const lazy = /"(.*?)"/g;
console.log(text.match(lazy)); // ['"value1"', '"value2"', '"value3"']
```

### 占有量词（某些引擎支持）

```javascript
// 占有量词 *+, ++, ?+：匹配后不回溯
const possessiveRegex = /a*+b/;
const text = 'aaab';
console.log(possessiveRegex.test(text)); // true

// 避免灾难性回溯
const dangerous = /a.*a.*a.*a.*a.*a/; // 可能性能很差
const safe = /a.*?a.*?a.*?a.*?a.*?a/; // 非贪婪更安全
```

## 回溯和性能优化

### 理解回溯

```javascript
// 简单的回溯示例
const regex = /\d+/;
const text = 'abc123def';
const match = regex.exec(text);
console.log(match[0]); // '123'

// 复杂的回溯场景
const complexRegex = /a(b|c)+d/;
const complexText = 'abcd';
console.log(complexRegex.test(complexText)); // true (尝试多种组合)
```

### 避免灾难性回溯

```javascript
// 危险的模式（指数级回溯）
const dangerous = /a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?a?aaaaaaaaaaaaaaaaaaaaaaa/;
const dangerousText = 'aaaaaaaaaaaaaaaaaaaaaaa';

// 优化的模式
const optimized = /a{20,}/;
console.log(optimized.test(dangerousText)); // true

// 使用原子分组（某些引擎支持）
const atomic = /(?>a+)/;
```

### 性能优化技巧

```javascript
// 1. 使用锚点限制匹配范围
const optimized1 = /^#\d+$/;
const text1 = '#123';
console.log(optimized1.test(text1)); // true

// 2. 使用字符类代替多个或操作
const bad = /a|b|c|d|e/;
const good = /[a-e]/;

// 3. 使用具体字符代替通配符
const bad2 = /.*hello.*/;
const good2 = /\w+hello\w+/;

// 4. 避免过度使用捕获组
const bad3 = /(\d+)-(\d+)-(\d+)/; // 三个捕获组
const good3 = /\d{4}-\d{2}-\d{2}/; // 无捕获组，需要时再加

// 5. 提前编译正则表达式
const compileRegex = /\b\w+\b/g;
for (let i = 0; i < 100; i++) {
  compileRegex.test('test'); // 重复使用编译好的正则
}
```

## 正则表达式标志详解

### g - 全局匹配

```javascript
// 全局匹配所有出现
const regex = /\d+/g;
const text = '123 abc 456 def 789';
console.log(text.match(regex)); // ['123', '456', '789']

// 使用 exec 多次匹配
const execRegex = /\d+/g;
let match;
while ((match = execRegex.exec(text)) !== null) {
  console.log(match[0], match.index);
}
```

### i - 忽略大小写

```javascript
// 不区分大小写
const regex = /javascript/i;
console.log(regex.test('JavaScript')); // true
console.log(regex.test('JAVASCRIPT')); // true
console.log(regex.test('javascript')); // true

// 配合全局使用
const globalRegex = /javascript/gi;
const text = 'JavaScript and javascript';
console.log(text.match(globalRegex)); // ['JavaScript', 'javascript']
```

### m - 多行模式

```javascript
// 多行模式影响 ^ 和 $
const regex1 = /^test/m;
const text1 = 'first line\ntest line\nanother line';
console.log(text1.match(regex1)[0]); // 'test' (匹配行首)

// 不使用多行模式
const regex2 = /^test/;
const text2 = 'first line\ntest line';
console.log(text2.match(regex2)); // null

// 结合使用
const multilineRegex = /^\s*#.*$/gm;
const code = '# comment\nfunction() {\n  # another comment\n}';
const lines = code.match(multilineRegex);
console.log(lines); // ['# comment', '# another comment']
```

### s - dotAll 模式（ES2018）

```javascript
// dotAll 模式：. 匹配包括换行符在内的所有字符
const regex1 = /a.*b/s;
const text1 = 'a\nb';
console.log(regex1.test(text1)); // true

// 不使用 dotAll
const regex2 = /a.*b/;
console.log(regex2.test(text1)); // false

// 使用 [\s\S] 的传统方法
const traditional = /a[\s\S]*b/;
console.log(traditional.test(text1)); // true
```

### u - Unicode 模式

```javascript
// Unicode 模式正确处理 Unicode 字符
const regex1 = /\u{1F600}/u;
console.log(regex1.test('😀')); // true

// 处理 surrogate pairs
const regex2 = /./u;
const emoji = '😀';
console.log(regex2.exec(emoji)[0]); // '😀'

// 不使用 u 模式
const regex3 = /./;
console.log(regex3.exec(emoji)[0]); // '\uD83D' (只匹配一半)
```

### y - sticky 模式

```javascript
// sticky 模式：从 lastIndex 开始匹配
const regex = /\d+/y;
const text = '123abc456';

regex.lastIndex = 0;
console.log(regex.exec(text)[0]); // '123'

regex.lastIndex = 6;
console.log(regex.exec(text)[0]); // '456'

// 匹配失败时 lastIndex 保持不变
regex.lastIndex = 3;
console.log(regex.exec(text)); // null
console.log(regex.lastIndex); // 3
```

## 高级应用场景

### 验证复杂格式

```javascript
// 验证 IPv4 地址
const ipv4Regex = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;
console.log(ipv4Regex.test('192.168.1.1')); // true
console.log(ipv4Regex.test('256.1.1.1')); // false

// 验证 URL
const urlRegex = /^(https?:\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;
console.log(urlRegex.test('https://example.com/path')); // true

// 验证电子邮件（简化版）
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
console.log(emailRegex.test('test@example.com')); // true
```

### 提复复杂数据

```javascript
// 提取 JSON 中的所有字符串
const jsonRegex = /"([^"\\]|\\.)*"/g;
const jsonStr = '{"name": "张三", "age": 25, "city": "北京"}';
const strings = jsonStr.match(jsonRegex);
console.log(strings); // ['"name"', '"张三"', '"age"', '"25"', '"city"', '"北京"']

// 提取 Markdown 链接
const markdownRegex = /\[([^\]]+)\]\(([^)]+)\)/g;
const markdown = '访问 [GitHub](https://github.com) 或 [Google](https://google.com)';
const links = [];
let match;
while ((match = markdownRegex.exec(markdown)) !== null) {
  links.push({ text: match[1], url: match[2] });
}
console.log(links); // [{text: 'GitHub', url: 'https://github.com'}, ...]
```

### 文本处理

```javascript
// 去除字符串首尾空格
const trimRegex = /^\s+|\s+$/g;
const text = '  hello world  ';
const trimmed = text.replace(trimRegex, '');
console.log(trimmed); // 'hello world'

// 将首字母大写
const capitalizeRegex = /\b\w/g;
const text2 = 'hello world';
const capitalized = text2.replace(capitalizeRegex, match => match.toUpperCase());
console.log(capitalized); // 'Hello World'

// 驼峰命名转换
const camelCaseRegex = /[-_\s]+(.)?/g;
const text3 = 'hello-world_test string';
const camelCase = text3.replace(camelCaseRegex, (match, char) => 
  char ? char.toUpperCase() : ''
);
console.log(camelCase); // 'helloWorldTestString'
```

### 日志分析

```javascript
// 解析日志格式
const logRegex = /^(\d{4}-\d{2}-\d{2}) (\d{2}:\d{2}:\d{2}) \[(\w+)\] (.*)$/;
const logLine = '2023-12-25 10:30:45 [INFO] User login successful';
const logMatch = logLine.match(logRegex);

console.log(logMatch[1]); // '2023-12-25' (日期)
console.log(logMatch[2]); // '10:30:45' (时间)
console.log(logMatch[3]); // 'INFO' (日志级别)
console.log(logMatch[4]); // 'User login successful' (消息)

// 提取错误信息
const errorRegex = /\[ERROR\] (.*) at line (\d+)/;
const errorLog = '[ERROR] Division by zero at line 42';
const errorMatch = errorLog.match(errorRegex);
console.log(errorMatch[1]); // 'Division by zero'
console.log(errorMatch[2]); // '42'
```

## 实际应用技巧

### 使用正则表达式工具

```javascript
// 使用正则表达式构建器（提高可读性）
const buildRegex = (pattern, flags = 'g') => new RegExp(pattern, flags);

const wordRegex = buildRegex('\\b\\w+\\b');
console.log('hello world'.match(wordRegex)); // ['hello', 'world']

// 使用函数生成正则表达式
function createTagRegex(tag) {
  return new RegExp(`<${tag}([^>]*)>(.*?)<\\/${tag}>`, 'gis');
}

const divRegex = createTagRegex('div');
const html = '<div class="main">Content</div>';
console.log(html.match(divRegex));
```

### 正则表达式测试和调试

```javascript
// 测试正则表达式
function testRegex(regex, tests) {
  console.log(`Testing: ${regex}`);
  tests.forEach(({ input, expected, description }) => {
    const result = regex.test(input);
    const passed = result === expected;
    console.log(`${passed ? '✓' : '✗'} ${description}: "${input}" → ${result} (expected: ${expected})`);
  });
}

testRegex(/^\d+$/, [
  { input: '123', expected: true, description: '纯数字' },
  { input: '123abc', expected: false, description: '包含字母' },
  { input: '', expected: false, description: '空字符串' }
]);
```

### 缓存编译的正则表达式

```javascript
// 缓存正则表达式以提高性能
const regexCache = new Map();

function getCompiledRegex(pattern, flags) {
  const key = `${pattern}|${flags}`;
  if (!regexCache.has(key)) {
    regexCache.set(key, new RegExp(pattern, flags));
  }
  return regexCache.get(key);
}

// 使用缓存的正则
const regex = getCompiledRegex('\\d+', 'g');
console.log('test 123 test 456'.match(regex));
```

## 注意事项

1. **性能考虑**：复杂的正则表达式可能性能很差，特别是在长文本上。

2. **回溯问题**：避免可能导致灾难性回溯的模式。

3. **可读性**：复杂的正则表达式很难阅读和维护，考虑添加注释。

4. **字符编码**：处理 Unicode 字符时使用 u 标志。

5. **边界情况**：考虑空字符串、特殊字符等边界情况。

6. **测试**：充分测试正则表达式，包括正向和负向测试用例。

7. **浏览器兼容**：某些高级特性在不同浏览器中支持程度不同。

8. **过度使用**：有时简单的字符串方法比正则表达式更高效。

## 最佳实践

1. **简单优先**：优先使用简单、可读的正则表达式。

2. **命名捕获组**：使用命名捕获组提高可读性。

3. **注释正则**：为复杂的正则表达式添加注释。

4. **测试工具**：使用在线工具测试和调试正则表达式。

5. **性能测试**：对性能敏感的代码进行基准测试。

6. **分步构建**：从简单的模式开始，逐步添加复杂性。

7. **文档记录**：为复杂的正则表达式编写文档说明。

8. **持续学习**：正则表达式有很多高级技巧，持续学习新特性。

## 总结

通过本文的学习，相信你已经对 JavaScript 正则表达式进阶技巧有了更深入的理解。掌握这些高级技巧可以帮助你：

1. **处理复杂的文本匹配需求**
2. **优化正则表达式性能**
3. **编写更可维护的正则表达式**
4. **在实际项目中高效应用正则表达式**

记住，好的正则表达式应该：
- **正确**：准确匹配目标文本
- **高效**：性能良好，避免不必要的回溯
- **可读**：代码清晰，易于理解和维护
- **可测试**：能够充分验证其正确性

持续练习和实践，你将能够熟练运用正则表达式解决各种复杂的文本处理问题。