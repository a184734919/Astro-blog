---
title: JavaScript ES6 特性概览
published: 2022-05-08
description: 'ES6 主要新特性总结的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

ES6（ECMAScript 2015）是 JavaScript 的一次重大更新，引入了许多新的语法特性和 API，极大地提升了开发效率和代码质量。本文将全面介绍 ES6 的主要新特性。

## let 和 const

### let 和 const 的基本用法

```javascript
// let - 块级作用域变量
let name = '张三';
name = '李四'; // 可以重新赋值

if (true) {
  let blockScoped = '块级作用域';
  console.log(blockScoped); // '块级作用域'
}

// console.log(blockScoped); // ReferenceError: blockScoped is not defined

// const - 块级作用域常量
const PI = 3.14159;
// PI = 3.14; // TypeError: Assignment to constant variable

// const 对象的属性可以修改
const user = { name: '张三' };
user.name = '李四'; // 可以修改对象属性
console.log(user); // { name: '李四' }

// const 数组的元素可以修改
const colors = ['red', 'green'];
colors.push('blue'); // 可以修改数组
console.log(colors); // ['red', 'green', 'blue']
```

### var vs let vs const

```javascript
// var - 函数作用域，存在变量提升
function testVar() {
  console.log(myVar); // undefined (变量提升)
  var myVar = 'value';
}

// let/const - 块级作用域，存在暂时性死区
function testLet() {
  // console.log(myLet); // ReferenceError: Cannot access 'myLet' before initialization
  let myLet = 'value';
}

// 推荐使用 let 和 const，避免使用 var
```

## 箭头函数

### 箭头函数的基本用法

```javascript
// 传统函数
function add(a, b) {
  return a + b;
}

// 箭头函数
const add2 = (a, b) => a + b;

// 简化语法
const square = x => x * x;
const sayHello = () => console.log('Hello!');

// 对象字面量返回
const createPerson = (name, age) => ({ name, age });
console.log(createPerson('张三', 25)); // { name: '张三', age: 25 }
```

### 箭头函数的特点

```javascript
// 没有 this 绑定，继承外层的 this
class Person {
  constructor(name) {
    this.name = name;
  }

  sayHello() {
    setTimeout(() => {
      console.log(`Hello, ${this.name}`); // this 指向 Person 实例
    }, 1000);
  }
}

const person = new Person('张三');
person.sayHello();

// 没有 arguments 对象
const showArgs = () => {
  // console.log(arguments); // ReferenceError: arguments is not defined
  console.log(...arguments); // 错误
};

// 使用剩余参数代替
const showArgs = (...args) => {
  console.log(args);
};
showArgs(1, 2, 3); // [1, 2, 3]

// 不能作为构造函数
const ArrowFunc = () => {};
// new ArrowFunc(); // TypeError: ArrowFunc is not a constructor
```

## 模板字符串

### 模板字符串的基本用法

```javascript
// 基本模板字符串
const name = '张三';
const greeting = `Hello, ${name}!`;
console.log(greeting); // 'Hello, 张三!'

// 多行字符串
const html = `
  <div class="container">
    <h1>Title</h1>
    <p>Content</p>
  </div>
`;

console.log(html);

// 表达式插值
const a = 10;
const b = 20;
const result = `${a} + ${b} = ${a + b}`;
console.log(result); // '10 + 20 = 30'

// 函数调用
const upperName = `Hello, ${name.toUpperCase()}!`;
console.log(upperName); // 'Hello, 张三!'

// 嵌套模板字符串
const format = (text) => `**${text}**`;
const formatted = `formatted: ${format('Hello')}`;
console.log(formatted); // 'formatted: **Hello**'
```

### 标签模板

```javascript
// 自定义标签函数
function highlight(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] !== undefined ? `<strong>${values[i]}</strong>` : '';
    return result + string + value;
  }, '');
}

const name = '张三';
const age = 25;
const highlighted = highlight`姓名: ${name}, 年龄: ${age}`;
console.log(highlighted); // '姓名: <strong>张三</strong>, 年龄: <strong>25</strong>'

// HTML 转义
function safeHtml(strings, ...values) {
  return strings.reduce((result, string, i) => {
    const value = values[i] !== undefined
      ? String(values[i]).replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
      : '';
    return result + string + value;
  }, '');
}

const userInput = '<script>alert("XSS")</script>';
const safe = safeHtml`用户输入: ${userInput}`;
console.log(safe); // '用户输入: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;'
```

