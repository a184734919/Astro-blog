---
title: React State
published: 2023-07-11
description: '组件状态管理的详细介绍和学习笔记'
image: ''
tags: ["React","State"]
category: 'React'
draft: false
lang: 'zh-CN'
---

# React State

## 概述

State 是 React 组件内部的可变数据，它驱动组件的重新渲染。与 Props 不同，State 是组件私有的，完全由组件内部控制。理解 State 的使用机制对于构建交互式的 React 应用至关重要。State 的管理方式随着 React 的发展不断演进，从类组件的 this.state 到函数组件的 Hooks，提供了更加灵活和强大的状态管理能力。

### State 的核心特性

**组件私有**：State 是组件内部的状态，外部无法直接访问或修改

**驱动渲染**：State 的变化会触发组件重新渲染

**异步更新**：State 更新是异步的，React 会批量处理更新以提高性能

**不可变性**：State 应该被视为不可变的，更新时需要创建新的状态值

**函数式更新**：支持基于前一个状态的函数式更新

## 核心概念

### 1. State 的基本使用

```jsx
import { useState } from 'react';

// 1. 基本的 State 使用
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(count - 1)}>
        Decrement
      </button>
      <button onClick={() => setCount(0)}>
        Reset
      </button>
    </div>
  );
}

// 2. 对象 State
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0,
    address: {
      city: '',
      country: ''
    }
  });

  const updateName = (name) => {
    setUser(prev => ({ ...prev, name }));
  };

  const updateAddress = (city, country) => {
    setUser(prev => ({
      ...prev,
      address: { ...prev.address, city, country }
    }));
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
        placeholder="Name"
      />
      <p>Name: {user.name}</p>
      <p>Email: {user.email}</p>
    </div>
  );
}

// 3. 数组 State
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build a project', completed: false }
  ]);

  const addTodo = (text) => {
    const newTodo = {
      id: Date.now(),
      text,
      completed: false
    };
    setTodos([...todos, newTodo]);
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const deleteTodo = (id) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{
              textDecoration: todo.completed ? 'line-through' : 'none'
            }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>
              Delete
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

### 2. State 更新的正确方式

```jsx
function StateUpdatePatterns() {
  // 1. 函数式更新（推荐）
  const [count, setCount] = useState(0);

  const handleIncrement = () => {
    setCount(prevCount => prevCount + 1); // 基于前一个状态更新
  };

  // 2. 对象状态的更新
  const [user, setUser] = useState({
    name: 'John',
    age: 30,
    email: 'john@example.com'
  });

  // ✅ 正确：创建新对象
  const updateUserName = (name) => {
    setUser(prev => ({ ...prev, name }));
  };

  const updateMultipleFields = () => {
    setUser(prev => ({
      ...prev,
      name: 'Jane',
      age: 25,
      email: 'jane@example.com'
    }));
  };

  // ❌ 错误：直接修改原对象
  const badUpdate = () => {
    user.name = 'Jane'; // 不要这样做
    setUser(user);
  };

  // 3. 数组状态的更新
  const [items, setItems] = useState([1, 2, 3]);

  // 添加元素
  const addItem = (item) => {
    setItems(prev => [...prev, item]);
  };

  // 删除元素
  const removeItem = (index) => {
    setItems(prev => prev.filter((_, i) => i !== index));
  };

  // 更新元素
  const updateItem = (index, newValue) => {
    setItems(prev =>
      prev.map((item, i) => i === index ? newValue : item)
    );
  };

  // 4. 嵌套状态的更新
  const [nestedState, setNestedState] = useState({
    user: {
      profile: {
        name: 'John',
        age: 30
      },
      settings: {
        theme: 'light',
        language: 'en'
      }
    }
  });

  const updateNestedName = (name) => {
    setNestedState(prev => ({
      ...prev,
      user: {
        ...prev.user,
        profile: {
          ...prev.user.profile,
          name
        }
      }
    }));
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={handleIncrement}>Increment</button>
    </div>
  );
}
```

### 3. 多个 State 的管理

```jsx
function MultipleStates() {
  // 1. 独立的 State
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState('');
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [errors, setErrors] = useState({});

  // 2. 相关的 State 合并
  const [form, setForm] = useState({
    name: '',
    email: '',
    age: '',
    errors: {}
  });

  const handleFieldChange = (field, value) => {
    setForm(prev => ({
      ...prev,
      [field]: value,
      errors: {
        ...prev.errors,
        [field]: '' // 清除字段错误
      }
    }));
  };

  // 3. 使用 useReducer 管理复杂状态
  const initialState = {
    users: [],
    loading: false,
    error: null,
    currentPage: 1,
    itemsPerPage: 10
  };

  function reducer(state, action) {
    switch (action.type) {
      case 'FETCH_START':
        return { ...state, loading: true, error: null };
      case 'FETCH_SUCCESS':
        return {
          ...state,
          loading: false,
          users: action.payload,
          error: null
        };
      case 'FETCH_ERROR':
        return {
          ...state,
          loading: false,
          error: action.payload
        };
      case 'SET_PAGE':
        return { ...state, currentPage: action.payload };
      default:
        return state;
    }
  }

  const [state, dispatch] = useReducer(reducer, initialState);

  const fetchUsers = async (page) => {
    dispatch({ type: 'FETCH_START' });
    try {
      const response = await fetch(`/api/users?page=${page}`);
      const data = await response.json();
      dispatch({ type: 'FETCH_SUCCESS', payload: data });
    } catch (error) {
      dispatch({ type: 'FETCH_ERROR', payload: error.message });
    }
  };

  return (
    <div>
      {/* 组件内容 */}
    </div>
  );
}
```

### 4. State 初始化

```jsx
function StateInitialization() {
  // 1. 简单初始化
  const [count, setCount] = useState(0);

  // 2. 函数初始化（只执行一次）
  const [expensiveValue, setExpensiveValue] = useState(() => {
    console.log('Expensive calculation');
    return calculateExpensiveValue();
  });

  function calculateExpensiveValue() {
    // 复杂的计算逻辑
    return Math.random() * 1000;
  }

  // 3. 基于 Props 的初始化
  function ChildComponent({ initialValue }) {
    const [value, setValue] = useState(initialValue);

    // 当 props 变化时更新 state
    React.useEffect(() => {
      setValue(initialValue);
    }, [initialValue]);

    return <div>Value: {value}</div>;
  }

  // 4. 从 localStorage 初始化
  const [theme, setTheme] = useState(() => {
    const savedTheme = localStorage.getItem('theme');
    return savedTheme || 'light';
  });

  // 5. 复杂对象的初始化
  const [userProfile, setUserProfile] = useState(() => {
    const savedProfile = localStorage.getItem('userProfile');
    if (savedProfile) {
      return JSON.parse(savedProfile);
    }
    return {
      name: '',
      email: '',
      preferences: {
        theme: 'light',
        language: 'en',
        notifications: true
      }
    };
  });

  return (
    <div>
      <p>Count: {count}</p>
      <p>Theme: {theme}</p>
    </div>
  );
}
```

## 基本用法

### 1. 表单状态管理

```jsx
function FormStateManagement() {
  const [formData, setFormData] = useState({
    username: '',
    email: '',
    password: '',
    confirmPassword: '',
    agreeTerms: false
  });

  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  // 处理字段变化
  const handleChange = (event) => {
    const { name, value, type, checked } = event.target;

    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));

    // 实时验证
    if (touched[name]) {
      validateField(name, type === 'checkbox' ? checked : value);
    }
  };

  // 处理字段失焦
  const handleBlur = (event) => {
    const { name } = event.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    validateField(name, formData[name]);
  };

  // 字段验证
  const validateField = (name, value) => {
    let error = '';

    switch (name) {
      case 'username':
        if (!value.trim()) {
          error = 'Username is required';
        } else if (value.length < 3) {
          error = 'Username must be at least 3 characters';
        }
        break;

      case 'email':
        if (!value.trim()) {
          error = 'Email is required';
        } else if (!/\S+@\S+\.\S+/.test(value)) {
          error = 'Invalid email format';
        }
        break;

      case 'password':
        if (!value) {
          error = 'Password is required';
        } else if (value.length < 8) {
          error = 'Password must be at least 8 characters';
        }
        break;

      case 'confirmPassword':
        if (value !== formData.password) {
          error = 'Passwords do not match';
        }
        break;

      case 'agreeTerms':
        if (!value) {
          error = 'You must agree to the terms';
        }
        break;

      default:
        break;
    }

    setErrors(prev => ({ ...prev, [name]: error }));
    return !error;
  };

  // 表单验证
  const validateForm = () => {
    const newErrors = {};
    let isValid = true;

    Object.keys(formData).forEach(field => {
      const error = validateField(field, formData[field]);
      if (error) {
        newErrors[field] = error;
        isValid = false;
      }
    });

    setErrors(newErrors);
    return isValid;
  };

  // 表单提交
  const handleSubmit = async (event) => {
    event.preventDefault();

    // 标记所有字段为已触摸
    setTouched(
      Object.keys(formData).reduce((acc, field) => ({
        ...acc,
        [field]: true
      }), {})
    );

    if (!validateForm()) {
      return;
    }

    setIsSubmitting(true);

    try {
      // 模拟 API 调用
      await new Promise(resolve => setTimeout(resolve, 2000));
      console.log('Form submitted:', formData);
      alert('Registration successful!');

      // 重置表单
      setFormData({
        username: '',
        email: '',
        password: '',
        confirmPassword: '',
        agreeTerms: false
      });
      setErrors({});
      setTouched({});
    } catch (error) {
      console.error('Submission error:', error);
      alert('Registration failed. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit} className="registration-form">
      <h2>Register</h2>

      <div className="form-group">
        <label htmlFor="username">Username *</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.username && errors.username ? 'error' : ''}
        />
        {touched.username && errors.username && (
          <span className="error-message">{errors.username}</span>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="email">Email *</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.email && errors.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="password">Password *</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.password && errors.password ? 'error' : ''}
        />
        {touched.password && errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>

      <div className="form-group">
        <label htmlFor="confirmPassword">Confirm Password *</label>
        <input
          type="password"
          id="confirmPassword"
          name="confirmPassword"
          value={formData.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
          className={touched.confirmPassword && errors.confirmPassword ? 'error' : ''}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>

      <div className="form-group checkbox-group">
        <label>
          <input
            type="checkbox"
            name="agreeTerms"
            checked={formData.agreeTerms}
            onChange={handleChange}
            onBlur={handleBlur}
          />
          I agree to the terms and conditions
        </label>
        {touched.agreeTerms && errors.agreeTerms && (
          <span className="error-message">{errors.agreeTerms}</span>
        )}
      </div>

      <button
        type="submit"
        disabled={isSubmitting}
        className="submit-button"
      >
        {isSubmitting ? 'Registering...' : 'Register'}
      </button>
    </form>
  );
}
```

### 2. 列表和筛选状态

```jsx
function ListWithFilters() {
  const [items, setItems] = useState([
    { id: 1, name: 'Apple', category: 'fruit', price: 1.99, inStock: true },
    { id: 2, name: 'Banana', category: 'fruit', price: 0.99, inStock: true },
    { id: 3, name: 'Carrot', category: 'vegetable', price: 1.49, inStock: true },
    { id: 4, name: 'Milk', category: 'dairy', price: 3.99, inStock: false },
    { id: 5, name: 'Bread', category: 'bakery', price: 2.99, inStock: true },
    { id: 6, name: 'Cheese', category: 'dairy', price: 4.99, inStock: true },
  ]);

  const [filters, setFilters] = useState({
    category: 'all',
    search: '',
    priceRange: 'all',
    inStockOnly: false
  });

  const [sortConfig, setSortConfig] = useState({
    key: 'name',
    direction: 'ascending'
  });

  const [selectedItems, setSelectedItems] = useState(new Set());

  // 应用筛选
  const filteredItems = React.useMemo(() => {
    return items.filter(item => {
      // 分类筛选
      if (filters.category !== 'all' && item.category !== filters.category) {
        return false;
      }

      // 搜索筛选
      if (filters.search && !item.name.toLowerCase().includes(filters.search.toLowerCase())) {
        return false;
      }

      // 价格筛选
      if (filters.priceRange !== 'all') {
        const [min, max] = filters.priceRange.split('-').map(Number);
        if (item.price < min || item.price > max) {
          return false;
        }
      }

      // 库存筛选
      if (filters.inStockOnly && !item.inStock) {
        return false;
      }

      return true;
    });
  }, [items, filters]);

  // 应用排序
  const sortedItems = React.useMemo(() => {
    const sorted = [...filteredItems];
    sorted.sort((a, b) => {
      if (a[sortConfig.key] < b[sortConfig.key]) {
        return sortConfig.direction === 'ascending' ? -1 : 1;
      }
      if (a[sortConfig.key] > b[sortConfig.key]) {
        return sortConfig.direction === 'ascending' ? 1 : -1;
      }
      return 0;
    });
    return sorted;
  }, [filteredItems, sortConfig]);

  // 处理筛选变化
  const handleFilterChange = (key, value) => {
    setFilters(prev => ({ ...prev, [key]: value }));
  };

  // 处理排序变化
  const handleSort = (key) => {
    setSortConfig(prev => ({
      key,
      direction: prev.key === key && prev.direction === 'ascending'
        ? 'descending'
        : 'ascending'
    }));
  };

  // 处理项目选择
  const handleItemSelect = (id) => {
    setSelectedItems(prev => {
      const newSelected = new Set(prev);
      if (newSelected.has(id)) {
        newSelected.delete(id);
      } else {
        newSelected.add(id);
      }
      return newSelected;
    });
  };

  // 全选/取消全选
  const handleSelectAll = () => {
    if (selectedItems.size === sortedItems.length) {
      setSelectedItems(new Set());
    } else {
      setSelectedItems(new Set(sortedItems.map(item => item.id)));
    }
  };

  // 获取唯一分类
  const categories = React.useMemo(() => {
    const uniqueCategories = [...new Set(items.map(item => item.category))];
    return ['all', ...uniqueCategories];
  }, [items]);

  return (
    <div className="list-with-filters">
      <h2>Product List</h2>

      {/* 筛选器 */}
      <div className="filters">
        <div className="filter-group">
          <label>Category:</label>
          <select
            value={filters.category}
            onChange={(e) => handleFilterChange('category', e.target.value)}
          >
            {categories.map(category => (
              <option key={category} value={category}>
                {category === 'all' ? 'All Categories' : category}
              </option>
            ))}
          </select>
        </div>

        <div className="filter-group">
          <label>Search:</label>
          <input
            type="text"
            value={filters.search}
            onChange={(e) => handleFilterChange('search', e.target.value)}
            placeholder="Search products..."
          />
        </div>

        <div className="filter-group">
          <label>Price Range:</label>
          <select
            value={filters.priceRange}
            onChange={(e) => handleFilterChange('priceRange', e.target.value)}
          >
            <option value="all">All Prices</option>
            <option value="0-2">Under $2</option>
            <option value="2-4">$2 - $4</option>
            <option value="4-100">$4+</option>
          </select>
        </div>

        <div className="filter-group">
          <label>
            <input
              type="checkbox"
              checked={filters.inStockOnly}
              onChange={(e) => handleFilterChange('inStockOnly', e.target.checked)}
            />
            In Stock Only
          </label>
        </div>
      </div>

      {/* 排序控制 */}
      <div className="sort-controls">
        <span>Sort by:</span>
        <button
          onClick={() => handleSort('name')}
          className={sortConfig.key === 'name' ? 'active' : ''}
        >
          Name {sortConfig.key === 'name' && (sortConfig.direction === 'ascending' ? '↑' : '↓')}
        </button>
        <button
          onClick={() => handleSort('price')}
          className={sortConfig.key === 'price' ? 'active' : ''}
        >
          Price {sortConfig.key === 'price' && (sortConfig.direction === 'ascending' ? '↑' : '↓')}
        </button>
        <button
          onClick={() => handleSort('category')}
          className={sortConfig.key === 'category' ? 'active' : ''}
        >
          Category {sortConfig.key === 'category' && (sortConfig.direction === 'ascending' ? '↑' : '↓')}
        </button>
      </div>

      {/* 列表 */}
      <div className="list-info">
        <span>Total: {sortedItems.length} items</span>
        {selectedItems.size > 0 && (
          <span>Selected: {selectedItems.size} items</span>
        )}
      </div>

      <div className="items-grid">
        {sortedItems.map(item => (
          <div
            key={item.id}
            className={`item-card ${selectedItems.has(item.id) ? 'selected' : ''} ${!item.inStock ? 'out-of-stock' : ''}`}
          >
            <input
              type="checkbox"
              checked={selectedItems.has(item.id)}
              onChange={() => handleItemSelect(item.id)}
              disabled={!item.inStock}
            />
            <h3>{item.name}</h3>
            <p>Category: {item.category}</p>
            <p>Price: ${item.price.toFixed(2)}</p>
            {!item.inStock && <p className="out-of-stock-label">Out of Stock</p>}
          </div>
        ))}
      </div>

      {sortedItems.length === 0 && (
        <div className="empty-state">
          <p>No products found matching your filters.</p>
        </div>
      )}
    </div>
  );
}
```

### 3. 异步状态管理

```jsx
function AsyncStateManagement() {
  const [state, setState] = useState({
    data: null,
    loading: false,
    error: null,
    lastUpdated: null
  });

  const [retryCount, setRetryCount] = useState(0);

  const fetchData = async (showLoading = true) => {
    try {
      if (showLoading) {
        setState(prev => ({ ...prev, loading: true, error: null }));
      }

      const response = await fetch('/api/data');
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }

      const data = await response.json();

      setState({
        data,
        loading: false,
        error: null,
        lastUpdated: new Date()
      });

      setRetryCount(0);
    } catch (error) {
      setState({
        data: null,
        loading: false,
        error: error.message,
        lastUpdated: null
      });
    }
  };

  const retry = () => {
    setRetryCount(prev => prev + 1);
    fetchData(true);
  };

  // 组件挂载时获取数据
  React.useEffect(() => {
    fetchData();
  }, []);

  // 定时刷新
  React.useEffect(() => {
    const interval = setInterval(() => {
      fetchData(false);
    }, 30000); // 30秒刷新一次

    return () => clearInterval(interval);
  }, []);

  return (
    <div className="async-component">
      {state.loading && (
        <div className="loading-state">
          <div className="spinner"></div>
          <p>Loading data...</p>
        </div>
      )}

      {state.error && (
        <div className="error-state">
          <p>Error: {state.error}</p>
          <button onClick={retry}>
            Retry {retryCount > 0 && `(${retryCount})`}
          </button>
        </div>
      )}

      {state.data && !state.loading && (
        <div className="data-state">
          <h3>Data loaded successfully</h3>
          {state.lastUpdated && (
            <p>Last updated: {state.lastUpdated.toLocaleString()}</p>
          )}
          <pre>{JSON.stringify(state.data, null, 2)}</pre>
          <button onClick={() => fetchData()}>Refresh</button>
        </div>
      )}
    </div>
  );
}
```

## 实际应用

### 1. 购物车状态管理

```jsx
function ShoppingCart() {
  const [cart, setCart] = useState({
    items: [],
    couponCode: '',
    discount: 0,
    shippingMethod: 'standard',
    isProcessing: false
  });

  const [showCouponInput, setShowCouponInput] = useState(false);

  // 添加商品到购物车
  const addToCart = (product) => {
    setCart(prev => {
      const existingItem = prev.items.find(item => item.id === product.id);

      if (existingItem) {
        // 更新数量
        return {
          ...prev,
          items: prev.items.map(item =>
            item.id === product.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      } else {
        // 添加新商品
        return {
          ...prev,
          items: [...prev.items, { ...product, quantity: 1 }]
        };
      }
    });
  };

  // 更新商品数量
  const updateQuantity = (productId, newQuantity) => {
    if (newQuantity <= 0) {
      removeFromCart(productId);
      return;
    }

    setCart(prev => ({
      ...prev,
      items: prev.items.map(item =>
        item.id === productId
          ? { ...item, quantity: newQuantity }
          : item
      )
    }));
  };

  // 从购物车移除商品
  const removeFromCart = (productId) => {
    setCart(prev => ({
      ...prev,
      items: prev.items.filter(item => item.id !== productId)
    }));
  };

  // 应用优惠券
  const applyCoupon = () => {
    // 模拟优惠券验证
    const couponDiscounts = {
      'SAVE10': 0.10,
      'SAVE20': 0.20,
      'SAVE30': 0.30
    };

    const discount = couponDiscounts[cart.couponCode.toUpperCase()] || 0;
    setCart(prev => ({ ...prev, discount }));
  };

  // 计算总价
  const cartTotals = React.useMemo(() => {
    const subtotal = cart.items.reduce((sum, item) => {
      return sum + (item.price * item.quantity);
    }, 0);

    const discountAmount = subtotal * cart.discount;
    const subtotalAfterDiscount = subtotal - discountAmount;

    let shippingCost = 0;
    if (cart.shippingMethod === 'standard') {
      shippingCost = subtotalAfterDiscount >= 50 ? 0 : 5.99;
    } else if (cart.shippingMethod === 'express') {
      shippingCost = 12.99;
    } else if (cart.shippingMethod === 'overnight') {
      shippingCost = 24.99;
    }

    const total = subtotalAfterDiscount + shippingCost;

    return {
      subtotal,
      discountAmount,
      shippingCost,
      total
    };
  }, [cart.items, cart.discount, cart.shippingMethod]);

  // 结账
  const handleCheckout = async () => {
    setCart(prev => ({ ...prev, isProcessing: true }));

    try {
      // 模拟结账过程
      await new Promise(resolve => setTimeout(resolve, 2000));

      console.log('Checkout:', {
        items: cart.items,
        totals: cartTotals,
        couponCode: cart.couponCode,
        shippingMethod: cart.shippingMethod
      });

      alert('Order placed successfully!');
      setCart({
        items: [],
        couponCode: '',
        discount: 0,
        shippingMethod: 'standard',
        isProcessing: false
      });
    } catch (error) {
      console.error('Checkout error:', error);
      alert('Checkout failed. Please try again.');
      setCart(prev => ({ ...prev, isProcessing: false }));
    }
  };

  return (
    <div className="shopping-cart">
      <h2>Shopping Cart</h2>

      {cart.items.length === 0 ? (
        <div className="empty-cart">
          <p>Your cart is empty</p>
          <button onClick={() => addToCart(sampleProduct)}>
            Add Sample Product
          </button>
        </div>
      ) : (
        <>
          {/* 购物车项目 */}
          <div className="cart-items">
            {cart.items.map(item => (
              <div key={item.id} className="cart-item">
                <div className="item-info">
                  <h3>{item.name}</h3>
                  <p>Price: ${item.price.toFixed(2)}</p>
                </div>

                <div className="item-quantity">
                  <button
                    onClick={() => updateQuantity(item.id, item.quantity - 1)}
                    disabled={item.quantity <= 1}
                  >
                    -
                  </button>
                  <span>{item.quantity}</span>
                  <button
                    onClick={() => updateQuantity(item.id, item.quantity + 1)}
                  >
                    +
                  </button>
                </div>

                <div className="item-total">
                  <p>${(item.price * item.quantity).toFixed(2)}</p>
                </div>

                <button
                  onClick={() => removeFromCart(item.id)}
                  className="remove-button"
                >
                  ×
                </button>
              </div>
            ))}
          </div>

          {/* 优惠券 */}
          <div className="coupon-section">
            {showCouponInput ? (
              <div className="coupon-input">
                <input
                  type="text"
                  value={cart.couponCode}
                  onChange={(e) => setCart(prev => ({ ...prev, couponCode: e.target.value }))}
                  placeholder="Enter coupon code"
                />
                <button onClick={applyCoupon}>Apply</button>
                <button onClick={() => setShowCouponInput(false)}>Cancel</button>
              </div>
            ) : (
              <button onClick={() => setShowCouponInput(true)}>
                {cart.discount > 0
                  ? `Coupon Applied (${(cart.discount * 100).toFixed(0)}% off)`
                  : 'Apply Coupon'
                }
              </button>
            )}
          </div>

          {/* 配送方式 */}
          <div className="shipping-section">
            <h3>Shipping Method</h3>
            <div className="shipping-options">
              <label>
                <input
                  type="radio"
                  name="shipping"
                  value="standard"
                  checked={cart.shippingMethod === 'standard'}
                  onChange={(e) => setCart(prev => ({ ...prev, shippingMethod: e.target.value }))}
                />
                Standard (3-5 days)
                {cartTotals.shippingCost === 0 ? ' (Free)' : ` ($5.99)`}
              </label>
              <label>
                <input
                  type="radio"
                  name="shipping"
                  value="express"
                  checked={cart.shippingMethod === 'express'}
                  onChange={(e) => setCart(prev => ({ ...prev, shippingMethod: e.target.value }))}
                />
                Express (1-2 days) ($12.99)
              </label>
              <label>
                <input
                  type="radio"
                  name="shipping"
                  value="overnight"
                  checked={cart.shippingMethod === 'overnight'}
                  onChange={(e) => setCart(prev => ({ ...prev, shippingMethod: e.target.value }))}
                />
                Overnight ($24.99)
              </label>
            </div>
          </div>

          {/* 订单摘要 */}
          <div className="order-summary">
            <h3>Order Summary</h3>
            <div className="summary-row">
              <span>Subtotal:</span>
              <span>${cartTotals.subtotal.toFixed(2)}</span>
            </div>
            {cart.discount > 0 && (
              <div className="summary-row discount">
                <span>Discount:</span>
                <span>-${cartTotals.discountAmount.toFixed(2)}</span>
              </div>
            )}
            <div className="summary-row">
              <span>Shipping:</span>
              <span>
                {cartTotals.shippingCost === 0 ? 'Free' : `$${cartTotals.shippingCost.toFixed(2)}`}
              </span>
            </div>
            <div className="summary-row total">
              <span>Total:</span>
              <span>${cartTotals.total.toFixed(2)}</span>
            </div>
          </div>

          {/* 结账按钮 */}
          <button
            onClick={handleCheckout}
            disabled={cart.isProcessing || cart.items.length === 0}
            className="checkout-button"
          >
            {cart.isProcessing ? 'Processing...' : 'Checkout'}
          </button>
        </>
      )}
    </div>
  );
}
```

## 注意事项

### 1. 避免 State 不必要的更新

```jsx
// ❌ 错误：每次渲染都创建新的对象/函数
function BadComponent() {
  const [state, setState] = useState({});

  const handleClick = () => {
    setState({}); // 创建了新对象，可能导致不必要的重新渲染
  };

  return <button onClick={handleClick}>Click</button>;
}

// ✅ 正确：合理使用 useCallback 和 useMemo
function GoodComponent() {
  const [state, setState] = useState({});

  const handleClick = useCallback(() => {
    setState(prev => ({ ...prev })); // 使用函数式更新
  }, []);

  const memoizedValue = useMemo(() => {
    return expensiveCalculation(state);
  }, [state]);

  return <button onClick={handleClick}>Click</button>;
}
```

### 2. State 更新的异步特性

```jsx
function AsyncStateUpdates() {
  const [count, setCount] = useState(0);

  // ❌ 错误：假设状态立即更新
  const badIncrement = () => {
    setCount(count + 1);
    console.log(count); // 输出旧的值
  };

  // ✅ 正确：使用函数式更新
  const goodIncrement = () => {
    setCount(prev => {
      const newCount = prev + 1;
      console.log(newCount); // 输出新的值
      return newCount;
    });
  };

  // ❌ 错误：多次更新时使用旧值
  const badMultipleIncrements = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
    // 只会增加 1，而不是 3
  };

  // ✅ 正确：使用函数式更新
  const goodMultipleIncrements = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);
    // 会增加 3
  };

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={goodIncrement}>Increment</button>
      <button onClick={goodMultipleIncrements}>Increment 3x</button>
    </div>
  );
}
```

### 3. 避免 State 派生冗余

```jsx
// ❌ 错误：存储可以派生的状态
function BadComponent({ items }) {
  const [items, setItems] = useState([]);
  const [filteredItems, setFilteredItems] = useState([]);
  const [sortedItems, setSortedItems] = useState([]);

  // 需要手动同步所有状态
  const handleFilter = (filter) => {
    const filtered = items.filter(item => item.category === filter);
    setFilteredItems(filtered);
    // 还需要更新 sortedItems...
  };

  return <div>{/* 渲染逻辑 */}</div>;
}

