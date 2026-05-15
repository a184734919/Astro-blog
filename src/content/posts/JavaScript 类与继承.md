---
title: JavaScript 类与继承
published: 2022-05-27
description: 'class 语法和继承实现的详细介绍和学习笔记'
image: ''
tags: ["JS"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 的 class 语法是 ES6 引入的语法糖，它基于原型继承机制，提供了更清晰、更易用的面向对象编程方式。类（Class）是对象的模板，用于创建具有相同属性和方法的对象。

## 类的基本概念

### 什么是类

类是创建对象的模板，它定义了对象的属性和方法。JavaScript 的 class 实际上是构造函数和原型链的语法糖，底层仍然基于原型继承。

### 为什么使用类

- **代码组织**：更好地组织相关代码
- **继承机制**：方便实现代码复用
- **类型安全**：配合 TypeScript 提供更好的类型检查
- **可读性**：代码结构更清晰易懂

## 类的基本语法

### 定义类

```javascript
// 基本类定义
class Person {
  // 构造函数
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // 实例方法
  sayHello() {
    console.log(`Hello, my name is ${this.name}`);
  }

  // 实例方法
  getInfo() {
    return `${this.name} is ${this.age} years old`;
  }
}

// 创建实例
const person1 = new Person('张三', 25);
const person2 = new Person('李四', 30);

person1.sayHello(); // 'Hello, my name is 张三'
console.log(person1.getInfo()); // '张三 is 25 years old'
```

### 类声明和表达式

```javascript
// 类声明
class Animal {
  constructor(name) {
    this.name = name;
  }
}

// 类表达式（匿名）
const MyClass = class {
  constructor(value) {
    this.value = value;
  }
};

// 类表达式（具名）
const NamedClass = class MyClass {
  constructor() {
    this.className = 'MyClass';
  }
};
```

### 类的成员

```javascript
class Example {
  // 实例属性（在构造函数中定义）
  constructor() {
    this.instanceProperty = 'instance';
  }

  // 实例方法
  instanceMethod() {
    return 'instance method';
  }

  // 静态属性
  static staticProperty = 'static';

  // 静态方法
  static staticMethod() {
    return 'static method';
  }

  // 访问器属性
  get accessor() {
    return 'getter value';
  }

  set accessor(value) {
    console.log('setter called with:', value);
  }
}

console.log(Example.staticProperty); // 'static'
console.log(Example.staticMethod()); // 'static method'

const example = new Example();
console.log(example.accessor); // 'getter value'
example.accessor = 'new value'; // 'setter called with: new value'
```

## 构造函数

### 构造函数的作用

构造函数在创建类实例时自动调用，用于初始化对象的属性。

```javascript
class User {
  constructor(username, email) {
    this.username = username;
    this.email = email;
    this.createdAt = new Date();
  }

  getGreeting() {
    return `Hello, ${this.username}!`;
  }
}

const user = new User('zhangsan', 'zhangsan@example.com');
console.log(user.username); // 'zhangsan'
console.log(user.email); // 'zhangsan@example.com'
console.log(user.createdAt); // 创建时间
```

### 默认构造函数

如果不提供构造函数，类会自动提供一个默认的空构造函数。

```javascript
class DefaultConstructor {
  // 没有显式定义构造函数
}

const instance = new DefaultConstructor();
console.log(instance); // DefaultConstructor {}
```

### 构造函数注意事项

```javascript
// 1. 子类必须调用 super()
class Parent {
  constructor(name) {
    this.name = name;
  }
}

class Child extends Parent {
  constructor(name, age) {
    super(name); // 必须调用 super()
    this.age = age;
  }
}

// 2. 构造函数中不要返回值（除非返回对象）
class BadExample {
  constructor() {
    this.value = 1;
    return { other: 'object' }; // 返回对象会覆盖实例
  }
}

const bad = new BadExample();
console.log(bad.value); // undefined
console.log(bad.other); // 'object'
```

## 实例方法

### 定义实例方法

```javascript
class Calculator {
  constructor(initialValue = 0) {
    this.value = initialValue;
  }

  // 加法
  add(num) {
    this.value += num;
    return this;
  }

  // 减法
  subtract(num) {
    this.value -= num;
    return this;
  }

  // 乘法
  multiply(num) {
    this.value *= num;
    return this;
  }

  // 获取结果
  getResult() {
    return this.value;
  }
}

// 链式调用
const calc = new Calculator(10);
const result = calc
  .add(5)
  .subtract(3)
  .multiply(2)
  .getResult();

console.log(result); // 24
```

### 方法的特点

```javascript
class Example {
  method1() {
    console.log('method1');
  }

  method2() {
    // 方法内部不能枚举
    console.log('method2');
  }
}

// 方法不可枚举
const example = new Example();
for (let key in example) {
  console.log(key); // 不会输出任何方法名
}

// 方法在原型上
console.log(example.method1 === Example.prototype.method1); // true
```

## 静态方法和属性

### 静态方法

静态方法通过 `static` 关键字定义，只能通过类调用，不能通过实例调用。

```javascript
class MathUtils {
  // 静态方法
  static add(a, b) {
    return a + b;
  }

  static multiply(a, b) {
    return a * b;
  }

  static divide(a, b) {
    if (b === 0) {
      throw new Error('Division by zero');
    }
    return a / b;
  }

  // 静态方法可以调用其他静态方法
  static calculate(a, b, c) {
    return this.add(this.multiply(a, b), c);
  }
}

// 通过类调用静态方法
console.log(MathUtils.add(1, 2)); // 3
console.log(MathUtils.multiply(3, 4)); // 12
console.log(MathUtils.calculate(2, 3, 4)); // 10

// 不能通过实例调用
const utils = new MathUtils();
// utils.add(1, 2); // TypeError: utils.add is not a function
```

### 静态属性

```javascript
class Counter {
  // 静态属性
  static count = 0;

  constructor() {
    // 每次创建实例时增加计数
    Counter.count++;
  }

  static getCount() {
    return Counter.count;
  }

  static reset() {
    Counter.count = 0;
  }
}

const c1 = new Counter();
const c2 = new Counter();
const c3 = new Counter();

console.log(Counter.getCount()); // 3
Counter.reset();
console.log(Counter.getCount()); // 0
```

### 实际应用：工厂模式

```javascript
class UserFactory {
  // 静态工厂方法
  static createAdmin(name) {
    const user = new User(name);
    user.role = 'admin';
    user.permissions = ['read', 'write', 'delete'];
    return user;
  }

  static createRegularUser(name) {
    const user = new User(name);
    user.role = 'user';
    user.permissions = ['read'];
    return user;
  }

  static createGuest(name) {
    const user = new User(name);
    user.role = 'guest';
    user.permissions = [];
    return user;
  }
}

class User {
  constructor(name) {
    this.name = name;
    this.role = '';
    this.permissions = [];
  }
}

// 使用工厂方法
const admin = UserFactory.createAdmin('Admin');
const user = UserFactory.createRegularUser('User');
const guest = UserFactory.createGuest('Guest');
```

## Getter 和 Setter

### 基本语法

```javascript
class Circle {
  constructor(radius) {
    this._radius = radius; // 使用 _ 前缀表示私有属性（约定）
  }

  // Getter
  get radius() {
    return this._radius;
  }

  // Setter
  set radius(value) {
    if (value < 0) {
      throw new Error('半径不能为负数');
    }
    this._radius = value;
  }

  // 计算属性
  get diameter() {
    return this._radius * 2;
  }

  get area() {
    return Math.PI * this._radius ** 2;
  }

  get circumference() {
    return 2 * Math.PI * this._radius;
  }
}

const circle = new Circle(5);

console.log(circle.radius); // 5
console.log(circle.diameter); // 10
console.log(circle.area); // 78.53981633974483
console.log(circle.circumference); // 31.41592653589793

circle.radius = 10;
console.log(circle.radius); // 10

// circle.radius = -5; // Error: 半径不能为负数
```

### 数据验证

```javascript
class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }

  get celsius() {
    return this._celsius;
  }

  set celsius(value) {
    if (typeof value !== 'number') {
      throw new Error('温度必须是数字');
    }
    if (value < -273.15) {
      throw new Error('温度不能低于绝对零度');
    }
    this._celsius = value;
  }

  get fahrenheit() {
    return (this._celsius * 9/5) + 32;
  }

  set fahrenheit(value) {
    this.celsius = (value - 32) * 5/9;
  }

  get kelvin() {
    return this._celsius + 273.15;
  }
}

const temp = new Temperature(25);
console.log(temp.celsius); // 25
console.log(temp.fahrenheit); // 77
console.log(temp.kelvin); // 298.15

temp.fahrenheit = 100;
console.log(temp.celsius); // 37.77777777777778
```

### 只读属性

```javascript
class ReadOnly {
  constructor(value) {
    this._value = value;
  }

  get value() {
    return this._value;
  }

  // 不提供 setter，实现只读属性
}

const ro = new ReadOnly('immutable');
console.log(ro.value); // 'immutable'

// ro.value = 'new value'; // 静默失败（非严格模式）
```

## 类的继承

### extends 关键字

使用 `extends` 关键字创建子类，子类继承父类的所有属性和方法。

```javascript
// 父类
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }

  eat() {
    return `${this.name} is eating`;
  }
}

// 子类
class Dog extends Animal {
  constructor(name, breed) {
    super(name); // 调用父类构造函数
    this.breed = breed;
  }

  speak() {
    return `${this.name} barks`;
  }

  fetch() {
    return `${this.name} is fetching the ball`;
  }
}

const dog = new Dog('Buddy', 'Golden Retriever');
console.log(dog.speak()); // 'Buddy barks'
console.log(dog.eat()); // 'Buddy is eating' (继承的方法)
console.log(dog.fetch()); // 'Buddy is fetching the ball'
```

### super 关键字

`super` 关键字用于调用父类的构造函数和方法。

```javascript
class Vehicle {
  constructor(brand, model) {
    this.brand = brand;
    this.model = model;
    this.speed = 0;
  }

  accelerate(amount) {
    this.speed += amount;
    return `Accelerating by ${amount} km/h`;
  }

  brake(amount) {
    this.speed = Math.max(0, this.speed - amount);
    return `Braking by ${amount} km/h`;
  }

  getInfo() {
    return `${this.brand} ${this.model} - Speed: ${this.speed} km/h`;
  }
}

class Car extends Vehicle {
  constructor(brand, model, fuelType) {
    super(brand, model); // 调用父类构造函数
    this.fuelType = fuelType;
  }

  // 重写父类方法
  accelerate(amount) {
    // 调用父类方法并添加额外逻辑
    const result = super.accelerate(amount);
    return `${result} (car)`;
  }

  // 添加新方法
  refuel(liters) {
    return `Refueling ${liters} liters of ${this.fuelType}`;
  }

  // 使用父类方法
  getInfo() {
    const baseInfo = super.getInfo();
    return `${baseInfo} - Fuel: ${this.fuelType}`;
  }
}

const car = new Car('Toyota', 'Camry', 'petrol');
console.log(car.accelerate(50)); // 'Accelerating by 50 km/h (car)'
console.log(car.refuel(45)); // 'Refueling 45 liters of petrol'
console.log(car.getInfo()); // 'Toyota Camry - Speed: 50 km/h - Fuel: petrol'
```

### 方法重写

子类可以重写父类的方法，实现自己的逻辑。

```javascript
class Shape {
  constructor(color) {
    this.color = color;
  }

  getArea() {
    throw new Error('子类必须实现 getArea 方法');
  }

  getPerimeter() {
    throw new Error('子类必须实现 getPerimeter 方法');
  }

  toString() {
    return `Shape - Color: ${this.color}`;
  }
}

class Rectangle extends Shape {
  constructor(color, width, height) {
    super(color);
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }

  getPerimeter() {
    return 2 * (this.width + this.height);
  }

  toString() {
    return `Rectangle - Color: ${this.color}, Width: ${this.width}, Height: ${this.height}`;
  }
}

class Circle extends Shape {
  constructor(color, radius) {
    super(color);
    this.radius = radius;
  }

  getArea() {
    return Math.PI * this.radius ** 2;
  }

  getPerimeter() {
    return 2 * Math.PI * this.radius;
  }

  toString() {
    return `Circle - Color: ${this.color}, Radius: ${this.radius}`;
  }
}

const rectangle = new Rectangle('red', 10, 5);
const circle = new Circle('blue', 7);

console.log(rectangle.getArea()); // 50
console.log(circle.getArea()); // 153.93804002589985
console.log(rectangle.toString()); // 'Rectangle - Color: red, Width: 10, Height: 5'
```

## 私有字段和方法

### 私有字段（#）

使用 `#` 前缀定义私有字段，这是 ES2022 引入的标准语法。

```javascript
class BankAccount {
  // 私有字段
  #balance;
  #accountNumber;
  #pin;

  constructor(accountNumber, initialBalance, pin) {
    this.#accountNumber = accountNumber;
    this.#balance = initialBalance;
    this.#pin = pin;
  }

  // 公共方法访问私有字段
  deposit(amount) {
    if (amount <= 0) {
      throw new Error('存款金额必须大于0');
    }
    this.#balance += amount;
    return this.#getBalance();
  }

  withdraw(amount, pin) {
    if (pin !== this.#pin) {
      throw new Error('PIN码错误');
    }
    if (amount > this.#balance) {
      throw new Error('余额不足');
    }
    this.#balance -= amount;
    return this.#getBalance();
  }

  getBalance() {
    return this.#getBalance();
  }

  // 私有方法
  #getBalance() {
    return {
      accountNumber: this.#accountNumber,
      balance: this.#balance
    };
  }

  // 私有 getter
  get #formattedBalance() {
    return `¥${this.#balance.toFixed(2)}`;
  }
}

