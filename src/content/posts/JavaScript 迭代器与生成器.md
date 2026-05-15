---
title: JavaScript 迭代器与生成器
published: 2022-06-22
description: 'Iterator 和 Generator 的原理的详细介绍和学习笔记'
image: ''
tags: ["JS"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 迭代器与生成器是 ES6 引入的重要特性，提供了一种统一的方式来遍历数据结构和生成序列。迭代器协议定义了标准的遍历接口，而生成器函数则提供了一种更简洁、更强大的方式来创建迭代器。

## 迭代器

### 什么是迭代器

迭代器是一种特殊的对象，它实现了迭代器协议，能够按照顺序访问集合中的元素。

### 迭代器协议

迭代器协议包含两个核心协议：

#### 可迭代协议

一个对象是可迭代的，如果它实现了 `Symbol.iterator` 方法，该方法返回一个迭代器对象。

```javascript
// 可迭代协议
const myIterable = {
  [Symbol.iterator]() {
    return {
      // 返回迭代器对象
    };
  }
};

// 检查对象是否可迭代
console.log(typeof myIterable[Symbol.iterator]); // 'function'
```

#### 迭代器协议

迭代器对象必须实现 `next()` 方法，该方法返回包含 `value` 和 `done` 属性的对象。

```javascript
const iterator = {
  next() {
    return {
      value: 'next value',
      done: false
    };
  }
};
```

### 创建迭代器

```javascript
// 手动创建迭代器
function createIterator(items) {
  let index = 0;

  return {
    [Symbol.iterator]() {
      return this;
    },
    next() {
      if (index < items.length) {
        return {
          value: items[index++],
          done: false
        };
      } else {
        return {
          value: undefined,
          done: true
        };
      }
    }
  };
}

// 使用
const iterator = createIterator([1, 2, 3]);

console.log(iterator.next()); // { value: 1, done: false }
console.log(iterator.next()); // { value: 2, done: false }
console.log(iterator.next()); // { value: 3, done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

### for...of 循环

`for...of` 循环是使用可迭代对象的最简单方式：

```javascript
// 使用 for...of 遍历可迭代对象
const myIterable = createIterator([1, 2, 3]);

for (const item of myIterable) {
  console.log(item);
}
// 输出: 1, 2, 3

// 遍历数组
const array = [1, 2, 3];
for (const item of array) {
  console.log(item);
}

// 遍历字符串
const str = 'hello';
for (const char of str) {
  console.log(char);
}

// 遍历 Map
const map = new Map([['a', 1], ['b', 2]]);
for (const [key, value] of map) {
  console.log(key, value);
}

// 遍历 Set
const set = new Set([1, 2, 3]);
for (const item of set) {
  console.log(item);
}
```

### 内置可迭代对象

JavaScript 中许多内置对象都实现了可迭代协议：

```javascript
// Array
const arr = [1, 2, 3];
console.log(arr[Symbol.iterator]()); // Array Iterator

// String
const str = 'hello';
console.log(str[Symbol.iterator]()); // String Iterator

// Map
const map = new Map();
console.log(map[Symbol.iterator]()); // Map Iterator

// Set
const set = new Set();
console.log(set[Symbol.iterator]()); // Set Iterator

// TypedArray
const typedArray = new Uint8Array([1, 2, 3]);
console.log(typedArray[Symbol.iterator]()); // Uint8Array Iterator

// arguments 对象
function testArgs() {
  console.log(arguments[Symbol.iterator]()); // Arguments Iterator
}
testArgs(1, 2, 3);

// NodeList
const nodeList = document.querySelectorAll('div');
console.log(nodeList[Symbol.iterator]()); // NodeList Iterator
```

### 解构赋值和展开运算符

可迭代对象可以与解构赋值和展开运算符配合使用：

```javascript
// 解构赋值
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// 展开运算符
const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];
console.log(arr2); // [1, 2, 3, 4, 5]

// 可迭代对象的展开
const set = new Set([1, 2, 3]);
const arr = [...set];
console.log(arr); // [1, 2, 3]

// Map 展开为键值对数组
const map = new Map([['a', 1], ['b', 2]]);
const mapArr = [...map];
console.log(mapArr); // [['a', 1], ['b', 2]]
```

## 生成器

### 什么是生成器

生成器是一种特殊的函数，可以暂停执行并在稍后恢复。使用 `function*` 语法定义，使用 `yield` 关键字产生值。

### 基本语法

```javascript
// 定义生成器函数
function* myGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

// 创建生成器对象
const gen = myGenerator();

// 使用 next() 获取值
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

### yield 表达式

