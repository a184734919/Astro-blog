---
title: JavaScript 流程控制语句
published: 2022-01-20
description: 'if、switch、for、while 循环详解的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript 流程控制语句是前端开发中的重要内容，是实现代码逻辑跳转、条件执行、循环迭代的核心，也是编写可维护、高效代码的基础。无论是简单的表单验证，还是复杂的业务逻辑处理，都离不开流程控制语句的灵活运用。本文将详细介绍 if、switch 条件语句，以及 for、while 循环语句的核心用法、示例演示和学习笔记，帮助初学者快速掌握并灵活运用。
## 核心概念
理解核心概念是掌握任何技术的基础，JavaScript 流程控制语句的核心本质是「改变代码的执行顺序」，主要分为两大类别：
- **条件判断语句**：根据指定条件，决定代码是否执行、执行哪一段代码，核心是「选择」，代表语句有 if、switch。
- **循环语句**：根据指定条件，重复执行某一段代码，核心是「重复」，代表语句有 for、while、do...while。
流程控制的核心价值的是让代码摆脱「自上而下的顺序执行」的局限，根据业务需求实现灵活的逻辑控制，提升代码的复用性和可读性。
## 基本用法
以下是各类流程控制语句的基础示例代码，附带详细注释，便于理解和直接复用：
### 1. if 条件语句（单条件、多条件）
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
```
### 2. switch 条件语句（多分支匹配）
```javascript
// 匹配不同的状态值，执行对应逻辑
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
```
### 3. for 循环（固定次数循环）
```javascript
// 循环打印 1-10 的数字
for (let i = 1; i <= 10; i++) {
  console.log(i); // 每次循环执行，i自增1，直到i>10时终止
}

// 遍历数组
let arr = ['js', 'html', 'css'];
for (let i = 0; i < arr.length; i++) {
  console.log('数组元素：', arr[i]);
}
```
### 4. while 循环（未知次数循环，先判断后执行）
```javascript
// 循环打印 1-5 的数字
let num = 1;
while (num <= 5) {
  console.log(num);
  num++; // 必须有自增/自减，否则会陷入死循环
}

// 结合条件终止循环
let count = 0;
while (count < 3) {
  console.log('循环次数：', count + 1);
  count++;
}
```
### 5. do...while 循环（未知次数循环，先执行后判断）
```javascript
// 无论条件是否成立，至少执行一次
let x = 0;
do {
  console.log('执行次数：', x + 1);
  x++;
} while (x < 3);
```
## 实际应用
在实际项目中，我们需要根据具体场景灵活运用流程控制语句，以下是几个常见的应用场景示例，帮助快速衔接理论与实践：
- **表单验证**：使用 if 语句判断用户输入是否合法（如手机号、密码长度），提示对应错误信息。
- **数据遍历**：使用 for/while 循环遍历接口返回的数组，渲染页面列表（如商品列表、用户列表）。
- **状态切换**：使用 switch 语句根据后端返回的状态码，显示不同的页面提示（如订单状态：待支付、已支付、已取消）。
- **循环执行任务**：使用 while 循环实现倒计时功能，直到倒计时结束后执行后续操作（如发送验证码倒计时）。
示例：表单验证（简单版）
```javascript
function checkForm() {
  let username = document.getElementById('username').value;
  let password = document.getElementById('password').value;

  // 验证用户名不为空
  if (!username) {
    alert('请输入用户名');
    return false;
  }
  // 验证密码长度不小于6位
  if (password.length < 6) {
    alert('密码长度不能小于6位');
    return false;
  }
  // 验证通过，提交表单
  alert('表单验证通过，准备提交');
  return true;
}
```
## 注意事项
1. **注意边界情况**：判断条件时，需考虑边界值（如 => 与 > 的区别），避免出现逻辑漏洞（如判断成绩合格时，漏判 60 分的情况）；循环中注意初始值和终止条件，防止漏循环或多循环。
2. **考虑性能影响**：避免不必要的循环嵌套（如双重 for 循环嵌套，数据量大时会严重影响性能）；循环中尽量减少 DOM 操作，可先将数据处理完成后再批量渲染。
3. **遵循最佳实践**：if 语句多分支（超过3个）时，优先使用 switch 语句，提升代码可读性；循环中必须有终止条件，避免陷入死循环；switch 语句中不要遗漏 break，防止 case 穿透；使用 let/const 声明循环变量，避免变量污染。
4. **代码可读性**：复杂条件判断可拆分为多个变量，提升代码可读性；循环和条件语句的缩进要规范，便于后续维护。
## 总结
通过本文的学习，相信你已经对 JavaScript 流程控制语句有了更深入的理解。流程控制是 JavaScript 基础中的核心，if、switch 负责「选择」，for、while 负责「重复」，二者结合可以实现绝大多数前端业务逻辑。
学习的关键在于多练习、多应用，建议结合实际场景编写代码（如实现一个简单的倒计时、表单验证），加深对知识点的记忆和理解。后续可进一步学习流程控制的进阶用法（如 break/continue 的使用、循环嵌套技巧），逐步提升代码编写能力。
如果有疑问或补充，欢迎在评论区交流讨论，一起夯实 JavaScript 基础 ✨