const account = new BankAccount('123456789', 1000, '1234');

console.log(account.deposit(500)); // { accountNumber: '123456789', balance: 1500 }
console.log(account.withdraw(200, '1234')); // { accountNumber: '123456789', balance: 1300 }

// account.#balance; // SyntaxError: Private field '#balance' must be declared in an enclosing class
```

### 私有方法

```javascript
class PasswordManager {
  #passwords = new Map();

  #hash(password) {
    // 简单哈希函数（实际项目中应该使用更安全的方法）
    return password.split('').reduce((acc, char) => {
      return acc + char.charCodeAt(0);
    }, 0);
  }

  #validatePassword(password) {
    return password.length >= 8;
  }

  setPassword(service, password) {
    if (!this.#validatePassword(password)) {
      throw new Error('密码长度必须至少8位');
    }
    const hashed = this.#hash(password);
    this.#passwords.set(service, hashed);
  }

  checkPassword(service, password) {
    const hashed = this.#hash(password);
    const stored = this.#passwords.get(service);
    return hashed === stored;
  }
}

const pm = new PasswordManager();
pm.setPassword('email', 'mysecretpassword');
console.log(pm.checkPassword('email', 'mysecretpassword')); // true
console.log(pm.checkPassword('email', 'wrongpassword')); // false
```

## 静态块

### 基本语法

静态块在类初始化时执行，用于初始化静态属性。

```javascript
class Database {
  static connection;
  static config;