## 解构赋值

### 数组解构

```javascript
// 基本解构
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// 跳过元素
const [first, , third] = [1, 2, 3];
console.log(first, third); // 1 3

// 剩余元素
const [head, ...rest] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(rest); // [2, 3, 4, 5]

// 默认值
const [x = 1, y = 2] = [undefined, 10];
console.log(x, y); // 1 10

// 交换变量
let m = 1;
let n = 2;
[m, n] = [n, m];
console.log(m, n); // 2 1
```

### 对象解构

```javascript
// 基本解构
const { name, age } = { name: '张三', age: 25 };
console.log(name, age); // '张三' 25

// 重命名
const { name: userName, age: userAge } = { name: '张三', age: 25 };
console.log(userName, userAge); // '张三' 25

// 默认值
const { name = '默认', age = 0 } = {};
console.log(name, age); // '默认' 0

// 嵌套解构
const user = {
  info: {
    name: '张三',
    age: 25
  }
};
const { info: { name, age } } = user;
console.log(name, age); // '张三' 25

// 剩余属性
const { a, b, ...rest } = { a: 1, b: 2, c: 3, d: 4 };
console.log(a, b); // 1 2
console.log(rest); // { c: 3, d: 4 }
```

### 函数参数解构

```javascript
// 对象参数解构
function createUser({ name, age = 18, email }) {
  console.log(name, age, email);
}

createUser({ name: '张三', email: 'test@example.com' });
// '张三' 18 'test@example.com'

// 数组参数解构
function sum([a, b, c]) {
  return a + b + c;
}

console.log(sum([1, 2, 3])); // 6
```

## 默认参数

### 函数默认参数

```javascript
// 基本默认参数
function greet(name = 'World') {
  console.log(`Hello, ${name}!`);
}

greet(); // 'Hello, World!'
greet('张三'); // 'Hello, 张三!'

// 多个默认参数
function createPerson(name, age = 18, gender = '未知') {
  return { name, age, gender };
}

console.log(createPerson('张三')); // { name: '张三', age: 18, gender: '未知' }

// 默认参数可以是表达式
function createId(prefix = 'user', suffix = Date.now()) {
  return `${prefix}-${suffix}`;
}

console.log(createId()); // 'user-1234567890'
console.log(createId('admin')); // 'admin-1234567890'

// 参数解构配合默认值
function fetchUser({ id, name = '匿名', age = 0 } = {}) {
  return { id, name, age };
}

console.log(fetchUser({ id: 1, name: '张三' })); // { id: 1, name: '张三', age: 0 }
```

## 展开运算符和剩余参数

### 展开运算符

```javascript
// 数组展开
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// 数组复制
const original = [1, 2, 3];
const copy = [...original];
console.log(copy); // [1, 2, 3]

// 对象展开
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };
const merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 函数调用展开
function add(a, b, c) {
  return a + b + c;
}

const numbers = [1, 2, 3];
console.log(add(...numbers)); // 6

// 数组去重
const withDuplicates = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(withDuplicates)];
console.log(unique); // [1, 2, 3]
```

### 剩余参数

```javascript
// 剩余参数收集
function sum(...numbers) {
  return numbers.reduce((total, num) => total + num, 0);
}

console.log(sum(1, 2, 3)); // 6
console.log(sum(1, 2, 3, 4, 5)); // 15

// 混合使用
function greet(greeting, ...names) {
  names.forEach(name => {
    console.log(`${greeting}, ${name}!`);
  });
}

greet('Hello', '张三', '李四', '王五');
// 'Hello, 张三!'
// 'Hello, 李四!'
// 'Hello, 王五!'
```

## 类和继承

### 类的基本语法

