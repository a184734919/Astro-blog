---
title: JavaScript 深拷贝与浅拷贝
published: 2022-07-24
description: '对象拷贝的实现方法的详细介绍和学习笔记'
image: ''
tags: ["JS"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

在 JavaScript 中,对象是引用类型。直接赋值只是复制引用,而不是对象本身。深拷贝和浅拷贝是两种不同的对象复制方式,各有适用的场景。

## 基本概念

### 赋值 vs 浅拷贝 vs 深拷贝

```javascript
// 1. 赋值: 只是复制引用
const original = { name: 'Alice', age: 25 };
const assigned = original;

assigned.name = 'Bob';
console.log(original.name); // 'Bob' - 原对象被修改

// 2. 浅拷贝: 复制一层对象
const original = { name: 'Alice', details: { age: 25 } };
const shallow = { ...original };

shallow.name = 'Bob';
shallow.details.age = 26;
console.log(original.name); // 'Alice' - 基本类型不受影响
console.log(original.details.age); // 26 - 嵌套对象被修改

// 3. 深拷贝: 完全复制对象及其嵌套对象
const original = { name: 'Alice', details: { age: 25 } };
const deep = JSON.parse(JSON.stringify(original));

deep.name = 'Bob';
deep.details.age = 26;
console.log(original.name); // 'Alice'
console.log(original.details.age); // 25 - 原对象完全不受影响
```

## 浅拷贝方法

### 1. 展开运算符 (Spread Operator)

```javascript
const original = { a: 1, b: 2, c: 3 };
const copy = { ...original };

console.log(copy); // { a: 1, b: 2, c: 3 }
console.log(original === copy); // false - 是新对象

// 合并对象
const obj1 = { a: 1 };
const obj2 = { b: 2 };
const merged = { ...obj1, ...obj2 }; // { a: 1, b: 2 }
```

### 2. Object.assign()

```javascript
const original = { a: 1, b: 2 };
const copy = Object.assign({}, original);

console.log(copy); // { a: 1, b: 2 }
console.log(original === copy); // false

// 合并多个对象
const merged = Object.assign({}, obj1, obj2, obj3);

// 注意: 会修改目标对象
const target = {};
Object.assign(target, { a: 1 });
console.log(target); // { a: 1 }
```

### 3. Array.from() 和 slice()

```javascript
// 数组的浅拷贝
const original = [1, 2, 3, 4, 5];

// 方法1: slice()
const copy1 = original.slice();
console.log(copy1); // [1, 2, 3, 4, 5]

// 方法2: Array.from()
const copy2 = Array.from(original);
console.log(copy2); // [1, 2, 3, 4, 5]

// 方法3: 展开运算符
const copy3 = [...original];
console.log(copy3); // [1, 2, 3, 4, 5]
```

### 4. 数组 map/filter 等方法

```javascript
const original = [1, 2, 3, 4, 5];

// map 返回新数组
const mapped = original.map(x => x);
console.log(mapped); // [1, 2, 3, 4, 5]

// filter 返回新数组
const filtered = original.filter(() => true);
console.log(filtered); // [1, 2, 3, 4, 5]
```

## 深拷贝方法

### 1. JSON.parse(JSON.stringify())

```javascript
const original = {
  name: 'Alice',
  age: 25,
  hobbies: ['reading', 'swimming'],
  address: {
    city: 'Beijing',
    country: 'China'
  }
};

const copy = JSON.parse(JSON.stringify(original));

copy.name = 'Bob';
copy.hobbies.push('coding');
copy.address.city = 'Shanghai';

console.log(original.name); // 'Alice'
console.log(original.hobbies); // ['reading', 'swimming']
console.log(original.address.city); // 'Beijing'
```

**限制:**

```javascript
// 1. 无法复制函数
const obj1 = {
  name: 'Alice',
  greet: function() { console.log('Hello'); }
};
const copy1 = JSON.parse(JSON.stringify(obj1));
console.log(copy1.greet); // undefined

// 2. 无法复制 undefined
const obj2 = {
  name: 'Alice',
  age: undefined
};
const copy2 = JSON.parse(JSON.stringify(obj2));
console.log(copy2.age); // null

// 3. 无法复制 Symbol
const obj3 = {
  name: 'Alice',
  [Symbol('id')]: 123
};
const copy3 = JSON.parse(JSON.stringify(obj3));
console.log(Object.keys(copy3)); // ['name']

// 4. 无法复制循环引用
const obj4 = { name: 'Alice' };
obj4.self = obj4;
const copy4 = JSON.parse(JSON.stringify(obj4)); // TypeError
```

### 2. 结构化克隆算法

```javascript
// 使用 MessageChannel
function structuredClone(obj) {
  return new Promise(resolve => {
    const { port1, port2 } = new MessageChannel();
    port2.onmessage = event => resolve(event.data);
    port1.postMessage(obj);
  });
}

// 使用示例
const original = {
  name: 'Alice',
  date: new Date(),
  buffer: new ArrayBuffer(8),
  regex: /pattern/g
};

structuredClone(original).then(copy => {
  console.log(copy);
  // 支持更多类型: Date, ArrayBuffer, RegExp, Map, Set 等
});

// 或者使用浏览器原生 API (如果可用)
const copy = structuredClone(original);
```

### 3. 递归实现深拷贝

```javascript
function deepClone(obj, map = new WeakMap()) {
  // 处理基本类型和 null/undefined
  if (obj === null || typeof obj !== 'object') {
    return obj;
  }
  
  // 处理循环引用
  if (map.has(obj)) {
    return map.get(obj);
  }
  
  // 处理日期
  if (obj instanceof Date) {
    return new Date(obj);
  }
  
  // 处理正则表达式
  if (obj instanceof RegExp) {
    return new RegExp(obj);
  }
  
  // 处理 Map
  if (obj instanceof Map) {
    const copy = new Map();
    map.set(obj, copy);
    obj.forEach((value, key) => {
      copy.set(deepClone(key, map), deepClone(value, map));
    });
    return copy;
  }
  
  // 处理 Set
  if (obj instanceof Set) {
    const copy = new Set();
    map.set(obj, copy);
    obj.forEach(value => {
      copy.add(deepClone(value, map));
    });
    return copy;
  }
  
  // 处理数组
  if (Array.isArray(obj)) {
    const copy = [];
    map.set(obj, copy);
    for (let i = 0; i < obj.length; i++) {
      copy[i] = deepClone(obj[i], map);
    }
    return copy;
  }
  
  // 处理对象
  const copy = Object.create(Object.getPrototypeOf(obj));
  map.set(obj, copy);
  
  for (const key in obj) {
    if (obj.hasOwnProperty(key)) {
      copy[key] = deepClone(obj[key], map);
    }
  }
  
  return copy;
}

// 使用示例
const original = {
  name: 'Alice',
  date: new Date(),
  regex: /pattern/g,
  nested: { value: 123 },
  arr: [1, 2, 3]
};

// 创建循环引用
original.self = original;

const copy = deepClone(original);
console.log(copy); // 完整拷贝,包括循环引用
```

### 4. 使用 Lodash 库

```javascript
import _ from 'lodash';

const original = {
  name: 'Alice',
  details: {
    age: 25,
    address: {
      city: 'Beijing'
    }
  }
};

const copy = _.cloneDeep(original);

copy.details.address.city = 'Shanghai';
console.log(original.details.address.city); // 'Beijing'
```

## 性能比较

```javascript
// 性能测试
function testPerformance() {
  const largeObject = {
    data: Array(10000).fill(null).map((_, i) => ({
      id: i,
      nested: { value: i * 2 }
    }))
  };
  
  // 1. JSON.parse(JSON.stringify())
  console.time('JSON');
  const jsonCopy = JSON.parse(JSON.stringify(largeObject));
  console.timeEnd('JSON');
  
  // 2. 手动递归
  console.time('Recursive');
  const recursiveCopy = deepClone(largeObject);
  console.timeEnd('Recursive');
  
  // 3. Lodash
  console.time('Lodash');
  const lodashCopy = _.cloneDeep(largeObject);
  console.timeEnd('Lodash');
}

// 一般来说:
// - JSON 方法: 最快,但功能有限
// - Lodash: 功能全面,性能较好
// - 手动实现: 最灵活,可根据需求优化
```

## 实际应用场景

### 1. Redux 状态更新

```javascript
// 使用展开运算符进行浅拷贝
function reducer(state, action) {
  switch (action.type) {
    case 'UPDATE_USER':
      return {
        ...state,
        user: {
          ...state.user,
          name: action.payload.name
        }
      };
    default:
      return state;
  }
}
```

### 2. 表单数据处理

```javascript
// 复制表单初始值
const initialForm = {
  name: '',
  email: '',
  password: '',
  preferences: {
    newsletter: true,
    notifications: false
  }
};

// 深拷贝以避免修改初始值
const formData = deepClone(initialForm);

// 用户修改表单
formData.name = 'Alice';
formData.preferences.newsletter = false;

// 重置表单
function resetForm() {
  Object.assign(formData, deepClone(initialForm));
}
```

### 3. 缓存实现

```javascript
class Cache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, value) {
    // 深拷贝存储,避免外部修改
    this.cache.set(key, deepClone(value));
  }
  
  get(key) {
    // 深拷贝返回,避免缓存被修改
    const value = this.cache.get(key);
    return value ? deepClone(value) : undefined;
  }
}
```

### 4. 不可变数据操作

```javascript
// 使用浅拷贝创建新的不可变对象
const state = {
  users: [
    { id: 1, name: 'Alice', active: true },
    { id: 2, name: 'Bob', active: false }
  ],
  loading: false
};

// 更新用户状态
const newState = {
  ...state,
  users: state.users.map(user =>
    user.id === 1 ? { ...user, active: false } : user
  )
};

console.log(state.users[0].active); // true
console.log(newState.users[0].active); // false
```

## 最佳实践

### 1. 选择合适的方法

```javascript
// 简单对象: 使用展开运算符
const simpleCopy = { ...original };

// 需要函数/undefined/Symbol: 使用深拷贝库
const complexCopy = _.cloneDeep(original);

// 需要循环引用: 使用 WeakMap 的递归实现
const cyclicCopy = deepClone(original);
```

### 2. 考虑性能

```javascript
// 避免不必要的深拷贝
function processData(data) {
  // 如果不需要修改原始数据,直接使用
  const result = data.map(item => item.value * 2);
  return result;
}

// 只在需要修改时才拷贝
function updateData(data, id, newValue) {
  const copy = [...data];
  const index = copy.findIndex(item => item.id === id);
  if (index !== -1) {
    copy[index] = { ...copy[index], value: newValue };
  }
  return copy;
}
```

### 3. 使用不可变库

```javascript
import { produce } from 'immer';

// 使用 Immer 简化不可变操作
const nextState = produce(currentState, draft => {
  draft.users.push({ id: 3, name: 'Charlie' });
  draft.users[0].name = 'Alice Updated';
});

// Immer 内部处理了所有拷贝逻辑
```

### 4. 自定义拷贝逻辑

```javascript
// 针对特定场景优化拷贝
function customClone(obj) {
  if (!obj || typeof obj !== 'object') {
    return obj;
  }
  
  // 针对特定类型特殊处理
  if (obj.isClonable) {
    return obj.clone();
  }
  
  // 默认行为
  return deepClone(obj);
}
```

## 常见陷阱

### 1. 误以为 = 是拷贝

```javascript
const obj1 = { a: 1 };
const obj2 = obj1; // 这不是拷贝,是引用赋值
obj2.a = 2;
console.log(obj1.a); // 2 - 原对象被修改
```

### 2. 忘记处理特殊类型

```javascript
// JSON 方法会丢失函数
const obj = {
  name: 'Alice',
  greet: () => console.log('Hello')
};

const copy = JSON.parse(JSON.stringify(obj));
copy.greet(); // TypeError: copy.greet is not a function
```

### 3. 循环引用导致错误

```javascript
const obj = { name: 'Alice' };
obj.self = obj;

const copy = JSON.parse(JSON.stringify(obj)); // TypeError
// 需要使用支持循环引用的深拷贝方法
```

### 4. 性能问题

```javascript
// 避免对大对象频繁深拷贝
function processLargeData(data) {
  // 不好的做法: 每次都深拷贝
  const copy = deepClone(data);
  // 处理 copy
}

// 更好的做法: 只在必要时拷贝
function processLargeData(data) {
  // 直接处理,如果需要修改再考虑拷贝
  return processData(data);
}
```

## 总结

深拷贝与浅拷贝的选择要点:

1. **浅拷贝**: 使用展开运算符、Object.assign 等,适合简单对象
2. **深拷贝**: 使用 JSON 方法、递归实现、Lodash 等,适合嵌套对象
3. **方法选择**: 根据数据类型和性能需求选择合适的方法
4. **性能考虑**: 避免不必要的拷贝,只在实际需要时才执行
5. **特殊处理**: 注意函数、循环引用、Symbol 等特殊情况
6. **实际应用**: 在状态管理、表单处理、缓存等场景中合理使用
7. **最佳实践**: 结合不可变数据模式,使用专门的库简化操作

理解深浅拷贝的原理和应用场景,有助于编写更安全、更高效的 JavaScript 代码。