  // 静态块
  static {
    console.log('静态块执行');

    // 初始化静态属性
    this.config = {
      host: 'localhost',
      port: 5432,
      database: 'mydb'
    };

    // 可以执行复杂逻辑
    this.connection = this.#createConnection();
  }

  static #createConnection() {
    console.log('创建数据库连接');
    return { connected: true };
  }

  static connect() {
    return this.connection;
  }
}

console.log(Database.connect()); // { connected: true }
```

### 多个静态块

```javascript
class Config {
  static settings = {};

  static {
    console.log('第一个静态块');
    this.settings.apiKey = 'abc123';
    this.settings.debug = true;
  }

  static {
    console.log('第二个静态块');
    this.settings.timeout = 5000;
    this.settings.retryAttempts = 3;
  }
}

console.log(Config.settings);
// { apiKey: 'abc123', debug: true, timeout: 5000, retryAttempts: 3 }
```

## 原型链和继承原理

### 理解原型链

```javascript
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  bark() {
    return 'Woof!';
  }
}

const dog = new Dog('Rex');

// 原型链关系
console.log(dog instanceof Dog); // true
console.log(dog instanceof Animal); // true
console.log(dog instanceof Object); // true

console.log(Dog.prototype === Object.getPrototypeOf(dog)); // true
console.log(Animal.prototype === Object.getPrototypeOf(Dog.prototype)); // true
console.log(Object.prototype === Object.getPrototypeOf(Animal.prototype)); // true
```

### 方法查找

```javascript
class Parent {
  method() {
    console.log('Parent method');
  }
}

