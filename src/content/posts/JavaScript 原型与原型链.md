---
title: JavaScript 原型与原型链
published: 2022-02-21
description: '深入理解原型继承机制、原型链查找与面向对象编程的最佳实践'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 原型与原型链

## 概述

JavaScript 的原型继承机制是其最独特的特性之一，与传统的类继承不同，它提供了更灵活的对象创建和继承方式。理解原型和原型链是掌握 JavaScript 面向对象编程的基础，也是深入理解 JavaScript 语言本质的关键。

## 原型基础

### 原型的概念

```javascript
// 每个函数都有一个 prototype 属性
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

// 每个实例都有一个 __proto__ 属性指向构造函数的 prototype
const person = new Person('张三');

console.log(person.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true

// 访问原型方法的多种方式
person.greet();                    // 通过实例调用
person.__proto__.greet();          // 通过 __proto__ 调用（不推荐）
Person.prototype.greet.call(person); // 通过 call 调用
```

### 原型的层次结构

```javascript
// 原型的原型链
console.log(Object.prototype);                    // 最顶层原型
console.log(Person.prototype.__proto__);          // Person.prototype 的原型
console.log(Object.prototype.__proto__);          // null - 原型链终点

// 验证原型链
function verifyPrototypeChain(obj) {
  let proto = Object.getPrototypeOf(obj);
  const chain = [obj.constructor.name];

  while (proto) {
    const name = proto.constructor ? proto.constructor.name : 'null';
    chain.push(name);
    proto = Object.getPrototypeOf(proto);
  }

  return chain.join(' -> ');
}

console.log(verifyPrototypeChain(person));
// "Person -> Object -> null"
```

### 原型的属性查找

```javascript
// 属性查找顺序：实例 -> 原型 -> 原型的原型 -> ...
function Animal(species) {
  this.species = species;
}

Animal.prototype.getInfo = function() {
  return `I am a ${this.species}`;
};

const dog = new Animal('dog');
console.log(dog.species);  // 'dog' - 从实例中找到
console.log(dog.getInfo()); // 'I am a dog' - 从原型中找到

// 修改原型属性
Animal.prototype.getInfo = function() {
  return `This is a ${this.species}`;
};
console.log(dog.getInfo()); // 'This is a dog' - 原型方法已被修改

// 实例属性覆盖原型属性
function Vehicle(type) {
  this.type = type;
}

Vehicle.prototype.wheels = 4;  // 原型属性

const car = new Vehicle('car');
console.log(car.wheels);       // 4 - 从原型获取

car.wheels = 6;                // 实例属性覆盖原型
console.log(car.wheels);       // 6 - 从实例获取

delete car.wheels;
console.log(car.wheels);       // 4 - 删除实例属性后，回到原型
```

## 原型继承

### 构造函数继承

```javascript
// 简单的构造函数继承
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'green', 'blue'];
}

Parent.prototype.sayName = function() {
  console.log(`My name is ${this.name}`);
};

function Child(name, age) {
  Parent.call(this, name); // 继承实例属性
  this.age = age;
}

// 继承原型方法
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;

// 添加子类特有方法
Child.prototype.sayAge = function() {
  console.log(`I am ${this.age} years old`);
};

const child = new Child('小明', 10);
child.sayName();  // "My name is 小明"
child.sayAge();   // "I am 10 years old"
console.log(child instanceof Child);    // true
console.log(child instanceof Parent);   // true
```

### 组合继承

```javascript
// 组合继承 = 构造函数继承 + 原型继承
function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'green', 'blue'];
}

SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  // 继承实例属性
  SuperType.call(this, name);
  this.age = age;
}

// 继承原型方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;

// 添加子类方法
SubType.prototype.sayAge = function() {
  console.log(this.age);
};

const instance1 = new SubType('张三', 20);
instance1.colors.push('black');
console.log(instance1.colors); // ['red', 'green', 'blue', 'black']

const instance2 = new SubType('李四', 25);
console.log(instance2.colors); // ['red', 'green', 'blue'] - 互不影响
```

### 原型式继承

```javascript
// Object.create() 实现原型式继承
const person = {
  name: '默认名称',
  friends: ['张三', '李四'],

  sayName() {
    console.log(this.name);
  }
};

// 创建新对象，以 person 为原型
const anotherPerson = Object.create(person);
anotherPerson.name = '王五';
anotherPerson.friends.push('赵六');

const yetAnotherPerson = Object.create(person);
yetAnotherPerson.name = '赵七';

console.log(person.friends); // ['张三', '李四', '赵六']
console.log(anotherPerson.friends); // ['张三', '李四', '赵六']
console.log(yetAnotherPerson.friends); // ['张三', '李四', '赵六']

// 问题：引用类型被共享
```