```javascript
// 定义类
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // 实例方法
  sayHello() {
    console.log(`Hello, I'm ${this.name}`);
  }

  // 静态方法
  static createAnonymous() {
    return new Person('Anonymous', 0);
  }

  // getter
  get info() {
    return `${this.name} is ${this.age} years old`;
  }

  // setter
  set age(value) {
    if (value < 0) {
      throw new Error('Age cannot be negative');
    }
    this._age = value;
  }
}

const person = new Person('张三', 25);
person.sayHello(); // 'Hello, I'm 张三'
console.log(person.info); // '张三 is 25 years old'
const anonymous = Person.createAnonymous();
```

### 继承

```javascript
// 继承
class Student extends Person {
  constructor(name, age, grade) {
    super(name, age); // 调用父类构造函数
    this.grade = grade;
  }

  study() {
    console.log(`${this.name} is studying in grade ${this.grade}`);
  }

  // 重写父类方法
  sayHello() {
    super.sayHello(); // 调用父类方法
    console.log(`I'm a student in grade ${this.grade}`);
  }
}

const student = new Student('李四', 18, 12);
student.sayHello(); // 'Hello, I'm 李四' + 'I'm a student in grade 12'
student.study(); // '李四 is studying in grade 12'
```

## 模块化

### 导出和导入

```javascript
// math.js - 导出模块
export const PI = 3.14159;

export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export default class Calculator {
  add(a, b) {
    return a + b;
  }
}

// main.js - 导入模块
import Calculator from './math.js';
import { add, subtract, PI } from './math.js';
import * as Math from './math.js';

const calculator = new Calculator();
console.log(calculator.add(1, 2)); // 3
console.log(add(3, 4)); // 7
console.log(Math.PI); // 3.14159
```

## Promise

### Promise 基本用法

```javascript
// 创建 Promise
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    const success = true;
    if (success) {
      resolve('操作成功');
    } else {
      reject('操作失败');
    }
  }, 1000);
});

// 使用 Promise
promise
  .then(result => console.log(result))
  .catch(error => console.error(error))
  .finally(() => console.log('完成'));
```

## Symbol

### Symbol 基本用法

```javascript
// 创建 Symbol
const sym1 = Symbol('description');
const sym2 = Symbol('description');

console.log(sym1 === sym2); // false (每个 Symbol 都是唯一的)

// 作为对象属性
const mySymbol = Symbol('id');
const obj = {
  [mySymbol]: 'value',
  name: '张三'
};

console.log(obj[mySymbol]); // 'value'
console.log(obj.name); // '张三'

// Symbol 属性不会被普通方法遍历
console.log(Object.keys(obj)); // ['name']
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]

// 内置 Symbol
class MyClass {
  constructor(value) {
    this.value = value;
  }

  [Symbol.toStringTag] = 'MyClass';
}

const instance = new MyClass(42);
console.log(Object.prototype.toString.call(instance)); // '[object MyClass]'
```

## Proxy 和 Reflect

### Proxy 基本用法

```javascript
// 创建 Proxy
const handler = {
  get(target, prop) {
    console.log(`Getting ${prop}`);
    return target[prop];
  },

  set(target, prop, value) {
    console.log(`Setting ${prop} to ${value}`);
    target[prop] = value;
  }
};

const target = { name: '张三' };
const proxy = new Proxy(target, handler);

console.log(proxy.name); // 'Getting name' -> '张三'
proxy.name = '李四'; // 'Setting name to 李四'

// 数据验证
const validator = {
  set(obj, prop, value) {
    if (prop === 'age' && (typeof value !== 'number' || value < 0)) {
      throw new TypeError('Age must be a positive number');
    }
    obj[prop] = value;
    return true;
  }
};

const person = new Proxy({}, validator);
person.age = 25; // OK
// person.age = -5; // TypeError
```

## Map 和 Set

### Map 基本用法

```javascript
// 创建 Map
const map = new Map();

// 设置值
map.set('name', '张三');
map.set(1, '数字键');
map.set(true, '布尔键');

// 获取值
console.log(map.get('name')); // '张三'

// 检查键是否存在
console.log(map.has('name')); // true

// 删除键
map.delete(1);

// 获取大小
console.log(map.size); // 2

