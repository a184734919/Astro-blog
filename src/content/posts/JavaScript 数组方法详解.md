---
title: JavaScript 数组方法详解
description: 深入解析 JavaScript 数组方法，包括基础方法、查找方法、转换方法等，提供实际应用场景和最佳实践
pubDate: 2024-01-01
tags: ["JS基础","数组"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 数组方法详解

## 概述

JavaScript 数组提供了丰富的内置方法，这些方法使得数组操作变得简洁而强大。掌握这些方法不仅能提高代码的可读性，还能显著提升开发效率。本文将详细介绍各种数组方法的使用方法和实际应用场景，从基础方法到高级技巧，帮助你全面掌握数组操作的艺术。

## 核心概念

理解核心概念是掌握任何技术的基础，JavaScript 数组方法的核心特性和分类如下：

- **数组方法的本质**：JS 数组内置的函数，用于简化数组的操作（如遍历、修改、筛选、排序等），无需手动编写循环逻辑
- **核心分类**：
  - **遍历方法**：forEach, map, filter, reduce
  - **修改方法**：push, pop, shift, unshift, splice
  - **查找方法**：find, findIndex, indexOf, includes
  - **排序方法**：sort, reverse
  - **转换方法**：join, toString, flat, flatMap
  - **测试方法**：some, every

- **关键区分**：部分方法会「改变原数组」（如 push、pop），部分方法「不改变原数组」（如 map、filter），这是使用数组方法的核心注意点
- **回调函数共性**：多数数组方法（如 forEach、map）支持传入回调函数，回调函数会遍历数组的每一项，接收三个参数（当前项、索引、原数组），按需使用
- **性能考虑**：不同方法的性能特性不同，在大数据量场景下需要选择合适的方法

## 基本用法

### 一、遍历方法（最常用场景）

#### forEach - 基础遍历

```javascript
const arr = [1, 2, 3, 4];

// 基础用法：遍历数组，无返回值
arr.forEach((item, index, arr) => {
  console.log(`索引${index}：${item}`);
});
// 依次输出：索引0：1、索引1：2、索引2：3、索引3：4

// 注意：forEach 无法使用 break 跳出循环
// 如果需要跳出循环，使用 for...of 或 for 循环
for (const item of arr) {
  if (item === 3) break; // 可以使用 break
  console.log(item);
}
```

#### map - 映射转换

```javascript
const arr = [1, 2, 3, 4];

// 基础映射：每一项乘以2
const doubled = arr.map(item => item * 2);
console.log(doubled); // [2, 4, 6, 8]
console.log(arr); // [1, 2, 3, 4]（原数组不变）

// 对象数组映射
const users = [
  { firstName: 'John', lastName: 'Doe' },
  { firstName: 'Jane', lastName: 'Smith' }
];

const fullNames = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ['John Doe', 'Jane Smith']

// 创建新对象结构
const userSummary = users.map(user => ({
  id: user.id,
  displayName: `${user.firstName} ${user.lastName}`
}));
```

#### some - 存在性检查

```javascript
const arr = [1, 2, 3, 4];

// 检查是否有偶数
const hasEven = arr.some(item => item % 2 === 0);
console.log(hasEven); // true（2、4是偶数）

// 对象数组检查
const users = [
  { name: '张三', age: 25, active: true },
  { name: '李四', age: 30, active: false }
];

const hasActiveUser = users.some(user => user.active);
console.log(hasActiveUser); // true
```

#### every - 全部性检查

```javascript
const arr = [1, 2, 3, 4];

// 检查是否全是偶数
const allEven = arr.every(item => item % 2 === 0);
console.log(allEven); // false（1、3是奇数）

// 检查是否都大于0
const allPositive = arr.every(item => item > 0);
console.log(allPositive); // true

// 实际应用：表单验证
const formData = {
  username: 'john',
  email: 'john@example.com',
  password: 'password123'
};

const validationRules = [
  field => formData[field] && formData[field].length > 0
];

const isValid = Object.keys(formData).every((field, index) =>
  validationRules[index](field)
);
console.log(isValid); // true
```

### 二、修改方法（改变原数组）

#### push - 末尾添加

```javascript
const arr = [1, 2, 3];

// 添加单个元素
const newLength = arr.push(4);
console.log(arr); // [1, 2, 3, 4]
console.log(newLength); // 4

// 添加多个元素
arr.push(5, 6, 7);
console.log(arr); // [1, 2, 3, 4, 5, 6, 7]

// 实际应用：构建结果数组
function collectResults(...items) {
  const results = [];
  items.forEach(item => {
    if (item.isValid) {
      results.push(item.value);
    }
  });
  return results;
}
```

#### pop - 末尾删除

```javascript
const arr = [1, 2, 3, 4, 5];

// 删除并返回最后一个元素
const lastElement = arr.pop();
console.log(lastElement); // 5
console.log(arr); // [1, 2, 3, 4]

// 实际应用：栈数据结构
class Stack {
  constructor() {
    this.items = [];
  }

  push(element) {
    return this.items.push(element);
  }

  pop() {
    if (this.isEmpty()) {
      throw new Error('Stack is empty');
    }
    return this.items.pop();
  }

  isEmpty() {
    return this.items.length === 0;
  }

  peek() {
    if (this.isEmpty()) {
      throw new Error('Stack is empty');
    }
    return this.items[this.items.length - 1];
  }
}
```

#### shift - 开头删除

```javascript
const arr = [1, 2, 3, 4, 5];

// 删除并返回第一个元素
const firstElement = arr.shift();
console.log(firstElement); // 1
console.log(arr); // [2, 3, 4, 5]

// 注意：shift 操作会影响所有元素的索引，性能较差
// 对于大数组，考虑使用其他数据结构或方法
```

#### unshift - 开头添加

```javascript
const arr = [3, 4, 5];

// 在开头添加元素
const newLength = arr.unshift(1, 2);
console.log(arr); // [1, 2, 3, 4, 5]
console.log(newLength); // 5

// 实际应用：添加日志时间戳
const logs = ['用户登录', '查看页面'];
const timestampedLogs = [...logs];
timestampedLogs.unshift(new Date().toISOString());
console.log(timestampedLogs);
```

#### splice - 万能修改方法

```javascript
const arr = [1, 2, 3, 4, 5];

// 删除元素：splice(起始索引, 删除个数)
const deleted = arr.splice(2, 2); // 从索引2开始删除2个元素
console.log(arr);     // [1, 2, 5]
console.log(deleted); // [3, 4]

// 添加元素：splice(起始索引, 0, 要添加的元素)
arr.splice(2, 0, 3, 4); // 在索引2处插入3, 4
console.log(arr); // [1, 2, 3, 4, 5]

// 替换元素：splice(起始索引, 删除个数, 要添加的元素)
arr.splice(1, 2, 10, 20); // 替换索引1-2的元素
console.log(arr); // [1, 10, 20, 4, 5]

// 实际应用：批量操作数组
function removeByCondition(array, condition) {
  let i = array.length;
  while (i--) {
    if (condition(array[i])) {
      array.splice(i, 1);
    }
  }
  return array;
}

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
removeByCondition(numbers, num => num % 2 === 0);
console.log(numbers); // [1, 3, 5, 7, 9]
```

### 三、查找方法

#### indexOf - 查找元素位置

```javascript
const fruits = ['苹果', '香蕉', '橙子', '葡萄'];

// 查找元素位置
const index = fruits.indexOf('橙子');
console.log(index); // 2

// 未找到返回 -1
const notFound = fruits.indexOf('西瓜');
console.log(notFound); // -1

// 从指定位置开始查找
const fromIndex = fruits.indexOf('葡萄', 2);
console.log(fromIndex); // 3

// 实际应用：检查元素是否存在
function hasElement(array, element) {
  return array.indexOf(element) !== -1;
}

// 更好的方式：使用 includes
function hasElementModern(array, element) {
  return array.includes(element);
}
```

#### includes - 存在性检查

```javascript
const numbers = [1, 2, 3, 4, 5];

// 检查是否包含某个元素
console.log(numbers.includes(3));  // true
console.log(numbers.includes(10)); // false

// 从指定位置开始检查
console.log(numbers.includes(3, 3)); // false（从索引3开始查找）

// 处理 NaN
const arrayWithNaN = [1, 2, NaN, 4];
console.log(arrayWithNaN.includes(NaN)); // true

// 实际应用：权限检查
function hasPermission(userPermissions, requiredPermission) {
  return userPermissions.includes(requiredPermission) ||
         userPermissions.includes('admin');
}

const userPermissions = ['read', 'write'];
console.log(hasPermission(userPermissions, 'read'));   // true
console.log(hasPermission(userPermissions, 'delete')); // false
```

#### find - 查找符合条件的元素

```javascript
const users = [
  { id: 1, name: '张三', age: 25 },
  { id: 2, name: '李四', age: 30 },
  { id: 3, name: '王五', age: 35 }
];

// 查找第一个年龄大于30的用户
const user = users.find(user => user.age > 30);
console.log(user); // { id: 3, name: '王五', age: 35 }

// 查找特定ID的用户
const foundUser = users.find(user => user.id === 2);
console.log(foundUser); // { id: 2, name: '李四', age: 30 }

// 未找到返回 undefined
const notFound = users.find(user => user.age > 40);
console.log(notFound); // undefined

// 实际应用：查找和更新数据
function updateUser(users, userId, updates) {
  const userIndex = users.findIndex(user => user.id === userId);
  if (userIndex !== -1) {
    users[userIndex] = { ...users[userIndex], ...updates };
    return users[userIndex];
  }
  return null;
}
```

#### findIndex - 查找元素位置

```javascript
const numbers = [10, 20, 30, 40, 50];

// 查找第一个大于25的元素索引
const index = numbers.findIndex(num => num > 25);
console.log(index); // 2

// 查找偶数的索引
const evenIndex = numbers.findIndex(num => num % 2 === 0);
console.log(evenIndex); // 0

// 未找到返回 -1
const notFoundIndex = numbers.findIndex(num => num > 100);
console.log(notFoundIndex); // -1

// 实际应用：删除元素
function removeById(array, id) {
  const index = array.findIndex(item => item.id === id);
  if (index !== -1) {
    array.splice(index, 1);
    return true;
  }
  return false;
}
```

### 四、筛选和转换方法

#### filter - 筛选元素

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 筛选偶数
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// 筛选大于5的元素
const greaterThanFive = numbers.filter(num => num > 5);
console.log(greaterThanFive); // [6, 7, 8, 9, 10]

// 对象数组筛选
const products = [
  { name: '商品A', price: 100, category: '电子' },
  { name: '商品B', price: 200, category: '服装' },
  { name: '商品C', price: 150, category: '电子' }
];

const electronics = products.filter(product => product.category === '电子');
const expensive = products.filter(product => product.price > 120);

// 实际应用：高级筛选
function filterProducts(products, filters) {
  return products.filter(product => {
    if (filters.priceRange) {
      const { min, max } = filters.priceRange;
      if (min && product.price < min) return false;
      if (max && product.price > max) return false;
    }

    if (filters.categories && filters.categories.length > 0) {
      if (!filters.categories.includes(product.category)) return false;
    }

    if (filters.searchTerm) {
      const term = filters.searchTerm.toLowerCase();
      const searchableText = `${product.name} ${product.description}`.toLowerCase();
      if (!searchableText.includes(term)) return false;
    }

    return true;
  });
}
```

#### reduce - 归约计算

```javascript
const numbers = [1, 2, 3, 4, 5];

// 数组求和
const sum = numbers.reduce((acc, num) => acc + num, 0);
console.log(sum); // 15

// 数组求最大值
const max = numbers.reduce((acc, num) => Math.max(acc, num), -Infinity);
console.log(max); // 5

// 数组转对象
const fruits = ['苹果', '香蕉', '橙子'];
const fruitObject = fruits.reduce((acc, fruit, index) => {
  acc[index] = fruit;
  return acc;
}, {});

console.log(fruitObject); // { 0: '苹果', 1: '香蕉', 2: '橙子' }

// 复杂对象处理
const orders = [
  { id: 1, total: 100 },
  { id: 2, total: 200 },
  { id: 3, total: 150 }
];

const orderStats = orders.reduce((acc, order) => {
  acc.totalRevenue += order.total;
  acc.orderCount += 1;
  acc.averageTotal = acc.totalRevenue / acc.orderCount;
  return acc;
}, { totalRevenue: 0, orderCount: 0, averageTotal: 0 });

console.log(orderStats); // { totalRevenue: 450, orderCount: 3, averageTotal: 150 }

// 实际应用：数据分组
function groupBy(array, key) {
  return array.reduce((acc, item) => {
    const groupKey = item[key];
    if (!acc[groupKey]) {
      acc[groupKey] = [];
    }
    acc[groupKey].push(item);
    return acc;
  }, {});
}

const users = [
  { name: '张三', department: '技术' },
  { name: '李四', department: '市场' },
  { name: '王五', department: '技术' }
];

const groupedUsers = groupBy(users, 'department');
console.log(groupedUsers);
// {
//   '技术': [{ name: '张三', department: '技术' }, { name: '王五', department: '技术' }],
//   '市场': [{ name: '李四', department: '市场' }]
// }
```

### 五、排序方法

#### sort - 数组排序

```javascript
// 字符串排序
const fruits = ['香蕉', '苹果', '橙子', '葡萄'];
fruits.sort();
console.log(fruits); // ['苹果', '橙子', '葡萄', '香蕉']

// 数字排序（需要比较函数）
const numbers = [10, 2, 8, 4, 6];
numbers.sort((a, b) => a - b); // 升序
console.log(numbers); // [2, 4, 6, 8, 10]

numbers.sort((a, b) => b - a); // 降序
console.log(numbers); // [10, 8, 6, 4, 2]

// 对象数组排序
const users = [
  { name: '张三', age: 25 },
  { name: '李四', age: 30 },
  { name: '王五', age: 28 }
];

users.sort((a, b) => a.age - b.age);
console.log(users); // 按年龄升序排列

// 多条件排序
const products = [
  { name: 'A', price: 100, rating: 4.5 },
  { name: 'B', price: 100, rating: 4.8 },
  { name: 'C', price: 150, rating: 4.2 }
];

products.sort((a, b) => {
  if (a.price !== b.price) {
    return a.price - b.price; // 先按价格排序
  }
  return b.rating - a.rating; // 价格相同时按评分降序
});

// 实际应用：通用排序函数
function sortArray(array, sortConfig) {
  return [...array].sort((a, b) => {
    for (const { key, direction = 'asc' } of sortConfig) {
      const valueA = a[key];
      const valueB = b[key];

      if (valueA === valueB) continue;

      let comparison = 0;
      if (typeof valueA === 'string' && typeof valueB === 'string') {
        comparison = valueA.localeCompare(valueB);
      } else {
        comparison = valueA > valueB ? 1 : -1;
      }

      return direction === 'asc' ? comparison : -comparison;
    }
    return 0;
  });
}
```

#### reverse - 反转数组

```javascript
const numbers = [1, 2, 3, 4, 5];
numbers.reverse();
console.log(numbers); // [5, 4, 3, 2, 1]

// 注意：reverse() 会修改原数组
const original = [1, 2, 3];
const reversed = [...original].reverse(); // 创建副本再反转
console.log(original);  // [1, 2, 3] - 原数组未变
console.log(reversed);  // [3, 2, 1]

// 实际应用：字符串反转
function reverseString(str) {
  return str.split('').reverse().join('');
}

console.log(reverseString('hello')); // 'olleh'
```

### 六、其他实用方法

#### slice - 数组切片

```javascript
const numbers = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];

