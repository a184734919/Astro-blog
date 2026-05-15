---
title: JavaScript 函数基础
published: 2022-01-26
description: '函数定义、参数、返回值详解的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript 函数基础是前端开发中的重要内容，是实现代码复用、逻辑封装的核心手段，也是编写模块化、可维护代码的基础。无论是简单的工具函数，还是复杂的业务逻辑处理，都离不开函数的灵活运用。本文将详细介绍 JavaScript 函数的定义方式、参数传递、返回值使用等核心知识点，搭配示例演示和学习笔记，帮助初学者快速掌握函数基础并灵活运用。
## 核心概念
理解核心概念是掌握任何技术的基础，JavaScript 函数的核心本质是「封装一段可重复执行的代码块」，通过函数名调用，实现特定的功能，主要核心概念如下：
- **函数定义**：声明一个函数，明确函数的名称、参数和执行逻辑，是函数使用的前提。
- **函数参数**：函数调用时传入的数据，用于接收外部输入，让函数实现灵活复用（分为形参和实参）。
- **函数返回值**：函数执行完成后，返回给调用者的结果，通过 return 语句实现，没有 return 则默认返回 undefined。
- **函数调用**：通过函数名+括号的方式，执行函数内部的代码，可传入实参，接收返回值。
函数的核心价值是「代码复用」和「逻辑分离」，将重复的代码封装成函数，可减少冗余，提升代码可读性和维护性。
## 基本用法
以下是 JavaScript 函数的基础用法，涵盖多种定义方式、参数传递和返回值使用，附带详细注释，便于理解和直接复用：
### 1. 函数的三种定义方式
#### （1）函数声明式（最常用）
```javascript
// 语法：function 函数名(形参1, 形参2, ...) { 执行逻辑 }
function sayHello(name) {
  console.log(`Hello, ${name}!`); // 函数内部执行逻辑
}
// 函数调用：函数名(实参1, 实参2, ...)
sayHello('JavaScript'); // 输出：Hello, JavaScript!
```
#### （2）函数表达式（匿名/具名）
```javascript
// 匿名函数表达式：将匿名函数赋值给变量
const add = function(a, b) {
  return a + b; // 返回两个数的和
};
console.log(add(2, 3)); // 调用函数，输出：5

// 具名函数表达式：函数有名称，可用于内部调用
const multiply = function multiplyNum(a, b) {
  return a * b;
};
console.log(multiply(4, 5)); // 输出：20
```
#### （3）箭头函数（ES6 新增，简洁高效）
```javascript
// 语法：(形参1, 形参2, ...) => { 执行逻辑 }
// 单个形参可省略括号，单个语句可省略大括号和return
const subtract = (a, b) => a - b;
console.log(subtract(10, 4)); // 输出：6

// 多个语句需加括号和return
const getFullName = (firstName, lastName) => {
  const fullName = `${firstName} ${lastName}`;
  return fullName;
};
console.log(getFullName('张', '三')); // 输出：张三
```
### 2. 函数参数（形参 vs 实参）
```javascript
// 形参：函数定义时声明的参数，用于接收实参
// 实参：函数调用时传入的参数，传递给形参
function getSum(num1, num2) { // num1、num2 是形参
  return num1 + num2;
}
const result = getSum(10, 20); // 10、20 是实参
console.log(result); // 输出：30

// 注意：实参和形参数量可不一致
console.log(getSum(5)); // 形参num2未接收实参，默认是undefined，输出：NaN
console.log(getSum(5, 10, 15)); // 实参多余，多余部分被忽略，输出：15

// 形参默认值：避免实参缺失导致的异常
function getProduct(num1 = 1, num2 = 1) {
  return num1 * num2;
}
console.log(getProduct(5)); // 输出：5（num2使用默认值1）
```
### 3. 函数返回值（return 语句）
```javascript
// return 用于返回函数执行结果，执行return后，函数立即终止
function checkAge(age) {
  if (age >= 18) {
    return '成年'; // 返回结果，函数终止，后续代码不执行
  }
  return '未成年'; // 若age<18，执行此return
}
console.log(checkAge(20)); // 输出：成年
console.log(checkAge(16)); // 输出：未成年

// 无return的函数，默认返回undefined
function noReturn() {
  console.log('无返回值函数');
}
console.log(noReturn()); // 输出：无返回值函数 + undefined
```
## 实际应用
在实际项目中，函数的应用场景非常广泛，核心是实现代码复用和逻辑封装，以下是几个常见的应用场景示例，帮助快速衔接理论与实践：
- **工具函数封装**：将常用的操作（如数据格式化、数值计算）封装成函数，方便多次调用（如时间格式化、数组去重）。
- **业务逻辑拆分**：将复杂的业务逻辑拆分成多个小函数，每个函数负责一个具体功能，提升代码可读性和可维护性（如表单验证拆分多个验证函数）。
- **事件处理函数**：前端页面的交互（如点击、输入），通常通过绑定函数来实现响应逻辑（如按钮点击弹窗、输入框内容校验）。
- **模块化开发**：将不同功能的函数分类封装，实现代码模块化，便于后续维护和扩展。
示例：表单验证（拆分工具函数）
```javascript
// 封装验证用户名的函数
function checkUsername(username) {
  // 验证用户名不为空且长度在3-10位
  if (!username) return '用户名不能为空';
  if (username.length < 3 || username.length > 10) return '用户名长度需在3-10位之间';
 return '验证通过';
}

// 封装验证密码的函数
function checkPassword(password) {
  if (!password) return '密码不能为空';
  if (password.length < 6) return '密码长度不能小于6位';
  return '验证通过';
}

// 总验证函数，调用上面两个工具函数
function checkForm(username, password) {
  const usernameMsg = checkUsername(username);
  const passwordMsg = checkPassword(password);
  if (usernameMsg !== '验证通过') return usernameMsg;
  if (passwordMsg !== '验证通过') return passwordMsg;
  return '表单验证通过，可提交';
}

// 调用函数验证
console.log(checkForm('js123', '123456')); // 输出：表单验证通过，可提交
console.log(checkForm('js', '12345')); // 输出：用户名长度需在3-10位之间
```
## 注意事项
1. 注意边界情况：函数参数需考虑缺失、类型错误等边界，可通过设置默认值、添加类型判断，避免函数执行异常（如传入非数值类型进行计算）；return 语句需注意位置，避免逻辑遗漏。
2. 考虑性能影响：避免在函数内部重复声明变量、避免嵌套过多函数，减少不必要的计算；箭头函数不绑定 this，避免在需要 this 指向的场景（如对象方法）中使用。
3. 遵循最佳实践：函数命名需规范，采用小驼峰命名法（如 checkUsername），明确函数功能；函数功能需单一，一个函数只做一件事，便于复用和维护；避免函数过于冗长，复杂逻辑可拆分成多个小函数。
4. 避免变量污染：函数内部声明的变量（用 let/const）是局部变量，仅在函数内部有效；避免使用全局变量，防止变量污染，影响其他函数执行。
## 总结
通过本文的学习，相信你已经对 JavaScript 函数基础有了更深入的理解。函数是 JavaScript 中的核心知识点，其核心价值在于代码复用和逻辑封装，掌握函数的定义、参数、返回值用法，是编写高效、可维护前端代码的基础。
学习的关键在于多练习、多应用，建议结合实际场景封装函数（如工具函数、业务函数），加深对知识点的记忆和理解。后续可进一步学习函数的进阶用法（如函数作用域、闭包、this 指向），逐步提升代码编写能力。
如果有疑问或补充，欢迎在评论区交流讨论，一起夯实 JavaScript 基础 ✨