```javascript
function* generator() {
  console.log('开始');
  const a = yield 1;
  console.log('收到:', a);
  const b = yield 2;
  console.log('收到:', b);
  return '结束';
}

const gen = generator();

console.log(gen.next()); // 开始, { value: 1, done: false }
console.log(gen.next('Hello')); // 收到: Hello, { value: 2, done: false }
console.log(gen.next('World')); // 收到: World, { value: '结束', done: true }
```

### yield* 表达式

`yield*` 用于委托给另一个可迭代对象：

```javascript
// 嵌套生成器
function* innerGenerator() {
  yield 'a';
  yield 'b';
}

function* outerGenerator() {
  yield 'x';
  yield* innerGenerator();
  yield 'y';
}

for (const item of outerGenerator()) {
  console.log(item);
}
// 输出: x, a, b, y

// yield* 与数组的配合
function* arrayGenerator() {
  yield* [1, 2, 3];
  yield* 'abc';
}

for (const item of arrayGenerator()) {
  console.log(item);
}
// 输出: 1, 2, 3, a, b, c
```

### 生成器的方法

#### return() 方法

`return()` 方法可以提前终止生成器：

```javascript
function* generator() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = generator();

console.log(gen.next()); // { value: 1, done: false }
console.log(gen.return('finished')); // { value: 'finished', done: true }
console.log(gen.next()); // { value: undefined, done: true }
```

#### throw() 方法

`throw()` 方法可以在生成器内部抛出异常：

```javascript
function* generator() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (e) {
    console.log('捕获异常:', e);
    yield 'error';
  }
}

const gen = generator();

console.log(gen.next()); // { value: 1, done: false }
console.log(gen.throw(new Error('出错了')));
// 捕获异常: Error: 出错了
// { value: 'error', done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

## 生成器的应用场景

### 1. 创建无限序列

```javascript
// 生成无限斐波那契数列
function* fibonacci() {
  let [prev, curr] = [0, 1];
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

const fib = fibonacci();
console.log(fib.next().value); // 1
console.log(fib.next().value); // 1
console.log(fib.next().value); // 2
console.log(fib.next().value); // 3
console.log(fib.next().value); // 5

// 生成无限自然数
function* naturalNumbers() {
  let n = 1;
  while (true) {
    yield n++;
  }
}

// 取前10个
const naturals = naturalNumbers();
const firstTen = Array.from({ length: 10 }, () => naturals.next().value);
console.log(firstTen); // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
```

### 2. 异步流程控制

```javascript
// 使用生成器简化异步操作
function* fetchUser() {
  const response = yield fetch('/api/user');
  const user = yield response.json();
  return user;
}

// 生成器执行器
function runGenerator(generator) {
  const gen = generator();

  function step(value) {
    const result = gen.next(value);

    if (result.done) {
      return result.value;
    }

    return Promise.resolve(result.value).then(step);
  }

  return step();
}

// 使用
runGenerator(fetchUser()).then(user => {
  console.log(user);
});

// 更复杂的异步流程
function* loadDashboard() {
  const user = yield fetch('/api/user').then(r => r.json());
  const posts = yield fetch(`/api/posts/${user.id}`).then(r => r.json());
  const comments = yield fetch(`/api/comments/${posts[0].id}`).then(r => r.json());
  
  return { user, posts, comments };
}

runGenerator(loadDashboard()).then(data => {
  console.log(data);
});
```

### 3. 状态管理

```javascript
// 简单的状态机
function* stateMachine() {
  let state = 'idle';

  while (true) {
    const action = yield state;

    switch (state) {
      case 'idle':
        if (action === 'start') {
          state = 'running';
        }
        break;
      case 'running':
        if (action === 'pause') {
          state = 'paused';
        } else if (action === 'stop') {
          state = 'idle';
        }
        break;
      case 'paused':
        if (action === 'resume') {
          state = 'running';
        } else if (action === 'stop') {
          state = 'idle';
        }
        break;
    }
  }
}

const machine = stateMachine();
console.log(machine.next()); // { value: 'idle', done: false }
console.log(machine.next('start')); // { value: 'running', done: false }
console.log(machine.next('pause')); // { value: 'paused', done: false }
console.log(machine.next('resume')); // { value: 'running', done: false }
console.log(machine.next('stop')); // { value: 'idle', done: false }
```

### 4. 树的遍历

```javascript
// 树结构遍历
class TreeNode {
  constructor(value, children = []) {
    this.value = value;
    this.children = children;
  }
}

// 深度优先遍历
function* dfs(node) {
  yield node.value;
  for (const child of node.children) {
    yield* dfs(child);
  }
}

// 广度优先遍历
function* bfs(root) {
  const queue = [root];
  
  while (queue.length > 0) {
    const node = queue.shift();
    yield node.value;
    queue.push(...node.children);
  }
}

// 创建树
const tree = new TreeNode('A', [
  new TreeNode('B', [
    new TreeNode('D'),
    new TreeNode('E')
  ]),
  new TreeNode('C', [
    new TreeNode('F')
  ])
]);

console.log([...dfs(tree)]); // ['A', 'B', 'D', 'E', 'C', 'F']
console.log([...bfs(tree)]); // ['A', 'B', 'C', 'D', 'E', 'F']
```

### 5. 数据分页加载

```javascript
// 模拟分页数据加载
function* paginatedData(fetchFn, pageSize = 10) {
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const data = yield fetchFn(page, pageSize);
    
    if (data && data.length > 0) {
      yield* data;
      page++;
      hasMore = data.length === pageSize;
    } else {
      hasMore = false;
    }
  }
}

