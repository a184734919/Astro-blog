---
title: JavaScript 运算符全解析
published: 2022-01-13
description: '深入理解 JavaScript 中的算术、比较、逻辑、赋值与位运算符'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 运算符全解析

## 概述

运算符是 JavaScript 中最基础也是最核心的语法之一。

无论是变量计算、条件判断，还是复杂逻辑控制，都离不开各种运算符的配合使用。

本文将系统讲解：

- 算术运算符
- 比较运算符
- 逻辑运算符
- 赋值运算符
- 位运算符
- 三元运算符
- 运算符优先级

帮助你彻底理解 JavaScript 运算符体系。

---

# 算术运算符

算术运算符用于数学计算。

## 加法 +

```javascript
const a = 10;
const b = 20;

console.log(a + b); // 30
```

字符串拼接：

```javascript
console.log('Hello' + ' World');
```

---

## 减法 -

```javascript
console.log(20 - 10); // 10
```

---

## 乘法 *

```javascript
console.log(5 * 2); // 10
```

---

## 除法 /

```javascript
console.log(10 / 2); // 5
```

---

## 取余 %

```javascript
console.log(10 % 3); // 1
```

常用于：

- 判断奇偶
- 分页计算
- 循环轮询

---

## 自增 ++

```javascript
let num = 1;

num++;

console.log(num); // 2
```

### 前置自增

```javascript
let a = 1;

console.log(++a); // 2
```

### 后置自增

```javascript
let b = 1;

console.log(b++); // 1
console.log(b);   // 2
```

---

# 比较运算符

比较运算符返回布尔值。

## 大于 >

```javascript
console.log(10 > 5); // true
```

---

## 小于 <

```javascript
console.log(3 < 1); // false
```

---

## 大于等于 >=

```javascript
console.log(5 >= 5); // true
```

---

## 小于等于 <=

```javascript
console.log(2 <= 1); // false
```

---

## 相等 ==

```javascript
console.log(1 == '1'); // true
```

会发生隐式类型转换。

---

## 严格相等 ===

```javascript
console.log(1 === '1'); // false
```

推荐开发中统一使用 `===`。

---

## 不等于 !=

```javascript
console.log(1 != '2'); // true
```

---

## 严格不等于 !==

```javascript
console.log(1 !== '1'); // true
```

---

# 逻辑运算符

## 逻辑与 &&

```javascript
console.log(true && true); // true
console.log(true && false); // false
```

特点：

- 遇到 false 立即停止
- 返回最后一个值

```javascript
console.log(1 && 2); // 2
```

---

## 逻辑或 ||

```javascript
console.log(true || false); // true
```

特点：

- 遇到 true 立即停止

```javascript
console.log(0 || '默认值');
```

常用于默认值设置。

---

## 逻辑非 !

```javascript
console.log(!true); // false
```

双重取反：

```javascript
console.log(!!1); // true
```

常用于布尔转换。

---

# 赋值运算符

## 基础赋值

```javascript
let num = 10;
```

---

## 加法赋值 +=

```javascript
let a = 1;

a += 2;

console.log(a); // 3
```

---

## 减法赋值 -=

```javascript
let a = 10;

a -= 3;

console.log(a); // 7
```

---

## 乘法赋值 *=

```javascript
let a = 5;

a *= 2;

console.log(a); // 10
```

---

## 除法赋值 /=

```javascript
let a = 20;

a /= 4;

console.log(a); // 5
```

---

# 位运算符

位运算符直接操作二进制。

## 按位与 &

```javascript
console.log(5 & 1);
```

---

## 按位或 |

```javascript
console.log(5 | 1);
```

---

## 按位异或 ^

```javascript
console.log(5 ^ 1);
```

特点：

- 相同为 0
- 不同为 1

---

## 左移 <<

```javascript
console.log(1 << 2); // 4
```

相当于乘以 2 的幂。

---

## 右移 >>

```javascript
console.log(8 >> 1); // 4
```

相当于除以 2。

---

# 三元运算符

语法：

```javascript
condition ? value1 : value2
```

示例：

```javascript
const age = 18;

const result = age >= 18 ? '成年人' : '未成年';

console.log(result);
```

适用于简单条件判断。

---

# 运算符优先级

示例：

```javascript
console.log(1 + 2 * 3);
```

结果：

```javascript
7
```

因为乘法优先级高于加法。

---

## 使用括号提高可读性

```javascript
console.log((1 + 2) * 3);
```

开发中推荐：

- 不依赖记忆优先级
- 使用括号增强代码可读性

---

# 实际应用

## 1. 表单校验

```javascript
if (username && password) {
  console.log('验证通过');
}
```

---

## 2. 默认值处理

```javascript
const name = inputName || '游客';
```

---

## 3. 权限判断

```javascript
const role = 'admin';

role === 'admin'
  ? console.log('管理员')
  : console.log('普通用户');
```

---

# 常见面试题

## == 和 === 有什么区别？

### ==

会自动转换类型。

```javascript
1 == '1'; // true
```

### ===

严格比较：

- 值相等
- 类型也必须相等

```javascript
1 === '1'; // false
```

---

## || 和 && 的返回值是什么？

它们不一定返回布尔值。

```javascript
console.log(1 && 2); // 2
console.log(0 || 100); // 100
```

---

## 为什么 0.1 + 0.2 !== 0.3？

因为 JavaScript 使用 IEEE754 浮点数标准。

```javascript
console.log(0.1 + 0.2); // 0.30000000000000004
```

---

# 注意事项

## 避免滥用 ==

推荐统一使用：

```javascript
===
```

---

## 注意短路运算

```javascript
fn && fn();
```

虽然简洁，但可读性需要注意。

---

## 小心浮点数精度问题

金融场景建议：

- 使用整数计算
- 使用精度库

---

# 最佳实践

## 使用 === 替代 ==

提高代码稳定性。

---

## 使用括号增强可读性

```javascript
const result = (a + b) * c;
```

---

## 谨慎使用位运算

位运算虽然性能高，但可读性较差。

---

# 总结

JavaScript 运算符是开发中的核心基础能力。

本文系统讲解了：

- 算术运算符
- 比较运算符
- 逻辑运算符
- 赋值运算符
- 位运算符
- 三元运算符
- 运算符优先级

熟练掌握这些知识后，你将能够：

- 更准确地编写逻辑代码
- 避免隐式转换陷阱
- 提高代码可读性
- 提升调试效率

后续建议继续深入学习：

- JavaScript 类型转换机制
- 执行上下文
- 闭包
- 原型链
- ES6+ 新特性
