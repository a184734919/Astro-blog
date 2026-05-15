---
title: JavaScript 数组方法详解
published: 2022-02-27
description: '常用数组方法的用法和场景的详细介绍和学习笔记'
image: ''
tags: ["JS基础","数组"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript 数组方法详解是前端开发中的重要内容，数组作为 JS 中最常用的数据结构之一，其内置方法贯穿日常开发的方方面面。掌握常用数组方法的用法、特性和适用场景，能大幅提升开发效率，减少冗余代码，同时避免因方法使用不当导致的逻辑错误。本文将分类介绍常用数组方法，结合详细示例、使用场景和学习笔记，帮助大家系统掌握数组方法，实现灵活运用。
## 核心概念
理解核心概念是掌握任何技术的基础，JavaScript 数组方法的核心特性和分类如下，聚焦重点、贴合学习笔记调性，便于快速梳理思路：
- 数组方法的本质：JS 数组内置的函数，用于简化数组的操作（如遍历、修改、筛选、排序等），无需手动编写循环逻辑。
- 核心分类（按功能）：遍历方法、修改方法、筛选方法、排序方法、拼接方法、查找方法，不同类别对应不同开发场景。
- 关键区分：部分方法会「改变原数组」（如 push、pop），部分方法「不改变原数组」（如 map、filter），这是使用数组方法的核心注意点，避免误操作原数据。
- 回调函数共性：多数数组方法（如 forEach、map）支持传入回调函数，回调函数会遍历数组的每一项，接收三个参数（当前项、索引、原数组），按需使用。
## 基本用法
以下按分类整理常用数组方法，每个方法搭配简洁示例、核心功能说明和注释，覆盖基础用法，便于直接理解和复用，贴合学习笔记的实用性：
### 一、遍历方法（最常用场景）
用于遍历数组每一项，执行指定逻辑，核心方法：forEach、map、some、every
```javascript
// 1. forEach：遍历数组，无返回值，仅执行回调逻辑（最基础遍历）
const arr = [1, 2, 3, 4];
arr.forEach((item, index, arr) => {
  console.log(`索引${index}：${item}`); // 依次输出：索引0：1、索引1：2、索引2：3、索引3：4

// 2. map：遍历数组，返回一个新数组，新数组元素是回调函数的返回值（映射转换）
const newArr = arr.map(item => item * 2); // 每一项乘以2
console.log(newArr); // 输出：[2, 4, 6, 8]（不改变原数组）
console.log(arr); // 输出：[1, 2, 3, 4]（原数组不变）

// 3. some：遍历数组，判断是否有至少一项满足回调条件，返回布尔值
const hasEven = arr.some(item => item % 2 === 0); // 判断是否有偶数
console.log(hasEven); // 输出：true（2、4是偶数）

// 4. every：遍历数组，判断是否所有项都满足回调条件，返回布尔值
const allEven = arr.every(item => item % 2 === 0); // 判断是否全是偶数
console.log(allEven); // 输出：false（1、3是奇数）
```
### 二、修改方法（改变原数组）
用于修改数组本身（添加、删除、替换元素），核心方法：push、pop、shift、unshift、splice
```javascript
const arr = [1, 2, 3];

// 1. push：在数组末尾添加一个/多个元素，返回新数组长度（改变原数组）
const pushLen = arr.push(4, 5); // 添加4、5
console.log(arr); // 输出：[1, 2, 3, 4, 5]
console.log(pushLen); // 输出：5（新数组长度）

// 2. pop：删除数组末尾最后一个元素，返回被删除的元素（改变原数组）
const popItem = arr.pop(); // 删除末尾的5
console.log(arr); // 输出：[1, 2, 3, 4]
console.log(popItem); // 输出：5（被删除的元素）

// 3. shift：删除数组开头第一个元素，返回被删除的元素（改变原数组）
const shiftItem = arr.shift(); // 删除开头的1
console.log(arr); // 输出：[2, 3, 4]
console.log(shiftItem); // 输出：1（被删除的元素）

// 4. unshift：在数组开头添加一个/多个元素，返回新数组长度（改变原数组）
const unshiftLen = arr.unshift(0, 1); // 添加0、1
console.log(arr); // 输出：[0, 1, 2, 3, 4]
console.log(unshiftLen); // 输出：5（新数组长度）

// 5. splice：万能修改方法，可删除、添加、替换元素（改变原数组）
// 语法：splice(起始索引, 删除个数, 要添加的元素)
arr.splice(2, 1, 9); // 从索引2开始，删除1个元素，添加9
console.log(arr); // 输出：[0, 1, 9, 3, 4]
```
### 三、筛选方法（不改变原数组）
用于筛选数组中满足条件的元素，返回新数组，核心方法：filter、slice
```javascript
const arr = [1, 2, 3, 4, 5, 6];

// 1. filter：筛选满足回调条件的元素，返回新数组（不改变原数组）
const evenArr = arr.filter(item => item % 2 === 0); // 筛选偶数
console.log(evenArr); // 输出：[2, 4, 6]
console.log(arr); // 输出：[1, 2, 3, 4, 5, 6]（原数组不变）

// 2. slice：截取数组的一部分，返回新数组（不改变原数组）
// 语法：slice(起始索引, 结束索引)，结束索引不包含自身
const sliceArr1 = arr.slice(1, 4); // 截取索引1到3的元素
const sliceArr2 = arr.slice(2); // 截取从索引2到末尾的元素
console.log(sliceArr1); // 输出：[2, 3, 4]
console.log(sliceArr2); // 输出：[3, 4, 5, 6]
```
### 四、其他常用方法
涵盖排序、拼接、查找、求和等高频场景，核心方法：sort、concat、find、findIndex、reduce
```javascript
const arr1 = [3, 1, 4, 2];
const arr2 = [5, 6, 7];

// 1. sort：对数组进行排序，默认按字符串Unicode排序（改变原数组）
arr1.sort((a, b) => a - b); // 数字升序排序（关键：传入比较函数）
console.log(arr1); // 输出：[1, 2, 3, 4]

// 2. concat：拼接两个/多个数组，返回新数组（不改变原数组）
const concatArr = arr1.concat(arr2); // 拼接arr1和arr2
console.log(concatArr); // 输出：[1, 2, 3, 4, 5, 6, 7]

// 3. find：查找数组中第一个满足条件的元素，返回该元素（找不到返回undefined）
const findItem = concatArr.find(item => item > 4); // 查找第一个大于4的元素
console.log(findItem); // 输出：5

// 4. findIndex：查找数组中第一个满足条件的元素的索引，返回索引（找不到返回-1）
const findIdx = concatArr.findIndex(item => item > 4); // 查找第一个大于4的元素的索引
console.log(findIdx); // 输出：4

// 5. reduce：累加/累乘，返回最终计算结果（不改变原数组）
const sum = concatArr.reduce((prev, curr) => prev + curr, 0); // 数组求和，初始值为0
console.log(sum); // 输出：28（1+2+3+4+5+6+7）
```
## 实际应用
在实际项目中，我们需要根据具体场景灵活运用数组方法，以下是几个高频实际应用场景，结合综合示例，衔接基础用法与项目实践，便于直接参考复用：
### 场景1：数据筛选与格式化（最常用）
需求：从接口返回的用户列表中，筛选出年龄大于18岁的用户，并只保留姓名、年龄、手机号三个字段，格式化数据结构。
```javascript
// 模拟接口返回的用户列表
const userList = [
  { name: '张三', age: 17, phone: '13800138000', gender: '男' },
  { name: '李四', age: 20, phone: '13900139000', gender: '女' },
  { name: '王五', age: 22, phone: '13700137000', gender: '男' },
  { name: '赵六', age: 16, phone: '13600136000', gender: '女' }
];
// 筛选+格式化：filter筛选，map格式化字段
const adultUser = userList
  .filter(user => user.age > 18) // 筛选18岁以上用户
  .map(user => ({ // 只保留需要的字段
    name: user.name,
    age: user.age,
    phone: user.phone
  }));
console.log(adultUser); // 输出：[{name: '李四', age:20, phone: '13900139000'}, {name: '王五', age:22, phone: '13700137000'}]
```
### 场景2：数组去重与排序
需求：对一个包含重复元素的数组去重，然后按升序排序。
```javascript
const arr = [3, 1, 2, 3, 5, 2, 4, 1];
// 去重：利用filter+indexOf，保留第一个出现的元素
const uniqueArr = arr.filter((item, index) => arr.indexOf(item) === index);
// 排序：升序排序
const sortedArr = uniqueArr.sort((a, b) => a - b);
console.log(sortedArr); // 输出：[1, 2, 3, 4, 5]
```
### 场景3：数组累加与统计
需求：统计数组中所有偶数的和，以及偶数的个数。
```javascript
const arr = [1, 2, 3, 4, 5, 6, 7, 8];
// 利用reduce实现累加+统计，prev是累加器，存储上一次的计算结果
const result = arr.reduce((prev, curr) => {
  if (curr % 2 === 0) { // 判断当前项是否为偶数
    prev.sum += curr; // 累加偶数和
    prev.count++; // 统计偶数个数
  }
  return prev;
}, { sum: 0, count: 0 }); // 初始值：对象，包含sum（和）和count（个数）
console.log(result); // 输出：{ sum: 20, count: 4 }（2+4+6+8=20，共4个偶数）
```
### 场景4：数组拼接与截取（分页场景）
需求：将两个数组拼接，截取前5个元素作为第一页数据。
```javascript
const page1 = [1, 2, 3]; // 第一部分数据
const page2 = [4, 5, 6, 7]; // 第二部分数据
// 拼接数组，截取前5个元素
const firstPage = page1.concat(page2).slice(0, 5);
console.log(firstPage); // 输出：[1, 2, 3, 4, 5]
```
## 注意事项
1. 注意边界情况：使用 slice、splice 时，起始索引和结束索引需注意范围，避免出现负索引（负索引表示从末尾开始计数）；find、findIndex 方法找不到目标时，分别返回 undefined 和 -1，需做异常处理。
2. 考虑性能影响：forEach、map 等遍历方法在数组长度极大（万级以上）时，性能略低于 for 循环，需根据数组规模选择；splice 方法会改变原数组，频繁使用可能影响性能，可考虑用 slice 替代（不改变原数组）。
3. 遵循最佳实践：区分「改变原数组」和「不改变原数组」的方法，避免误操作原数据（如需修改原数组，建议先做备份）；排序时，数字排序必须传入比较函数（(a,b) => a - b），否则会按字符串排序导致异常；回调函数中尽量简化逻辑，提升可读性和性能。
4. 避免常见误区：不要混淆 filter 和 map 的作用（filter 用于筛选，map 用于映射转换）；不要误以为 concat 会改变原数组（实际返回新数组）；reduce 的初始值按需设置，避免数组为空时出现 NaN 等异常。
## 总结
通过本文的学习，相信你已经对 JavaScript 数组方法详解有了更深入的理解。数组方法是前端开发的基础工具，核心在于根据场景选择合适的方法，牢记「是否改变原数组」这一关键区分，灵活搭配使用不同方法，能大幅提升开发效率。
本文整理的均为日常开发中最常用的数组方法，涵盖遍历、修改、筛选、排序等核心场景，结合示例和实际应用，便于大家快速上手和复用。后续可结合项目需求，多练习综合运用（如 filter+map+reduce 组合），加深对数组方法的理解，夯实 JavaScript 基础。