// 提取子数组
const firstFive = numbers.slice(0, 5); // [0, 1, 2, 3, 4]
const lastThree = numbers.slice(-3);    // [7, 8, 9]
const middle = numbers.slice(3, 7);     // [3, 4, 5, 6]

// 复制数组
const copy = numbers.slice(); // 完整复制数组

// 实际应用：分页数据处理
function paginate(array, page = 1, pageSize = 10) {
  const startIndex = (page - 1) * pageSize;
  const endIndex = startIndex + pageSize;
  return {
    data: array.slice(startIndex, endIndex),
    total: array.length,
    page: page,
    totalPages: Math.ceil(array.length / pageSize)
  };
}

const allUsers = Array.from({ length: 45 }, (_, i) => ({ id: i + 1, name: `用户${i + 1}` }));
const page1 = paginate(allUsers, 1, 10);
console.log(page1);
```

#### concat - 数组合并

```javascript
const array1 = [1, 2, 3];
const array2 = [4, 5, 6];
const array3 = [7, 8, 9];

const combined = array1.concat(array2, array3);
console.log(combined); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// 也可以使用展开运算符
const combinedSpread = [...array1, ...array2, ...array3];
```

#### join - 数组转字符串

```javascript
const words = ['Hello', 'World', 'JavaScript'];
const sentence = words.join(' '); // 'Hello World JavaScript'