### 寄生式继承

```javascript
// 寄生式继承 = 原型式继承 + 增强对象
function createAnother(original) {
  const clone = Object.create(original);

  // 增强对象
  clone.sayHi = function() {
    console.log('Hi');
  };

  return clone;
}

const person = {
  name: '张三'
};

const anotherPerson = createAnother(person);
anotherPerson.sayHi();  // 'Hi'
console.log(anotherPerson.name); // '张三'
```

### 寄生组合继承

```javascript
// 寄生组合继承 = 最优继承方式
function inheritPrototype(subType, superType) {
  const prototype = Object.create(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}

function SuperType(name) {
  this.name = name;
  this.colors = ['red', 'green', 'blue'];
}

SuperType.prototype.sayName = function() {
  console.log(this.name);
};

function SubType(name, age) {
  SuperType.call(this, name);
  this.age = age;
}

// 使用寄生组合继承
inheritPrototype(SubType, SuperType);

SubType.prototype.sayAge = function() {
  console.log(this.age);
};

const instance = new SubType('小明', 18);
instance.sayName();  // '小明'
instance.sayAge();   // 18
```

## ES6 类语法

### 类的基本使用

```javascript
// ES6 类语法
class Animal {
  constructor(name) {
    this.name = name;
  }

  speak() {
    console.log(`${this.name} makes a noise.`);
  }

  static getSpecies() {
    return 'Unknown';
  }
}

class Dog extends Animal {
  constructor(name, breed) {
    super(name);
    this.breed = breed;
  }

  speak() {
    console.log(`${this.name} barks.`);
  }

  getBreed() {
    return this.breed;
  }

  static getSpecies() {
    return 'Canine';
  }
}

const dog = new Dog('旺财', '金毛');
dog.speak();  // '旺财 barks.'
console.log(dog.getBreed()); // '金毛'
console.log(Dog.getSpecies()); // 'Canine'
console.log(Animal.getSpecies()); // 'Unknown'
```

### 类的原型本质

```javascript
// 类的底层仍然是原型
class Person {
  constructor(name) {
    this.name = name;
  }

  greet() {
    console.log(`Hello, ${this.name}`);
  }
}

const person = new Person('张三');

// 验证类的原型本质
console.log(typeof Person); // 'function'
console.log(Person.prototype); // { constructor: Person, greet: [Function] }
console.log(person.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true

// 类方法本质上是原型方法
console.log(person.greet === Person.prototype.greet); // true
```

### 私有字段和公共字段

