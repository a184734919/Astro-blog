---
title: JavaScript 对象解构与展开
published: 2022-03-25
description: '解构赋值和展开运算符的应用的详细介绍和学习笔记'
image: ''
tags: ["JS基础"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

JavaScript 对象解构与展开是 ES6 引入的重要特性，它们极大地简化了代码编写，提高了代码的可读性和可维护性。解构赋值允许我们从对象或数组中提取值并赋给变量，而展开运算符则提供了一种简洁的方式来展开对象和数组。

## 对象解构

### 基本解构

```javascript
// 基本对象解构
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 从对象中提取属性
const { name, age, email } = user;

console.log(name);  // '张三'
console.log(age);   // 25
console.log(email); // 'zhangsan@example.com'

// 等价于
// const name = user.name;
// const age = user.age;
// const email = user.email;
```

### 重命名变量

```javascript
const user = {
  name: '张三',
  age: 25
};

// 解构时重命名变量
const { name: userName, age: userAge } = user;

console.log(userName); // '张三'
console.log(userAge);  // 25

console.log(name);     // ReferenceError: name is not defined
console.log(age);      // ReferenceError: age is not defined
```

### 默认值

```javascript
const user = {
  name: '张三',
  age: 25
};

// 为解构的属性设置默认值
const { name, age, gender = '男', country = '中国' } = user;

console.log(name);    // '张三'
console.log(age);     // 25
console.log(gender);  // '男'
console.log(country); // '中国'

// 属性存在但值为 undefined 时使用默认值
const user2 = {
  name: '李四',
  email: undefined
};

const { name: name2, email: email2 = 'default@example.com' } = user2;
console.log(email2); // 'default@example.com'

// 属性值为 null 时不会使用默认值
const user3 = {
  name: '王五',
  email: null
};

const { name: name3, email: email3 = 'default@example.com' } = user3;
console.log(email3); // null
```

### 嵌套对象解构

```javascript
const user = {
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    street: '朝阳区',
    zipCode: '100000'
  }
};

// 解构嵌套对象
const {
  name,
  address: {
    city,
    street,
    zipCode
  }
} = user;

console.log(name);     // '张三'
console.log(city);     // '北京'
console.log(street);   // '朝阳区'
console.log(zipCode);  // '100000'

// console.log(address); // ReferenceError: address is not defined

// 保留外层对象
const {
  name: userName,
  address: userAddress,
  address: { city: userCity }
} = user;

console.log(userName);    // '张三'
console.log(userAddress); // { city: '北京', street: '朝阳区', zipCode: '100000' }
console.log(userCity);    // '北京'
```

### 解构剩余属性

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  phone: '13800138000',
  address: '北京'
};

// 解构部分属性，剩余属性收集到新对象
const { name, age, ...rest } = user;

console.log(name);  // '张三'
console.log(age);   // 25
console.log(rest);  // { email: 'zhangsan@example.com', phone: '13800138000', address: '北京' }
```

### 函数参数解构

```javascript
// 基本函数参数解构
function greetUser({ name, age }) {
  console.log(`你好, ${name}! 你今年 ${age} 岁了。`);
}

const user = { name: '张三', age: 25 };
greetUser(user); // '你好, 张三! 你今年 25 岁了。'

// 设置默认值
function createUser({ name, age = 18, gender = '未知' } = {}) {
  return { name, age, gender };
}

console.log(createUser({ name: '李四' }));
// { name: '李四', age: 18, gender: '未知' }

console.log(createUser({ name: '王五', age: 30, gender: '女' }));
// { name: '王五', age: 30, gender: '女' }

// 不传递参数时的默认值
console.log(createUser()); // { name: undefined, age: 18, gender: '未知' }
```

### 动态属性名解构

```javascript
const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
};

// 使用变量作为属性名
const key = 'name';
const { [key]: userName } = user;

console.log(userName); // '张三'

// 结合计算属性名
const propName = 'email';
const { [propName]: userEmail } = user;