const numbers = [1, 2, 3, 4, 5];
const csv = numbers.join(','); // '1,2,3,4,5'

// 实际应用：生成路径
const pathSegments = ['home', 'user', 'documents', 'file.txt'];
const filePath = pathSegments.join('/'); // 'home/user/documents/file.txt'

// 生成 URL 参数
const params = ['key1=value1', 'key2=value2', 'key3=value3'];
const queryString = params.join('&'); // 'key1=value1&key2=value2&key3=value3'
```

#### flat - 数组扁平化

```javascript
// 基础扁平化
const nested = [1, [2, [3, [4, 5]], 6], 7];
const flat1 = nested.flat();        // [1, 2, [3, [4, 5]], 6, 7] - 默认深度为 1
const flat2 = nested.flat(2);       // [1, 2, 3, [4, 5], 6, 7]
const flatInfinite = nested.flat(Infinity); // [1, 2, 3, 4, 5, 6, 7]

// 实际应用：处理嵌套数据结构
function flattenObject(obj, prefix = '') {
  const result = {};

  for (const key in obj) {
    const newKey = prefix ? `${prefix}.${key}` : key;
    if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
      Object.assign(result, flattenObject(obj[key], newKey));
    } else {
      result[newKey] = obj[key];
    }
  }

  return result;
}
```

#### flatMap - 映射并扁平化

```javascript
const sentences = ['Hello world', 'JavaScript is awesome'];