// 模拟 API
function fetchPage(page, size) {
  return new Promise(resolve => {
    setTimeout(() => {
      const total = 25;
      const start = (page - 1) * size;
      const data = [];

      for (let i = start; i < Math.min(start + size, total); i++) {
        data.push(`Item ${i + 1}`);
      }

      resolve(data);
    }, 100);
  });
}

// 使用
async function loadData() {
  const generator = paginatedData(fetchPage, 10);
  
  // 启动生成器
  generator.next();
  
  const allData = [];
  
  while (true) {
    const result = await generator.next();
    
    if (result.done) {
      break;
    }
    
    if (Array.isArray(result.value)) {
      // 分页数据
      allData.push(...result.value);
    } else {
      // Promise
      const nextResult = await generator.next();
      if (nextResult.done) break;
    }
  }
  
  console.log('所有数据:', allData);
}
```

## 异步迭代器和异步生成器

### 异步迭代器

异步迭代器是迭代器的异步版本，返回 Promise：

```javascript
// 异步迭代器对象
const asyncIterator = {
  [Symbol.asyncIterator]() {
    let i = 0;
    const items = [1, 2, 3, 4, 5];
    
    return {
      next() {
        return new Promise(resolve => {
          setTimeout(() => {
            if (i < items.length) {
              resolve({
                value: items[i++],
                done: false
              });
            } else {
              resolve({
                value: undefined,
                done: true
              });
            }
          }, 100);
        });
      }
    };
  }
};

// 使用 for-await-of
(async () => {
  for await (const item of asyncIterator) {
    console.log(item);
  }
})();
```

### 异步生成器

异步生成器函数使用 `async function*` 语法：

```javascript
// 异步生成器
async function* asyncGenerator() {
  yield Promise.resolve(1);
  await new Promise(resolve => setTimeout(resolve, 100));
  yield Promise.resolve(2);
  yield Promise.resolve(3);
}

// 使用 for-await-of
(async () => {
  for await (const value of asyncGenerator()) {
    console.log(value);
  }
})();

// 异步生成器的实际应用
async function* fetchUsers(ids) {
  for (const id of ids) {
    const response = await fetch(`/api/users/${id}`);
    const user = await response.json();
    yield user;
  }
}

// 使用
(async () => {
  const userIds = [1, 2, 3];
  
  for await (const user of fetchUsers(userIds)) {
    console.log(user);
  }
})();
```

### 实际应用场景

```javascript
// 1. 分页获取数据
async function* fetchPaginatedData(url, limit = 10) {
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`${url}?page=${page}&limit=${limit}`);
    const data = await response.json();

    yield* data.items;

    hasMore = data.hasMore;
    page++;
  }
}

// 使用
(async () => {
  for await (const item of fetchPaginatedData('/api/items')) {
    console.log(item);
  }
})();

// 2. 文件流处理
async function* readLines(file) {
  const reader = file.stream().getReader();
  const decoder = new TextDecoder();
  let buffer = '';

  while (true) {
    const { done, value } = await reader.read();

    if (done) break;

    buffer += decoder.decode(value, { stream: true });
    const lines = buffer.split('\n');
    buffer = lines.pop();

    for (const line of lines) {
      if (line) yield line;
    }
  }

  if (buffer) yield buffer;
}

// 3. 数据库游标
async function* queryWithCursor(query, batchSize = 100) {
  let cursor = null;

  while (true) {
    const result = await query.execute({ cursor, limit: batchSize });
    const { data, nextCursor } = result;

    yield* data;

    if (!nextCursor || data.length === 0) {
      break;
    }

    cursor = nextCursor;
  }
}
```

## 实际应用案例

### 1. 实现自定义数据结构

```javascript
// 自定义范围类
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  [Symbol.iterator]() {
    let current = this.start;
    return {
      next: () => {
        if (current >= this.end) {
          return { done: true, value: undefined };
        }
        const value = current;
        current += this.step;
        return { done: false, value };
      }
    };
  }
}

// 使用
const range = new Range(0, 10, 2);
console.log([...range]); // [0, 2, 4, 6, 8]