console.log(userEmail); // 'zhangsan@example.com'
```

## 数组解构

### 基本数组解构

```javascript
const colors = ['red', 'green', 'blue', 'yellow'];

// 按顺序解构
const [first, second, third] = colors;

console.log(first);  // 'red'
console.log(second); // 'green'
console.log(third);  // 'blue'

// 等价于
// const first = colors[0];
// const second = colors[1];
// const third = colors[2];
```

### 跳过元素

```javascript
const colors = ['red', 'green', 'blue', 'yellow', 'purple'];

// 跳过某些元素
const [first, , third, , fifth] = colors;

console.log(first);  // 'red'
console.log(third);  // 'blue'
console.log(fifth);  // 'purple'

// 只取第一个和最后一个
const [head, ...middle, tail] = colors;
// 注意：这种写法会报错，rest 参数必须是最后一个

// 正确的方式
const [head, ...rest] = colors;
console.log(head); // 'red'
console.log(rest); // ['green', 'blue', 'yellow', 'purple']
```

### 默认值

```javascript
const numbers = [1, 2];

// 设置默认值
const [a, b, c = 3, d = 4] = numbers;

console.log(a); // 1
console.log(b); // 2
console.log(c); // 3
console.log(d); // 4

// 解构 undefined
const [x = 10, y = 20, z = 30] = [undefined, , null];
console.log(x); // 10 (undefined 使用默认值)
console.log(y); // 20 (跳过，使用默认值)
console.log(z); // null (null 不使用默认值)
```

### 剩余元素

```javascript
const numbers = [1, 2, 3, 4, 5];

// 获取前几个元素，剩余元素收集到数组
const [first, second, ...rest] = numbers;

console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]

// 交换变量
let a = 1;
let b = 2;
[a, b] = [b, a];
console.log(a); // 2
console.log(b); // 1
```

### 嵌套数组解构

```javascript
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

// 解构嵌套数组
const [[a, b, c], [d, e, f], [g, h, i]] = matrix;

console.log(a); // 1
console.log(e); // 5
console.log(i); // 9

// 选择性解构
const [
  [firstRowFirst, , firstRowThird],
  [, secondRowSecond, ]
] = matrix;

console.log(firstRowFirst);  // 1
console.log(firstRowThird);  // 3
console.log(secondRowSecond); // 5
```

### 数组解构的实际应用

```javascript
// 返回多个值
function getCoordinates() {
  return [100, 200];
}

const [x, y] = getCoordinates();
console.log(x, y); // 100 200

// 解构正则表达式结果
const dateStr = '2025-01-14';
const regex = /(\d{4})-(\d{2})-(\d{2})/;
const [, year, month, day] = regex.exec(dateStr);

console.log(year);  // '2025'
console.log(month); // '01'
console.log(day);   // '14'

// 函数返回多个处理后的值
function processData(data) {
  const sum = data.reduce((a, b) => a + b, 0);
  const avg = sum / data.length;
  const max = Math.max(...data);
  const min = Math.min(...data);

  return [sum, avg, max, min];
}

const [total, average, maximum, minimum] = processData([1, 2, 3, 4, 5]);
console.log(total);    // 15
console.log(average);  // 3
console.log(maximum);  // 5
console.log(minimum);  // 1
```

## 对象展开

### 基本展开

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// 展开对象
const merged = { ...obj1, ...obj2 };

console.log(merged); // { a: 1, b: 2, c: 3, d: 4 }

// 展开并添加新属性
const extended = { ...obj1, e: 5, f: 6 };
console.log(extended); // { a: 1, b: 2, e: 5, f: 6 }
```

### 对象合并

```javascript
const defaults = {
  theme: 'light',
  language: 'zh-CN',
  timeout: 5000
};

const userConfig = {
  theme: 'dark',
  timeout: 10000
};

// 合并配置，后面的覆盖前面的
const config = { ...defaults, ...userConfig };

console.log(config);
// { theme: 'dark', language: 'zh-CN', timeout: 10000 }

// 局部更新对象
const user = {
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    street: '朝阳区'
  }
};

const updatedUser = { ...user, age: 26 };
console.log(updatedUser);
// { name: '张三', age: 26, address: { city: '北京', street: '朝阳区' } }
```

