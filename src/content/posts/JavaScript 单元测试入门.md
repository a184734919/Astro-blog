---
title: JavaScript 单元测试入门
published: 2022-10-21
description: 'Jest 测试框架基础的详细介绍和学习笔记'
image: ''
tags: ["测试"]
category: 'JavaScript'
draft: false
lang: 'zh-CN'
---

## 概述

单元测试是软件开发中的重要实践，它可以帮助我们确保代码的正确性、提高代码质量、减少回归错误。Jest 是目前最流行的 JavaScript 测试框架之一，本文将详细介绍 Jest 的使用方法和测试最佳实践。

## 核心概念

### 单元测试的价值

1. **代码质量保证**：发现和修复 bug，确保代码行为符合预期
2. **重构安全网**：为代码重构提供安全保障
3. **文档作用**：测试用例本身就是代码的使用文档
4. **设计指导**：可测试的代码通常设计更好

### Jest 的核心特性

- **零配置**：开箱即用，无需复杂配置
- **快照测试**：支持 UI 组件的快照测试
- **Mock 功能**：内置强大的 mock 系统
- **并行测试**：自动并行运行测试，提高测试速度
- **覆盖率报告**：内置代码覆盖率工具
- **断言库**：内置丰富的断言方法

## 基本用法

### Jest 基础配置

```bash
# 安装 Jest
npm install --save-dev jest

# 或使用 yarn
yarn add --dev jest

# 在 package.json 中添加测试脚本
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  }
}
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'node',
  coverageDirectory: 'coverage',
  collectCoverageFrom: [
    'src/**/*.js',
    '!src/**/*.test.js',
    '!src/**/*.spec.js'
  ],
  testMatch: [
    '**/__tests__/**/*.js',
    '**/?(*.)+(spec|test).js'
  ],
  moduleFileExtensions: ['js', 'json'],
  verbose: true
};
```

### 基础测试用例

```javascript
// math.js - 被测试的函数
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error('Division by zero');
  }
  return a / b;
}

module.exports = { add, subtract, multiply, divide };
```

```javascript
// math.test.js - 测试文件
const { add, subtract, multiply, divide } = require('./math');

describe('数学函数测试', () => {
  // 基础断言
  describe('add 函数', () => {
    test('应该正确相加两个正数', () => {
      expect(add(2, 3)).toBe(5);
    });

    test('应该正确相加负数', () => {
      expect(add(-2, -3)).toBe(-5);
    });

    test('应该正确相加正数和负数', () => {
      expect(add(2, -3)).toBe(-1);
    });
  });

  describe('subtract 函数', () => {
    test('应该正确相减', () => {
      expect(subtract(5, 3)).toBe(2);
    });
  });

  describe('multiply 函数', () => {
    test('应该正确相乘', () => {
      expect(multiply(3, 4)).toBe(12);
    });
  });

  describe('divide 函数', () => {
    test('应该正确相除', () => {
      expect(divide(10, 2)).toBe(5);
    });

    test('除数为零时应该抛出错误', () => {
      expect(() => divide(10, 0)).toThrow('Division by zero');
    });
  });
});
```

### 异步测试

```javascript
// asyncFunctions.js
async function fetchData() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({ data: 'success' });
    }, 100);
  });
}

async function fetchWithError() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error('Network error'));
    }, 100);
  });
}

function callbackStyle(callback) {
  setTimeout(() => {
    callback(null, 'success');
  }, 100);
}

function callbackStyleError(callback) {
  setTimeout(() => {
    callback(new Error('Callback error'));
  }, 100);
}

module.exports = {
  fetchData,
  fetchWithError,
  callbackStyle,
  callbackStyleError
};
```