// 将每个句子分割为单词，然后扁平化
const words = sentences.flatMap(sentence => sentence.split(' '));
console.log(words); // ['Hello', 'world', 'JavaScript', 'is', 'awesome']

// 实际应用：提取嵌套数据
const orders = [
  { id: 1, items: [{ name: 'A' }, { name: 'B' }] },
  { id: 2, items: [{ name: 'C' }] },
  { id: 3, items: [{ name: 'D' }, { name: 'E' }, { name: 'F' }] }
];

// 提取所有订单项
const allItems = orders.flatMap(order =>
  order.items.map(item => ({ ...item, orderId: order.id }))
);

console.log(allItems);
/*
[
  { name: 'A', orderId: 1 },
  { name: 'B', orderId: 1 },
  { name: 'C', orderId: 2 },
  { name: 'D', orderId: 3 },
  { name: 'E', orderId: 3 },
  { name: 'F', orderId: 3 }
]
*/
```

## 实际应用

### 1. 数据处理管道

```javascript
// 复杂数据处理流程
function processUserMetrics(rawData) {
  return rawData
    .filter(user => user.isActive) // 筛选活跃用户
    .map(user => ({
      ...user,
      age: calculateAge(user.birthDate),
      engagement: calculateEngagement(user.activities)
    }))
    .sort((a, b) => b.engagement - a.engagement) // 按参与度排序
    .slice(0, 10); // 取前10名
}

