---
title: JavaScript 数组高阶方法
description: 深入理解 map、filter、reduce 等高阶函数的原理、应用和最佳实践
pubDate: 2024-01-02
tags: ["JS基础","数组","高阶函数"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

# JavaScript 数组高阶方法

## 概述

JavaScript 数组高阶方法是函数式编程的重要体现，它们以函数作为参数或返回函数，为我们提供了强大的数据处理能力。核心的高阶方法包括 `map`、`filter`、`reduce` 等，它们不仅能替代传统的循环语句，还能通过链式调用构建复杂的数据处理管道，使代码更加简洁、声明式和易于维护。

## 核心概念

### 什么是高阶函数

高阶函数是指满足以下任一条件的函数：

1. **接收函数作为参数**：如 `map`、`filter`、`reduce` 等
2. **返回函数作为结果**：如函数柯里化、高阶组件等

```javascript
// 高阶函数示例
function withLogger(fn) {
  return function(...args) {
    console.log('调用函数:', fn.name, '参数:', args);
    const result = fn.apply(this, args);
    console.log('函数结果:', result);
    return result;
  };
}

const add = (a, b) => a + b;
const loggedAdd = withLogger(add);
loggedAdd(2, 3); // 输出日志并返回 5
```

### 数组高阶方法的核心特性

- **不可变性**：不修改原数组，返回新数组或结果
- **声明式**：描述"做什么"而不是"怎么做"
- **可组合**：支持方法链式调用
- **函数式**：支持纯函数和副作用隔离

### 三大核心方法对比

| 方法 | 用途 | 返回值 | 是否改变原数组 |
|------|------|--------|----------------|
| map | 映射转换 | 新数组 | 否 |
| filter | 筛选过滤 | 新数组 | 否 |
| reduce | 归约计算 | 单一值 | 否 |

## 基本用法

### 1. map - 映射转换

#### 基础用法

```javascript
const numbers = [1, 2, 3, 4, 5];

// 简单映射：每个元素乘以 2
const doubled = numbers.map(num => num * 2);
console.log(doubled); // [2, 4, 6, 8, 10]

// 对象数组映射
const users = [
  { firstName: 'John', lastName: 'Doe' },
  { firstName: 'Jane', lastName: 'Smith' }
];

const fullNames = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(fullNames); // ['John Doe', 'Jane Smith']
```

#### 高级映射技巧

```javascript
// 索引映射：为元素添加索引信息
const items = ['苹果', '香蕉', '橙子'];
const indexedItems = items.map((item, index) => ({
  id: index + 1,
  name: item,
  selected: false
}));
console.log(indexedItems);
// [{ id: 1, name: '苹果', selected: false }, ...]

// 条件映射：根据条件进行不同的转换
const scores = [85, 92, 78, 95, 88];
const grades = scores.map(score => {
  if (score >= 90) return { score, grade: 'A' };
  if (score >= 80) return { score, grade: 'B' };
  return { score, grade: 'C' };
});

// 对象转换：提取和重组对象属性
const products = [
  { id: 1, name: 'iPhone', price: 999, category: 'phone' },
  { id: 2, name: 'MacBook', price: 1999, category: 'computer' }
];

const productCards = products.map(({ id, name, price }) => ({
  productId: id,
  productName: name,
  priceFormatted: `$${price.toFixed(2)}`,
  inStock: true
}));
```

#### 实际应用场景

```javascript
// API 数据格式转换
function transformApiResponse(apiResponse) {
  return apiResponse.map(item => ({
    id: item.id,
    title: item.title,
    subtitle: item.description,
    image: item.imageUrl,
    metadata: {
      created: new Date(item.createdAt),
      updated: new Date(item.updatedAt),
      author: item.authorName
    },
    stats: {
      views: item.viewCount || 0,
      likes: item.likeCount || 0,
      comments: item.commentCount || 0
    }
  }));
}

// 数据库记录到前端模型的转换
function toUserModels(dbRecords) {
  return dbRecords.map(record => ({
    id: record.user_id,
    username: record.username,
    email: record.email,
    profile: {
      firstName: record.first_name,
      lastName: record.last_name,
      avatar: record.avatar_url
    },
    permissions: record.permissions.split(','),
    createdAt: new Date(record.created_at),
    isActive: record.status === 'active'
  }));
}
```

### 2. filter - 筛选过滤

#### 基础用法

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 筛选偶数
const evens = numbers.filter(num => num % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// 筛选大于 5 的数
const greaterThanFive = numbers.filter(num => num > 5);
console.log(greaterThanFive); // [6, 7, 8, 9, 10]

// 对象数组筛选
const users = [
  { name: '张三', age: 25, active: true },
  { name: '李四', age: 17, active: true },
  { name: '王五', age: 30, active: false }
];

const activeAdults = users.filter(user => user.active && user.age >= 18);
console.log(activeAdults); // [{ name: '张三', age: 25, active: true }]
```

#### 高级筛选技巧

```javascript
// 多条件筛选
const products = [
  { name: 'iPhone', price: 999, category: 'phone', stock: 10 },
  { name: 'Samsung', price: 799, category: 'phone', stock: 0 },
  { name: 'MacBook', price: 1999, category: 'computer', stock: 5 }
];

const availablePhones = products.filter(product =>
  product.category === 'phone' &&
  product.stock > 0 &&
  product.price < 1000
);

// 使用辅助函数构建复杂筛选条件
function buildFilter(...conditions) {
  return (item) => conditions.every(condition => condition(item));
}

const expensiveItems = products.filter(buildFilter(
  item => item.price > 1000,
  item => item.stock > 0,
  item => item.category !== 'accessory'
));

// 动态筛选器
function createFilter(filters) {
  return (item) => {
    return Object.entries(filters).every(([key, value]) => {
      if (typeof value === 'function') {
        return value(item[key]);
      }
      return item[key] === value;
    });
  };
}

const productFilter = createFilter({
  category: 'phone',
  price: price => price < 1000,
  stock: stock => stock > 0
});

const filteredProducts = products.filter(productFilter);
```

#### 实际应用场景

```javascript
// 电商商品筛选
function filterProducts(products, filters) {
  return products.filter(product => {
    // 价格范围筛选
    if (filters.priceRange) {
      const { min, max } = filters.priceRange;
      if (min && product.price < min) return false;
      if (max && product.price > max) return false;
    }

    // 类别筛选
    if (filters.categories && filters.categories.length > 0) {
      if (!filters.categories.includes(product.category)) return false;
    }

    // 品牌筛选
    if (filters.brands && filters.brands.length > 0) {
      if (!filters.brands.includes(product.brand)) return false;
    }

    // 评分筛选
    if (filters.minRating && product.rating < filters.minRating) {
      return false;
    }

    // 搜索关键词
    if (filters.searchTerm) {
      const term = filters.searchTerm.toLowerCase();
      const searchableText = `${product.name} ${product.description}`.toLowerCase();
      if (!searchableText.includes(term)) return false;
    }

    return true;
  });
}

// 使用示例
const allProducts = [
  { name: 'iPhone 15', price: 799, category: 'electronics', brand: 'Apple', rating: 4.5, description: 'Latest smartphone' },
  { name: 'Samsung S24', price: 699, category: 'electronics', brand: 'Samsung', rating: 4.3, description: 'Android flagship' },
  { name: 'MacBook Pro', price: 1999, category: 'electronics', brand: 'Apple', rating: 4.8, description: 'Professional laptop' }
];

const filtered = filterProducts(allProducts, {
  categories: ['electronics'],
  priceRange: { min: 500, max: 1000 },
  minRating: 4.0,
  searchTerm: 'phone'
});

// 数据清洗：去除无效数据
function cleanData(rawData) {
  return rawData.filter(item => {
    // 检查必要字段
    if (!item.id || !item.name) return false;

    // 检查数据类型
    if (typeof item.id !== 'number') return false;

    // 检查数据范围
    if (item.age && (item.age < 0 || item.age > 150)) return false;

    // 检查格式
    if (item.email && !item.email.includes('@')) return false;

    return true;
  });
}
```

### 3. reduce - 归约计算

#### 基础用法

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
```

#### 高级归约技巧

```javascript
// 数据分组
const users = [
  { name: '张三', department: '技术' },
  { name: '李四', department: '市场' },
  { name: '王五', department: '技术' },
  { name: '赵六', department: '市场' }
];

const groupedByDept = users.reduce((acc, user) => {
  const dept = user.department;
  if (!acc[dept]) {
    acc[dept] = [];
  }
  acc[dept].push(user);
  return acc;
}, {});

console.log(groupedByDept);
// {
//   '技术': [{ name: '张三', department: '技术' }, { name: '王五', department: '技术' }],
//   '市场': [{ name: '李四', department: '市场' }, { name: '赵六', department: '市场' }]
// }

// 多维度统计
const orders = [
  { id: 1, product: 'A', price: 100, quantity: 2, category: 'electronics' },
  { id: 2, product: 'B', price: 50, quantity: 3, category: 'books' },
  { id: 3, product: 'A', price: 100, quantity: 1, category: 'electronics' }
];

const stats = orders.reduce((acc, order) => {
  // 总销售额
  acc.totalRevenue += order.price * order.quantity;

  // 按类别统计
  if (!acc.byCategory[order.category]) {
    acc.byCategory[order.category] = {
      revenue: 0,
      count: 0
    };
  }
  acc.byCategory[order.category].revenue += order.price * order.quantity;
  acc.byCategory[order.category].count += 1;

  // 按产品统计
  if (!acc.byProduct[order.product]) {
    acc.byProduct[order.product] = {
      totalQuantity: 0,
      totalRevenue: 0
    };
  }
  acc.byProduct[order.product].totalQuantity += order.quantity;
  acc.byProduct[order.product].totalRevenue += order.price * order.quantity;

  return acc;
}, {
  totalRevenue: 0,
  byCategory: {},
  byProduct: {}
});

// 数组去重
function uniqueBy(array, key) {
  return array.reduce((acc, item) => {
    const keyValue = item[key];
    if (!acc.seen.has(keyValue)) {
      acc.seen.add(keyValue);
      acc.result.push(item);
    }
    return acc;
  }, { seen: new Set(), result: [] }).result;
}

const uniqueUsers = uniqueBy(users, 'name');
```

#### 实际应用场景

```javascript
// 数据透视表
function createPivotTable(data, rowKey, colKey, valueKey, aggregation = 'sum') {
  return data.reduce((acc, item) => {
    const row = item[rowKey];
    const col = item[colKey];
    const value = item[valueKey];

    if (!acc[row]) {
      acc[row] = {};
    }
    if (!acc[row][col]) {
      acc[row][col] = 0;
    }

    switch (aggregation) {
      case 'sum':
        acc[row][col] += value;
        break;
      case 'count':
        acc[row][col] += 1;
        break;
      case 'avg':
        // 平均值需要额外处理
        if (!acc[row][`${col}_count`]) {
          acc[row][`${col}_count`] = 0;
        }
        acc[row][`${col}_count`] += 1;
        acc[row][col] = (acc[row][col] * (acc[row][`${col}_count`] - 1) + value) / acc[row][`${col}_count`];
        break;
    }

    return acc;
  }, {});
}

// 使用示例
const salesData = [
  { region: 'East', product: 'A', quarter: 'Q1', revenue: 100 },
  { region: 'East', product: 'B', quarter: 'Q1', revenue: 150 },
  { region: 'West', product: 'A', quarter: 'Q1', revenue: 120 },
  { region: 'East', product: 'A', quarter: 'Q2', revenue: 110 }
];

const pivotTable = createPivotTable(salesData, 'region', 'product', 'revenue');
console.log(pivotTable);
// {
//   'East': { 'A': 210, 'B': 150 },
//   'West': { 'A': 120 }
// }

// 复杂的数据转换管道
function processDataPipeline(rawData) {
  return rawData
    .filter(item => item.isValid) // 过滤无效数据
    .map(item => ({
      ...item,
      processedValue: item.value * 1.1, // 转换数值
      category: categorize(item.value) // 分类
    }))
    .reduce((acc, item) => {
      // 按类别分组并统计
      if (!acc[item.category]) {
        acc[item.category] = {
          count: 0,
          total: 0,
          items: []
        };
      }
      acc[item.category].count++;
      acc[item.category].total += item.processedValue;
      acc[item.category].items.push(item);
      return acc;
    }, {});
}

function categorize(value) {
  if (value < 50) return 'low';
  if (value < 100) return 'medium';
  return 'high';
}
```

## 方法组合与链式调用

### 基础链式调用

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// 筛选偶数，乘以 3，然后求和
const result = numbers
  .filter(num => num % 2 === 0)  // [2, 4, 6, 8, 10]
  .map(num => num * 3)            // [6, 12, 18, 24, 30]
  .reduce((sum, num) => sum + num, 0); // 90

console.log(result); // 90
```

### 实际业务场景

```javascript
// 电商订单处理
function processOrders(orders) {
  return orders
    .filter(order => order.status === 'completed') // 筛选已完成订单
    .map(order => ({
      ...order,
      totalAmount: order.items.reduce((sum, item) =>
        sum + (item.price * item.quantity), 0
      )
    }))
    .filter(order => order.totalAmount > 100) // 筛选金额大于100的订单
    .sort((a, b) => b.totalAmount - a.totalAmount) // 按金额降序排列
    .slice(0, 10); // 取前10名
}

// 用户数据分析
function analyzeUserActivity(users, activities) {
  return users
    .filter(user => user.isActive) // 筛选活跃用户
    .map(user => {
      const userActivities = activities.filter(
        activity => activity.userId === user.id
      );

      return {
        ...user,
        activityCount: userActivities.length,
        totalDuration: userActivities.reduce(
          (sum, activity) => sum + activity.duration,
          0
        ),
        avgSessionDuration: userActivities.length > 0
          ? userActivities.reduce(
              (sum, activity) => sum + activity.duration,
              0
            ) / userActivities.length
          : 0
      };
    })
    .sort((a, b) => b.totalDuration - a.totalDuration) // 按总时长排序
    .map(user => ({
      username: user.username,
      engagementScore: calculateEngagementScore(
        user.activityCount,
        user.totalDuration,
        user.avgSessionDuration
      )
    }));
}

function calculateEngagementScore(count, totalDuration, avgDuration) {
  return (count * 0.3) + (totalDuration * 0.4) + (avgDuration * 0.3);
}
```

### 高级组合模式

```javascript
// 数据处理管道构建器
class DataPipeline {
  constructor(data) {
    this.data = data;
    this.operations = [];
  }

  filter(predicate) {
    this.operations.push(data => data.filter(predicate));
    return this;
  }

  map(transformer) {
    this.operations.push(data => data.map(transformer));
    return this;
  }

  sort(comparator) {
    this.operations.push(data => [...data].sort(comparator));
    return this;
  }

  groupBy(key) {
    this.operations.push(data => {
      return data.reduce((groups, item) => {
        const groupKey = item[key];
        if (!groups[groupKey]) {
          groups[groupKey] = [];
        }
        groups[groupKey].push(item);
        return groups;
      }, {});
    });
    return this;
  }

  execute() {
    return this.operations.reduce((data, operation) => operation(data), this.data);
  }
}

// 使用示例
const users = [
  { name: '张三', age: 25, department: '技术' },
  { name: '李四', age: 30, department: '市场' },
  { name: '王五', age: 28, department: '技术' }
];

const result = new DataPipeline(users)
  .filter(user => user.age > 25)
  .map(user => ({ ...user, ageGroup: user.age > 30 ? 'senior' : 'mid' }))
  .groupBy('ageGroup')
  .execute();

console.log(result);
// {
//   'mid': [{ name: '李四', age: 30, department: '市场', ageGroup: 'mid' }, ...]
// }
```

## 性能优化与最佳实践

### 1. 避免不必要的中间数组

```javascript
// 不好的做法：创建多个中间数组
const badResult = data
  .filter(item => item.active)
  .map(item => item.value)
  .filter(value => value > 10)
  .map(value => value * 2);

// 好的做法：减少中间数组
const goodResult = data
  .filter(item => item.active && item.value > 10)
  .map(item => item.value * 2);

// 更好的做法：使用 reduce 一次性完成
const bestResult = data.reduce((acc, item) => {
  if (item.active && item.value > 10) {
    acc.push(item.value * 2);
  }
  return acc;
}, []);
```

### 2. 合理使用方法顺序

```javascript
const largeArray = Array.from({ length: 100000 }, (_, i) => ({
  id: i,
  value: Math.random() * 100,
  active: Math.random() > 0.5
}));

// 先过滤再映射，减少后续处理的数据量
const optimized = largeArray
  .filter(item => item.active) // 先过滤
  .filter(item => item.value > 50) // 再过滤
  .map(item => item.value * 2) // 最后映射
  .slice(0, 100); // 只取前100个
```

### 3. 缓存和记忆化

```javascript
// 缓存计算结果
function memoizedMap(array, transformer) {
  const cache = new Map();

  return array.map(item => {
    const cacheKey = JSON.stringify(item);
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }

    const result = transformer(item);
    cache.set(cacheKey, result);
    return result;
  });
}

// 缓存筛选结果
function createCachedFilter() {
  const cache = new Map();

  return function(array, predicate) {
    const cacheKey = array.join(',') + predicate.toString();
    if (cache.has(cacheKey)) {
      return cache.get(cacheKey);
    }

    const result = array.filter(predicate);
    cache.set(cacheKey, result);
    return result;
  };
}
```

### 4. 处理大数据集

```javascript
// 分批处理大数据集
function processLargeDataset(dataset, batchSize = 1000, processor) {
  const results = [];

  for (let i = 0; i < dataset.length; i += batchSize) {
    const batch = dataset.slice(i, i + batchSize);
    const processedBatch = processor(batch);
    results.push(...processedBatch);

    // 让浏览器有时间处理其他任务
    if (i % (batchSize * 10) === 0) {
      await new Promise(resolve => setTimeout(resolve, 0));
    }
  }

  return results;
}

// 使用示例
const largeDataset = Array.from({ length: 100000 }, (_, i) => ({
  id: i,
  value: Math.random() * 100
}));

const processedData = await processLargeDataset(
  largeDataset,
  1000,
  batch => batch
    .filter(item => item.value > 50)
    .map(item => item.value * 2)
);
```

## 错误处理和边界情况

### 1. 处理空数组

```javascript
// 安全的 reduce
function safeReduce(array, reducer, initialValue) {
  if (array.length === 0) {
    return initialValue !== undefined ? initialValue : null;
  }
  return array.reduce(reducer, initialValue);
}

// 安全的 filter
function safeFilter(array, predicate) {
  if (!Array.isArray(array)) {
    console.warn('Expected array, got:', typeof array);
    return [];
  }
  return array.filter(predicate);
}
```

### 2. 处理异常值

```javascript
// 防御性编程
function robustMap(array, transformer) {
  return array.map((item, index) => {
    try {
      return transformer(item, index);
    } catch (error) {
      console.error(`Error processing item at index ${index}:`, error);
      return null; // 或返回默认值
    }
  }).filter(item => item !== null); // 过滤掉处理失败的项
}

// 处理 undefined 和 null
function cleanFilter(array, predicate) {
  return array
    .filter(item => item != null) // 过滤掉 null 和 undefined
    .filter(predicate);
}
```

### 3. 类型安全

```javascript
// 类型检查的 map
function typedMap(array, transformer, expectedType) {
  return array.map((item, index) => {
    const result = transformer(item, index);
    if (typeof result !== expectedType) {
      throw new TypeError(
        `Expected ${expectedType} at index ${index}, got ${typeof result}`
      );
    }
    return result;
  });
}

// 使用示例
const numbers = [1, 2, 3, 4, 5];
const strings = typedMap(numbers, num => String(num), 'string');
console.log(strings); // ['1', '2', '3', '4', '5']
```

## 实际项目应用案例

### 1. 电商购物车系统

```javascript
class ShoppingCart {
  constructor(items = []) {
    this.items = items;
  }

  // 添加商品
  addItem(product, quantity = 1) {
    const existingItem = this.items.find(item => item.id === product.id);

    if (existingItem) {
      existingItem.quantity += quantity;
    } else {
      this.items.push({
        ...product,
        quantity,
        addedAt: new Date()
      });
    }

    return this;
  }

  // 移除商品
  removeItem(productId) {
    this.items = this.items.filter(item => item.id !== productId);
    return this;
  }

  // 更新数量
  updateQuantity(productId, quantity) {
    const item = this.items.find(item => item.id === productId);
    if (item) {
      item.quantity = Math.max(0, quantity);
    }
    return this;
  }

  // 计算总价
  getTotal() {
    return this.items.reduce((total, item) => {
      return total + (item.price * item.quantity);
    }, 0);
  }

  // 应用折扣
  applyDiscount(discountFunction) {
    this.items = this.items.map(item => ({
      ...item,
      finalPrice: discountFunction(item.price, item.quantity)
    }));
    return this;
  }

  // 获取商品列表
  getItems() {
    return this.items
      .filter(item => item.quantity > 0)
      .map(item => ({
        id: item.id,
        name: item.name,
        price: item.price,
        finalPrice: item.finalPrice || item.price,
        quantity: item.quantity,
        subtotal: (item.finalPrice || item.price) * item.quantity
      }));
  }

  // 获取摘要信息
  getSummary() {
    const items = this.getItems();
    const subtotal = this.getTotal();
    const totalItems = items.reduce((sum, item) => sum + item.quantity, 0);

    return {
      itemCount: totalItems,
      uniqueItems: items.length,
      subtotal,
      items
    };
  }
}

// 使用示例
const cart = new ShoppingCart();

cart
  .addItem(
    { id: 1, name: 'iPhone', price: 999 },
    2
  )
  .addItem(
    { id: 2, name: 'MacBook', price: 1999 },
    1
  )
  .applyDiscount((price, quantity) => {
    if (quantity >= 2) {
      return price * 0.9; // 10% 折扣
    }
    return price;
  });

const summary = cart.getSummary();
console.log(summary);
```

### 2. 数据分析和报表系统

```javascript
class DataAnalyzer {
  constructor(data) {
    this.data = data;
  }

  // 按条件筛选
  filterBy(predicate) {
    this.data = this.data.filter(predicate);
    return this;
  }

  // 转换数据
  transform(transformer) {
    this.data = this.data.map(transformer);
    return this;
  }

  // 分组统计
  groupBy(groupKey, valueKey, aggregation = 'sum') {
    const grouped = this.data.reduce((acc, item) => {
      const key = item[groupKey];
      const value = item[valueKey];

      if (!acc[key]) {
        acc[key] = { count: 0, sum: 0, values: [] };
      }

      acc[key].count++;
      acc[key].sum += value;
      acc[key].values.push(value);

      return acc;
    }, {});

    // 计算聚合值
    Object.keys(grouped).forEach(key => {
      const group = grouped[key];
      group.average = group.sum / group.count;
      group.min = Math.min(...group.values);
      group.max = Math.max(...group.values);
    });

    return grouped;
  }

  // 时间序列分析
  analyzeTimeSeries(dateKey, valueKey) {
    const sortedData = [...this.data].sort((a, b) =>
      new Date(a[dateKey]) - new Date(b[dateKey])
    );

    return {
      trend: this.calculateTrend(sortedData, valueKey),
      movingAverage: this.calculateMovingAverage(sortedData, valueKey, 7),
      growth: this.calculateGrowth(sortedData, valueKey)
    };
  }

  calculateTrend(data, valueKey) {
    const values = data.map(item => item[valueKey]);
    const n = values.length;

    // 简单线性回归计算趋势
    const sumX = (n * (n - 1)) / 2;
    const sumY = values.reduce((sum, val) => sum + val, 0);
    const sumXY = values.reduce((sum, val, i) => sum + (i * val), 0);
    const sumX2 = (n * (n - 1) * (2 * n - 1)) / 6;

    const slope = (n * sumXY - sumX * sumY) / (n * sumX2 - sumX * sumX);
    const intercept = (sumY - slope * sumX) / n;

    return { slope, intercept };
  }

  calculateMovingAverage(data, valueKey, windowSize) {
    const values = data.map(item => item[valueKey]);
    const movingAverages = [];

    for (let i = windowSize - 1; i < values.length; i++) {
      const window = values.slice(i - windowSize + 1, i + 1);
      const average = window.reduce((sum, val) => sum + val, 0) / windowSize;
      movingAverages.push({
        date: data[i][dateKey],
        average
      });
    }

    return movingAverages;
  }

  calculateGrowth(data, valueKey) {
    const values = data.map(item => item[valueKey]);
    const growthRates = [];

    for (let i = 1; i < values.length; i++) {
      const previousValue = values[i - 1];
      const currentValue = values[i];
      const growthRate = ((currentValue - previousValue) / previousValue) * 100;

      growthRates.push({
        date: data[i][dateKey],
        previousValue,
        currentValue,
        growthRate
      });
    }

    return growthRates;
  }
}

// 使用示例
const salesData = [
  { date: '2024-01-01', product: 'A', revenue: 100, region: 'East' },
  { date: '2024-01-02', product: 'A', revenue: 120, region: 'East' },
  { date: '2024-01-03', product: 'A', revenue: 110, region: 'East' },
  { date: '2024-01-01', product: 'B', revenue: 80, region: 'West' },
  { date: '2024-01-02', product: 'B', revenue: 90, region: 'West' }
];

const analyzer = new DataAnalyzer(salesData);

// 按地区分组统计
const regionalStats = analyzer.groupBy('region', 'revenue');
console.log(regionalStats);

// 时间序列分析
const timeAnalysis = analyzer.analyzeTimeSeries('date', 'revenue');
console.log(timeAnalysis);
```

### 3. 实时数据流处理

```javascript
class DataStreamProcessor {
  constructor(options = {}) {
    this.buffer = [];
    this.maxBufferSize = options.maxBufferSize || 1000;
    this.processors = [];
    this.filters = [];
  }

  // 添加数据
  addData(data) {
    this.buffer.push(...data);

    // 保持缓冲区大小
    if (this.buffer.length > this.maxBufferSize) {
      this.buffer = this.buffer.slice(-this.maxBufferSize);
    }

    return this;
  }

  // 添加过滤器
  addFilter(filter) {
    this.filters.push(filter);
    return this;
  }

  // 添加处理器
  addProcessor(processor) {
    this.processors.push(processor);
    return this;
  }

  // 处理数据流
  process() {
    let processedData = [...this.buffer];

    // 应用所有过滤器
    this.filters.forEach(filter => {
      processedData = processedData.filter(filter);
    });

    // 应用所有处理器
    this.processors.forEach(processor => {
      processedData = processedData.map(processor);
    });

    return processedData;
  }

  // 获取统计信息
  getStatistics() {
    const processedData = this.process();

    return {
      totalItems: this.buffer.length,
      filteredItems: processedData.length,
      filterRate: ((this.buffer.length - processedData.length) / this.buffer.length) * 100,
      averageValue: processedData.reduce((sum, item) => sum + (item.value || 0), 0) / processedData.length
    };
  }

  // 清空缓冲区
  clear() {
    this.buffer = [];
    return this;
  }
}

// 使用示例
const processor = new DataStreamProcessor({ maxBufferSize: 500 });

processor
  .addFilter(item => item.value > 10) // 过滤值小于等于10的项
  .addFilter(item => item.category === 'important') // 只保留重要类别
  .addProcessor(item => ({
    ...item,
    processed: true,
    timestamp: new Date().toISOString()
  }));

// 添加实时数据
processor.addData([
  { id: 1, value: 15, category: 'important' },
  { id: 2, value: 5, category: 'normal' },
  { id: 3, value: 25, category: 'important' }
]);

// 处理数据
const results = processor.process();
console.log(results);

// 获取统计信息
const stats = processor.getStatistics();
console.log(stats);
```

## 调试和测试技巧

### 1. 调试链式调用

```javascript
// 添加调试日志
function debug(label) {
  return function(data) {
    console.log(`${label}:`, data);
    return data;
  };
}

// 使用示例
const result = data
  .filter(item => item.active)
  .pipe(debug('After filter')) // 调试过滤后的数据
  .map(item => item.value * 2)
  .pipe(debug('After map')) // 调试映射后的数据
  .reduce((sum, val) => sum + val, 0);
```

### 2. 单元测试高阶函数

```javascript
// 测试 map 函数
describe('Array map', () => {
  test('should double each number', () => {
    const numbers = [1, 2, 3, 4, 5];
    const doubled = numbers.map(num => num * 2);
    expect(doubled).toEqual([2, 4, 6, 8, 10]);
  });

  test('should handle empty array', () => {
    const empty = [];
    const result = empty.map(num => num * 2);
    expect(result).toEqual([]);
  });

  test('should preserve index in callback', () => {
    const items = ['a', 'b', 'c'];
    const withIndex = items.map((item, index) => ({ item, index }));
    expect(withIndex).toEqual([
      { item: 'a', index: 0 },
      { item: 'b', index: 1 },
      { item: 'c', index: 2 }
    ]);
  });
});

// 测试 filter 函数
describe('Array filter', () => {
  test('should filter even numbers', () => {
    const numbers = [1, 2, 3, 4, 5, 6];
    const evens = numbers.filter(num => num % 2 === 0);
    expect(evens).toEqual([2, 4, 6]);
  });

  test('should return empty array if no matches', () => {
    const numbers = [1, 3, 5];
    const evens = numbers.filter(num => num % 2 === 0);
    expect(evens).toEqual([]);
  });
});

// 测试 reduce 函数
describe('Array reduce', () => {
  test('should sum numbers', () => {
    const numbers = [1, 2, 3, 4, 5];
    const sum = numbers.reduce((acc, num) => acc + num, 0);
    expect(sum).toBe(15);
  });

  test('should handle empty array with initial value', () => {
    const empty = [];
    const result = empty.reduce((acc, num) => acc + num, 0);
    expect(result).toBe(0);
  });
});
```

## 总结

JavaScript 数组高阶方法是现代前端开发的重要工具，它们不仅提供了强大的数据处理能力，还体现了函数式编程的思想。通过掌握 `map`、`filter`、`reduce` 等核心方法，你可以：

1. **编写更简洁的代码**：用声明式的方式描述数据处理逻辑
2. **提高代码可读性**：方法链式调用让代码意图更加清晰
3. **增强代码可维护性**：纯函数和不可变性让代码更易于测试和维护
4. **提升开发效率**：减少样板代码，专注于业务逻辑

### 学习建议：

1. **基础掌握**：熟悉每个高阶方法的基本用法和参数
2. **组合练习**：多练习方法的组合使用，理解数据流
3. **性能意识**：了解不同方法的性能特性，合理选择
4. **实际应用**：在真实项目中应用这些方法，解决实际问题
5. **持续优化**：不断重构和优化代码，提升代码质量

通过不断练习和实践，你将能够熟练运用数组高阶方法，编写出更加优雅、高效的 JavaScript 代码。