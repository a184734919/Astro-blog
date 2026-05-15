---
title: JavaScript Set 和 Map
published: 2022-03-31
description: '新的数据结构 Set 和 Map的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript Set 和 Map 是 ES6 引入的两种新的数据结构，它们为 JavaScript 提供了更强大、更灵活的数据处理能力。Set 是值的集合，Map 是键值对的集合，它们都能保持插入顺序，并且提供了丰富的方法来操作数据。

## Set 数据结构

### 什么是 Set

Set 是值的集合，其中的值是唯一的，没有重复的值。Set 可以存储任何类型的值，无论是原始值还是对象引用。

### 创建 Set

```javascript
// 创建空 Set
const emptySet = new Set();

// 使用数组创建 Set
const setFromArray = new Set([1, 2, 3, 4, 5]);
console.log(setFromArray); // Set(5) {1, 2, 3, 4, 5}

// 使用字符串创建 Set
const setFromString = new Set('hello');
console.log(setFromString); // Set(4) {'h', 'e', 'l', 'o'}

// 自动去重
const setWithDuplicates = new Set([1, 2, 2, 3, 3, 3, 4]);
console.log(setWithDuplicates); // Set(4) {1, 2, 3, 4}

// 存储不同类型的值
const mixedSet = new Set([1, 'hello', { a: 1 }, [1, 2]]);
console.log(mixedSet); // Set(4) {1, 'hello', {a: 1}, [1, 2]}
```

### Set 基本方法

```javascript
const mySet = new Set();

// add(value) - 添加值
mySet.add(1);
mySet.add(2);
mySet.add(3);
mySet.add(2); // 重复添加，不会生效
console.log(mySet); // Set(3) {1, 2, 3}

// has(value) - 检查值是否存在
console.log(mySet.has(1)); // true
console.log(mySet.has(4)); // false

// delete(value) - 删除值
mySet.delete(2);
console.log(mySet); // Set(2) {1, 3}
console.log(mySet.has(2)); // false

// clear() - 清空 Set
mySet.clear();
console.log(mySet); // Set(0) {}

// size 属性 - 获取元素数量
const newSet = new Set([1, 2, 3, 4, 5]);
console.log(newSet.size); // 5
```

### Set 遍历方法

```javascript
const mySet = new Set(['a', 'b', 'c', 'd']);

// forEach() - 遍历每个元素
mySet.forEach((value, key, set) => {
  console.log(value, key); // 注意：Set 中 value 和 key 相同
});
// a a
// b b
// c c
// d d

// for...of 遍历
for (const value of mySet) {
  console.log(value);
}
// a
// b
// c
// d

// values() - 获取所有值的迭代器
const valuesIterator = mySet.values();
console.log([...valuesIterator]); // ['a', 'b', 'c', 'd']

// keys() - 获取所有键的迭代器（Set 中键和值相同）
const keysIterator = mySet.keys();
console.log([...keysIterator]); // ['a', 'b', 'c', 'd']

// entries() - 获取键值对迭代器
const entriesIterator = mySet.entries();
console.log([...entriesIterator]); // [['a', 'a'], ['b', 'b'], ['c', 'c'], ['d', 'd']]
```

### Set 转换为数组

```javascript
const mySet = new Set([1, 2, 3, 4, 5]);

// 转换为数组
const arr = [...mySet];
console.log(arr); // [1, 2, 3, 4, 5]

// 使用 Array.from()
const arr2 = Array.from(mySet);
console.log(arr2); // [1, 2, 3, 4, 5]

// 数组去重
const arrWithDuplicates = [1, 2, 2, 3, 3, 3, 4, 4, 5];
const uniqueArr = [...new Set(arrWithDuplicates)];
console.log(uniqueArr); // [1, 2, 3, 4, 5]

// 字符串去重
const str = 'hello world';
const uniqueChars = [...new Set(str)].join('');
console.log(uniqueChars); // 'helo wrd'
```

### Set 操作

