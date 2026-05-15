---
title: JavaScript this 指向详解
published: 2022-02-14
description: '不同场景下 this 的绑定规则的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 概述
JavaScript this 指向详解是前端开发中的重要内容，也是 JS 基础中的核心难点之一。this 是函数执行时自动生成的一个隐式变量，其指向并非固定不变，而是**取决于函数的调用方式**，而非函数的定义位置。理解 this 指向的绑定规则，能有效避免因 this 指向混乱导致的逻辑错误，是编写可维护、高质量 JS 代码的基础。本文将详细介绍不同场景下 this 的绑定规则、核心概念、基本用法、实际应用及注意事项，搭配示例演示和学习笔记，帮助初学者快速掌握 this 指向的核心逻辑并灵活运用。
## 核心概念
理解核心概念是掌握任何技术的基础，JavaScript 中 this 的核心定义及关键特性如下，聚焦重点、不冗余，贴合学习笔记调性：
- **this 的本质**：函数执行时的「上下文对象」，指向函数调用时所处的环境，用于访问当前执行环境中的变量、函数和DOM元素。
- **核心原则**：this 指向「谁调用函数，就指向谁」，这是判断 this 指向的核心口诀，不同调用场景对应不同的绑定规则。
- **绑定规则分类**：根据函数调用方式，this 绑定主要分为 4 类——默认绑定、隐式绑定、显式绑定、new 绑定，覆盖所有开发场景。
- **特殊说明**：箭头函数没有自己的 this，其 this 指向定义时所在的外层作用域的 this，不遵循上述 4 类绑定规则，是 this 指向中的特殊场景。
## 基本用法
以下是 JavaScript 中 this 指向的基本用法，按绑定规则分类演示，搭配详细注释和对比示例，清晰呈现不同场景下 this 的指向，便于理解和直接复用：
### 1. 默认绑定（全局环境/普通函数调用）
当函数以「普通函数」形式调用（无明确调用者）时，this 指向全局对象（浏览器环境为 window，Node 环境为 global）；严格模式下，默认绑定的 this 为 undefined。
```javascript
// 示例1：非严格模式，普通函数调用
function fn() {
  console.log(this); // 指向 window（浏览器环境）
  console.log(this.name); // 访问 window 的 name 属性
}
window.name = '全局window';
fn(); // 输出：Window { ... } 、 全局window

// 示例2：严格模式，普通函数调用
function fnStrict() {
  'use strict'; // 开启严格模式
  console.log(this); // 输出：undefined
}
fnStrict(); // 严格模式下，默认绑定的 this 为 undefined
```
### 2. 隐式绑定（对象方法调用）
当函数作为「对象的方法」调用时，this 指向调用该方法的对象（即函数的直接调用者）。
```javascript
const obj = {
  name: 'obj对象',
 sayName: function() {
    console.log(this); // 指向调用者 obj
    console.log(this.name); // 输出：obj对象
  }
};
// 函数作为obj的方法调用，this指向obj
obj.sayName(); 

// 注意：若将方法赋值给变量，再调用，会变成普通函数调用（默认绑定）
const say = obj.sayName;
say(); // this 指向 window，输出：undefined（window无name属性时）
```
### 3. 显式绑定（call/apply/bind 调用）
通过 call、apply、bind 三个方法，可手动指定 this 的指向，不受调用方式的影响，是开发中常用的手动控制 this 指向的方式。
```javascript
function fn() {
 console.log(this.name); // 输出手动指定的this对象的name属性
}
const obj1 = { name: 'obj1' };
const obj2 = { name: 'obj2' };

// 1. call 方法：参数逐个传入
fn.call(obj1); // 输出：obj1（this指向obj1）
// 2. apply 方法：参数以数组形式传入
fn.apply(obj2); // 输出：obj2（this指向obj2）
// 3. bind 方法：返回一个新函数，this 永久绑定到指定对象，需手动调用
const fnBind = fn.bind(obj1);
fnBind(); // 输出：obj1（this始终指向obj1，不受调用方式影响）
```
### 4. new 绑定（构造函数调用）
当函数作为「构造函数」，通过 new 关键字调用时，this 指向新创建的实例对象。
```javascript
// 构造函数
function Person(name) {
  this.name = name; // this 指向新创建的实例
  this.sayName = function() {
    console.log(this.name); // this 依然指向当前实例
  }
}
// new 关键字创建实例，this指向实例p1
const p1 = new Person('张三');
p1.sayName(); // 输出：张三
// this 指向新实例p2
const p2 = new Person('李四');
p2.sayName(); // 输出：李四
```
### 5. 特殊场景：箭头函数的 this 指向
箭头函数没有自己的 this，其 this 指向「定义时所在的外层作用域的 this」，而非调用时的上下文，且无法通过 call、apply、bind 手动修改。
```javascript
const obj = {
  name: 'obj',
  fn: () => {
    console.log(this); // 指向外层作用域的this（window，浏览器环境）
    console.log(this.name); // 输出：undefined
  }
};
obj.fn(); // 箭头函数定义时外层作用域是全局，this指向window

// 嵌套场景：箭头函数的this继承外层函数的this
const obj2 = {
  name: 'obj2',
  outer: function() {
    const inner = () => {
      console.log(this); // 继承外层函数outer的this，指向obj2
      console.log(this.name); // 输出：obj2
    };
    inner();
  }
};
obj2.outer();
```
## 实际应用
在实际项目中，this 指向的应用场景无处不在，核心是根据调用方式判断 this 指向，灵活运用绑定规则解决实际问题，以下是几个高频应用场景示例：
- **对象方法调用**：在对象中定义方法，通过 this 访问对象自身的属性和方法（如组件中的方法访问组件实例的属性）。
- **事件处理函数**：DOM 事件绑定中，this 指向触发事件的 DOM 元素（默认情况），可通过 bind 手动修改 this 指向。
- **构造函数/类组件**：通过 new 绑定，让 this 指向实例，实现实例的属性和方法复用（如 React 类组件、原生构造函数）。
- **回调函数优化**：使用箭头函数解决回调函数中 this 指向混乱的问题（如定时器、数组方法回调）。
示例1：DOM 事件处理中的 this 指向
```javascript
// 获取按钮元素
const btn = document.getElementById('btn');
// 事件处理函数，默认this指向触发事件的btn元素
btn.addEventListener('click', function() {
  console.log(this); // 指向 btn 元素
  this.style.color = 'red'; // 操作当前按钮的样式
});

// 若需让this指向其他对象（如全局），使用bind
btn.addEventListener('click', function() {
  console.log(this); // 指向 window
}.bind(window));
```
示例2：定时器回调中 this 指向优化
```javascript
const obj = {
  name: '张三',
  showName: function() {
    // 传统回调：this指向window，无法访问obj的name
    setTimeout(function() {
      console.log(this.name); // 输出：undefined
    }, 1000);

    // 箭头函数回调：this继承外层作用域的this（obj）
 setTimeout(() => {
      console.log(this.name); // 输出：张三
    }, 2000);
  }
};
obj.showName();
```
## 注意事项
1. 注意边界情况：普通函数赋值给变量后调用，会触发默认绑定（this指向全局/undefined），而非隐式绑定；箭头函数无法通过 call、apply、bind 修改 this 指向，修改后依然指向外层作用域的 this。
2. 考虑性能影响：bind 方法会返回一个新函数，频繁使用会增加内存开销；箭头函数虽然简化了 this 指向，但在需要动态 this 指向的场景（如对象方法）中使用，会导致逻辑异常，增加维护成本。
3. 遵循最佳实践：判断 this 指向时，优先根据「谁调用函数」的原则，结合绑定规则分析；对象方法优先使用传统函数（保证 this 指向对象），回调函数优先使用箭头函数（避免 this 混乱）；构造函数中避免使用箭头函数作为方法（this 会指向外层全局）。
4. 避免常见误区：不要认为 this 指向函数本身，也不要认为 this 指向函数的定义环境，核心是「调用方式决定 this 指向」；严格模式下，默认绑定的 this 为 undefined，避免依赖 window 对象导致的异常。
## 总结
通过本文的学习，相信你已经对 JavaScript this 指向详解有了更深入的理解。this 指向的核心是「调用方式决定指向」，掌握默认绑定、隐式绑定、显式绑定、new 绑定这 4 类核心规则，以及箭头函数的特殊 this 指向，就能解决绝大多数开发中的 this 相关问题。
学习的关键在于多结合实际场景分析，多练习不同调用方式下的 this 指向，牢记核心口诀和边界情况，避免常见误区。后续可结合闭包、类组件等知识点，进一步深化对 this 指向的理解，夯实 JavaScript 进阶基础。