function calculateAge(birthDate) {
  const today = new Date();
  const birth = new Date(birthDate);
  let age = today.getFullYear() - birth.getFullYear();
  const monthDiff = today.getMonth() - birth.getMonth();
  if (monthDiff < 0 || (monthDiff === 0 && today.getDate() < birth.getDate())) {
    age--;
  }
  return age;
}

function calculateEngagement(activities) {
  return activities.reduce((total, activity) => total + activity.score, 0);
}
```

### 2. 表格数据处理

```javascript
// 处理表格数据的完整示例
function processTableData(rawData, options = {}) {
  return {
    processed: rawData
      .filter(row => {
        // 过滤条件
        if (options.filter) {
          return Object.entries(options.filter).every(([key, value]) =>
            row[key] === value
          );
        }
        return true;
      })
      .map(row => {
        // 数据转换
        return {
          id: row.id,
          name: row.name || 'Unknown',
          status: row.active ? 'Active' : 'Inactive',
          date: new Date(row.created_at).toLocaleDateString(),
          amount: parseFloat(row.amount).toFixed(2)
        };
      })
      .sort((a, b) => {
        // 排序
        const sortBy = options.sortBy || 'name';
        const direction = options.sortDirection === 'desc' ? -1 : 1;
        return a[sortBy] > b[sortBy] ? direction : -direction;
      }),

    summary: rawData.reduce((acc, row) => {
      acc.total++;
      acc.totalAmount += parseFloat(row.amount);
      acc.activeCount += row.active ? 1 : 0;
      return acc;
    }, { total: 0, totalAmount: 0, activeCount: 0 })
  };
}
```

### 3. 数据统计和分析

```javascript
// 数据分析工具
class DataAnalyzer {
  constructor(data) {
    this.data = data;
  }