### 属性覆盖顺序

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

// 后面的属性覆盖前面的
const merged1 = { ...obj1, ...obj2 };
console.log(merged1); // { a: 1, b: 3, c: 4 }

const merged2 = { ...obj2, ...obj1 };
console.log(merged2); // { a: 1, b: 2, c: 4 }

// 直接属性覆盖
const merged3 = { a: 1, b: 2, b: 3 };
console.log(merged3); // { a: 1, b: 3 }
```

### 条件性展开

```javascript
const baseConfig = {
  apiUrl: 'https://api.example.com',
  timeout: 5000
};

const isDevelopment = true;
const debug = false;

// 根据条件展开属性
const config = {
  ...baseConfig,
  ...(isDevelopment && { mode: 'development' }),
  ...(debug && { logLevel: 'debug' })
};

console.log(config);
// { apiUrl: 'https://api.example.com', timeout: 5000, mode: 'development' }

// 使用三元运算符
const config2 = {
  ...baseConfig,
  mode: isDevelopment ? 'development' : 'production'
};
console.log(config2.mode); // 'development'
```

## 数组展开

### 基本展开

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// 展开数组
const merged = [...arr1, ...arr2];
console.log(merged); // [1, 2, 3, 4, 5, 6]

// 展开并添加新元素
const extended = [...arr1, 4, 5, 6];
console.log(extended); // [1, 2, 3, 4, 5, 6]
```

### 数组复制

```javascript
const original = [1, 2, 3];

// 浅拷贝数组
const copy = [...original];

console.log(copy);    // [1, 2, 3]
console.log(original === copy); // false

// 修改拷贝不影响原数组
copy.push(4);
console.log(original); // [1, 2, 3]
console.log(copy);     // [1, 2, 3, 4]

// 与 slice 对比
const copy2 = original.slice();
console.log(copy2); // [1, 2, 3]
```

### 数组拼接

```javascript
const arr1 = [1, 2];
const arr2 = [3, 4];
const arr3 = [5, 6];

// 多个数组拼接
const concatenated = [...arr1, ...arr2, ...arr3];
console.log(concatenated); // [1, 2, 3, 4, 5, 6]

// 与 concat 对比
const result = arr1.concat(arr2, arr3);
console.log(result); // [1, 2, 3, 4, 5, 6]

// 混合使用展开和普通元素
const mixed = [0, ...arr1, 3, ...arr2, 7];
console.log(mixed); // [0, 1, 2, 3, 3, 4, 7]
```

### 在函数调用中展开

```javascript
function sum(a, b, c, d) {
  return a + b + c + d;
}

const numbers = [1, 2, 3, 4];

// 展开数组作为函数参数
const result = sum(...numbers);
console.log(result); // 10

// 等价于
// sum(1, 2, 3, 4)

// 部分展开
const result2 = sum(1, ...[2, 3], 4);
console.log(result2); // 10

// Math.max 示例
const numbers2 = [1, 5, 3, 9, 2];
const max = Math.max(...numbers2);
console.log(max); // 9

// Math.min 示例
const min = Math.min(...numbers2);
console.log(min); // 1
```

### 字符串展开

```javascript
const str = 'Hello';

// 将字符串展开为字符数组
const chars = [...str];
console.log(chars); // ['H', 'e', 'l', 'l', 'o']

// 与 split 对比
const chars2 = str.split('');
console.log(chars2); // ['H', 'e', 'l', 'l', 'o']

// 区别：展开能正确处理 Unicode 字符
const emoji = '😀🎉';
const emojiArray = [...emoji];
console.log(emojiArray); // ['😀', '🎉']

const emojiArray2 = emoji.split('');
console.log(emojiArray2); // ['\ud83d', '\ude00', '\ud83c', '\udf89']
```

### Set 和 Map 展开