```javascript
// 私有字段（ES2022）
class BankAccount {
  #balance = 0;  // 私有字段
  owner;

  constructor(owner, initialBalance) {
    this.owner = owner;
    this.#balance = initialBalance;
  }

  deposit(amount) {
    if (amount > 0) {
      this.#balance += amount;
      console.log(`Deposited ${amount}. New balance: ${this.#balance}`);
    }
  }

  withdraw(amount) {
    if (amount > 0 && amount <= this.#balance) {
      this.#balance -= amount;
      console.log(`Withdrew ${amount}. New balance: ${this.#balance}`);
    }
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount('张三', 1000);
account.deposit(500);
account.withdraw(200);
console.log(account.getBalance()); // 1300

// account.#balance = 0;  // SyntaxError - 无法从外部访问私有字段
```

### Getter 和 Setter

```javascript
class Temperature {
  constructor(celsius) {
    this._celsius = celsius;
  }

  get celsius() {
    return this._celsius;
  }

  set celsius(value) {
    if (value < -273.15) {
      throw new Error('Temperature below absolute zero');
    }
    this._celsius = value;
  }

  get fahrenheit() {
    return (this._celsius * 9/5) + 32;
  }

  set fahrenheit(value) {
    this._celsius = (value - 32) * 5/9;
  }
}

const temp = new Temperature(25);
console.log(temp.celsius);    // 25
console.log(temp.fahrenheit); // 77

temp.fahrenheit = 100;
console.log(temp.celsius);    // 37.78
```

## 原型链的深度应用

### 方法继承和覆盖

```javascript
// 基类
class Shape {
  constructor(color) {
    this.color = color;
  }

  draw() {
    console.log(`Drawing a ${this.color} shape`);
  }

  getArea() {
    throw new Error('getArea must be implemented');
  }
}

class Rectangle extends Shape {
  constructor(width, height, color) {
    super(color);
    this.width = width;
    this.height = height;
  }

  draw() {
    super.draw(); // 调用父类方法
    console.log(`Rectangle ${this.width}x${this.height}`);
  }

  getArea() {
    return this.width * this.height;
  }
}

class Circle extends Shape {
  constructor(radius, color) {
    super(color);
    this.radius = radius;
  }

  draw() {
    console.log(`Circle with radius ${this.radius}`);
  }

  getArea() {
    return Math.PI * this.radius * this.radius;
  }
}

const rectangle = new Rectangle(5, 10, 'red');
rectangle.draw();
console.log(rectangle.getArea()); // 50

const circle = new Circle(7, 'blue');
circle.draw();
console.log(circle.getArea()); // 153.94
```

### Mixin 模式

```javascript
// Mixin 模式 - 多重继承
const Flyable = {
  fly() {
    console.log(`${this.name} is flying`);
  }
};

const Swimmable = {
  swim() {
    console.log(`${this.name} is swimming`);
  }
};

class Animal {
  constructor(name) {
    this.name = name;
  }
}

// 使用 Object.assign 实现 Mixin
class Duck extends Animal {}
Object.assign(Duck.prototype, Flyable, Swimmable);

const duck = new Duck('Daffy');
duck.fly();  // 'Daffy is flying'
duck.swim(); // 'Daffy is swimming'

// 或者使用类表达式
class Bird {
  constructor(name) {
    this.name = name;
  }
}

function applyMixins(targetClass, ...mixins) {
  mixins.forEach(mixin => {
    Object.getOwnPropertyNames(mixin).forEach(name => {
      const descriptor = Object.getOwnPropertyDescriptor(mixin, name);
      Object.defineProperty(targetClass.prototype, name, descriptor);
    });
  });
}

class Eagle {
  constructor(name) {
    this.name = name;
  }
}

applyMixins(Eagle, Flyable);

const eagle = new Eagle('Eddie');
eagle.fly();  // 'Eddie is flying'
```

### 原型链操作

```javascript
// 修改原型链
const originalPrototype = {
  greet() {
    console.log('Hello from original');
  }
};

const enhancedPrototype = Object.create(originalPrototype, {
  greet: {
    value: function() {
      console.log('Hello from enhanced');
      // 调用原型链上的方法
      Object.getPrototypeOf(this).greet.call(this);
    },
    writable: true,
    configurable: true,
    enumerable: true
  },

  farewell: {
    value: function() {
      console.log('Goodbye from enhanced');
    },
    writable: true,
    configurable: true,
    enumerable: true
  }
});

const obj = Object.create(enhancedPrototype);
obj.greet();    // 'Hello from enhanced' then 'Hello from original'
obj.farewell(); // 'Goodbye from enhanced'

// 动态修改原型
function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, I'm ${this.name}`);
};

const person = new Person('张三');
person.greet(); // 'Hello, I'm 张三'

// 动态添加方法
Person.prototype.farewell = function() {
  console.log(`Goodbye, I was ${this.name}`);
};

person.farewell(); // 'Goodbye, I was 张三'

// 注意：频繁修改原型会影响性能
```

## 性能优化

### 原型链查找优化

```javascript
// 原型链查找会沿着原型链向上查找，链越长越慢
class Level1 {
  method1() { console.log('Level 1'); }
}

class Level2 extends Level1 {
  method2() { console.log('Level 2'); }
}

class Level3 extends Level2 {
  method3() { console.log('Level 3'); }
}

class Level4 extends Level3 {
  method4() { console.log('Level 4'); }
}

const obj = new Level4();

// 查找 method1 需要遍历整个原型链
obj.method1();  // Level4 -> Level3 -> Level2 -> Level1 -> Object

// 优化：缓存经常使用的方法
class Optimized {
  constructor() {
    // 缓存原型方法
    this.cachedMethod = this.someMethod.bind(this);
  }

  someMethod() {
    console.log('Method called');
  }
}

const optimized = new Optimized();
optimized.cachedMethod(); // 直接调用，无需原型查找
```

### 避免深层次继承

```javascript
// 不好：过深的继承层次
class Animal { }
class Mammal extends Animal { }
class Primate extends Mammal { }
class Hominid extends Primate { }
class Human extends Hominid { }
class Adult extends Human { }
class AdultMale extends Adult { }