class Child extends Parent {
  method() {
    console.log('Child method');
  }

  callParentMethod() {
    super.method(); // 显式调用父类方法
  }
}

const child = new Child();

child.method(); // 'Child method' (方法重写)
child.callParentMethod(); // 'Parent method' (通过 super 调用)
```

## 与构造函数的对比

### 构造函数方式

```javascript
// 传统构造函数方式
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.sayHello = function() {
  console.log(`Hello, my name is ${this.name}`);
};

Person.staticMethod = function() {
  console.log('Static method');
};

const person = new Person('张三', 25);
person.sayHello(); // 'Hello, my name is 张三'
Person.staticMethod(); // 'Static method'
```

### class 方式

```javascript
// 现代 class 方式
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  sayHello() {
    console.log(`Hello, my name is ${this.name}`);
  }

  static staticMethod() {
    console.log('Static method');
  }
}

const person = new Person('张三', 25);
person.sayHello(); // 'Hello, my name is 张三'
Person.staticMethod(); // 'Static method'
```

### 主要区别

```javascript
// 1. 提升（hoisting）
const p1 = new Person('Test'); // ReferenceError
class Person { }

function Person2() { }
const p2 = new Person2(); // 正常工作

// 2. 严格模式
class StrictClass {
  method() {
    // class 内部自动使用严格模式
    // this = null; // TypeError
  }
}