```javascript
// Set 展开
const mySet = new Set([1, 2, 3, 3, 2, 1]);
const setArray = [...mySet];
console.log(setArray); // [1, 2, 3] (自动去重)

// 使用展开创建数组去重函数
function unique(array) {
  return [...new Set(array)];
}

console.log(unique([1, 2, 2, 3, 3, 3])); // [1, 2, 3]

// Map 展开
const myMap = new Map([
  ['name', '张三'],
  ['age', 25]
]);

const mapEntries = [...myMap];
console.log(mapEntries); // [['name', '张三'], ['age', 25]]
```

## 混合解构

### 对象和数组混合解构

```javascript
const data = {
  id: 1,
  name: '张三',
  scores: [85, 90, 78, 92],
  info: {
    age: 25,
    email: 'zhangsan@example.com'
  }
};

// 混合解构
const {
  id,
  name,
  scores: [math, chinese, english, physics],
  info: { age, email }
} = data;

console.log(id);       // 1
console.log(name);     // '张三'
console.log(math);     // 85
console.log(english);  // 78
console.log(age);      // 25
console.log(email);    // 'zhangsan@example.com'
```

### 复杂数据结构解构

```javascript
const apiResponse = {
  status: 'success',
  data: {
    users: [
      { id: 1, name: '张三' },
      { id: 2, name: '李四' }
    ],
    pagination: {
      page: 1,
      pageSize: 10,
      total: 100
    }
  }
};

// 解构复杂数据
const {
  status,
  data: {
    users: [
      { name: firstUserName },
      { name: secondUserName }
    ],
    pagination: { page, pageSize, total }
  }
} = apiResponse;

console.log(status);         // 'success'
console.log(firstUserName);  // '张三'
console.log(secondUserName); // '李四'
console.log(page);           // 1
console.log(total);          // 100
```

## 实际应用

### 1. 配置管理

```javascript
// 默认配置
const defaultConfig = {
  api: {
    baseUrl: 'https://api.example.com',
    timeout: 5000
  },
  ui: {
    theme: 'light',
    language: 'zh-CN'
  }
};

// 用户配置
const userConfig = {
  api: {
    timeout: 10000
  },
  ui: {
    theme: 'dark'
  }
};

// 深度合并配置
function deepMerge(target, source) {
  const result = { ...target };

  for (const key in source) {
    if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
      result[key] = deepMerge(target[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  }

  return result;
}

const finalConfig = deepMerge(defaultConfig, userConfig);
console.log(finalConfig);
```

### 2. 状态更新

```javascript
// React 状态更新示例
const initialState = {
  user: {
    name: '张三',
    age: 25
  },
  settings: {
    theme: 'light',
    notifications: true
  }
};

// 更新嵌套状态
function updateState(state, path, value) {
  const keys = path.split('.');
  const newState = { ...state };
  let current = newState;

  for (let i = 0; i < keys.length - 1; i++) {
    current[keys[i]] = { ...current[keys[i]] };
    current = current[keys[i]];
  }

  current[keys[keys.length - 1]] = value;
  return newState;
}

const updatedState = updateState(initialState, 'user.age', 26);
console.log(updatedState);
```

### 3. 数据转换

```javascript
// API 数据转换
const apiData = [
  { user_id: 1, user_name: '张三', user_age: 25 },
  { user_id: 2, user_name: '李四', user_age: 30 }
];

// 转换为驼峰命名
const transformedData = apiData.map(({ user_id, user_name, user_age }) => ({
  id: user_id,
  name: user_name,
  age: user_age
}));

console.log(transformedData);
// [
//   { id: 1, name: '张三', age: 25 },
//   { id: 2, name: '李四', age: 30 }
// ]
```

### 4. 组件属性处理

```javascript
// 函数组件 props 处理
function Button({ title, icon, size = 'medium', variant = 'primary', ...rest }) {
  return {
    text: title,
    icon,
    size,
    variant,
    ...rest
  };
}

const buttonProps = Button({
  title: '提交',
  icon: 'send',
  size: 'large',
  className: 'custom-class',
  disabled: true
});

console.log(buttonProps);
// {
//   text: '提交',
//   icon: 'send',
//   size: 'large',
//   variant: 'primary',
//   className: 'custom-class',
//   disabled: true
// }
```