```javascript
const setA = new Set([1, 2, 3, 4]);
const setB = new Set([3, 4, 5, 6]);

// 并集
const union = new Set([...setA, ...setB]);
console.log(union); // Set(6) {1, 2, 3, 4, 5, 6}

// 交集
const intersection = new Set([...setA].filter(x => setB.has(x)));
console.log(intersection); // Set(2) {3, 4}

// 差集 (A - B)
const difference = new Set([...setA].filter(x => !setB.has(x)));
console.log(difference); // Set(2) {1, 2}

// 对称差集 (A ∪ B) - (A ∩ B)
const symmetricDifference = new Set([
  ...[...setA].filter(x => !setB.has(x)),
  ...[...setB].filter(x => !setA.has(x))
]);
console.log(symmetricDifference); // Set(4) {1, 2, 5, 6}

// 判断子集
function isSubset(setA, setB) {
  return [...setA].every(x => setB.has(x));
}

console.log(isSubset(new Set([1, 2]), setA)); // true
console.log(isSubset(new Set([1, 6]), setA)); // false
```

### Set 的相等性

```javascript
// Set 使用 SameValueZero 算法判断相等性
const set = new Set();

// 原始值的比较
set.add(1);
set.add('1');
console.log(set); // Set(2) {1, '1'}

// NaN 的比较
set.add(NaN);
set.add(NaN);
console.log(set); // Set(3) {1, '1', NaN} // NaN 等于 NaN

// 对象的比较
const obj1 = { a: 1 };
const obj2 = { a: 1 };
set.add(obj1);
set.add(obj2);
console.log(set); // Set(5) {1, '1', NaN, {a: 1}, {a: 1}}

// 相同对象引用
const obj3 = obj1;
set.add(obj3);
console.log(set); // Set(5) {1, '1', NaN, {a: 1}, {a: 1}} // 没有添加
```

## Map 数据结构

### 什么是 Map

Map 是键值对的集合，其中的键可以是任意类型的值，包括对象。与 Object 不同，Map 的键不会转换为字符串，而是保持原始类型。

### 创建 Map

```javascript
// 创建空 Map
const emptyMap = new Map();

// 使用数组数组创建 Map
const mapFromArray = new Map([
  ['name', '张三'],
  ['age', 25],
  ['email', 'zhangsan@example.com']
]);
console.log(mapFromArray);
// Map(3) {'name' => '张三', 'age' => 25, 'email' => 'zhangsan@example.com'}

// 使用对象作为键
const objKey = { id: 1 };
const mapWithObjectKey = new Map([
  [objKey, '这是对象键'],
  ['stringKey', '这是字符串键']
]);
console.log(mapWithObjectKey.get(objKey)); // '这是对象键'

// 使用函数作为键
const funcKey = function() {};
const mapWithFuncKey = new Map([
  [funcKey, '函数键']
]);
console.log(mapWithFuncKey.get(funcKey)); // '函数键'
```

### Map 基本方法

```javascript
const myMap = new Map();

// set(key, value) - 设置键值对
myMap.set('name', '张三');
myMap.set('age', 25);
myMap.set('email', 'zhangsan@example.com');
console.log(myMap);
// Map(3) {'name' => '张三', 'age' => 25, 'email' => 'zhangsan@example.com'}

// get(key) - 获取值
console.log(myMap.get('name')); // '张三'
console.log(myMap.get('address')); // undefined

// has(key) - 检查键是否存在
console.log(myMap.has('age')); // true
console.log(myMap.has('phone')); // false

// delete(key) - 删除键值对
myMap.delete('email');
console.log(myMap); // Map(2) {'name' => '张三', 'age' => 25}
console.log(myMap.has('email')); // false

// clear() - 清空 Map
myMap.clear();
console.log(myMap); // Map(0) {}

// size 属性 - 获取键值对数量
const newMap = new Map([
  ['a', 1],
  ['b', 2],
  ['c', 3]
]);
console.log(newMap.size); // 3
```

### Map 遍历方法

```javascript
const myMap = new Map([
  ['name', '张三'],
  ['age', 25],
  ['email', 'zhangsan@example.com']
]);

// forEach() - 遍历每个键值对
myMap.forEach((value, key, map) => {
  console.log(`${key}: ${value}`);
});
// name: 张三
// age: 25
// email: zhangsan@example.com

// for...of 遍历
for (const [key, value] of myMap) {
  console.log(`${key}: ${value}`);
}

// keys() - 获取所有键的迭代器
const keysIterator = myMap.keys();
console.log([...keysIterator]); // ['name', 'age', 'email']

// values() - 获取所有值的迭代器
const valuesIterator = myMap.values();
console.log([...valuesIterator]); // ['张三', 25, 'zhangsan@example.com']

// entries() - 获取键值对迭代器
const entriesIterator = myMap.entries();
console.log([...entriesIterator]);
// [['name', '张三'], ['age', 25], ['email', 'zhangsan@example.com']]
```