```javascript
// asyncFunctions.test.js
const {
  fetchData,
  fetchWithError,
  callbackStyle,
  callbackStyleError
} = require('./asyncFunctions');

describe('异步函数测试', () => {
  // Promise 测试
  describe('fetchData', () => {
    test('应该成功返回数据', async () => {
      const result = await fetchData();
      expect(result).toEqual({ data: 'success' });
    });

    test('使用 .resolves 断言', () => {
      return expect(fetchData()).resolves.toEqual({ data: 'success' });
    });
  });

  describe('fetchWithError', () => {
    test('应该抛出错误', async () => {
      await expect(fetchWithError()).rejects.toThrow('Network error');
    });

    test('使用 .rejects 断言', () => {
      return expect(fetchWithError()).rejects.toThrow('Network error');
    });
  });

  // 回调函数测试
  describe('callbackStyle', () => {
    test('应该成功回调', (done) => {
      callbackStyle((error, result) => {
        expect(error).toBeNull();
        expect(result).toBe('success');
        done();
      });
    });
  });

  describe('callbackStyleError', () => {
    test('应该错误回调', (done) => {
      callbackStyleError((error) => {
        expect(error).toBeInstanceOf(Error);
        expect(error.message).toBe('Callback error');
        done();
      });
    });
  });
});
```

## 实际应用

### Mock 和 Spy

```javascript
// userService.js
const axios = require('axios');

class UserService {
  constructor(httpClient = axios) {
    this.httpClient = httpClient;
  }

  async getUser(id) {
    const response = await this.httpClient.get(`/api/users/${id}`);
    return response.data;
  }

  async createUser(userData) {
    const response = await this.httpClient.post('/api/users', userData);
    return response.data;
  }

  formatUserName(user) {
    return `${user.firstName} ${user.lastName}`;
  }
}

module.exports = UserService;
```

```javascript
// userService.test.js
const UserService = require('./userService');
const axios = require('axios');

jest.mock('axios');

describe('UserService 测试', () => {
  let userService;

  beforeEach(() => {
    userService = new UserService();
    jest.clearAllMocks();
  });

  describe('getUser', () => {
    test('应该成功获取用户', async () => {
      const mockUser = { id: 1, name: 'Alice' };
      axios.get.mockResolvedValue({ data: mockUser });

      const result = await userService.getUser(1);

      expect(result).toEqual(mockUser);
      expect(axios.get).toHaveBeenCalledWith('/api/users/1');
      expect(axios.get).toHaveBeenCalledTimes(1);
    });

    test('应该处理 API 错误', async () => {
      axios.get.mockRejectedValue(new Error('Network error'));

      await expect(userService.getUser(1)).rejects.toThrow('Network error');
    });
  });

  describe('createUser', () => {
    test('应该成功创建用户', async () => {
      const userData = { name: 'Bob' };
      const createdUser = { id: 2, ...userData };
      axios.post.mockResolvedValue({ data: createdUser });

      const result = await userService.createUser(userData);

      expect(result).toEqual(createdUser);
      expect(axios.post).toHaveBeenCalledWith('/api/users', userData);
    });
  });

  describe('formatUserName', () => {
    test('应该正确格式化用户名', () => {
      const user = { firstName: 'John', lastName: 'Doe' };
      expect(userService.formatUserName(user)).toBe('John Doe');
    });
  });
});

// 使用 jest.spyOn 监控方法调用
describe('方法监控测试', () => {
  test('应该监控方法调用', () => {
    const userService = new UserService();
    const formatSpy = jest.spyOn(userService, 'formatUserName');

    const user = { firstName: 'Alice', lastName: 'Smith' };
    const result = userService.formatUserName(user);

    expect(result).toBe('Alice Smith');
    expect(formatSpy).toHaveBeenCalledWith(user);
    expect(formatSpy).toHaveBeenCalledTimes(1);

    formatSpy.mockRestore(); // 恢复原始方法
  });
});
```

### 测试 React 组件

```javascript
// UserCard.jsx
import React from 'react';

const UserCard = ({ user, onClick }) => {
  return (
    <div className="user-card" data-testid="user-card">
      <h2 data-testid="user-name">{user.name}</h2>
      <p data-testid="user-email">{user.email}</p>
      <button onClick={() => onClick(user.id)}>View Profile</button>
    </div>
  );
};

export default UserCard;
```

