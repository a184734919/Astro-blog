---
title: JavaScript 数组高阶方法
published: 2022-03-05
description: 'map、filter、reduce 等高阶函数的详细介绍和学习笔记'
image: ''
tags: ["JS基础","数组"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript 数组高阶方法是前端开发中的重要内容，也是提升代码简洁度、可读性和可维护性的核心工具。数组高阶方法本质是接收函数作为参数的数组内置函数，核心作用是简化数组遍历、筛选、转换、累加等操作，替代传统 for 循环，让代码更具逻辑性和优雅性。本文将重点讲解最常用的三个高阶方法——map、filter、reduce，结合详细示例、核心原理、学习笔记和实际应用场景，帮助大家系统掌握，实现灵活运用。
## 核心概念
理解核心概念是掌握任何技术的基础，JavaScript 数组高阶方法的核心特性和关键定义如下，聚焦重点、贴合学习笔记调性，便于快速梳理思路：
- 高阶方法的定义：接收一个或多个函数作为参数，并且/或者返回一个函数的数组方法，本文重点讲解「接收回调函数作为参数」的核心高阶方法（map、filter、reduce）。
- 回调函数共性：三个方法的回调函数均会遍历数组每一项，默认接收三个参数——当前项（item）、当前索引（index）、原数组（arr），可根据需求选择性使用。
- 核心特性：三个方法均「不改变原数组」，而是返回一个新值（新数组或计算结果），这是区别于普通数组方法（如 push、splice）的关键，也是开发中避免误操作原数据的核心优势。
- 核心区别：map 用于「映射转换」（一对一转换，返回新数组），filter 用于「筛选过滤」（按条件筛选，返回新数组），reduce 用于「累加计算」（汇总成一个结果，返回任意类型值）。
## 基本用法
以下分别讲解 map、filter、reduce 三个核心高阶方法的基本用法，每个方法搭配简洁示例、核心功能说明、回调参数解析和详细注释，覆盖基础场景，便于直接理解和复用，贴合学习笔记的实用性：
### 一、map 方法（映射转换）
核心功能：遍历数组每一项，通过回调函数对每一项进行转换，返回一个与原数组长度相同的新数组，新数组的元素是回调函数的返回值。
```javascript
// 基本语法：arr.map((item, index, arr) => { 转换逻辑; return 转换后的值; })
const arr = [1, 2, 3, 4, 5];

// 示例1：简单映射（每一项乘以2）
const doubleArr = arr.map(item => item * 2); // 省略index和arr（无需使用时可省略）
console.log(doubleArr); // 输出：[2, 4, 6, 8, 10]（新数组，长度与原数组一致）
console.log(arr); // 输出：[1, 2, 3, 4, 5]（原数组不变）

// 示例2：复杂映射（对象数组格式化）
const userList = [
  { name: '张三', age: 20 },
  { name: '李四', age: 22 },
  { name: '王五', age: 18 }
];
// 提取所有用户的姓名，组成新数组
const nameList = userList.map((item, index) => {
  console.log(`索引${index}的用户：${item.name}`); // 可使用index
  return item.name; // 返回转换后的值（姓名）
});
console.log(nameList); // 输出：['张三', '李四', '王五']
```
### 二、filter 方法（筛选过滤）
核心功能：遍历数组每一项，通过回调函数判断每一项是否满足指定条件，返回一个包含所有满足条件元素的新数组（新数组长度 ≤ 原数组长度）。
```javascript
// 基本语法：arr.filter((item, index, arr) => { 判断逻辑; return 布尔值; })
// 回调返回true：当前项保留到新数组；返回false：当前项不保留
const arr = [1, 2, 3, 4, 5, 6];

// 示例1：筛选偶数
const evenArr = arr.filter(item => item % 2 === 0); // 简洁写法，省略多余参数
console.log(evenArr); // 输出：[2, 4, 6]（仅保留满足条件的元素）

// 示例2：筛选对象数组（筛选年龄≥20的用户）
const userList = [
  { name: '张三', age: 20, gender: '男' },
  { name: '李四', age: 18, gender: '女' },
  { name: '王五', age: 22, gender: '男' },
  { name: '赵六', age: 19, gender: '女' }
];
const adultUser = userList.filter((item, index, arr) => {
  // 多条件筛选：年龄≥20 且 性别为男
  return item.age >= 20 && item.gender === '男';
});
console.log(adultUser); // 输出：[{name: '张三', age:20, gender: '男'}, {name: '王五', age:22, gender: '男'}]
```
### 三、reduce 方法（累加计算）
核心功能：遍历数组每一项，通过回调函数对数组元素进行累加、累乘、汇总等操作，最终返回一个单一的计算结果（可是数字、对象、数组等任意类型），功能最灵活。
```javascript
// 基本语法：arr.reduce((prev, curr, index, arr) => { 计算逻辑; return 计算后的值; }, 初始值)
// 关键参数：prev（上一次计算的结果）、curr（当前项）、初始值（可选，不写则默认以数组第一项为初始值）
const arr = [1, 2, 3, 4, 5];

// 示例1：数组求和（最基础用法）
// 初始值设为0，避免数组为空时返回NaN
const sum = arr.reduce((prev, curr) => {
  return prev + curr; // 上一次的和 + 当前项
}, 0);
console.log(sum); // 输出：15（1+2+3+4+5）

// 示例2：数组累乘
const product = arr.reduce((prev, curr) => prev * curr, 1); // 初始值设为1
console.log(product); // 输出：120（1×2×3×4×5）

// 示例3：复杂汇总（统计对象数组中各性别的人数）
const userList = [
  { name: '张三', gender: '男' },
 { name: '李四', gender: '女' },
  { name: '王五', gender: '男' },
  { name: '赵六', gender: '女' },
  { name: '孙七', gender: '男' }
];
// 初始值设为对象，用于存储统计结果
const genderCount = userList.reduce((prev, curr) => {
  // 判断当前性别是否已在prev中存在
  if (prev[curr.gender]) {
 prev[curr.gender]++; // 存在则计数+1
  } else {
    prev[curr.gender] = 1; // 不存在则初始化计数为1
  }
  return prev; // 返回当前的统计结果
}, {});
console.log(genderCount); // 输出：{ 男: 3, 女: 2 }
```
## 实际应用
在实际项目中，map、filter、reduce 三个高阶方法很少单独使用，更多是组合运用，解决复杂的数据处理场景。以下是几个高频实际应用场景，结合综合示例，衔接基础用法与项目实践，便于直接参考复用：
### 场景1：数据筛选+格式化（最常用）
需求：从接口返回的商品列表中，筛选出价格≥100元的商品，仅保留商品名称、价格、图片三个字段，并将价格保留2位小数。
```javascript
// 模拟接口返回的商品列表
const goodsList = [
  { name: '手机', price: 1999.99, img: 'phone.jpg', stock: 50 },
  { name: '耳机', price: 89.9, img: 'earphone.jpg', stock: 100 },
  { name: '平板', price: 2999.5, img: 'pad.jpg', stock: 30 },
  { name: '充电器', price: 49.9, img: 'charger.jpg', stock: 200 }
];
// 组合使用filter（筛选）+ map（格式化）
const formatGoods = goodsList
  .filter(goods => goods.price >= 100) // 筛选价格≥100的商品
  .map(goods => ({ // 格式化字段，价格保留2位小数
    name: goods.name,
    price: goods.price.toFixed(2), // 价格格式化
    img: goods.img
  }));
console.log(formatGoods);
// 输出：[{name: '手机', price: '1999.99', img: 'phone.jpg'}, {name: '平板', price: '2999.50', img: 'pad.jpg'}]
```
### 场景2：数据筛选+累加计算
需求：筛选出数组中大于10的元素，计算这些元素的总和与平均值。
```javascript
const arr = [5, 12, 8, 20, 15, 7, 25];
// 组合使用filter（筛选）+ reduce（累加+统计）
const result = arr
  .filter(item => item > 10) // 筛选大于10的元素，得到[12,20,15,25]
  .reduce((prev, curr) => {
    prev.sum += curr; // 累加总和
    prev.count++; // 统计筛选后元素的个数
    prev.average = (prev.sum / prev.count).toFixed(1); // 计算平均值（保留1位小数）
    return prev;
  }, { sum: 0, count: 0, average: 0 }); // 初始值包含总和、个数、平均值
console.log(result); // 输出：{ sum: 72, count: 4, average: '18.0' }
```
### 场景3：三层组合（筛选+映射+累加）
需求：从用户列表中，筛选出成年用户（年龄≥18），提取用户的消费金额，计算所有成年用户的总消费金额。
```javascript
// 模拟用户列表
const userList = [
  { name: '张三', age: 20, consume: 350 },
  { name: '李四', age: 17, consume: 200 },
  { name: '王五', age: 25, consume: 800 },
  { name: '赵六', age: 19, consume: 450 }
];
// 组合使用filter（筛选成年用户）+ map（提取消费金额）+ reduce（累加总消费）
const totalConsume = userList
  .filter(user => user.age >= 18) // 筛选成年用户
  .map(user => user.consume) // 提取消费金额，得到[350, 800, 450]
  .reduce((prev, curr) => prev + curr, 0); // 累加总消费
console.log(totalConsume); // 输出：1600（350+800+450）
```
## 注意事项
1. 注意边界情况：reduce 方法若不设置初始值，当数组为空时会报错，建议始终设置初始值；map 方法返回的新数组长度与原数组一致，即使回调返回 undefined，也会保留该位置的 undefined；filter 方法若没有满足条件的元素，会返回空数组，需做空值处理。
2. 考虑性能影响：三个高阶方法均会遍历数组，嵌套使用（如 filter+map+reduce）会遍历多次数组，若数组长度极大（万级以上），性能会略低于传统 for 循环，可根据数组规模优化；避免在回调函数中编写复杂逻辑，影响执行效率。
3. 遵循最佳实践：始终牢记三个方法「不改变原数组」的特性，无需担心误操作原数据；回调函数尽量简洁，可使用箭头函数简化写法（无需绑定 this 时）；map 用于转换、filter 用于筛选、reduce 用于汇总，避免混用（如用 map 筛选元素、用 filter 转换元素）。
4. 避免常见误区：不要误以为 map 会筛选元素（map 仅做转换，不筛选，即使返回 false 也会保留该元素）；不要忘记 reduce 的初始值（尤其数组可能为空时）；不要在 filter 回调中修改原数组元素（虽不改变原数组结构，但会污染原数据）。
## 总结
通过本文的学习，相信你已经对 JavaScript 数组高阶方法有了更深入的理解。map、filter、reduce 作为最核心的数组高阶方法，各自有明确的定位和用途：map 负责「转换」，filter 负责「筛选」，reduce 负责「汇总」，三者组合使用能解决绝大多数前端数据处理场景。
掌握这些高阶方法，不仅能替代繁琐的 for 循环，让代码更简洁优雅，还能提升代码的可读性和可维护性，是前端开发必备的基础技能。后续可结合项目需求，多练习三者的组合运用，加深对高阶方法的理解，同时注意避开常见误区，规范使用方法，夯实 JavaScript 数组操作基础。