### Map 与 Object 的对比

```javascript
// 创建 Object 和 Map
const obj = {
  name: '张三',
  age: 25,
  [Symbol('id')]: 123
};

const map = new Map([
  ['name', '张三'],
  ['age', 25],
  [Symbol('id'), 123]
]);

// 键的类型
console.log(obj[123]); // undefined (键会转换为字符串)
console.log(map.get(123)); // undefined

const numKey = 123;
map.set(numKey, 'number key');
console.log(map.get(123)); // 'number key'

// 键的顺序
const obj2 = { 2: 'two', 1: 'one' };
const map2 = new Map([[2, 'two'], [1, 'one']]);
console.log(Object.keys(obj2)); // ['1', '2'] (数字键按升序)
console.log([...map2.keys()]); // [2, 1] (保持插入顺序)

// 大小
console.log(Object.keys(obj).length); // 2
console.log(map.size); // 3

// 遍历
for (const key in obj) {
  console.log(key);
}
// name
// age

for (const [key, value] of map) {
  console.log(key, value);
}
// name 张三
// age 25
// Symbol(id) 123
```

### Map 的实际应用

```javascript
// 1. 频率统计
function countFrequency(arr) {
  const frequency = new Map();

  for (const item of arr) {
    frequency.set(item, (frequency.get(item) || 0) + 1);
  }

  return frequency;
}

const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];
const fruitCount = countFrequency(fruits);
console.log(fruitCount);
// Map(3) {'apple' => 3, 'banana' => 2, 'orange' => 1}

// 2. 缓存函数结果
const cache = new Map();

function expensiveCalculation(n) {
  if (cache.has(n)) {
    console.log('从缓存获取');
    return cache.get(n);
  }

  console.log('计算中...');
  const result = n * n;
  cache.set(n, result);
  return result;
}

console.log(expensiveCalculation(5)); // 计算中... 25
console.log(expensiveCalculation(5)); // 从缓存获取 25

// 3. 对象关联数据
const user1 = { id: 1, name: '张三' };
const user2 = { id: 2, name: '李四' };

const userData = new Map();
userData.set(user1, { role: 'admin', permissions: ['read', 'write'] });
userData.set(user2, { role: 'user', permissions: ['read'] });

console.log(userData.get(user1));
// { role: 'admin', permissions: ['read', 'write'] }

// 4. 双向映射
class BiDirectionalMap {
  constructor() {
    this.forward = new Map();
    this.backward = new Map();
  }

  set(key, value) {
    this.forward.set(key, value);
    this.backward.set(value, key);
  }

  get(key) {
    return this.forward.get(key);
  }

  getKey(value) {
    return this.backward.get(value);
  }
}

const bidiMap = new BiDirectionalMap();
bidiMap.set(1, 'one');
bidiMap.set(2, 'two');

console.log(bidiMap.get(1)); // 'one'
console.log(bidiMap.getKey('one')); // 1
```

## WeakSet 和 WeakMap

### WeakSet

WeakSet 是 Set 的弱引用版本，只能存储对象引用。WeakSet 中的对象引用是弱引用，不会阻止垃圾回收。

```javascript
// 创建 WeakSet
const weakSet = new WeakSet();

let obj1 = { name: '张三' };
let obj2 = { name: '李四' };

// 添加对象
weakSet.add(obj1);
weakSet.add(obj2);

console.log(weakSet.has(obj1)); // true
console.log(weakSet.has(obj2)); // true

// WeakSet 只能存储对象
// weakSet.add(1); // TypeError: Invalid value used in weak set

// 删除对象
weakSet.delete(obj1);
console.log(weakSet.has(obj1)); // false

// 弱引用 - 对象被回收后，WeakSet 中的引用也会被移除
obj2 = null; // 对象可以被垃圾回收

// WeakSet 的限制
// 1. 不能遍历
// 2. 没有 size 属性
// 3. 不能清空（没有 clear 方法）
// 4. 只能存储对象
```

### WeakMap

WeakMap 是 Map 的弱引用版本，键必须是对象。WeakMap 中的键是弱引用，不会阻止垃圾回收。

