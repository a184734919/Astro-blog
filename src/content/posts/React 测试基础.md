---
title: React 测试基础
published: 2023-10-17
description: 'Jest 和 React Testing Library的详细介绍和学习笔记'
image: ''
tags: ["React","测试"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

React 测试是确保 React 应用质量和稳定性的重要手段。通过测试，我们可以：

- 验证组件行为的正确性
- 防止回归错误
- 提供组件使用文档
- 支持安全重构
- 提升代码质量

React 测试主要使用 Jest 作为测试框架，React Testing Library (RTL) 作为组件测试工具。

---

## 核心概念

### 测试金字塔

```
        /\
       /  \
      /E2E \      端到端测试 - 少量，测试核心用户流程
     /------\
    / Integration\  集成测试 - 适量，测试组件交互
   /------------\
  /   Unit Tests  \ 单元测试 - 大量，测试独立功能
 /----------------\
```

### React Testing Library 核心理念

1. **用户视角**：测试用户如何使用组件，而不是实现细节
2. **真实交互**：模拟真实的用户操作
3. **可访问性**：鼓励使用可访问的查询方式
4. **重构友好**：关注行为而非实现

---

## 基本用法

### 1. 环境搭建

```bash
# 安装依赖
npm install --save-dev jest @testing-library/react @testing-library/jest-dom @testing-library/user-event jest-environment-jsdom
```

```javascript
// jest.config.js
module.exports = {
  testEnvironment: 'jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    '\\.(css|less|scss|sass)$': 'identity-obj-proxy',
    '\\.(jpg|jpeg|png|gif|webp|svg)$': '<rootDir>/__mocks__/fileMock.js'
  },
  transform: {
    '^.+\\.(js|jsx|ts|tsx)$': 'babel-jest'
  },
  collectCoverageFrom: [
    'src/**/*.{js,jsx,ts,tsx}',
    '!src/**/*.d.ts',
    '!src/**/*.stories.{js,jsx,ts,tsx}',
    '!src/**/__tests__/**'
  ],
  coverageThreshold: {
    global: {
      branches: 70,
      functions: 70,
      lines: 70,
      statements: 70
    }
  }
};
```

```javascript
// jest.setup.js
import '@testing-library/jest-dom';

// 全局 mock
global.matchMedia = jest.fn().mockImplementation(query => ({
  matches: false,
  media: query,
  onchange: null,
  addListener: jest.fn(),
  removeListener: jest.fn(),
  addEventListener: jest.fn(),
  removeEventListener: jest.fn(),
  dispatchEvent: jest.fn()
}));
```

### 2. 基础组件测试

```jsx
// components/Button.jsx
import React from 'react';

const Button = ({ children, onClick, disabled, variant = 'primary' }) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled}
      className={`btn btn-${variant}`}
      data-testid="button"
    >
      {children}
    </button>
  );
};

export default Button;
```

```jsx
// components/__tests__/Button.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import Button from '../Button';

describe('Button 组件', () => {
  describe('基础渲染', () => {
    test('应该渲染按钮文本', () => {
      render(<Button>点击我</Button>);
      expect(screen.getByText('点击我')).toBeInTheDocument();
    });

    test('应该渲染按钮元素', () => {
      render(<Button>按钮</Button>);
      const button = screen.getByTestId('button');
      expect(button).toBeInTheDocument();
      expect(button.tagName).toBe('BUTTON');
    });
  });

  describe('点击事件', () => {
    test('应该触发 onClick 事件', () => {
      const handleClick = jest.fn();
      render(<Button onClick={handleClick}>点击我</Button>);

      const button = screen.getByTestId('button');
      fireEvent.click(button);

      expect(handleClick).toHaveBeenCalledTimes(1);
    });

    test('禁用状态时不应该触发点击', () => {
      const handleClick = jest.fn();
      render(<Button onClick={handleClick} disabled>
        点击我
      </Button>);

      const button = screen.getByTestId('button');
      fireEvent.click(button);

      expect(handleClick).not.toHaveBeenCalled();
    });
  });

  describe('样式和属性', () => {
    test('应该应用正确的 variant 类名', () => {
      const { container } = render(<Button variant="secondary">按钮</Button>);
      const button = container.querySelector('.btn-secondary');
      expect(button).toBeInTheDocument();
    });

    test('禁用时应该有 disabled 属性', () => {
      render(<Button disabled>按钮</Button>);
      const button = screen.getByTestId('button');
      expect(button).toBeDisabled();
    });

    test('非禁用时不应该有 disabled 属性', () => {
      render(<Button>按钮</Button>);
      const button = screen.getByTestId('button');
      expect(button).not.toBeDisabled();
    });
  });

  describe('可访问性', () => {
    test('应该可以通过文本查询找到按钮', () => {
      render(<Button aria-label="提交表单">提交</Button>);
      expect(screen.getByLabelText('提交表单')).toBeInTheDocument();
    });

    test('应该可以通过角色找到按钮', () => {
      render(<Button>点击我</Button>);
      expect(screen.getByRole('button', { name: '点击我' })).toBeInTheDocument();
    });
  });
});
```

### 3. 表单组件测试

```jsx
// components/Form.jsx
import React, { useState } from 'react';

const Form = ({ onSubmit }) => {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    onSubmit(formData);
    setFormData({ name: '', email: '', message: '' });
  };

  return (
    <form onSubmit={handleSubmit} data-testid="form">
      <div>
        <label htmlFor="name">姓名</label>
        <input
          id="name"
          name="name"
          type="text"
          value={formData.name}
          onChange={handleChange}
          required
          data-testid="name-input"
        />
      </div>

      <div>
        <label htmlFor="email">邮箱</label>
        <input
          id="email"
          name="email"
          type="email"
          value={formData.email}
          onChange={handleChange}
          required
          data-testid="email-input"
        />
      </div>

      <div>
        <label htmlFor="message">消息</label>
        <textarea
          id="message"
          name="message"
          value={formData.message}
          onChange={handleChange}
          data-testid="message-input"
        />
      </div>

      <button type="submit" data-testid="submit-button">
        提交
      </button>
    </form>
  );
};

export default Form;
```

```jsx
// components/__tests__/Form.test.jsx
import React from 'react';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Form from '../Form';

describe('Form 组件', () => {
  const mockOnSubmit = jest.fn();

  beforeEach(() => {
    mockOnSubmit.mockClear();
  });

  describe('表单渲染', () => {
    test('应该渲染所有表单字段', () => {
      render(<Form onSubmit={mockOnSubmit} />);

      expect(screen.getByLabelText('姓名')).toBeInTheDocument();
      expect(screen.getByLabelText('邮箱')).toBeInTheDocument();
      expect(screen.getByLabelText('消息')).toBeInTheDocument();
      expect(screen.getByRole('button', { name: '提交' })).toBeInTheDocument();
    });

    test('应该有正确的输入类型', () => {
      render(<Form onSubmit={mockOnSubmit} />);

      expect(screen.getByLabelText('姓名')).toHaveAttribute('type', 'text');
      expect(screen.getByLabelText('邮箱')).toHaveAttribute('type', 'email');
    });
  });

  describe('表单输入', () => {
    test('应该更新 name 字段的值', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      const nameInput = screen.getByLabelText('姓名');
      await user.type(nameInput, '张三');

      expect(nameInput).toHaveValue('张三');
    });

    test('应该更新 email 字段的值', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      const emailInput = screen.getByLabelText('邮箱');
      await user.type(emailInput, 'zhangsan@example.com');

      expect(emailInput).toHaveValue('zhangsan@example.com');
    });

    test('应该更新 message 字段的值', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      const messageInput = screen.getByLabelText('消息');
      await user.type(messageInput, '这是一条测试消息');

      expect(messageInput).toHaveValue('这是一条测试消息');
    });
  });

  describe('表单提交', () => {
    test('应该提交表单数据', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      // 填写表单
      await user.type(screen.getByLabelText('姓名'), '张三');
      await user.type(screen.getByLabelText('邮箱'), 'zhangsan@example.com');
      await user.type(screen.getByLabelText('消息'), '测试消息');

      // 提交表单
      await user.click(screen.getByRole('button', { name: '提交' }));

      // 验证提交数据
      expect(mockOnSubmit).toHaveBeenCalledTimes(1);
      expect(mockOnSubmit).toHaveBeenCalledWith({
        name: '张三',
        email: 'zhangsan@example.com',
        message: '测试消息'
      });
    });

    test('提交后应该重置表单', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      await user.type(screen.getByLabelText('姓名'), '张三');
      await user.click(screen.getByRole('button', { name: '提交' }));

      expect(screen.getByLabelText('姓名')).toHaveValue('');
    });

    test('必填字段为空时不应该提交', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      // 不填写必填字段直接提交
      await user.click(screen.getByRole('button', { name: '提交' }));

      expect(mockOnSubmit).not.toHaveBeenCalled();
    });
  });

  describe('表单验证', () => {
    test('应该验证必填字段', () => {
      render(<Form onSubmit={mockOnSubmit} />);

      const nameInput = screen.getByLabelText('姓名');
      const emailInput = screen.getByLabelText('邮箱');

      expect(nameInput).toBeRequired();
      expect(emailInput).toBeRequired();
    });

    test('应该接受有效的邮箱格式', async () => {
      const user = userEvent.setup();
      render(<Form onSubmit={mockOnSubmit} />);

      const emailInput = screen.getByLabelText('邮箱');

      await user.type(emailInput, 'invalid-email');
      expect(emailInput).toBeInvalid();

      await user.clear(emailInput);
      await user.type(emailInput, 'valid@example.com');
      expect(emailInput).toBeValid();
    });
  });
});
```

---

## 实际应用

### 1. 列表组件测试

```jsx
// components/UserList.jsx
import React from 'react';

const UserList = ({ users, onEdit, onDelete, loading }) => {
  if (loading) {
    return <div data-testid="loading">加载中...</div>;
  }

  if (users.length === 0) {
    return <div data-testid="empty">暂无用户</div>;
  }

  return (
    <ul data-testid="user-list">
      {users.map(user => (
        <li key={user.id} data-testid={`user-${user.id}`}>
          <span>{user.name}</span>
          <span>{user.email}</span>
          <button onClick={() => onEdit(user)}>编辑</button>
          <button onClick={() => onDelete(user.id)}>删除</button>
        </li>
      ))}
    </ul>
  );
};

export default UserList;
```

```jsx
// components/__tests__/UserList.test.jsx
import React from 'react';
import { render, screen, fireEvent } from '@testing-library/react';
import UserList from '../UserList';

describe('UserList 组件', () => {
  const mockUsers = [
    { id: 1, name: '张三', email: 'zhangsan@example.com' },
    { id: 2, name: '李四', email: 'lisi@example.com' }
  ];

  const mockOnEdit = jest.fn();
  const mockOnDelete = jest.fn();

  beforeEach(() => {
    mockOnEdit.mockClear();
    mockOnDelete.mockClear();
  });

  describe('加载状态', () => {
    test('应该显示加载状态', () => {
      render(
        <UserList
          users={[]}
          loading={true}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      expect(screen.getByTestId('loading')).toHaveTextContent('加载中...');
    });

    test('加载时不应该渲染用户列表', () => {
      render(
        <UserList
          users={mockUsers}
          loading={true}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      expect(screen.queryByTestId('user-list')).not.toBeInTheDocument();
    });
  });

  describe('空状态', () => {
    test('应该显示空状态', () => {
      render(
        <UserList
          users={[]}
          loading={false}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      expect(screen.getByTestId('empty')).toHaveTextContent('暂无用户');
    });
  });

  describe('用户列表渲染', () => {
    test('应该渲染所有用户', () => {
      render(
        <UserList
          users={mockUsers}
          loading={false}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      expect(screen.getByText('张三')).toBeInTheDocument();
      expect(screen.getByText('李四')).toBeInTheDocument();
    });

    test('应该渲染正确数量的用户', () => {
      render(
        <UserList
          users={mockUsers}
          loading={false}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      const userItems = screen.getAllByRole('listitem');
      expect(userItems).toHaveLength(2);
    });
  });

  describe('用户操作', () => {
    test('应该触发编辑操作', () => {
      render(
        <UserList
          users={mockUsers}
          loading={false}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      const editButton = screen.getAllByText('编辑')[0];
      fireEvent.click(editButton);

      expect(mockOnEdit).toHaveBeenCalledWith(mockUsers[0]);
    });

    test('应该触发删除操作', () => {
      render(
        <UserList
          users={mockUsers}
          loading={false}
          onEdit={mockOnEdit}
          onDelete={mockOnDelete}
        />
      );

      const deleteButton = screen.getAllByText('删除')[0];
      fireEvent.click(deleteButton);

      expect(mockOnDelete).toHaveBeenCalledWith(mockUsers[0].id);
    });
  });
});
```

### 2. 异步组件测试

```jsx
// components/UserProfile.jsx
import React, { useState, useEffect } from 'react';

const UserProfile = ({ userId, apiClient }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        const data = await apiClient.getUser(userId);
        setUser(data);
        setError(null);
      } catch (err) {
        setError('加载用户信息失败');
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId, apiClient]);

  if (loading) {
    return <div data-testid="loading">加载中...</div>;
  }

  if (error) {
    return <div data-testid="error">{error}</div>;
  }

  if (!user) {
    return <div data-testid="not-found">用户不存在</div>;
  }

  return (
    <div data-testid="user-profile">
      <h2>{user.name}</h2>
      <p>邮箱: {user.email}</p>
      <p>电话: {user.phone}</p>
    </div>
  );
};

export default UserProfile;
```

```jsx
// components/__tests__/UserProfile.test.jsx
import React from 'react';
import { render, screen, waitFor } from '@testing-library/react';
import UserProfile from '../UserProfile';

describe('UserProfile 组件', () => {
  const mockApiClient = {
    getUser: jest.fn()
  };

  beforeEach(() => {
    mockApiClient.getUser.mockClear();
  });

  describe('加载状态', () => {
    test('应该显示加载状态', () => {
      mockApiClient.getUser.mockImplementation(() => new Promise(() => {}));

      render(<UserProfile userId="1" apiClient={mockApiClient} />);

      expect(screen.getByTestId('loading')).toHaveTextContent('加载中...');
    });
  });

  describe('成功加载', () => {
    test('应该显示用户信息', async () => {
      const mockUser = {
        id: '1',
        name: '张三',
        email: 'zhangsan@example.com',
        phone: '1234567890'
      };

      mockApiClient.getUser.mockResolvedValue(mockUser);

      render(<UserProfile userId="1" apiClient={mockApiClient} />);

      // 等待异步操作完成
      await waitFor(() => {
        expect(screen.getByTestId('user-profile')).toBeInTheDocument();
      });

      expect(screen.getByText('张三')).toBeInTheDocument();
      expect(screen.getByText('邮箱: zhangsan@example.com')).toBeInTheDocument();
      expect(screen.getByText('电话: 1234567890')).toBeInTheDocument();
    });

    test('应该调用正确的 API', async () => {
      mockApiClient.getUser.mockResolvedValue({ id: '1', name: '张三' });

      render(<UserProfile userId="1" apiClient={mockApiClient} />);

      await waitFor(() => {
        expect(mockApiClient.getUser).toHaveBeenCalledTimes(1);
        expect(mockApiClient.getUser).toHaveBeenCalledWith('1');
      });
    });
  });

  describe('错误处理', () => {
    test('应该显示错误信息', async () => {
      mockApiClient.getUser.mockRejectedValue(new Error('Network error'));

      render(<UserProfile userId="1" apiClient={mockApiClient} />);

      await waitFor(() => {
        expect(screen.getByTestId('error')).toHaveTextContent('加载用户信息失败');
      });
    });

    test('用户不存在时应该显示提示', async () => {
      mockApiClient.getUser.mockResolvedValue(null);

      render(<UserProfile userId="1" apiClient={mockApiClient} />);

      await waitFor(() => {
        expect(screen.getByTestId('not-found')).toHaveTextContent('用户不存在');
      });
    });
  });

  describe('重新加载', () => {
    test('userId 变化时应该重新加载', async () => {
      mockApiClient.getUser
        .mockResolvedValueOnce({ id: '1', name: '张三' })
        .mockResolvedValueOnce({ id: '2', name: '李四' });

      const { rerender } = render(
        <UserProfile userId="1" apiClient={mockApiClient} />
      );

      await waitFor(() => {
        expect(screen.getByText('张三')).toBeInTheDocument();
      });

      rerender(<UserProfile userId="2" apiClient={mockApiClient} />);

      await waitFor(() => {
        expect(screen.getByText('李四')).toBeInTheDocument();
      });

      expect(mockApiClient.getUser).toHaveBeenCalledTimes(2);
    });
  });
});
```

### 3. 自定义 Hook 测试

```jsx
// hooks/useFetch.js
import { useState, useEffect } from 'react';

const useFetch = (url, options = {}) => {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url, options);
        const json = await response.json();
        setData(json);
        setError(null);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url, JSON.stringify(options)]);

  return { data, loading, error };
};

export default useFetch;
```

```jsx
// hooks/__tests__/useFetch.test.js
import { renderHook, waitFor } from '@testing-library/react';
import useFetch from '../useFetch';

// Mock fetch
global.fetch = jest.fn();

describe('useFetch Hook', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  test('应该返回初始状态', () => {
    fetch.mockImplementation(() => new Promise(() => {}));

    const { result } = renderHook(() => useFetch('/api/data'));

    expect(result.current.loading).toBe(true);
    expect(result.current.data).toBeNull();
    expect(result.current.error).toBeNull();
  });

  test('应该成功获取数据', async () => {
    const mockData = { id: 1, name: '测试数据' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toEqual(mockData);
    expect(result.current.error).toBeNull();
  });

  test('应该处理错误', async () => {
    const errorMessage = 'Network error';
    fetch.mockRejectedValueOnce(new Error(errorMessage));

    const { result } = renderHook(() => useFetch('/api/data'));

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.data).toBeNull();
    expect(result.current.error).toBe(errorMessage);
  });

  test('URL 变化时应该重新获取数据', async () => {
    fetch
      .mockResolvedValueOnce({ ok: true, json: async () => ({ id: 1 }) })
      .mockResolvedValueOnce({ ok: true, json: async () => ({ id: 2 }) });

    const { result, rerender } = renderHook(
      ({ url }) => useFetch(url),
      { initialProps: { url: '/api/data1' } }
    );

    await waitFor(() => {
      expect(result.current.data).toEqual({ id: 1 });
    });

    rerender({ url: '/api/data2' });

    await waitFor(() => {
      expect(result.current.data).toEqual({ id: 2 });
    });

    expect(fetch).toHaveBeenCalledTimes(2);
  });
});
```

### 4. 上下文提供者测试

```jsx
// context/AuthContext.jsx
import React, { createContext, useContext, useState } from 'react';

const AuthContext = createContext();

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [isAuthenticated, setIsAuthenticated] = useState(false);

  const login = (userData) => {
    setUser(userData);
    setIsAuthenticated(true);
  };

  const logout = () => {
    setUser(null);
    setIsAuthenticated(false);
  };

  return (
    <AuthContext.Provider value={{ user, isAuthenticated, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
};

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};
```

```jsx
// context/__tests__/AuthContext.test.jsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { AuthProvider, useAuth } from '../AuthContext';

describe('AuthContext', () => {
  const TestComponent = () => {
    const { user, isAuthenticated, login, logout } = useAuth();

    return (
      <div>
        <div data-testid="is-authenticated">
          {isAuthenticated ? '已认证' : '未认证'}
        </div>
        <div data-testid="user-name">
          {user ? user.name : '无用户'}
        </div>
        <button onClick={() => login({ name: '张三' })}>
          登录
        </button>
        <button onClick={logout}>
          退出
        </button>
      </div>
    );
  };

  const renderWithAuth = (component) => {
    return render(<AuthProvider>{component}</AuthProvider>);
  };

  describe('初始状态', () => {
    test('应该提供初始认证状态', () => {
      renderWithAuth(<TestComponent />);

      expect(screen.getByTestId('is-authenticated')).toHaveTextContent('未认证');
      expect(screen.getByTestId('user-name')).toHaveTextContent('无用户');
    });
  });

  describe('登录功能', () => {
    test('应该能够登录', () => {
      renderWithAuth(<TestComponent />);

      const loginButton = screen.getByText('登录');
      loginButton.click();

      expect(screen.getByTestId('is-authenticated')).toHaveTextContent('已认证');
      expect(screen.getByTestId('user-name')).toHaveTextContent('张三');
    });
  });

  describe('退出功能', () => {
    test('应该能够退出', () => {
      renderWithAuth(<TestComponent />);

      // 先登录
      screen.getByText('登录').click();
      expect(screen.getByTestId('is-authenticated')).toHaveTextContent('已认证');

      // 然后退出
      screen.getByText('退出').click();
      expect(screen.getByTestId('is-authenticated')).toHaveTextContent('未认证');
      expect(screen.getByTestId('user-name')).toHaveTextContent('无用户');
    });
  });
});
```

### 5. 路由组件测试

```jsx
// components/ProtectedRoute.jsx
import React from 'react';
import { Navigate } from 'react-router-dom';

const ProtectedRoute = ({ children, isAuthenticated }) => {
  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }
  return children;
};

export default ProtectedRoute;
```

```jsx
// components/__tests__/ProtectedRoute.test.jsx
import React from 'react';
import { render, screen } from '@testing-library/react';
import { MemoryRouter, Routes, Route } from 'react-router-dom';
import ProtectedRoute from '../ProtectedRoute';

describe('ProtectedRoute 组件', () => {
  const TestComponent = () => <div data-testid="protected-content">受保护的内容</div>;

  test('未认证时应该重定向到登录页', () => {
    render(
      <MemoryRouter initialEntries={['/protected']}>
        <Routes>
          <Route
            path="/protected"
            element={
              <ProtectedRoute isAuthenticated={false}>
                <TestComponent />
              </ProtectedRoute>
            }
          />
          <Route path="/login" element={<div data-testid="login-page">登录页</div>} />
        </Routes>
      </MemoryRouter>
    );

    expect(screen.getByTestId('login-page')).toBeInTheDocument();
    expect(screen.queryByTestId('protected-content')).not.toBeInTheDocument();
  });

  test('已认证时应该渲染受保护的内容', () => {
    render(
      <MemoryRouter initialEntries={['/protected']}>
        <Routes>
          <Route
            path="/protected"
            element={
              <ProtectedRoute isAuthenticated={true}>
                <TestComponent />
              </ProtectedRoute>
            }
          />
          <Route path="/login" element={<div data-testid="login-page">登录页</div>} />
        </Routes>
      </MemoryRouter>
    );

    expect(screen.getByTestId('protected-content')).toBeInTheDocument();
    expect(screen.queryByTestId('login-page')).not.toBeInTheDocument();
  });
});
```

---

## 注意事项

### 1. 测试最佳实践

```jsx
// ✅ 好的测试：测试用户行为
test('用户应该能够添加商品到购物车', () => {
  render(<ProductCard product={mockProduct} />);
  const addToCartButton = screen.getByRole('button', { name: /加入购物车/i });
  userEvent.click(addToCartButton);
  expect(screen.getByText('已添加到购物车')).toBeInTheDocument();
});

// ❌ 不好的测试：测试实现细节
test('应该调用 addToCart 函数', () => {
  const mockAddToCart = jest.fn();
  render(<ProductCard product={mockProduct} onAddToCart={mockAddToCart} />);
  const button = screen.getByTestId('add-to-cart-button');
  fireEvent.click(button);
  expect(mockAddToCart).toHaveBeenCalledWith(mockProduct);
});
```

### 2. 选择正确的查询方法

```jsx
// 推荐的查询优先级（按推荐顺序）：
// 1. 用户交互相关
screen.getByRole('button', { name: /提交/i })
screen.getByLabelText('邮箱')
screen.getByPlaceholderText('请输入密码')

// 2. 文本内容相关
screen.getByText('欢迎使用')
screen.getByTitle('工具提示')

// 3. 测试 ID（最后选择）
screen.getByTestId('submit-button')

// 避免使用
// screen.getByClassName('btn-primary')  // 测试实现细节
// screen.querySelector('#submit')       // 测试实现细节
```

### 3. 测试覆盖率

```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch"
  },
  "jest": {
    "collectCoverageFrom": [
      "src/**/*.{js,jsx}",
      "!src/**/*.test.{js,jsx}",
      "!src/index.js"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
```

### 4. Mock 和 Spy

```jsx
// Mock 外部依赖
jest.mock('axios', () => ({
  get: jest.fn(),
  post: jest.fn()
}));

// 监控函数调用
const mockFunction = jest.fn();

// 清除 mock
beforeEach(() => {
  mockFunction.mockClear();
});

// 模拟返回值
mockFunction.mockReturnValue('result');
mockFunction.mockResolvedValue('async-result');
mockFunction.mockRejectedValue(new Error('error'));
```

---

## 总结

React 测试是保证应用质量的重要手段：

### 核心要点

1. **用户视角**：测试用户如何使用组件
2. **真实交互**：模拟真实的用户操作
3. **测试金字塔**：平衡单元测试、集成测试和 E2E 测试
4. **覆盖率目标**：追求有意义的测试覆盖率

### 最佳实践

- 使用 React Testing Library 而不是 Enzyme
- 关注行为而非实现细节
- 测试用户交互和集成场景
- 保持测试简洁和可维护
- 适当的 mock 和 spy

### 常见场景

- 表单测试
- 异步操作测试
- 事件处理测试
- 路由测试
- 上下文测试

通过掌握 React 测试，你可以构建更可靠、更易维护的 React 应用。记住测试不仅仅是验证功能，更是设计和文档的重要组成部分。