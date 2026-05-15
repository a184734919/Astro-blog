---
title: JavaScript Symbol 类型
published: 2022-06-03
description: 'Symbol 的使用场景和方法的详细介绍和学习笔记'
image: ''
tags: ["JS"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

Symbol 是 ES6 引入的一种新的原始数据类型，表示独一无二的值。它是 JavaScript 第七种数据类型（前六种是：undefined、null、布尔值、字符串、数值、对象）。

## 核心概念

Symbol 的主要特点是唯一性，即使创建时传入相同的描述符，两个 Symbol 也不相等。这使得 Symbol 非常适合作为对象属性名，避免属性名冲突。

## 创建 Symbol

### Symbol() 函数

```javascript
// 创建 Symbol
const sym1 = Symbol();
const sym2 = Symbol();

console.log(sym1 === sym2); // false
console.log(sym1); // Symbol()

// 带描述的 Symbol
const sym3 = Symbol('description');
const sym4 = Symbol('description');

console.log(sym3 === sym4); // false (描述相同但 Symbol 不同)
console.log(sym3); // Symbol(description)
console.log(sym4); // Symbol(description)

// 获取描述
console.log(sym3.description); // 'description'

// Symbol 不能使用 new 关键字
// const sym = new Symbol(); // TypeError
```

### Symbol.for() 和 Symbol.keyFor()

```javascript
// Symbol.for() - 在全局 Symbol 注册表中查找或创建
const globalSym1 = Symbol.for('key');
const globalSym2 = Symbol.for('key');

console.log(globalSym1 === globalSym2); // true

// Symbol() 创建的 Symbol 不会被注册
const localSym = Symbol('key');
const globalSym3 = Symbol.for('key');

console.log(localSym === globalSym3); // false

// Symbol.keyFor() - 获取全局 Symbol 的 key
console.log(Symbol.keyFor(globalSym1)); // 'key'
console.log(Symbol.keyFor(localSym)); // undefined (非全局 Symbol)
```

### 内置 Symbol 值

JavaScript 提供了一些内置的 Symbol 值，用于语言内部的机制：

```javascript
// Symbol.iterator - 定义对象的默认迭代器
const myIterable = {
  [Symbol.iterator]: function* () {
    yield 1;
    yield 2;
    yield 3;
  }
};

console.log([...myIterable]); // [1, 2, 3]

// Symbol.toPrimitive - 对象到原始值的转换
const obj = {
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      return 42;
    }
    return 'hello';
  }
};

console.log(+obj); // 42
console.log(`${obj}`); // 'hello'

// Symbol.toStringTag - 自定义对象的 toString 输出
class MyClass {
  get [Symbol.toStringTag]() {
    return 'MyClass';
  }
}

console.log(Object.prototype.toString.call(new MyClass())); // '[object MyClass]'

// 其他内置 Symbol
console.log(Symbol.asyncIterator); // 异步迭代器
console.log(Symbol.hasInstance);   // instanceof 检查
console.log(Symbol.isConcatSpreadable); // 数组拼接
console.log(Symbol.match);         // String.match
console.log(Symbol.replace);       // String.replace
console.log(Symbol.search);        // String.search
console.log(Symbol.species);       // 创建派生对象
console.log(Symbol.split);         // String.split
console.log(Symbol.unscopables);   // with 环境
```

## Symbol 作为属性名

### 基本用法

```javascript
// Symbol 作为对象属性名
const sym = Symbol('id');

const obj = {
  [sym]: 123,
  name: '张三',
  age: 25
};

console.log(obj[sym]); // 123
console.log(obj.name); // '张三'

// Symbol 属性不会出现在 for...in 循环中
for (const key in obj) {
  console.log(key); // 'name', 'age' (没有 Symbol 属性)
}

// Symbol 属性不会被 Object.keys() 返回
console.log(Object.keys(obj)); // ['name', 'age']

// Symbol 属性不会被 Object.getOwnPropertyNames() 返回
console.log(Object.getOwnPropertyNames(obj)); // ['name', 'age']

// 获取所有 Symbol 属性
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]

// 获取所有属性（包括 Symbol）
console.log(Reflect.ownKeys(obj)); // ['name', 'age', Symbol(id)]
```

### 隐藏属性

```javascript
// 使用 Symbol 创建私有属性
const privateData = Symbol('privateData');

class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this[privateData] = {
      secret: '这是私密信息',
      token: 'abc123'
    };
  }

  getSecret() {
    return this[privateData].secret;
  }

  get token() {
    return this[privateData].token;
  }
}

const user = new User('张三', 25);

console.log(user.name); // '张三'
console.log(user.age); // 25
console.log(user.getSecret()); // '这是私密信息'
console.log(user.token); // 'abc123'

// 外部无法直接访问 privateData
console.log(user[privateData]); // undefined (privateData 不在外部作用域)
```

### 常量定义

```javascript
// 使用 Symbol 定义常量
const MODES = {
  READ: Symbol('read'),
  WRITE: Symbol('write'),
  EXECUTE: Symbol('execute')
};

function processFile(mode) {
  if (mode === MODES.READ) {
    console.log('读取文件');
  } else if (mode === MODES.WRITE) {
    console.log('写入文件');
  } else if (mode === MODES.EXECUTE) {
    console.log('执行文件');
  }
}

processFile(MODES.READ);   // '读取文件'
processFile(MODES.WRITE); // '写入文件'

// 无法通过字符串比较
// if (mode === 'read') { } // 永远不会为 true
```

## Symbol 的实际应用

### 1. 消除魔术字符串

```javascript
// 魔术字符串：直接使用字符串
function getArea(shape) {
  if (shape === 'circle') {
    return Math.PI * r * r;
  } else if (shape === 'rectangle') {
    return width * height;
  }
}

// 使用 Symbol 消除魔术字符串
const shapeType = {
  CIRCLE: Symbol('circle'),
  RECTANGLE: Symbol('rectangle'),
  TRIANGLE: Symbol('triangle')
};

function getArea(shape) {
  const { type, ...params } = shape;

  if (type === shapeType.CIRCLE) {
    const { r } = params;
    return Math.PI * r * r;
  } else if (type === shapeType.RECTANGLE) {
    const { width, height } = params;
    return width * height;
  }
}

const circle = { type: shapeType.CIRCLE, r: 10 };
const rectangle = { type: shapeType.RECTANGLE, width: 10, height: 20 };

console.log(getArea(circle));    // 314.159...
console.log(getArea(rectangle)); // 200
```

### 2. 定义类的私有方法

```javascript
// 使用 Symbol 定义私有方法
const privateMethod = Symbol('privateMethod');

class DataProcessor {
  constructor(data) {
    this.data = data;
  }

  [privateMethod]() {
    // 私有方法逻辑
    return this.data.map(item => item.toUpperCase());
  }

  process() {
    // 公有方法调用私有方法
    return this[privateMethod]();
  }
}

const processor = new DataProcessor(['hello', 'world']);
console.log(processor.process()); // ['HELLO', 'WORLD']

// 无法直接访问私有方法
// processor[privateMethod](); // ReferenceError
```

### 3. 元编程

```javascript
// 使用 Symbol.toPrimitive 自定义对象转换
class Money {
  constructor(amount, currency) {
    this.amount = amount;
    this.currency = currency;
  }

  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      return this.amount;
    } else if (hint === 'string') {
      return `${this.amount} ${this.currency}`;
    }
    return `${this.amount} ${this.currency}`;
  }
}

const money = new Money(100, 'USD');

console.log(String(money)); // '100 USD'
console.log(Number(money)); // 100
console.log(money + 0); // 100
console.log(`${money}`); // '100 USD'

// 使用 Symbol.iterator 使对象可迭代
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  [Symbol.iterator]() {
    let current = this.start;
    const end = this.end;
    const step = this.step;

    return {
      next() {
        if (current < end) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { done: true };
      }
    };
  }
}

const range = new Range(1, 5);
console.log([...range]); // [1, 2, 3, 4]

for (const num of range) {
  console.log(num); // 1, 2, 3, 4
}
```

### 4. 定义类的特殊行为

```javascript
// 使用 Symbol.toStringTag 自定义类型标签
class User {
  constructor(name) {
    this.name = name;
  }

  get [Symbol.toStringTag]() {
    return 'User';
  }
}

const user = new User('张三');
console.log(Object.prototype.toString.call(user)); // '[object User]'
console.log(user.toString()); // '[object User]'

// 使用 Symbol.hasInstance 自定义 instanceof 行为
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

const arr = [1, 2, 3];
console.log(arr instanceof MyArray); // true

// 使用 Symbol.species 自定义派生对象的构造函数
class MyArray2 extends Array {
  static get [Symbol.species]() {
    return Array;
  }
}

const myArr = new MyArray2(1, 2, 3);
const mapped = myArr.map(x => x * 2);

console.log(mapped instanceof MyArray2); // false
console.log(mapped instanceof Array); // true
```

### 5. 防止属性名冲突

```javascript
// 第三方库 A
const libraryA = (function() {
  const INTERNAL = Symbol('internal');

  return {
    create(data) {
      const obj = {};
      obj[INTERNAL] = data;
      return Object.defineProperty(obj, 'getData', {
        value: () => obj[INTERNAL]
      });
    }
  };
})();

// 第三方库 B
const libraryB = (function() {
  const INTERNAL = Symbol('internal');

  return {
    create(data) {
      const obj = {};
      obj[INTERNAL] = data;
      return Object.defineProperty(obj, 'getData', {
        value: () => obj[INTERNAL]
      });
    }
  };
})();

// 使用两个库
const objA = libraryA.create({ source: 'A' });
const objB = libraryB.create({ source: 'B' });

console.log(objA.getData()); // { source: 'A' }
console.log(objB.getData()); // { source: 'B' }

// 各自的 Symbol 不会冲突
```

## Symbol 的方法

### Object.getOwnPropertySymbols()

```javascript
const sym1 = Symbol('a');
const sym2 = Symbol('b');

const obj = {
  name: 'test',
  [sym1]: 'value1',
  [sym2]: 'value2',
  age: 25
};

// 获取所有 Symbol 属性
const symbols = Object.getOwnPropertySymbols(obj);
console.log(symbols); // [Symbol(a), Symbol(b)]

// 结合 Reflect.ownKeys 获取所有属性
const allKeys = Reflect.ownKeys(obj);
console.log(allKeys); // ['name', 'age', Symbol(a), Symbol(b)]
```

### Reflect.ownKeys()

```javascript
const sym = Symbol('sym');

const obj = {
  name: 'test',
  [sym]: 'value',
  age: 25
};

// 获取所有属性键（包括 Symbol）
const keys = Reflect.ownKeys(obj);
console.log(keys); // ['name', 'age', Symbol(sym)]

// 与 Object.keys() 对比
console.log(Object.keys(obj)); // ['name', 'age']
```

## 注意事项

1. **Symbol 不能使用 new**：Symbol 是原始值，不是对象，不能使用 new 操作符。

2. **Symbol 不能转换为数字**：Symbol 不能隐式转换为数字，会抛出错误。

```javascript
const sym = Symbol('test');
// Number(sym); // TypeError
// sym + 1; // TypeError
```

3. **Symbol 可以转换为字符串**：使用 String() 或 .toString() 方法。

```javascript
const sym = Symbol('test');
console.log(String(sym)); // 'Symbol(test)'
console.log(sym.toString()); // 'Symbol(test)'
```

4. **Symbol 不能用于算术运算**：Symbol 不能参与加、减、乘、除等运算。

5. **Symbol 作为键时的特殊性**：Symbol 属性不会被常规方法遍历，只能通过专门的 API 访问。

6. **Symbol 的描述是可选的**：创建 Symbol 时可以不提供描述。

7. **全局 Symbol 注册表**：使用 Symbol.for() 创建的 Symbol 可以在整个程序中共享。

8. **JSON.stringify() 忽略 Symbol**：Symbol 键的属性在 JSON 序列化时会被忽略。

```javascript
const sym = Symbol('id');
const obj = { [sym]: 123, name: 'test' };
console.log(JSON.stringify(obj)); // '{"name":"test"}'
```

## 最佳实践

1. **使用有意义的描述**：为 Symbol 提供清晰的描述，便于调试和维护。

2. **避免滥用 Symbol**：只在确实需要唯一属性键时使用 Symbol。

3. **封装 Symbol**：将 Symbol 定义在闭包或模块作用域中，实现真正的私有属性。

4. **使用内置 Symbol**：充分利用内置 Symbol 实现元编程功能。

5. **文档化 Symbol 属性**：在注释或文档中说明 Symbol 属性的用途。

6. **考虑兼容性**：在不支持 Symbol 的环境中提供 Polyfill。

7. **使用 Symbol 作为常量**：用 Symbol 定义一组常量，避免魔术字符串。

8. **合理使用全局 Symbol**：只有在需要在全局共享时才使用 Symbol.for()。

## 总结

Symbol 是 ES6 引入的重要特性，它提供了独一无二的数据类型，解决了属性名冲突的问题。通过理解 Symbol 的基本用法、作为属性名的特性、内置 Symbol 值以及实际应用场景，可以更好地利用这个强大的工具。

主要要点：
- Symbol 是原始数据类型，表示独一无二的值
- Symbol 非常适合作为对象属性名，避免冲突
- 可以使用 Symbol 实现私有属性、消除魔术字符串
- 内置 Symbol 值提供了元编程能力
- Symbol 属性不会被常规方法遍历
- 合理使用 Symbol 可以提高代码的可维护性和安全性

掌握 Symbol 的使用，有助于编写更安全、更优雅的 JavaScript 代码。