### 5. 工具函数

```javascript
// 对象选择器
function pick(obj, keys) {
  return keys.reduce((result, key) => {
    if (key in obj) {
      result[key] = obj[key];
    }
    return result;
  }, {});
}

const user = {
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  phone: '13800138000',
  address: '北京'
};

const selectedUser = pick(user, ['name', 'email']);
console.log(selectedUser); // { name: '张三', email: 'zhangsan@example.com' }

// 对象排除器
function omit(obj, keys) {
  const result = { ...obj };
  keys.forEach(key => {
    delete result[key];
  });
  return result;
}

const omitUser = omit(user, ['phone', 'address']);
console.log(omitUser); // { name: '张三', age: 25, email: 'zhangsan@example.com' }
```

## 性能考虑

### 展开运算符 vs Object.assign

```javascript
const obj1 = { a: 1, b: 2 };
const obj2 = { c: 3, d: 4 };

// 展开运算符
const merged1 = { ...obj1, ...obj2 };

// Object.assign
const merged2 = Object.assign({}, obj1, obj2);

// 性能比较：在大多数情况下，展开运算符性能更好
// 但对于大对象，Object.assign 可能更快

// 基准测试
function testSpread() {
  for (let i = 0; i < 10000; i++) {
    const result = { ...obj1, ...obj2 };
  }
}

function testAssign() {
  for (let i = 0; i < 10000; i++) {
    const result = Object.assign({}, obj1, obj2);
  }
}
```

### 避免不必要的展开

```javascript
// 不好的方式：不必要的展开
function processUser(user) {
  const newUser = { ...user }; // 不必要的拷贝
  newUser.processed = true;
  return newUser;
}

// 好的方式：直接创建新对象
function processUser(user) {
  return { ...user, processed: true };
}

// 不好的方式：大对象展开
const bigObject = { /* 大量数据 */ };
const result = { ...bigObject, newProp: 'value' };

// 好的方式：只添加需要的属性
const result = Object.assign({}, bigObject, { newProp: 'value' });
```

## 注意事项

1. **浅拷贝**：展开运算符只进行浅拷贝，嵌套对象仍共享引用。

2. **原型链**：展开运算符不会复制原型链上的属性。

3. **不可枚举属性**：展开运算符只会展开对象自身的可枚举属性。

4. **Symbol 属性**：展开运算符会展开 Symbol 属性。

5. **解构未定义值**：对 `null` 或 `undefined` 进行解构会抛出错误。

6. **循环引用**：展开运算符处理循环引用时会抛出错误。

7. **性能影响**：频繁使用展开运算符可能影响性能，特别是对大对象。

8. **剩余参数位置**：在解构中，剩余参数必须是最后一个。

## 最佳实践

1. **优先使用展开运算符**：相比 `Object.assign()` 和 `concat()`，展开运算符语法更简洁。

2. **合理使用默认值**：为解构的属性设置合理的默认值，提高代码健壮性。

3. **重命名有意义的变量**：解构时使用有意义的变量名，提高代码可读性。

4. **避免过度嵌套**：复杂的嵌套解构可能降低代码可读性。

5. **使用剩余参数**：使用剩余参数收集不需要的属性，便于后续处理。

6. **函数参数解构**：在函数参数中使用解构，使接口更清晰。

7. **条件性展开**：合理使用条件展开，避免不必要的属性。

8. **类型安全**：在 TypeScript 中使用类型注解，提高类型安全。

## 总结

通过本文的学习，相信你已经对 JavaScript 对象解构与展开有了更深入的理解。解构赋值和展开运算符是现代 JavaScript 开发中不可或缺的工具，它们大大简化了代码编写，提高了代码的可读性和可维护性。在实际开发中，合理运用这些特性，可以编写出更优雅、更高效的代码。记住要结合具体场景选择合适的方式，并注意性能和边界情况的处理。