// ✅ 正确：只在渲染时计算派生状态
function GoodComponent({ items }) {
  const [items, setItems] = useState([]);
  const [filter, setFilter] = useState('all');
  const [sortBy, setSortBy] = useState('name');

  // 使用 useMemo 缓存计算结果
  const filteredAndSortedItems = useMemo(() => {
    let result = items;

    if (filter !== 'all') {
      result = result.filter(item => item.category === filter);
    }

    result.sort((a, b) => {
      if (a[sortBy] < b[sortBy]) return -1;
      if (a[sortBy] > b[sortBy]) return 1;
      return 0;
    });

    return result;
  }, [items, filter, sortBy]);

  return (
    <div>
      {filteredAndSortedItems.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}
```

## 最佳实践

### 1. 状态结构设计

```jsx
// ✅ 好的做法：扁平化的状态结构
const [user, setUser] = useState({
  id: 1,
  name: 'John',
  email: 'john@example.com',
  preferences: {
    theme: 'light',
    language: 'en'
  }
});

// ✅ 好的做法：相关的状态分组
const [formState, setFormState] = useState({
  values: {
    username: '',
    email: '',
    password: ''
  },
  errors: {},
  touched: {},
  isSubmitting: false
});

// ✅ 好的做法：使用 ID 引用而不是嵌套
const [posts, setPosts] = useState([
  { id: 1, title: 'Post 1', authorId: 1 },
  { id: 2, title: 'Post 2', authorId: 2 }
]);

const [users, setUsers] = useState([
  { id: 1, name: 'John' },
  { id: 2, name: 'Jane' }
]);

// ❌ 不好的做法：过度嵌套的状态
const [badState, setBadState] = useState({
  data: {
    users: {
      list: [
        {
          id: 1,
          profile: {
            name: 'John',
            contact: {
              email: 'john@example.com'
            }
          }
        }
      ]
    }
  }
});
```

### 2. 自定义 Hooks 复用状态逻辑

```jsx
// 自定义 Hook：表单状态管理
function useForm(initialValues, validate) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});

  const handleChange = (event) => {
    const { name, value } = event.target;
    setValues(prev => ({ ...prev, [name]: value }));

    if (touched[name] && validate) {
      setErrors(prev => ({
        ...prev,
        [name]: validate(name, value)
      }));
    }
  };

  const handleBlur = (event) => {
    const { name } = event.target;
    setTouched(prev => ({ ...prev, [name]: true }));

    if (validate) {
      setErrors(prev => ({
        ...prev,
        [name]: validate(name, values[name])
      }));
    }
  };

  const reset = () => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  };

  const validateAll = () => {
    if (!validate) return true;

    const newErrors = {};
    let isValid = true;

    Object.keys(values).forEach(field => {
      const error = validate(field, values[field]);
      if (error) {
        newErrors[field] = error;
        isValid = false;
      }
    });

    setErrors(newErrors);
    setTouched(
      Object.keys(values).reduce((acc, field) => ({ ...acc, [field]: true }), {})
    );

    return isValid;
  };

  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    reset,
    validateAll
  };
}