// 更好：组合优于继承
class Animal {
  constructor(name) {
    this.name = name;
  }
}

class Walker {
  walk() {
    console.log(`${this.name} is walking`);
  }
}

class Speaker {
  speak(words) {
    console.log(`${this.name} says: ${words}`);
  }
}

class Human extends Animal {}
Object.assign(Human.prototype, Walker, Speaker);

const person = new Human('张三');
person.walk();      // '张三 is walking'
person.speak('Hello'); // '张三 says: Hello'
```

### 原型方法与实例方法

```javascript
// 原型方法：所有实例共享，节省内存
class Efficient {
  constructor(value) {
    this.value = value;
  }

  // 原型方法
  multiply(factor) {
    return this.value * factor;
  }
}

const efficient1 = new Efficient(10);
const efficient2 = new Efficient(20);

console.log(efficient1.multiply === efficient2.multiply); // true - 同一个方法

// 实例方法：每个实例都有自己的方法，占用更多内存
class Inefficient {
  constructor(value) {
    this.value = value;
    this.multiply = function(factor) {
      return this.value * factor;
    };
  }
}

const inefficient1 = new Inefficient(10);
const inefficient2 = new Inefficient(20);

console.log(inefficient1.multiply === inefficient2.multiply); // false - 不同方法
```

## 实际应用场景

### 1. 工具函数库

```javascript
// 使用原型创建工具库
class StringExtensions {
  constructor(str) {
    this.value = String(str);
  }

  capitalize() {
    return this.value.charAt(0).toUpperCase() + this.value.slice(1).toLowerCase();
  }

  reverse() {
    return this.value.split('').reverse().join('');
  }

  truncate(length) {
    if (this.value.length <= length) return this.value;
    return this.value.substring(0, length) + '...';
  }

  isEmpty() {
    return this.value.trim().length === 0;
  }
}

// 使用方式
const text = new StringExtensions('hello world');
console.log(text.capitalize()); // 'Hello world'
console.log(text.reverse());    // 'dlrow olleh'
console.log(text.truncate(5));  // 'hello...'
console.log(text.isEmpty());    // false
```

### 2. 数据模型

```javascript
// 使用原型创建数据模型
class BaseModel {
  constructor(data = {}) {
    this.id = data.id || this.generateId();
    this.createdAt = new Date();
    this.updatedAt = new Date();
  }

  generateId() {
    return Date.now().toString(36) + Math.random().toString(36).substr(2);
  }

  update(updates) {
    Object.assign(this, updates, { updatedAt: new Date() });
  }

  toJSON() {
    return {
      id: this.id,
      createdAt: this.createdAt,
      updatedAt: this.updatedAt,
      ...this.getData()
    };
  }

  getData() {
    return {};
  }
}

class User extends BaseModel {
  constructor(data = {}) {
    super(data);
    this.name = data.name || '';
    this.email = data.email || '';
    this.age = data.age || 0;
  }

  getData() {
    return {
      name: this.name,
      email: this.email,
      age: this.age
    };
  }

  validate() {
    const errors = [];

    if (!this.name || this.name.trim() === '') {
      errors.push('Name is required');
    }

    if (!this.email || !this.email.includes('@')) {
      errors.push('Valid email is required');
    }

    if (this.age < 0 || this.age > 120) {
      errors.push('Age must be between 0 and 120');
    }

    return errors;
  }
}

const user = new User({
  name: '张三',
  email: 'zhangsan@example.com',
  age: 25
});

console.log(user.validate()); // [] - 无错误

user.update({ age: 26 });
console.log(user.toJSON());
```

### 3. 组件系统

```javascript
// 使用原型创建组件系统
class Component {
  constructor(props = {}) {
    this.props = props;
    this.state = {};
    this.children = [];
  }

  setState(newState) {
    this.state = { ...this.state, ...newState };
    this.render();
  }

  addChild(child) {
    this.children.push(child);
  }

  render() {
    throw new Error('render must be implemented');
  }
}

class Button extends Component {
  constructor(props) {
    super(props);
    this.state = { isHovered: false, isClicked: false };
  }

  render() {
    const className = `button ${this.state.isHovered ? 'hovered' : ''} ${this.state.isClicked ? 'clicked' : ''}`;
    return `<button class="${className}">${this.props.text}</button>`;
  }

  onMouseEnter() {
    this.setState({ isHovered: true });
  }

  onMouseLeave() {
    this.setState({ isHovered: false });
  }