  // 计算平均值
  average(key) {
    const values = this.data.map(item => item[key]).filter(val => !isNaN(val));
    return values.reduce((sum, val) => sum + val, 0) / values.length;
  }

  // 计算中位数
  median(key) {
    const values = this.data.map(item => item[key]).filter(val => !isNaN(val)).sort((a, b) => a - b);
    const mid = Math.floor(values.length / 2);
    return values.length % 2 !== 0 ? values[mid] : (values[mid - 1] + values[mid]) / 2;
  }

  // 计算标准差
  standardDeviation(key) {
    const values = this.data.map(item => item[key]).filter(val => !isNaN(val));
    const avg = values.reduce((sum, val) => sum + val, 0) / values.length;
    const squareDiffs = values.map(value => Math.pow(value - avg, 2));
    return Math.sqrt(squareDiffs.reduce((sum, diff) => sum + diff, 0) / values.length);
  }

  // 分组统计
  groupBy(key, metricKey) {
    return this.data.reduce((groups, item) => {
      const groupKey = item[key];
      if (!groups[groupKey]) {
        groups[groupKey] = {
          count: 0,
          sum: 0,
          items: []
        };
      }
      groups[groupKey].count++;
      groups[groupKey].sum += item[metricKey] || 0;
      groups[groupKey].items.push(item);
      return groups;
    }, {});
  }
}

