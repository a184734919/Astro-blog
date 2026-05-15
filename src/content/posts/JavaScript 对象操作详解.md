---
title: JavaScript 对象操作详解
published: 2022-03-18
description: '对象创建、属性访问、遍历方法的详细介绍和学习笔记'
image: ''
tags: ["JS基础","对象"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 对象是 JavaScript 语言的核心数据结构，几乎所有的 JavaScript 值（除原始值外）都是对象。本文将详细介绍对象的创建、属性访问、遍历、操作以及相关的最佳实践。

## 对象基础

### 什么是对象

对象是键值对的集合，用于存储数据和功能：

```javascript
// 简单对象
let person = {
  name: '张三',
  age: 25,
  sayHello: function() {
    console.log('你好！');
  }
};

console.log(person.name); // "张三"
person.sayHello(); // "你好！"
```

### 对象的创建方式

#### 1. 对象字面量（推荐）

```javascript
// 空对象
let emptyObj = {};

// 带属性的对象
let user = {
  name: '李四',
  age: 30,
  'user-id': 12345  // 包含连字符的属性名需要引号
};
```

#### 2. 使用 Object 构造函数

```javascript
let obj = new Object();
obj.name = '王五';
obj.age = 28;

// 创建空对象
let emptyObj = new Object();
```

#### 3. 使用 Object.create()

```javascript
// 创建一个新对象，使用现有对象作为原型
let proto = {
  greet: function() {
    return `Hello, ${this.name}!`;
  }
};

let person = Object.create(proto);
person.name = '赵六';

console.log(person.greet()); // "Hello, 赵六!"
```

#### 4. 使用构造函数

```javascript
function Person(name, age) {
  this.name = name;
  this.age = age;
}

Person.prototype.greet = function() {
  return `Hello, ${this.name}!`;
};

let person1 = new Person('孙七', 35);
console.log(person1.greet()); // "Hello, 孙七!"
```

#### 5. 使用类（ES6）

```javascript
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  greet() {
    return `Hello, ${this.name}!`;
  }
}

let person2 = new Person('周八', 40);
console.log(person2.greet()); // "Hello, 周八!"
```

## 属性访问

### 点语法访问

```javascript
let user = {
  name: '张三',
  age: 25
};

console.log(user.name);  // "张三"
console.log(user.age);   // 25

// 动态属性名
let prop = 'name';
console.log(user[prop]); // "张三"
```

### 方括号语法访问

```javascript
let user = {
  name: '李四',
  'user-id': 12345,
  'first name': '王五'
};

console.log(user['name']);       // "李四"
console.log(user['user-id']);    // 12345
console.log(user['first name']); // "王五"

// 方括号支持变量
let key = 'name';
console.log(user[key]); // "李四"
```

### 可选链操作符（ES2020）

```javascript
let user = {
  profile: {
    name: '张三'
  }
};

// 安全访问嵌套属性
console.log(user.profile?.name);      // "张三"
console.log(user.address?.city);      // undefined
console.log(user.profile?.info?.age); // undefined

// 调用方法
console.log(user.getInfo?.());        // undefined

// 与空值合并操作符配合
let city = user.address?.city ?? '未知';
console.log(city); // "未知"
```

## 属性操作

### 添加和修改属性

```javascript
let user = {};

// 添加属性
user.name = '张三';
user.age = 25;
user['email'] = 'zhangsan@example.com';

// 修改属性
user.name = '李四';

// 批量添加属性
Object.assign(user, {
  phone: '13800138000',
  address: '北京市'
});

console.log(user);
```

### 删除属性

```javascript
let user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 删除属性
delete user.age;
delete user['email'];

console.log(user); // { name: '张三' }

// 注意：不能删除继承的属性
let obj = Object.create({ inherited: 'value' });
obj.own = 'own value';
delete obj.inherited; // 无效
delete obj.own; // 有效
```

### 检查属性存在

```javascript
let user = {
  name: '张三',
  age: 25,
  toString: function() {
    return this.name;
  }
};

// in 操作符 - 检查自有和继承属性
console.log('name' in user);      // true
console.log('toString' in user);  // true
console.log('address' in user);   // false

// hasOwnProperty - 只检查自有属性
console.log(user.hasOwnProperty('name'));      // true
console.log(user.hasOwnProperty('toString'));  // false
console.log(user.hasOwnProperty('address'));   // false

// Object.hasOwn - 更现代的方式（ES2022）
console.log(Object.hasOwn(user, 'name'));      // true
console.log(Object.hasOwn(user, 'toString'));  // false
```

## 对象遍历

### for...in 遍历

```javascript
let user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 遍历可枚举属性（包括继承的）
for (let key in user) {
  if (user.hasOwnProperty(key)) {  // 过滤继承属性
    console.log(`${key}: ${user[key]}`);
  }
}

// 输出:
// name: 张三
// age: 25
// email: zhangsan@example.com
```

### Object.keys() 遍历

```javascript
let user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 获取所有可枚举的自有属性名
let keys = Object.keys(user);
console.log(keys); // ['name', 'age', 'email']

// 遍历属性
Object.keys(user).forEach(key => {
  console.log(`${key}: ${user[key]}`);
});
```

### Object.values() 遍历

```javascript
let user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 获取所有可枚举的自有属性值
let values = Object.values(user);
console.log(values); // ['张三', 25, 'zhangsan@example.com']

// 遍历值
Object.values(user).forEach(value => {
  console.log(value);
});
```

### Object.entries() 遍历

```javascript
let user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 获取所有可枚举的自有属性的键值对
let entries = Object.entries(user);
console.log(entries); // [['name', '张三'], ['age', 25], ['email', 'zhangsan@example.com']]

// 遍历键值对
Object.entries(user).forEach(([key, value]) => {
  console.log(`${key}: ${value}`);
});

// 转换为 Map
let userMap = new Map(Object.entries(user));
console.log(userMap.get('name')); // '张三'
```

## 对象合并

### Object.assign()

```javascript
let target = { a: 1 };
let source1 = { b: 2, c: 3 };
let source2 = { c: 4, d: 5 };

// 合并对象
let result = Object.assign(target, source1, source2);
console.log(result); // { a: 1, b: 2, c: 4, d: 5 }

// 注意：会修改目标对象
console.log(target === result); // true

// 创建新对象
let newObj = Object.assign({}, source1, source2);
console.log(newObj); // { b: 2, c: 4, d: 5 }

// 注意：浅拷贝
let obj1 = { a: { b: 1 } };
let obj2 = Object.assign({}, obj1);
obj2.a.b = 2;
console.log(obj1.a.b); // 2
```

### 展开运算符（ES6）

```javascript
let obj1 = { a: 1, b: 2 };
let obj2 = { c: 3, d: 4 };

// 合并对象
let merged = { ...obj1, ...obj2 };
console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 后面的属性会覆盖前面的
let conflict = { ...obj1, a: 10, ...obj2 };
console.log(conflict); // { a: 10, b: 2, c: 3, d: 4 }

// 添加属性
let withNewProp = { ...obj1, e: 5 };
console.log(withNewProp); // { a: 1, b: 2, e: 5 }

// 注意：也是浅拷贝
let obj = { a: { b: 1 } };
let copy = { ...obj };
copy.a.b = 2;
console.log(obj.a.b); // 2
```

## 对象拷贝

### 浅拷贝

```javascript
let original = {
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    country: '中国'
  }
};

// 方法1：Object.assign()
let copy1 = Object.assign({}, original);

// 方法2：展开运算符
let copy2 = { ...original };

// 方法3：Object.create()
let copy3 = Object.create(
  Object.getPrototypeOf(original),
  Object.getOwnPropertyDescriptors(original)
);

console.log(copy1);
console.log(copy2);

// 验证浅拷贝
copy1.address.city = '上海';
console.log(original.address.city); // '上海' (被修改)
```

### 深拷贝

```javascript
let original = {
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    country: '中国'
  }
};

// 方法1：JSON 方法（简单对象）
let deepCopy1 = JSON.parse(JSON.stringify(original));
deepCopy1.address.city = '上海';
console.log(original.address.city); // '北京' (未被修改)

// 方法2：使用 structuredClone（ES2021）
let deepCopy2 = structuredClone(original);

// 方法3：手动实现深拷贝
function deepClone(obj, map = new WeakMap()) {
  // 处理原始值和 null
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }

  // 处理循环引用
  if (map.has(obj)) {
    return map.get(obj);
  }

  // 处理 Date
  if (obj instanceof Date) {
    return new Date(obj);
  }

  // 处理 Array
  if (Array.isArray(obj)) {
    const copy = [];
    map.set(obj, copy);
    return copy.map(item => deepClone(item, map));
  }

  // 处理 Object
  const copy = Object.create(Object.getPrototypeOf(obj));
  map.set(obj, copy);

  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      copy[key] = deepClone(obj[key], map);
    }
  }

  return copy;
}

let deepCopy3 = deepClone(original);
```

## 对象方法

### Object 静态方法

```javascript
let obj = {
  name: '张三',
  age: 25,
  [Symbol('id')]: 123
};

// Object.keys() - 获取所有可枚举属性名
console.log(Object.keys(obj)); // ['name', 'age']

// Object.values() - 获取所有可枚举属性值
console.log(Object.values(obj)); // ['张三', 25]

// Object.entries() - 获取所有可枚举属性的键值对
console.log(Object.entries(obj)); // [['name', '张三'], ['age', 25]]

// Object.getOwnPropertyNames() - 获取所有自有属性名（包括不可枚举）
console.log(Object.getOwnPropertyNames(obj)); // ['name', 'age']

// Object.getOwnPropertySymbols() - 获取所有 Symbol 属性
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]

// Object.getOwnPropertyDescriptors() - 获取所有属性描述符
console.log(Object.getOwnPropertyDescriptors(obj));

// Object.fromEntries() - 将键值对数组转换为对象
let entries = [['name', '李四'], ['age', 30]];
let newObj = Object.fromEntries(entries);
console.log(newObj); // { name: '李四', age: 30 }
```

### 对象实例方法

```javascript
let user = {
  name: '张三',
  age: 25,

  // 方法简写
  greet() {
    return `Hello, ${this.name}!`;
  },

  // 计算属性名
  ['say' + 'Hi']() {
    return `Hi, ${this.name}!`;
  }
};

console.log(user.greet());    // "Hello, 张三!"
console.log(user.sayHi());    // "Hi, 张三!"

// valueOf() - 返回对象的原始值
let numObj = {
  value: 42,
  valueOf() {
    return this.value;
  }
};
console.log(numObj + 10); // 52

// toString() - 返回对象的字符串表示
let strObj = {
  name: '张三',
  toString() {
    return `User: ${this.name}`;
  }
};
console.log(String(strObj)); // "User: 张三"
```

## 属性描述符

### 获取属性描述符

```javascript
let obj = {
  name: '张三',
  age: 25
};

// 获取单个属性的描述符
let nameDescriptor = Object.getOwnPropertyDescriptor(obj, 'name');
console.log(nameDescriptor);
// {
//   value: '张三',
//   writable: true,
//   enumerable: true,
//   configurable: true
// }

// 获取所有属性的描述符
let allDescriptors = Object.getOwnPropertyDescriptors(obj);
console.log(allDescriptors);
```

### 定义属性

```javascript
let obj = {};

// 定义单个属性
Object.defineProperty(obj, 'name', {
  value: '张三',
  writable: true,      // 是否可写
  enumerable: true,    // 是否可枚举
  configurable: true   // 是否可配置
});

// 定义多个属性
Object.defineProperties(obj, {
  age: {
    value: 25,
    writable: true,
    enumerable: true
  },
  _id: {
    value: 123,
    writable: true,
    enumerable: false  // 不可枚举
  }
});

console.log(Object.keys(obj)); // ['name', 'age']
console.log(obj._id); // 123 (仍可访问)
```

### 访问器属性

```javascript
let person = {
  firstName: '张',
  lastName: '三',

  // getter
  get fullName() {
    return `${this.firstName}${this.lastName}`;
  },

  // setter
  set fullName(name) {
    const parts = name.split(' ');
    this.firstName = parts[0];
    this.lastName = parts[1] || '';
  }
};

console.log(person.fullName); // '张三'

person.fullName = '李 四';
console.log(person.firstName); // '李'
console.log(person.lastName);  // '四'

// 使用 Object.defineProperty 定义访问器
let user = {
  _age: 25
};

Object.defineProperty(user, 'age', {
  get() {
    return this._age;
  },
  set(value) {
    if (value >= 0) {
      this._age = value;
    } else {
      console.log('年龄不能为负数');
    }
  }
});

user.age = 30;
console.log(user.age); // 30

user.age = -5; // "年龄不能为负数"
```

## 对象冻结和密封

### Object.preventExtensions()

```javascript
let obj = { a: 1 };

// 防止添加新属性
Object.preventExtensions(obj);

obj.b = 2; // 无效（严格模式下报错）
delete obj.a; // 可以删除
obj.a = 2; // 可以修改

console.log(Object.isExtensible(obj)); // false
```

### Object.seal()

```javascript
let obj = { a: 1, b: 2 };

// 密封对象：防止添加和删除属性
Object.seal(obj);

obj.c = 3; // 无效
delete obj.a; // 无效
obj.a = 10; // 可以修改

console.log(Object.isSealed(obj)); // true
```

### Object.freeze()

```javascript
let obj = { a: 1, b: 2 };

// 冻结对象：防止添加、删除和修改属性
Object.freeze(obj);

obj.c = 3; // 无效
delete obj.a; // 无效
obj.a = 10; // 无效

console.log(Object.isFrozen(obj)); // true

// 注意：只冻结第一层
let nested = { a: { b: 1 } };
Object.freeze(nested);
nested.a.b = 2; // 仍然可以修改嵌套对象
```

### 深度冻结

```javascript
function deepFreeze(obj) {
  // 冻结自身
  Object.freeze(obj);

  // 冻结所有属性
  Object.getOwnPropertyNames(obj).forEach(name => {
    const value = obj[name];
    if (typeof value === 'object' && value !== null) {
      deepFreeze(value);
    }
  });

  return obj;
}

let nested = { a: { b: 1 } };
deepFreeze(nested);
nested.a.b = 2; // 无效
```

## 实际应用

### 1. 配置管理

```javascript
// 默认配置
const defaultConfig = {
  theme: 'light',
  language: 'zh-CN',
  timeout: 5000,
  debug: false
};

// 用户配置
const userConfig = {
  theme: 'dark',
  debug: true
};

// 合并配置
const config = { ...defaultConfig, ...userConfig };
console.log(config); // { theme: 'dark', language: 'zh-CN', timeout: 5000, debug: true }
```

### 2. 数据验证

```javascript
function validateUser(user) {
  const requiredFields = ['name', 'email', 'age'];

  // 检查必填字段
  for (let field of requiredFields) {
    if (!Object.hasOwn(user, field)) {
      throw new Error(`缺少必填字段: ${field}`);
    }
  }

  // 验证数据类型
  if (typeof user.name !== 'string') {
    throw new Error('name 必须是字符串');
  }

  if (typeof user.email !== 'string' || !email.includes('@')) {
    throw new Error('email 格式不正确');
  }

  if (typeof user.age !== 'number' || user.age < 0) {
    throw new Error('age 必须是非负数');
  }

  return true;
}

try {
  const user = { name: '张三', email: 'test@example.com', age: 25 };
  validateUser(user);
  console.log('验证通过');
} catch (error) {
  console.error('验证失败:', error.message);
}
```

### 3. 对象映射

```javascript
const users = [
  { id: 1, name: '张三', age: 25 },
  { id: 2, name: '李四', age: 30 },
  { id: 3, name: '王五', age: 28 }
];

// 转换为对象映射
const userMap = Object.fromEntries(
  users.map(user => [user.id, user])
);

console.log(userMap);
// {
//   1: { id: 1, name: '张三', age: 25 },
//   2: { id: 2, name: '李四', age: 30 },
//   3: { id: 3, name: '王五', age: 28 }
// }

// 快速查找
console.log(userMap[2]); // { id: 2, name: '李四', age: 30 }
```

### 4. 数据转换

```javascript
const apiResponse = {
  user_id: 1,
  user_name: '张三',
  user_email: 'test@example.com'
};

// 转换为驼峰命名
function toCamelCase(obj) {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => {
      const camelKey = key.replace(/_([a-z])/g, (_, char) => char.toUpperCase());
      return [camelKey, value];
    })
  );
}

const formattedResponse = toCamelCase(apiResponse);
console.log(formattedResponse);
// { userId: 1, userName: '张三', userEmail: 'test@example.com' }
```

### 5. 深度比较

```javascript
function deepEqual(obj1, obj2) {
  // 检查原始值和 null
  if (obj1 === obj2) {
    return true;
  }

  // 检查类型
  if (typeof obj1 !== 'object' || obj1 === null ||
      typeof obj2 !== 'object' || obj2 === null) {
    return false;
  }

  // 检查键的数量
  const keys1 = Object.keys(obj1);
  const keys2 = Object.keys(obj2);

  if (keys1.length !== keys2.length) {
    return false;
  }

  // 递归比较每个属性
  for (let key of keys1) {
    if (!deepEqual(obj1[key], obj2[key])) {
      return false;
    }
  }

  return true;
}

const obj1 = { a: 1, b: { c: 2 } };
const obj2 = { a: 1, b: { c: 2 } };
const obj3 = { a: 1, b: { c: 3 } };

console.log(deepEqual(obj1, obj2)); // true
console.log(deepEqual(obj1, obj3)); // false
```

## 性能优化

### 1. 避免不必要的对象创建

```javascript
// 不好的方式
function createUser(name, age) {
  let user = {};
  user.name = name;
  user.age = age;
  user.greet = function() {
    return `Hello, ${this.name}!`;
  };
  return user;
}

// 好的方式 - 使用对象字面量
function createUser(name, age) {
  return {
    name,
    age,
    greet() {
      return `Hello, ${this.name}!`;
    }
  };
}
```

### 2. 使用原型减少内存占用

```javascript
// 不好的方式 - 每个实例都有自己的方法
function BadUser(name) {
  this.name = name;
  this.greet = function() {
    return `Hello, ${this.name}!`;
  };
}

// 好的方式 - 方法共享
function GoodUser(name) {
  this.name = name;
}

GoodUser.prototype.greet = function() {
  return `Hello, ${this.name}!`;
};

// 或者使用类
class User {
  constructor(name) {
    this.name = name;
  }

  greet() {
    return `Hello, ${this.name}!`;
  }
}
```

### 3. 避免过度使用 delete

```javascript
// 不好的方式 - 频繁删除属性
let obj = { a: 1, b: 2, c: 3 };
delete obj.a;
delete obj.b;

// 好的方式 - 创建新对象或重新赋值
let obj = { a: 1, b: 2, c: 3 };
obj = { c: 3 }; // 或使用解构
```

### 4. 使用 Object.freeze 优化性能

```javascript
// 冻结常量对象，防止意外修改
const CONFIG = Object.freeze({
  API_URL: 'https://api.example.com',
  TIMEOUT: 5000,
  VERSION: '1.0.0'
});

// 优化 - V8 可以优化对冻结对象的访问
```

## 注意事项

1. **引用类型**：对象是引用类型，赋值操作只是复制引用，而不是复制对象本身。

2. **浅拷贝 vs 深拷贝**：默认的拷贝方法是浅拷贝，需要深拷贝时要使用专门的实现。

3. **循环引用**：处理复杂对象时要注意循环引用问题，深拷贝时使用 WeakMap 来检测。

4. **原型链**：for...in 会遍历原型链上的属性，使用 hasOwnProperty 进行过滤。

5. **属性顺序**：在现代 JavaScript 中，字符串键按插入顺序排序，数字键按数字顺序排序。

6. **Symbol 属性**：Symbol 属性不会被常规遍历方法访问到。

7. **冻结是浅层的**：Object.freeze 只冻结第一层，嵌套对象仍可修改。

8. **避免修改原型**：不要在原型上添加或删除属性，这会影响所有实例。

## 最佳实践

1. **优先使用对象字面量**：创建对象时优先使用字面量语法，代码更简洁。

2. **使用展开运算符**：合并和拷贝对象时使用展开运算符 `...`。

3. **合理使用可选拆链**：访问嵌套属性时使用 `?.` 避免错误。

4. **封装私有属性**：使用闭包或 `#` 私有字段语法保护私有数据。

5. **使用 Object.freeze**：对于不应该修改的对象使用冻结。

6. **避免全局对象污染**：避免在全局对象上添加属性。

7. **使用 TypeScript**：在大型项目中使用 TypeScript 获得更好的类型安全。

8. **合理使用访问器**：使用 getter/setter 实现数据验证和计算属性。

## 总结

通过本文的学习，相信你已经对 JavaScript 对象操作有了更深入的理解。对象是 JavaScript 的核心，掌握对象的创建、操作、遍历和优化技巧对于编写高质量的 JavaScript 代码至关重要。在实际开发中，根据具体需求选择合适的方法，注意性能优化和代码可维护性。