  onClick() {
    this.setState({ isClicked: !this.state.isClicked });
    if (this.props.onClick) {
      this.props.onClick();
    }
  }
}

const button = new Button({
  text: 'Click me',
  onClick: () => console.log('Button clicked!')
});

console.log(button.render());
button.onMouseEnter();
console.log(button.render());
button.onClick();
console.log(button.render());
```

### 4. 插件系统

```javascript
// 使用原型实现插件系统
class PluginManager {
  constructor() {
    this.plugins = [];
  }

  register(plugin) {
    if (!plugin.name || typeof plugin.init !== 'function') {
      throw new Error('Invalid plugin');
    }
    this.plugins.push(plugin);
    plugin.init(this);
  }

  getPlugin(name) {
    return this.plugins.find(p => p.name === name);
  }

  hasPlugin(name) {
    return this.plugins.some(p => p.name === name);
  }
}

class Application extends PluginManager {
  constructor() {
    super();
    this.config = {};
    this.data = {};
  }

  setConfig(key, value) {
    this.config[key] = value;
  }

  getConfig(key) {
    return this.config[key];
  }

  setData(key, value) {
    this.data[key] = value;
  }

  getData(key) {
    return this.data[key];
  }
}

// 插件示例
const loggerPlugin = {
  name: 'logger',
  init(manager) {
    const originalSetConfig = manager.setConfig.bind(manager);
    manager.setConfig = function(key, value) {
      console.log(`Setting config: ${key} = ${value}`);
      return originalSetConfig(key, value);
    };
  }
};

const cachePlugin = {
  name: 'cache',
  cache: new Map(),
  init(manager) {
    const originalGetData = manager.getData.bind(manager);
    manager.getData = function(key) {
      if (this.cache.has(key)) {
        console.log(`Cache hit: ${key}`);
        return this.cache.get(key);
      }
      console.log(`Cache miss: ${key}`);
      return originalGetData(key);
    };
  },
  set(key, value) {
    this.cache.set(key, value);
  }
};

const app = new Application();
app.register(loggerPlugin);
app.register(cachePlugin);

app.setConfig('apiKey', 'secret123');  // "Setting config: apiKey = secret123"
app.setData('users', ['张三', '李四']); // "Cache miss: users"
cachePlugin.set('users', ['张三', '李四']);
app.getData('users');                 // "Cache hit: users"
```

## 调试和测试

### 原型链调试

```javascript
// 调试工具函数
function debugPrototypeChain(obj) {
  console.log('=== Prototype Chain ===');
  let current = obj;
  let level = 0;

  while (current) {
    const type = current.constructor ? current.constructor.name : typeof current;
    console.log(`Level ${level}: ${type}`);

    if (current.constructor) {
      console.log(`  Properties:`, Object.keys(current).filter(key => key !== 'constructor'));
    }

    current = Object.getPrototypeOf(current);
    level++;
  }

  console.log('=== End ===');
}

class A {}
class B extends A {}
class C extends B {}

const instance = new C();
debugPrototypeChain(instance);
// Level 0: C
// Level 1: B
// Level 2: A
// Level 3: Object
// Level 4: null
```

### 原型方法测试

```javascript
// 简单的测试框架
function describe(name, fn) {
  console.log(`\n${name}`);
  try {
    fn();
  } catch (error) {
    console.error('Test failed:', error.message);
  }
}

function it(description, fn) {
  try {
    fn();
    console.log(`  ✓ ${description}`);
  } catch (error) {
    console.error(`  ✗ ${description}`);
    console.error(`    Error: ${error.message}`);
  }
}

function expect(actual) {
  return {
    toBe(expected) {
      if (actual !== expected) {
        throw new Error(`Expected ${expected}, got ${actual}`);
      }
    },
    toEqual(expected) {
      if (JSON.stringify(actual) !== JSON.stringify(expected)) {
        throw new Error(`Expected ${JSON.stringify(expected)}, got ${JSON.stringify(actual)}`);
      }
    },
    toThrow(expectedMessage) {
      let threw = false;
      try {
        actual();
      } catch (error) {
        threw = true;
        if (error.message !== expectedMessage) {
          throw new Error(`Expected error "${expectedMessage}", got "${error.message}"`);
        }
      }
      if (!threw) {
        throw new Error(`Expected to throw "${expectedMessage}"`);
      }
    }
  };
}