// 使用示例
const salesData = [
  { product: 'A', category: '电子', price: 100, quantity: 5 },
  { product: 'B', category: '服装', price: 50, quantity: 10 },
  { product: 'C', category: '电子', price: 150, quantity: 3 }
];

const analyzer = new DataAnalyzer(salesData);
console.log(analyzer.average('price')); // 平均价格
console.log(analyzer.groupBy('category', 'price')); // 按类别分组
```

### 4. 数组去重和合并

```javascript
// 数组去重的多种方法
function uniqueArray(array) {
  // 方法1：使用 Set
  return [...new Set(array)];
}

function uniqueBy(array, key) {
  // 方法2：根据对象属性去重
  const seen = new Set();
  return array.filter(item => {
    const keyValue = item[key];
    if (seen.has(keyValue)) {
      return false;
    }
    seen.add(keyValue);
    return true;
  });
}

// 智能数组合并
function smartMerge(...arrays) {
  const merged = [];
  const seen = new Set();

  arrays.forEach(array => {
    array.forEach(item => {
      const key = typeof item === 'object' ? JSON.stringify(item) : item;
      if (!seen.has(key)) {
        seen.add(key);
        merged.push(item);
      }
    });
  });

  return merged;
}

// 使用示例
const arr1 = [1, 2, 3];
const arr2 = [3, 4, 5];
const arr3 = [5, 6, 7];

console.log(smartMerge(arr1, arr2, arr3)); // [1, 2, 3, 4, 5, 6, 7]
```

### 5. 缓存和记忆化

```javascript
// 使用数组方法实现缓存
class DataCache {
  constructor(maxSize = 100) {
    this.cache = [];
    this.maxSize = maxSize;
  }

  get(key) {
    return this.cache.find(item => item.key === key)?.value;
  }

  set(key, value) {
    const existingIndex = this.cache.findIndex(item => item.key === key);

    if (existingIndex !== -1) {
      // 更新现有项
      this.cache[existingIndex] = { key, value, timestamp: Date.now() };
    } else {
      // 添加新项
      this.cache.push({ key, value, timestamp: Date.now() });

      // 如果超出最大容量，删除最旧的项
      if (this.cache.length > this.maxSize) {
        this.cache.shift();
      }
    }
  }

  clear() {
    this.cache = [];
  }

  // 根据条件过滤缓存
  filter(predicate) {
    return this.cache.filter(predicate);
  }

  // 清理过期缓存
  cleanUp(maxAge) {
    const now = Date.now();
    this.cache = this.cache.filter(item =>
      now - item.timestamp < maxAge
    );
  }
}

// 记忆化函数
function memoize(fn, keyGenerator = (...args) => args.join('_')) {
  const cache = [];

  return function(...args) {
    const key = keyGenerator(...args);
    const cached = cache.find(item => item.key === key);

    if (cached) {
      return cached.value;
    }

    const result = fn.apply(this, args);
    cache.push({ key, value: result });
    return result;
  };
}

// 使用示例
const expensiveFunction = memoize((n) => {
  console.log('计算中...');
  return n * 2;
});

console.log(expensiveFunction(5)); // 计算中... 10
console.log(expensiveFunction(5)); // 10 (使用缓存)
```

## 性能注意事项

### 1. 方法选择对性能的影响

```javascript
const largeArray = Array.from({ length: 100000 }, (_, i) => i);

// 性能测试：不同方法的执行时间
console.time('forEach');
largeArray.forEach(item => item * 2);
console.timeEnd('forEach');

console.time('for');
for (let i = 0; i < largeArray.length; i++) {
  largeArray[i] * 2;
}
console.timeEnd('for');

console.time('for...of');
for (const item of largeArray) {
  item * 2;
}
console.timeEnd('for...of');

// 通常情况下，传统 for 循环性能最好，但在可读性和维护性方面，数组方法更优
```

### 2. 避免在循环中创建数组

```javascript
// 不好的做法
function processBad(items) {
  const results = [];
  items.forEach(item => {
    const temp = item.data.map(d => d.value); // 每次迭代都创建新数组
    results.push(...temp);
  });
  return results;
}