// 遍历
map.forEach((value, key) => {
  console.log(`${key}: ${value}`);
});

for (const [key, value] of map) {
  console.log(`${key}: ${value}`);
}
```

### Set 基本用法

```javascript
// 创建 Set
const set = new Set();

// 添加值
set.add(1);
set.add(2);
set.add(2); // 重复值会被忽略

// 检查值是否存在
console.log(set.has(1)); // true

// 删除值
set.delete(1);

// 获取大小
console.log(set.size); // 1

// 数组去重
const arr = [1, 2, 2, 3, 3, 3];
const unique = [...new Set(arr)];
console.log(unique); // [1, 2, 3]
```

## 数组新方法

### 数组新增方法

```javascript
// Array.from - 将类数组转换为数组
const arrayLike = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
const arr = Array.from(arrayLike);
console.log(arr); // ['a', 'b', 'c']

// Array.of - 创建数组
console.log(Array.of(1, 2, 3)); // [1, 2, 3]

// find - 查找符合条件的元素
const numbers = [1, 2, 3, 4, 5];
const found = numbers.find(num => num > 3);
console.log(found); // 4

// findIndex - 查找符合条件的索引
const index = numbers.findIndex(num => num > 3);
console.log(index); // 3

// includes - 检查数组是否包含某个值
console.log(numbers.includes(3)); // true

// entries - 返回键值对迭代器
for (const [index, value] of numbers.entries()) {
  console.log(index, value);
}

// keys - 返回键迭代器
for (const index of numbers.keys()) {
  console.log(index);
}

// values - 返回值迭代器
for (const value of numbers.values()) {
  console.log(value);
}
```

## 对象新方法

### 对象新增方法

```javascript
// Object.assign - 对象合并
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const merged = Object.assign({}, obj1, obj2);
console.log(merged); // { a: 1, b: 2 }

// Object.keys - 获取所有键
const obj = { name: '张三', age: 25 };
console.log(Object.keys(obj)); // ['name', 'age']

// Object.values - 获取所有值
console.log(Object.values(obj)); // ['张三', 25]

// Object.entries - 获取键值对
console.log(Object.entries(obj)); // [['name', '张三'], ['age', 25]]

// Object.fromEntries - 键值对转对象
const entries = [['name', '张三'], ['age', 25]];
const newObj = Object.fromEntries(entries);
console.log(newObj); // { name: '张三', age: 25 }

// 对象属性简写
const name = '张三';
const age = 25;
const person = { name, age };
console.log(person); // { name: '张三', age: 25 }

// 方法简写
const user = {
  name: '张三',
  sayHello() {
    console.log(`Hello, ${this.name}`);
  }
};
user.sayHello(); // 'Hello, 张三'

// 计算属性名
const propName = 'dynamic';
const dynamicObj = {
  [propName]: 'value'
};
console.log(dynamicObj); // { dynamic: 'value' }
```

## 注意事项

1. **选择合适的变量声明**：优先使用 const，需要重新赋值时使用 let，避免使用 var。

2. **理解箭头函数的限制**：箭头函数没有自己的 this，不适合作为对象方法或构造函数。

3. **模板字符串的安全使用**：在处理用户输入时，要注意 XSS 攻击风险。

4. **解构赋值的默认值**：为解构属性设置合理的默认值，避免 undefined 错误。

5. **模块化的重要性**：合理使用模块化，提高代码的可维护性和复用性。

6. **Promise 错误处理**：始终处理 Promise 的错误，避免未捕获的异常。

7. **Symbol 的唯一性**：理解 Symbol 的唯一性，适用于私有属性的场景。

8. **浏览器兼容性**：考虑目标浏览器的兼容性，必要时使用 Babel 转译。

## 总结

ES6 为 JavaScript 带来了许多重要的新特性，极大地提升了开发体验和代码质量。从 let/const 到箭头函数，从解构赋值到模板字符串，从类到模块化，每个特性都有其适用场景。掌握这些特性，能够编写出更简洁、更安全、更易维护的代码。在实际开发中，要根据具体需求选择合适的特性，遵循最佳实践，充分发挥 ES6 的优势。