// 测试示例
describe('Calculator', () => {
  class Calculator {
    add(a, b) { return a + b; }
    subtract(a, b) { return a - b; }
    multiply(a, b) { return a * b; }
    divide(a, b) {
      if (b === 0) throw new Error('Division by zero');
      return a / b;
    }
  }

  const calc = new Calculator();

  it('should add two numbers', () => {
    expect(calc.add(2, 3)).toBe(5);
  });

  it('should subtract two numbers', () => {
    expect(calc.subtract(5, 3)).toBe(2);
  });

  it('should multiply two numbers', () => {
    expect(calc.multiply(2, 3)).toBe(6);
  });

  it('should divide two numbers', () => {
    expect(calc.divide(6, 2)).toBe(3);
  });

  it('should throw on division by zero', () => {
    expect(() => calc.divide(6, 0)).toThrow('Division by zero');
  });
});
```

## 常见面试题

### 1. 原型和原型链是什么？

```javascript
// 原型是对象的内部属性，指向另一个对象
// 原型链是对象的继承链，用于属性查找

function Person(name) {
  this.name = name;
}

Person.prototype.greet = function() {
  console.log(`Hello, ${this.name}`);
};

const person = new Person('张三');

// 原型链查找过程
// person.greet()
// 1. 查找 person 对象本身 -> 没找到
// 2. 查找 person.__proto__ (Person.prototype) -> 找到了
console.log(person.__proto__ === Person.prototype); // true
```

### 2. 如何创建对象？

```javascript
// 1. 字面量
const obj1 = { name: '张三' };

// 2. 构造函数
function Person(name) {
  this.name = name;
}
const obj2 = new Person('李四');

// 3. Object.create
const proto = { greet() { console.log('Hello'); } };
const obj3 = Object.create(proto);

// 4. 工厂函数
function createPerson(name) {
  return { name };
}
const obj4 = createPerson('王五');

// 5. class
class Person {
  constructor(name) {
    this.name = name;
  }
}
const obj5 = new Person('赵六');
```

### 3. instanceof 的工作原理？

```javascript
// instanceof 检查构造函数的 prototype 是否出现在对象的原型链中
function myInstanceof(left, right) {
  let proto = Object.getPrototypeOf(left);
  let prototype = right.prototype;

  while (proto) {
    if (proto === prototype) return true;
    proto = Object.getPrototypeOf(proto);
  }

  return false;
}

// 使用示例
function Person() {}
const person = new Person();

console.log(person instanceof Person);        // true
console.log(myInstanceof(person, Person));   // true
console.log(person instanceof Object);        // true
console.log(myInstanceof(person, Object));   // true
```

### 4. new 操作符做了什么？

```javascript
// 模拟 new 操作符
function myNew(constructor, ...args) {
  // 1. 创建新对象
  const obj = Object.create(null);

  // 2. 绑定原型
  Object.setPrototypeOf(obj, constructor.prototype);

  // 3. 执行构造函数，绑定 this
  const result = constructor.apply(obj, args);

  // 4. 返回结果
  return result instanceof Object ? result : obj;
}

// 使用示例
function Person(name) {
  this.name = name;
}

const person = myNew(Person, '张三');
console.log(person.name); // '张三'
console.log(person instanceof Person); // true
```

### 5. 如何实现继承？

```javascript
// 寄生组合继承（最佳实践）
function inheritPrototype(subType, superType) {
  const prototype = Object.create(superType.prototype);
  prototype.constructor = subType;
  subType.prototype = prototype;
}

class SuperType {
  constructor(name) {
    this.name = name;
  }
}

class SubType extends SuperType {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}

// 或者使用 ES6 extends
class Child extends Parent {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
}
```

## 总结

JavaScript 的原型和原型链是语言的核心特性，理解它们对于掌握 JavaScript 至关重要。关键要点：

1. **原型基础**：理解 prototype 和 __proto__ 的区别和联系
2. **原型链**：掌握属性查找机制和继承原理
3. **继承方式**：了解各种继承方式的优缺点和适用场景
4. **ES6 类**：理解类语法的本质和与原型的关系
5. **性能优化**：避免深层继承，合理使用原型方法
6. **实际应用**：在工具库、数据模型、组件系统中的应用
7. **调试技巧**：掌握原型链调试方法

原型继承提供了比传统类继承更灵活的编程方式，但同时也要求开发者有更深的理解。在实际开发中，要根据项目需求选择合适的继承策略，避免过度复杂的设计。

随着 ES6+ 的普及，class 语法让原型继承更加易用，但理解其背后的原型原理仍然很重要。建议在实际项目中多练习，加深对原型和原型链的理解。