// 3. 枚举性
class ExampleClass {
  method() { }
}

function ExampleFunc() { }
ExampleFunc.prototype.method = function() { }

const instance1 = new ExampleClass();
const instance2 = new ExampleFunc();

for (let key in instance1) { }
// 不会输出 method

for (let key in instance2) {
  console.log(key); // 输出 method
}

console.log(Object.keys(instance1)); // []
console.log(Object.keys(instance2)); // ['method']
```

## 实际应用

### 1. 表单验证类

```javascript
class FormValidator {
  #rules = new Map();

  addRule(fieldName, rule) {
    if (!this.#rules.has(fieldName)) {
      this.#rules.set(fieldName, []);
    }
    this.#rules.get(fieldName).push(rule);
  }

  validate(data) {
    const errors = {};

    for (const [fieldName, rules] of this.#rules) {
      const value = data[fieldName];

      for (const rule of rules) {
        const result = rule(value);
        if (result !== true) {
          if (!errors[fieldName]) {
            errors[fieldName] = [];
          }
          errors[fieldName].push(result);
        }
      }
    }

    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }

  static required(message = '此字段为必填') {
    return (value) => {
      if (!value || (typeof value === 'string' && value.trim() === '')) {
        return message;
      }
      return true;
    };
  }

  static minLength(min, message) {
    return (value) => {
      if (value && value.length < min) {
        return message || `长度不能少于${min}个字符`;
      }
      return true;
    };
  }

  static email(message = '邮箱格式不正确') {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return (value) => {
      if (value && !emailRegex.test(value)) {
        return message;
      }
      return true;
    };
  }
}

// 使用
const validator = new FormValidator();

validator.addRule('username', FormValidator.required('用户名为必填'));
validator.addRule('username', FormValidator.minLength(3, '用户名至少3个字符'));
validator.addRule('email', FormValidator.required('邮箱为必填'));
validator.addRule('email', FormValidator.email());

const formData = {
  username: 'ab',
  email: 'invalid-email'
};

const result = validator.validate(formData);
console.log(result);
// {
//   isValid: false,
//   errors: {
//     username: ['用户名至少3个字符'],
//     email: ['邮箱格式不正确']
//   }
// }
```

### 2. 事件发射器类

```javascript
class EventEmitter {
  #events = new Map();

  on(event, listener) {
    if (!this.#events.has(event)) {
      this.#events.set(event, []);
    }
    this.#events.get(event).push(listener);
    return this;
  }

  off(event, listener) {
    if (!this.#events.has(event)) {
      return this;
    }

    const listeners = this.#events.get(event);
    const index = listeners.indexOf(listener);

    if (index > -1) {
      listeners.splice(index, 1);
    }

    return this;
  }

  emit(event, ...args) {
    if (!this.#events.has(event)) {
      return false;
    }

    const listeners = this.#events.get(event);

    for (const listener of listeners) {
      try {
        listener.apply(this, args);
      } catch (error) {
        console.error(`Error in event listener for '${event}':`, error);
      }
    }

    return true;
  }