```javascript
// 创建 WeakMap
const weakMap = new WeakMap();

let key1 = { id: 1 };
let key2 = { id: 2 };

// 设置键值对
weakMap.set(key1, 'value1');
weakMap.set(key2, 'value2');

console.log(weakMap.get(key1)); // 'value1'
console.log(weakMap.has(key1)); // true

// WeakMap 的键必须是对象
// weakMap.set('key', 'value'); // TypeError: Invalid value used as weak map key

// 删除键值对
weakMap.delete(key1);
console.log(weakMap.has(key1)); // false

// WeakMap 的应用：私有数据
const privateData = new WeakMap();

class Person {
  constructor(name, age) {
    privateData.set(this, { name, age });
  }

  getInfo() {
    const data = privateData.get(this);
    return `${data.name}, ${data.age}岁`;
  }
}

const person = new Person('张三', 25);
console.log(person.getInfo()); // '张三, 25岁'

// WeakMap 的限制
// 1. 不能遍历
// 2. 没有 size 属性
// 3. 不能清空（没有 clear 方法）
// 4. 键必须是对象
```

## 实际应用

### 1. 数组去重

```javascript
// 简单数组去重
function uniqueArray(arr) {
  return [...new Set(arr)];
}

const numbers = [1, 2, 2, 3, 3, 3, 4, 4, 4, 4];
console.log(uniqueArray(numbers)); // [1, 2, 3, 4]

// 对象数组去重（基于某个属性）
function uniqueObjects(arr, key) {
  const seen = new Set();
  return arr.filter(item => {
    const keyValue = item[key];
    if (seen.has(keyValue)) {
      return false;
    }
    seen.add(keyValue);
    return true;
  });
}

const users = [
  { id: 1, name: '张三' },
  { id: 2, name: '李四' },
  { id: 1, name: '张三' },
  { id: 3, name: '王五' }
];

console.log(uniqueObjects(users, 'id'));
// [{ id: 1, name: '张三' }, { id: 2, name: '李四' }, { id: 3, name: '王五' }]
```

### 2. 数据分组

```javascript
// 使用 Map 分组
function groupBy(arr, key) {
  const groups = new Map();

  for (const item of arr) {
    const groupKey = item[key];
    if (!groups.has(groupKey)) {
      groups.set(groupKey, []);
    }
    groups.get(groupKey).push(item);
  }

  return Object.fromEntries(groups);
}

const students = [
  { name: '张三', grade: 'A' },
  { name: '李四', grade: 'B' },
  { name: '王五', grade: 'A' },
  { name: '赵六', grade: 'C' }
];

const grouped = groupBy(students, 'grade');
console.log(grouped);
// {
//   A: [{ name: '张三', grade: 'A' }, { name: '王五', grade: 'A' }],
//   B: [{ name: '李四', grade: 'B' }],
//   C: [{ name: '赵六', grade: 'C' }]
// }
```

### 3. 缓存实现

```javascript
// 简单的 LRU 缓存
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) {
      return -1;
    }

    // 重新插入以更新顺序
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);

    return value;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // 删除最旧的项（第一个）
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }

    this.cache.set(key, value);
  }
}

const cache = new LRUCache(2);
cache.put(1, 'one');
cache.put(2, 'two');
console.log(cache.get(1)); // 'one'
cache.put(3, 'three'); // 删除 key 2
console.log(cache.get(2)); // -1
```

### 4. 图的遍历

```javascript
// 使用 Map 和 Set 实现图的遍历
class Graph {
  constructor() {
    this.vertices = new Map();
  }

  addVertex(vertex) {
    if (!this.vertices.has(vertex)) {
      this.vertices.set(vertex, new Set());
    }
  }

  addEdge(vertex1, vertex2) {
    this.addVertex(vertex1);
    this.addVertex(vertex2);
    this.vertices.get(vertex1).add(vertex2);
    this.vertices.get(vertex2).add(vertex1);
  }

  bfs(start) {
    const visited = new Set();
    const queue = [start];
    const result = [];

    visited.add(start);

    while (queue.length > 0) {
      const vertex = queue.shift();
      result.push(vertex);

      for (const neighbor of this.vertices.get(vertex)) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }

    return result;
  }
}

const graph = new Graph();
graph.addEdge('A', 'B');
graph.addEdge('A', 'C');
graph.addEdge('B', 'D');
graph.addEdge('C', 'D');

console.log(graph.bfs('A')); // ['A', 'B', 'C', 'D']
```

### 5. 模板引擎