// 使用自定义 Hook
function LoginForm() {
  const {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    validateAll,
    reset
  } = useForm(
    { username: '', password: '' },
    (field, value) => {
      switch (field) {
        case 'username':
          return !value ? 'Username is required' : '';
        case 'password':
          return !value ? 'Password is required' : '';
        default:
          return '';
      }
    }
  );

  const handleSubmit = (event) => {
    event.preventDefault();
    if (validateAll()) {
      console.log('Form submitted:', values);
      reset();
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <input
        name="username"
        value={values.username}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.username && errors.username && (
        <span className="error">{errors.username}</span>
      )}
      <input
        name="password"
        type="password"
        value={values.password}
        onChange={handleChange}
        onBlur={handleBlur}
      />
      {touched.password && errors.password && (
        <span className="error">{errors.password}</span>
      )}
      <button type="submit">Login</button>
    </form>
  );
}
```

### 3. 性能优化

```jsx
import { useState, useCallback, useMemo } from 'react';

// 1. 使用 useCallback 缓存事件处理器
function OptimizedComponent({ items }) {
  const [selectedItems, setSelectedItems] = useState(new Set());

  const handleToggle = useCallback((id) => {
    setSelectedItems(prev => {
      const newSelected = new Set(prev);
      if (newSelected.has(id)) {
        newSelected.delete(id);
      } else {
        newSelected.add(id);
      }
      return newSelected;
    });
  }, []);

  const handleSelectAll = useCallback(() => {
    setSelectedItems(prev =>
      prev.size === items.length ? new Set() : new Set(items.map(item => item.id))
    );
  }, [items]);

  // 2. 使用 useMemo 缓存计算结果
  const processedItems = useMemo(() => {
    return items.map(item => ({
      ...item,
      processed: true,
      timestamp: Date.now()
    }));
  }, [items]);

  const statistics = useMemo(() => {
    return {
      total: items.length,
      selected: selectedItems.size,
      percentage: items.length > 0
        ? (selectedItems.size / items.length) * 100
        : 0
    };
  }, [items, selectedItems]);

  // 3. 避免在渲染中创建新函数
  return (
    <div>
      <div className="statistics">
        <p>Total: {statistics.total}</p>
        <p>Selected: {statistics.selected}</p>
        <p>Percentage: {statistics.percentage.toFixed(1)}%</p>
      </div>

      <ul>
        {processedItems.map(item => (
          <li key={item.id}>
            <input
              type="checkbox"
              checked={selectedItems.has(item.id)}
              onChange={() => handleToggle(item.id)}
            />
            {item.name}
          </li>
        ))}
      </ul>

      <button onClick={handleSelectAll}>
        {selectedItems.size === items.length ? 'Deselect All' : 'Select All'}
      </button>
    </div>
  );
}
```

## 总结

React State 是构建交互式应用的核心，正确理解和使用 State 对于开发高质量的 React 应用至关重要。

### 关键要点

1. **不可变性**：State 应该被视为不可变的，更新时创建新的状态值
2. **函数式更新**：使用函数式更新避免异步更新带来的问题
3. **状态结构**：设计合理的状态结构，避免过度嵌套和冗余状态
4. **性能优化**：使用 useCallback、useMemo 等优化技术避免不必要的重新渲染
5. **代码复用**：通过自定义 Hooks 复用状态逻辑

### 学习建议

1. **掌握基础**：熟练掌握 useState 的各种使用场景
2. **理解机制**：深入理解 State 更新的异步机制和批量处理
3. **性能意识**：学习识别和优化性能问题
4. **模式学习**：掌握常见的状态管理模式和最佳实践
5. **工具使用**：学习使用 React DevTools 等工具调试状态问题

State 管理是 React 开发的基础技能，但也是最容易出问题的地方。通过持续的实践和对 React 内部机制的理解，你将能够编写出高效、可维护的状态管理代码。