// 生成器版本
class Range2 {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }

  *[Symbol.iterator]() {
    for (let i = this.start; i < this.end; i += this.step) {
      yield i;
    }
  }
}
```

### 2. 懒加载数据

```javascript
// 懒加载大型数据集
class LazyArray {
  constructor(source) {
    this.source = source;
  }

  *[Symbol.iterator]() {
    for (const item of this.source) {
      yield item;
    }
  }

  map(fn) {
    return new LazyArray(this._map(fn));
  }

  filter(fn) {
    return new LazyArray(this._filter(fn));
  }

  take(n) {
    return new LazyArray(this._take(n));
  }

  *[Symbol.iterator]() {
    for (const item of this.source) {
      yield item;
    }
  }

  // 辅助方法
  *_map(fn) {
    for (const item of this.source) {
      yield fn(item);
    }
  }

  *_filter(fn) {
    for (const item of this.source) {
      if (fn(item)) yield item;
    }
  }

  *_take(n) {
    let count = 0;
    for (const item of this.source) {
      if (count >= n) break;
      yield item;
      count++;
    }
  }
}

// 使用
const numbers = new LazyArray([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);
const result = numbers
  .map(x => x * 2)
  .filter(x => x > 10)
  .take(3);

console.log([...result]); // [12, 14, 16]
```

### 3. 实现观察者模式

```javascript
// 使用生成器实现观察者
function* observable() {
  const observers = [];
  
  while (true) {
    const action = yield;

    if (action.type === 'subscribe') {
      observers.push(action.observer);
    } else if (action.type === 'next') {
      observers.forEach(observer => observer(action.value));
    } else if (action.type === 'unsubscribe') {
      const index = observers.indexOf(action.observer);
      if (index > -1) {
        observers.splice(index, 1);
      }
    }
  }
}

// 使用
const obs = observable();
obs.next();

// 订阅观察者
obs.next({
  type: 'subscribe',
  observer: (value) => console.log('Observer 1:', value)
});

obs.next({
  type: 'subscribe',
  observer: (value) => console.log('Observer 2:', value)
});

// 发送数据
obs.next({
  type: 'next',
  value: 'Hello'
});

// 输出:
// Observer 1: Hello
// Observer 2: Hello
```

## 注意事项

1. **生成器的状态保持**：生成器函数会保持内部状态，不要在不需要时创建过多的生成器。

2. **内存泄漏**：未完成的生成器会持有作用域中的变量，确保正确完成或终止。

3. **异步生成器的错误处理**：异步生成器中的错误需要特别处理，使用 try-catch。

4. **for...of vs for...in**：注意区分 `for...of`（迭代值）和 `for...in`（迭代键）。

5. **迭代器的不可逆性**：迭代器只能前进，不能回退。

6. **生成器的单次性**：生成器对象只能遍历一次，如需多次遍历需要创建新的生成器。

7. **性能考虑**：对于简单的迭代，传统方法可能更快，生成器在复杂场景更有优势。

8. **this 绑定**：作为对象方法的生成器函数，需要注意 this 的绑定问题。

## 最佳实践

1. **合理使用生成器**：在需要懒加载、流式处理、复杂迭代时使用生成器。

2. **使用 for-await-of**：处理异步迭代时，优先使用 `for-await-of` 而不是手动调用 next()。

3. **实现可迭代协议**：自定义数据结构时，实现 `Symbol.iterator` 方法。

4. **生成器执行器**：使用生成器执行器简化异步操作（如 co 库）。

5. **资源清理**：使用 try-finally 确保生成器中的资源得到清理。

6. **避免嵌套生成器**：深层嵌套的生成器会降低代码可读性。

7. **文档说明**：为自定义的迭代器和生成器提供清晰的文档说明。

8. **测试覆盖**：确保迭代器和生成器的边界条件和异常情况都有测试覆盖。

## 总结

通过本文的学习，相信你已经对 JavaScript 迭代器与生成器有了更深入的理解。

### 核心要点

1. **迭代器协议**：实现了可迭代协议和迭代器协议的对象，可以统一遍历方式。

2. **生成器函数**：使用 `function*` 定义的函数，使用 `yield` 产生值，可以暂停和恢复执行。

3. **异步迭代器**：支持异步操作的迭代器，使用 `for-await-of` 遍历。

4. **应用场景**：
   - 无限序列的生成
   - 异步流程控制
   - 懒加载和流式处理
   - 自定义数据结构
   - 状态机和观察者模式

5. **最佳实践**：
   - 合理选择使用场景
   - 注意内存管理和资源清理
   - 提供清晰的文档
   - 编写充分的测试

迭代器和生成器是 JavaScript 中强大的特性，能够帮助我们编写更优雅、更高效的代码。掌握这些特性对于编写复杂的数据处理和异步逻辑非常有帮助。