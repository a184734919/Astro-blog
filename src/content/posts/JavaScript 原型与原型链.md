---
title: JavaScript 原型与原型链
published: 2022-02-21
description: '深入理解原型继承机制的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---
## 什么是原型
JavaScript 中的每个对象都有一个原型对象（prototype），对象从原型继承方法和属性。这是 JavaScript 实现继承的主要方式，也是区别于传统面向对象语言（如 Java）类继承的核心特性。
### 核心补充（学习笔记）
- 原型的本质：原型本身也是一个对象，每个对象（除 null 外）都会默认关联一个原型对象，这个关联关系通过内部属性（浏览器中可通过 __proto__ 访问）实现。
- 原型的作用：实现属性和方法的复用，避免重复定义，降低内存开销。例如，多个对象可以共享同一个原型上的方法，无需为每个对象单独定义。
- 关键区分：函数才有 prototype（原型对象），普通对象只有 __proto__（原型指向），二者本质都是对象，只是存在场景和作用不同。
## 原型链
当访问一个对象的属性或方法时，JavaScript 引擎会先在对象自身查找；若未找到，则会通过 __proto__ 指向的原型对象继续查找；若原型对象中仍未找到，会继续查找原型的原型，直到找到目标或抵达原型链的终点（null），这个层层查找的链条就是原型链。
以下示例清晰展示原型链的结构和查找机制，搭配详细注释：
```javascript
// 定义构造函数
function Person(name) {
  this.name = name; // 实例自身的属性
}
// 给构造函数的原型对象添加方法（所有实例共享该方法）
Person.prototype.sayHello = function() {
  console.log('Hello, ' + this.name);
};
// 通过 new 关键字创建实例
const person = new Person('张三');
// 调用实例方法（自身未定义，从原型查找）
person.sayHello(); // 'Hello, 张三'

// 原型链结构解析（核心重点）
// 1. 实例的 __proto__ 指向其构造函数的 prototype
console.log(person.__proto__ === Person.prototype); // true
// 2. 构造函数的 prototype 是一个对象，其 __proto__ 指向 Object.prototype（所有对象的顶层原型）
console.log(Person.prototype.__proto__ === Object.prototype); // true
// 3. Object.prototype 是原型链的顶层，其 __proto__ 指向 null（原型链终点）
console.log(Object.prototype.__proto__); // null
```
### 原型链核心特点
1.  原型链的终点是 null，当查找至 Object.prototype 仍未找到目标时，返回 undefined；
2.  所有对象最终都继承自 Object.prototype，因此所有对象都拥有 Object 原型上的方法（如 toString()、hasOwnProperty()）；
3.  实例与原型之间是“继承”关系，实例可以直接访问原型上的属性和方法，无需手动关联。
## 原型继承
原型继承是 JavaScript 实现继承的核心方式，通过让子类的原型指向父类的实例（或父类的原型），实现子类继承父类的属性和方法。以下是最常用、最规范的原型继承实现方式，搭配完整示例和解析：
```javascript
// 父类（Animal）：定义公共属性和方法
function Animal(name) {
  this.name = name; // 父类的实例属性
}
// 父类原型添加公共方法（所有子类实例共享）
Animal.prototype.eat = function() {
  console.log(this.name + ' is eating');
};

// 子类（Dog）：继承父类的属性和方法，并添加自身特有属性和方法
function Dog(name, breed) {
  // 调用父类构造函数，继承父类的实例属性（关键：绑定this为子类实例）
  Animal.call(this, name);
  this.breed = breed; // 子类自身的特有属性
}

// 核心：让子类的原型指向父类的原型，实现方法继承
// Object.create() 方法创建一个新对象，其 __proto__ 指向传入的对象（Animal.prototype）
Dog.prototype = Object.create(Animal.prototype);
// 修复子类原型的 constructor 指向（避免指向父类 Animal）
Dog.prototype.constructor = Dog;

// 子类原型添加自身特有方法
Dog.prototype.bark = function() {
  console.log(this.name + ' is barking');
};

// 测试：创建子类实例，验证继承效果
const dog = new Dog('旺财', '柯基');
dog.eat(); // '旺财 is eating'（继承父类方法）
dog.bark(); // '旺财 is barking'（子类自身方法）
console.log(dog.name); // '旺财'（继承父类属性）
console.log(dog.breed); // '柯基'（子类自身属性）
```
### 关键注意点
- 必须使用 Animal.call(this, name)：仅通过原型继承无法继承父类的实例属性，需调用父类构造函数并绑定子类 this，才能继承父类的实例属性；
- 必须修复 constructor 指向：Dog.prototype = Object.create(Animal.prototype) 会覆盖子类原型的 constructor，导致其指向 Animal，需手动修正为 Dog；
- 避免直接赋值原型：不要使用 Dog.prototype = Animal.prototype（浅拷贝），否则修改子类原型会同步修改父类原型，造成污染。
## ES6 class 语法
ES6 引入了 class 语法，本质是原型继承的语法糖（底层依然基于原型和原型链实现），简化了继承的写法，让代码更具可读性和规范性。以下是 class 语法实现继承的示例，对比原型继承，突出简洁性：
```javascript
// 父类：使用 class 定义
class Animal {
  // 构造函数：对应原型继承中的父类构造函数，定义实例属性
  constructor(name) {
    this.name = name;
  }
  // 类方法：对应父类原型上的方法，所有实例共享
  eat() {
    console.log(`${this.name} is eating`);
  }
}

// 子类：使用 extends 关键字继承父类
class Dog extends Animal {
  // 子类构造函数：必须调用 super()，对应原型继承中的 Animal.call(this, name)
  constructor(name, breed) {
    super(name); // 调用父类构造函数，继承父类实例属性
    this.breed = breed; // 子类特有属性
  }
  // 子类特有方法
  bark() {
    console.log(`${this.name} is barking`);
  }
}

// 测试实例
const dog = new Dog('旺财', '柯基');
dog.eat(); // '旺财 is eating'（继承父类方法）
dog.bark(); // '旺财 is barking'（子类方法）
```
### class 与原型继承的关联
- class 语法本质是原型继承的封装，class 定义的类对应原型继承中的构造函数；
- constructor 方法对应构造函数的函数体，用于定义实例属性；
- class 内定义的方法（如 eat、bark），会被挂载到类的 prototype 上，与原型继承中手动添加方法一致；
- extends 关键字对应原型继承中 Dog.prototype = Object.create(Animal.prototype)，super() 对应 Animal.call(this, name)。
## 实际应用
原型与原型链在实际开发中应用广泛，核心是利用原型实现属性和方法的复用、扩展对象功能，以下是几个高频应用场景示例：
### 场景1：扩展内置对象方法
通过修改内置对象（如 Array、String）的原型，添加自定义方法，实现功能扩展（注意：谨慎使用，避免污染内置原型）。
```javascript
// 给 Array 原型添加自定义方法，获取数组第一个元素
Array.prototype.first = function() {
  return this[0]; // this 指向调用该方法的数组实例
};
// 给 Array 原型添加自定义方法，获取数组最后一个元素
Array.prototype.last = function() {
  return this[this.length - 1];
};

// 测试使用
const arr = [1, 2, 3];
arr.first(); // 1（调用自定义原型方法）
arr.last(); // 3（调用自定义原型方法）
```
### 场景2：实现对象复用（模块化）
通过构造函数和原型，创建可复用的对象模板，减少重复代码，提升开发效率（如封装通用组件、工具类）。
```javascript
// 封装工具类（通过原型复用方法）
function Tool() {}
// 原型添加通用工具方法
Tool.prototype.formatTime = function(time) {
 // 简单的时间格式化逻辑（示例）
 return new Date(time).toLocaleString();
};
Tool.prototype.deepClone = function(obj) {
  return JSON.parse(JSON.stringify(obj));
};

// 实例化工具类，复用方法
const tool = new Tool();
console.log(tool.formatTime(Date.now())); // 格式化当前时间
console.log(tool.deepClone({ name: '测试' })); // 深拷贝对象
```
### 场景3：class 语法封装组件（前端框架常用）
在 React、Vue 等前端框架中，class 语法（基于原型）常用于封装组件，实现组件的属性和方法复用，简化代码结构。
## 总结
原型和原型链是 JavaScript 面向对象编程的基础，也是理解 JavaScript 继承机制的核心。其核心逻辑可总结为：
1.  每个对象（除 null 外）都有 __proto__，指向其原型对象，实现属性和方法的继承；
2.  函数有 prototype（原型对象），是其所有实例的原型，用于存放可共享的方法和属性；
3.  原型链是层层查找属性/方法的链条，终点为 null，实现了对象之间的继承关系；
4.  ES6 class 语法是原型继承的语法糖，简化了继承写法，底层依然依赖原型和原型链。
理解原型与原型链，不仅能帮助我们写出更高效、可复用的代码，还能深入理解 JavaScript 的底层机制，避开因原型链查找导致的 bug，为后续学习前端框架、封装工具类打下坚实基础。