```javascript
// UserCard.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import UserCard from './UserCard';

describe('UserCard 组件', () => {
  const mockUser = {
    id: 1,
    name: 'Alice Johnson',
    email: 'alice@example.com'
  };

  const mockOnClick = jest.fn();

  beforeEach(() => {
    mockOnClick.mockClear();
  });

  test('应该渲染用户信息', () => {
    render(<UserCard user={mockUser} onClick={mockOnClick} />);

    expect(screen.getByTestId('user-name')).toHaveTextContent('Alice Johnson');
    expect(screen.getByTestId('user-email')).toHaveTextContent('alice@example.com');
  });

  test('应该正确触发点击事件', () => {
    render(<UserCard user={mockUser} onClick={mockOnClick} />);

    const button = screen.getByRole('button', { name: /view profile/i });
    fireEvent.click(button);

    expect(mockOnClick).toHaveBeenCalledWith(1);
    expect(mockOnClick).toHaveBeenCalledTimes(1);
  });

  test('应该渲染用户卡片容器', () => {
    const { container } = render(<UserCard user={mockUser} onClick={mockOnClick} />);

    expect(container.querySelector('[data-testid="user-card"]')).toBeInTheDocument();
  });
});
```

### 测试覆盖率配置

```javascript
// package.json
{
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.js",
      "src/**/*.jsx",
      "!src/**/*.test.js",
      "!src/**/*.test.jsx",
      "!src/**/*.spec.js",
      "!src/**/*.spec.jsx",
      "!src/index.js",
      "!src/config/**"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    },
    "coverageReporters": [
      "text",
      "text-summary",
      "html",
      "lcov"
    ]
  }
}
```

### 测试辅助函数

```javascript
// testHelpers.js
// 创建测试数据生成器
function createMockUser(overrides = {}) {
  return {
    id: Math.random().toString(36).substr(2, 9),
    name: 'Test User',
    email: 'test@example.com',
    age: 25,
    ...overrides
  };
}

function createMockPost(overrides = {}) {
  return {
    id: Math.random().toString(36).substr(2, 9),
    title: 'Test Post',
    content: 'Test content',
    authorId: 'test-author-id',
    ...overrides
  };
}

// 等待异步操作完成
async function waitFor(condition, timeout = 5000) {
  const startTime = Date.now();
  while (Date.now() - startTime < timeout) {
    if (await condition()) {
      return true;
    }
    await new Promise(resolve => setTimeout(resolve, 100));
  }
  throw new Error(`Condition not met within ${timeout}ms`);
}

// 清理测试数据
function cleanupTestData() {
  // 清理测试创建的数据
}

module.exports = {
  createMockUser,
  createMockPost,
  waitFor,
  cleanupTestData
};
```

## 注意事项

1. **测试独立性**：每个测试应该独立运行，不依赖其他测试
2. **避免实现细节**：测试关注行为而非实现细节
3. **命名规范**：使用描述性的测试名称，清晰表达测试意图
4. **保持简洁**：每个测试只验证一个功能点
5. **及时更新**：代码变更时同步更新测试
6. **避免过度 Mock**：Mock 太多会导致测试脆弱
7. **测试覆盖率**：追求有意义的覆盖率，而非仅仅数字

## 总结

掌握 Jest 单元测试能够：

- **提高代码质量**：通过自动化测试发现 bug 和问题
- **增强信心**：重构和修改代码时更加自信
- **文档作用**：测试用例展示代码的使用方式
- **开发效率**：快速验证功能，减少手动测试时间
- **团队协作**：统一的测试标准便于团队协作

单元测试是现代软件开发的重要组成部分，投入时间学习和实践测试，将为你带来长期的收益。从简单的测试开始，逐步掌握更复杂的测试技巧，你的代码质量将得到显著提升。