  once(event, listener) {
    const onceWrapper = (...args) => {
      listener.apply(this, args);
      this.off(event, onceWrapper);
    };

    return this.on(event, onceWrapper);
  }

  removeAllListeners(event) {
    if (event) {
      this.#events.delete(event);
    } else {
      this.#events.clear();
    }
    return this;
  }
}

// 使用
const emitter = new EventEmitter();

emitter.on('data', (data) => {
  console.log('Data received:', data);
});

emitter.once('init', () => {
  console.log('Initialized once');
});

emitter.emit('init'); // 'Initialized once'
emitter.emit('init'); // 不会触发
emitter.emit('data', { id: 1, value: 'test' }); // 'Data received: { id: 1, value: 'test' }'
```

### 3. 状态管理类

```javascript
class Store {
  #state = {};
  #listeners = new Set();

  constructor(initialState = {}) {
    this.#state = JSON.parse(JSON.stringify(initialState));
  }

  getState() {
    return JSON.parse(JSON.stringify(this.#state));
  }

  setState(newState) {
    const prevState = this.getState();
    this.#state = { ...this.#state, ...newState };
    this.#notify(prevState, this.getState());
  }

  subscribe(listener) {
    this.#listeners.add(listener);

    // 返回取消订阅函数
    return () => {
      this.#listeners.delete(listener);
    };
  }

  #notify(prevState, currentState) {
    this.#listeners.forEach(listener => {
      try {
        listener(currentState, prevState);
      } catch (error) {
        console.error('Error in state listener:', error);
      }
    });
  }

  // Selector 方法
  select(selector) {
    return selector(this.getState());
  }
}

// 使用
const store = new Store({
  user: null,
  isLoading: false,
  error: null
});

const unsubscribe = store.subscribe((currentState, prevState) => {
  console.log('State changed from', prevState, 'to', currentState);
});

store.setState({ isLoading: true });

// 使用 selector
const isLoading = store.select(state => state.isLoading);
console.log(isLoading); // true

store.setState({ isLoading: false, user: { name: '张三' } });

unsubscribe(); // 取消订阅
```

## 注意事项

1. **类声明不会提升**：类声明不会像函数声明那样提升，必须先声明后使用。

2. **构造函数中必须调用 super()**：子类构造函数中必须先调用 `super()`，否则会报错。

3. **私有字段不能在类外访问**：私有字段只能在类内部访问，提供真正的封装。

4. **方法不可枚举**：类中定义的方法默认不可枚举。

5. **自动使用严格模式**：class 内部自动使用严格模式。

6. **this 绑定**：类方法中的 this 需要正确绑定，箭头函数会绑定外层 this。

7. **静态方法不能访问实例属性**：静态方法只能访问静态属性和方法。

8. **避免过度继承**：继承层次过深会影响代码可维护性，优先使用组合。

## 最佳实践

1. **单一职责原则**：每个类应该只有一个明确的职责。

2. **使用私有字段**：使用 `#` 私有字段保护内部状态，避免外部直接访问。

3. **命名规范**：
   - 类名使用 PascalCase（首字母大写）
   - 私有字段使用 `#` 前缀
   - 私有方法使用 `#` 前缀

4. **合理使用继承**：只在存在"is-a"关系时使用继承，"has-a"关系使用组合。

5. **提供清晰的接口**：通过 getter/setter 控制属性访问，添加验证逻辑。

6. **使用组合优于继承**：优先考虑组合模式，避免继承层次过深。

7. **文档和注释**：为类、方法、属性添加清晰的文档注释。

8. **使用静态方法**：将工具方法定义为静态方法，避免实例化。

## 总结

通过本文的学习，相信你已经对 JavaScript 类与继承有了更深入的理解。ES6 的 class 语法基于原型继承，提供了更清晰、更易用的面向对象编程方式。

主要要点：

- **class 是语法糖**：底层仍然是原型继承
- **继承使用 extends**：通过 extends 实现继承，使用 super 调用父类
- **支持私有字段**：使用 `#` 实现真正的私有属性
- **静态成员**：使用 static 定义静态方法和属性
- **访问器属性**：通过 getter/setter 控制属性访问

掌握类的使用和继承机制，能够编写出结构清晰、易于维护的面向对象代码。在实际项目中，合理使用类和继承，结合组合模式，可以构建出灵活、可扩展的代码架构。