// 好的做法
function processGood(items) {
  return items.flatMap(item => item.data.map(d => d.value));
}
```

### 3. 合理使用 reduce

```javascript
// 简单场景使用 map + filter 更清晰
const result = numbers
  .filter(n => n % 2 === 0)
  .map(n => n * 2);

// 复杂聚合操作使用 reduce
const stats = numbers.reduce((acc, n) => {
  if (n % 2 === 0) {
    acc.evenSum += n;
    acc.evenCount++;
  } else {
    acc.oddSum += n;
    acc.oddCount++;
  }
  return acc;
}, { evenSum: 0, evenCount: 0, oddSum: 0, oddCount: 0 });
```

## 注意事项

### 1. 注意边界情况

- **索引越界**：访问超出范围的索引不会抛出错误，而是返回 undefined
- **空数组处理**：reduce 方法在空数组时需要设置初始值
- **NaN 处理**：indexOf 无法找到 NaN，includes 可以正确处理

```javascript
const arr = [1, 2, 3];
console.log(arr[10]); // undefined

const emptyArr = [];
const sum = emptyArr.reduce((a, b) => a + b, 0); // 需要初始值，否则报错

const withNaN = [1, 2, NaN, 4];
console.log(withNaN.indexOf(NaN)); // -1
console.log(withNaN.includes(NaN)); // true
```

### 2. 考虑性能影响

- **forEach vs for**：在大数据量场景下，传统 for 循环性能更好
- **splice 性能**：splice 会移动数组元素，在大数组上性能较差
- **内存使用**：链式调用会创建多个中间数组，注意内存消耗

### 3. 遵循最佳实践

- **不可变性**：优先使用不修改原数组的方法（map, filter, reduce）
- **可读性**：在性能和可读性之间取得平衡，优先考虑代码的可维护性
- **方法链**：合理使用方法链，但避免过度嵌套影响可读性

```javascript
// 好的做法：清晰的方法链
const result = data
  .filter(item => item.active)
  .map(item => item.value)
  .reduce((sum, val) => sum + val, 0);

// 避免过度嵌套
const badResult = data
  .filter(item => {
    return item.details && item.details.info && item.details.info.active;
  })
  .map(item => {
    return item.data.map(d => d.value).filter(v => v > 0);
  })
  .flat();
```

### 4. 避免常见误区

- **不要混淆 filter 和 map**：filter 用于筛选，map 用于映射转换
- **不要误以为 concat 会改变原数组**：concat 返回新数组
- **不要忘记 reduce 的初始值**：尤其数组可能为空时
- **不要在 forEach 中使用 break**：使用 for...of 或 for 循环替代

## 总结

通过本文的学习，相信你已经对 JavaScript 数组方法有了更深入的理解。数组方法是前端开发的基础工具，核心在于根据场景选择合适的方法，牢记「是否改变原数组」这一关键区分，灵活搭配使用不同方法，能大幅提升开发效率。

### 核心要点回顾：

1. **掌握方法分类**：了解遍历、修改、查找、筛选、排序等不同类别的方法
2. **理解方法特性**：明确哪些方法改变原数组，哪些返回新数组
3. **性能考虑**：在大数据量场景下选择合适的方法
4. **实际应用**：结合业务场景灵活运用数组方法
5. **最佳实践**：优先考虑代码的可读性和可维护性

### 进阶学习建议：

- 深入学习函数式编程思想
- 掌握数组方法的组合使用技巧
- 了解 TypeScript 中数组的类型定义
- 学习性能优化和内存管理
- 探索数组方法的 polyfill 实现原理

持续练习和实践，你将能够熟练运用数组方法解决各种复杂的数据处理问题，编写出更加优雅、高效的 JavaScript 代码。