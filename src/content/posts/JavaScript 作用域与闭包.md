---
title: JavaScript 作用域与闭包
published: 2022-02-01
description: '作用域、作用域链及闭包的核心原理、失败使用方式与边界情况详细解析'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript 作用域与闭包是 JS 基础中的核心难点，也是前端面试高频考点，直接影响代码的执行顺序、变量访问权限和内存管理。作用域定义了变量的可访问范围，闭包则是基于作用域链形成的特殊现象，既能实现变量私有化，也容易因使用不当导致内存泄漏等问题。本文将聚焦作用域与闭包的核心概念，重点拆解**失败使用方式**和**边界情况**，搭配示例演示和学习笔记，帮助初学者避开坑点、精准掌握知识点。
## 核心概念
理解核心概念是避免使用失败的前提，作用域与闭包的核心定义及关联如下，不冗余、不延伸，聚焦关键要点：
- **作用域**：规定变量和函数的可访问范围，分为全局作用域、函数作用域（ES5）和块级作用域（ES6 新增，let/const 声明），核心是「隔离变量，避免污染」。
- **作用域链**：当访问一个变量时，JS 引擎会从当前作用域开始，逐级向上查找，直到找到变量或抵达全局作用域，形成的查找链条即为作用域链。
- **闭包**：函数嵌套时，内层函数引用了外层函数的局部变量，且内层函数被外层函数以外的地方引用，此时内层函数及其引用的外层变量构成闭包。核心是「保留外层函数的局部变量，使其不被垃圾回收」。
## 基本用法（极简铺垫，为失败方式和边界情况做支撑）
核心展示正确基础用法，对比后续失败案例，突出差异：
```javascript
// 1. 作用域基础用法（块级作用域 vs 函数作用域）
function fn() {
  var funcVar = '函数作用域变量'; // 函数作用域，外部不可访问
  let blockVar = '块级作用域变量'; // 块级作用域，仅当前函数内可访问
  if (true) {
    let innerBlock = '内层块级变量'; // 仅if块内可访问
    console.log(innerBlock); // 正常访问：内层块级变量
 }
  console.log(innerBlock); // 报错：innerBlock is not defined（边界情况铺垫）
}
fn();
console.log(funcVar); // 报错：funcVar is not defined

// 2. 闭包正确用法（实现变量私有化）
function createCounter() {
  let count = 0; // 外层函数局部变量，被内层函数引用
  // 内层函数引用外层变量，且被返回（外部可访问）
  return function() {
    count++; // 闭包保留count，不被垃圾回收
    return count;
  };
}
const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2（count被保留）
```
## 核心重点：失败使用方式（高频坑点）
结合实际开发场景，拆解作用域与闭包最易出错的使用方式，每个失败案例附带错误原因和正确修正，直观易懂：
### 1. 作用域相关失败使用方式
#### （1）混淆 var/let/const 作用域，导致变量污染或访问异常
```javascript
// 失败案例1：var 无块级作用域，导致循环中变量污染
for (var i = 0; i < 3; i++) {
 setTimeout(() => {
    console.log(i); // 输出：3, 3, 3（而非0,1,2）
  }, 100);
}
// 错误原因：var 声明的i是函数作用域（全局作用域），循环中i不断覆盖，定时器回调执行时i已变成3
// 正确修正：使用let声明，形成块级作用域，每次循环创建独立的i
for (let i = 0; i < 3; i++) {
  setTimeout(() => {
    console.log(i); // 输出：0, 1, 2
  }, 100);
}

// 失败案例2：误认为块级作用域可跨函数访问
if (true) {
  let blockVar = '块级变量';
}
console.log(blockVar); // 报错：blockVar is not defined（失败）
// 错误原因：let声明的变量具有块级作用域，仅在所在的if块内可访问，外部无法访问
// 正确修正：将变量声明在块级作用域外部，或通过函数返回暴露
let blockVar;
if (true) {
  blockVar = '块级变量';
}
console.log(blockVar); // 正常访问：块级变量
```
#### （2）忽略作用域链查找顺序，导致变量访问错误
```javascript
// 失败案例：内层变量与外层变量同名，误认为会访问外层变量
let name = '全局name';
function fn() {
  let name = '函数内name';
  function innerFn() {
    console.log(name); // 输出：函数内name（而非全局name）
  }
  innerFn();
}
fn(); // 失败点：误以为作用域链会优先查找外层（全局），实际优先查找当前作用域
// 错误原因：作用域链查找顺序是「当前作用域 → 外层作用域 → 全局作用域」，同名变量会覆盖上层
// 正确修正：若需访问外层同名变量，可通过全局对象（window）或改变变量名避免冲突
let name = '全局name';
function fn() {
  let name = '函数内name';
  function innerFn() {
    console.log(window.name); // 输出：全局name（浏览器环境）
  }
  innerFn();
}
fn();
```
### 2. 闭包相关失败使用方式
#### （1）滥用闭包，导致内存泄漏
```javascript
// 失败案例：闭包引用大体积对象，且长期不释放，导致内存泄漏
function createBigObj() {
  // 大体积对象（模拟场景，如大型DOM集合、海量数据）
  const bigObj = new Array(1000000).fill('large data');
  return function() {
    console.log(bigObj.length); // 闭包引用bigObj，使其无法被垃圾回收
  };
}
const getBigObjLen = createBigObj(); // 闭包被全局变量引用，长期存在
// 失败点：bigObj体积大，且被闭包长期引用，无法回收，导致内存占用过高、页面卡顿
// 正确修正：使用完毕后，手动解除引用，让垃圾回收机制回收
let getBigObjLen = createBigObj();
getBigObjLen(); // 使用完毕
getBigObjLen = null; // 解除引用，bigObj可被垃圾回收
```
#### （2）误认为闭包能改变外层变量的作用域
```javascript
// 失败案例：试图通过闭包访问外层函数中未声明的变量，或修改外层变量作用域
function outer() {
  let outerVar = '外层变量';
  return function inner() {
    outerVar = '修改后外层变量'; // 可修改外层变量（正确）
    console.log(innerVar); // 报错：innerVar is not defined（失败）
  };
  let innerVar = '内层未暴露变量'; // 声明在return之后，闭包无法访问
}
const innerFn = outer();
innerFn();
// 错误原因：1. 闭包只能访问外层函数「声明在闭包之前」的变量；2. 闭包无法改变变量的作用域，只能引用已存在的外层变量
// 正确修正：将需要被闭包访问的变量，声明在闭包之前
function outer() {
  let innerVar = '内层未暴露变量'; // 声明在闭包之前
  let outerVar = '外层变量';
  return function inner() {
    outerVar = '修改后外层变量';
    console.log(innerVar); // 正常访问：内层未暴露变量
 };
}
```
#### （3）闭包中使用this，导致指向异常
```javascript
// 失败案例：闭包中this指向混乱，误认为指向外层函数
const obj = {
  name: 'obj',
  outer: function() {
    return function inner() {
      console.log(this.name); // 输出：undefined（失败）
    };
  }
};
const innerFn = obj.outer();
innerFn(); // 失败点：误认为闭包this指向外层函数outer的this（obj），实际指向全局window
// 错误原因：闭包（inner函数）是独立函数，调用时无明确调用者，this默认指向全局window（非严格模式）
// 正确修正：提前保存外层this，或使用箭头函数（箭头函数无自身this，继承外层this）
const obj = {
  name: 'obj',
  outer: function() {
    const _this = this; // 保存外层this（obj）
    return function inner() {
      console.log(_this.name); // 输出：obj
    };
  }
};
```
## 核心重点：边界情况（易忽略场景）
聚焦作用域与闭包使用中易忽略的边界场景，明确每种场景的表现和注意事项，避免踩坑：
### 1. 作用域边界情况
#### （1）块级作用域与函数作用域的嵌套边界
```javascript
// 边界场景1：函数作用域内嵌套块级作用域，块级变量无法穿透到函数外部
function fn() {
  if (true) {
    let blockVar = '块级变量';
    var funcVar = '函数变量';
  }
  console.log(funcVar); // 正常访问：函数变量（var无块级作用域）
  console.log(blockVar); // 报错：blockVar is not defined（块级作用域边界）
}
fn();
// 注意：let/const的块级作用域，仅在当前代码块（{}）内有效，即使嵌套在函数内，也无法突破块级边界

// 边界场景2：全局作用域中，var与let/const的边界差异
var globalVar = 'var全局变量';
let globalLet = 'let全局变量';
console.log(window.globalVar); // 输出：var全局变量（var声明的全局变量挂载到window）
console.log(window.globalLet); // 输出：undefined（let声明的全局变量不挂载到window）
// 注意：这是全局作用域的关键边界，避免通过window访问let/const声明的全局变量
```
#### （2）作用域链的查找边界（终止条件）
```javascript
// 边界场景：访问不存在的变量，作用域链查找至全局作用域后终止，报错
function outer() {
  function inner() {
    console.log(undefinedVar); // 报错：undefinedVar is not defined
  }
  inner();
}
outer();
// 注意：作用域链的终止条件是「找到变量」或「抵达全局作用域」，若全局作用域也无该变量，直接报错，不会继续查找
```
### 2. 闭包边界情况
#### （1）闭包仅保留外层变量的引用，而非副本
```javascript
// 边界场景：闭包引用的是外层变量的引用，而非值的副本，外层变量修改后，闭包访问的是修改后的值
function outer() {
  let num = 10;
  const inner = () => {
    console.log(num); // 引用num，而非副本
  };
  num = 20; // 修改外层变量
  return inner;
}
const innerFn = outer();
innerFn(); // 输出：20（而非10）
// 注意：这是闭包的核心边界，容易误认为闭包保存的是变量的初始值，实际保存的是引用
```
#### （2）闭包无法访问外层函数的arguments对象
```javascript
// 边界场景：闭包无法直接访问外层函数的arguments对象，需提前保存
function outer() {
  const args = arguments; // 保存外层arguments
  return function inner() {
    console.log(arguments); // 访问的是inner自身的arguments（空）
    console.log(args); // 访问外层arguments（需提前保存）
 };
}
const innerFn = outer(1, 2, 3);
innerFn(); // 输出：Arguments(0) [] 和 Arguments(3) [1, 2, 3]
// 注意：闭包有自身的arguments对象（非箭头函数），无法直接访问外层函数的arguments，需提前保存到变量中
```
#### （3）箭头函数作为闭包时，无自身this和arguments
```javascript
// 边界场景：箭头函数作为闭包，继承外层this和作用域，无自身this和arguments
function outer() {
  return () => {
    console.log(this); // 继承外层this（outer的this）
    console.log(arguments); // 报错：arguments is not defined（箭头函数无arguments）
  };
}
const innerFn = outer(1, 2, 3);
innerFn(); // 报错：arguments is not defined
// 注意：箭头函数作为闭包时，需用剩余参数（...rest）替代arguments，避免报错
```
## 总结
作用域与闭包的核心难点，不在于概念本身，而在于**失败使用方式的规避**和**边界情况的把握**。本文重点拆解了开发中高频的失败场景（变量污染、内存泄漏、this指向异常等）和易忽略的边界情况，核心要点如下：
1.  区分var/let/const的作用域差异，避免因作用域混淆导致的访问异常；
2.  理解作用域链的查找顺序和终止条件，不盲目访问跨作用域变量；
3.  合理使用闭包，避免滥用导致内存泄漏，使用完毕后及时解除引用；
4.  牢记闭包的边界特性：保留变量引用而非副本，箭头函数闭包无自身this和arguments。
掌握这些要点，既能避开绝大多数坑点，也能更灵活地运用作用域与闭包实现变量私有化、模块化开发等需求。后续可结合实际项目场景多练习，加深对知识点的理解，夯实JS基础。