```javascript
// 使用 Map 存储模板变量
const templateCache = new Map();

function compileTemplate(template, data) {
  const cacheKey = template;

  if (templateCache.has(cacheKey)) {
    const fn = templateCache.get(cacheKey);
    return fn(data);
  }

  // 简单的模板编译
  const regex = /\{\{(\w+)\}\}/g;
  const fn = new Function('data', `
    return '${template.replace(regex, "' + (data['$1'] || '') + '")}'
  `);

  templateCache.set(cacheKey, fn);
  return fn(data);
}

const template = 'Hello, {{name}}! You are {{age}} years old.';
const data = { name: '张三', age: 25 };

console.log(compileTemplate(template, data)); // 'Hello, 张三! You are 25 years old.'
```

## 性能考虑

### Set vs Array

```javascript
// 查找性能比较
const largeSet = new Set(Array.from({ length: 100000 }, (_, i) => i));
const largeArray = Array.from({ length: 100000 }, (_, i) => i);

// Set 查找 - O(1)
console.time('Set查找');
largeSet.has(99999);
console.timeEnd('Set查找');

// Array 查找 - O(n)
console.time('Array查找');
largeArray.includes(99999);
console.timeEnd('Array查找');

// 添加元素
console.time('Set添加');
const testSet = new Set();
for (let i = 0; i < 100000; i++) {
  testSet.add(i);
}
console.timeEnd('Set添加');

console.time('Array添加');
const testArray = [];
for (let i = 0; i < 100000; i++) {
  testArray.push(i);
}
console.timeEnd('Array添加');
```

### Map vs Object

```javascript
// 性能比较
const largeMap = new Map();
const largeObj = {};

for (let i = 0; i < 100000; i++) {
  largeMap.set(i, `value${i}`);
  largeObj[i] = `value${i}`;
}

// Map 查找
console.time('Map查找');
largeMap.get(99999);
console.timeEnd('Map查找');

// Object 查找
console.time('Object查找');
largeObj[99999];
console.timeEnd('Object查找');

// Map 添加
console.time('Map添加');
const testMap = new Map();
for (let i = 0; i < 100000; i++) {
  testMap.set(i, `value${i}`);
}
console.timeEnd('Map添加');

// Object 添加
console.time('Object添加');
const testObj = {};
for (let i = 0; i < 100000; i++) {
  testObj[i] = `value${i}`;
}
console.timeEnd('Object添加');
```

## 注意事项

1. **Set 的唯一性**：Set 使用 SameValueZero 算法判断相等性，注意 NaN 的特殊性。

2. **Map 的键类型**：Map 的键可以是任意类型，包括对象和函数。

3. **WeakSet 和 WeakMap 的限制**：不能遍历、没有 size 属性、只能存储对象。

4. **内存泄漏**：WeakSet 和 WeakMap 可以避免内存泄漏，因为它们使用弱引用。

5. **性能权衡**：对于小型数据集，Object 和 Array 可能更快；对于大型数据集，Set 和 Map 性能更好。

6. **插入顺序**：Set 和 Map 保持插入顺序，而 Object 的键顺序在某些情况下可能不可预测。

7. **类型转换**：Object 的键会隐式转换为字符串，而 Map 不会。

8. **Symbol 键**：Map 可以使用 Symbol 作为键，但要注意不同的 Symbol 实例不相等。

## 最佳实践

1. **需要唯一值时使用 Set**：当需要存储唯一值集合时，优先使用 Set。

2. **需要键值对时使用 Map**：当需要频繁添加、删除、查找键值对时，使用 Map。

3. **对象作为键时使用 Map**：当需要使用对象作为键时，必须使用 Map。

4. **避免内存泄漏时使用 WeakMap**：当需要关联数据但不希望阻止垃圾回收时，使用 WeakMap。

5. **保持插入顺序时使用 Map**：当需要保持键值对的插入顺序时，使用 Map。

6. **数据去重时使用 Set**：使用 Set 可以轻松实现数组或字符串去重。

7. **缓存计算结果时使用 Map**：使用 Map 缓存函数结果可以提高性能。

8. **简单数据使用 Object**：对于简单的静态数据，Object 可能更合适。

## 总结

通过本文的学习，相信你已经对 JavaScript Set 和 Map 有了更深入的理解。Set 和 Map 是 ES6 引入的重要数据结构，它们提供了更强大、更灵活的数据处理能力。在实际开发中，根据具体需求选择合适的数据结构，可以大大提高代码的质量和性能。记住每种数据结构的特点和适用场景，合理运用 Set、Map、WeakSet 和 WeakMap，编写出更优雅、更高效的代码。