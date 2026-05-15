---
title: React JSX 语法
published: 2023-07-03
description: 'JSX 语法规则的详细介绍和学习笔记'
image: ''
tags: ["React","JSX"]
category: 'React'
draft: false
lang: 'zh-CN'
---

# React JSX 语法

## 概述

JSX (JavaScript XML) 是 React 的核心特性之一，它是一种语法扩展，允许我们在 JavaScript 代码中编写类似 HTML 的代码。JSX 不是必需的，但它极大地提升了 React 开发的体验和效率。通过 JSX，开发者可以用声明式的方式描述 UI 结构，让代码更加直观、易读和易维护。

### JSX 的本质

JSX 本质上是 JavaScript 的语法糖，在编译时会被 Babel 等工具转换为普通的 JavaScript 函数调用。React.createElement() 是底层的渲染函数，而 JSX 提供了更友好的语法。

```jsx
// JSX 语法
const element = <h1>Hello, World!</h1>;

// 编译后的 JavaScript
const element = React.createElement('h1', null, 'Hello, World!');
```

## 核心概念

### 1. JSX 与 HTML 的区别

虽然 JSX 看起来很像 HTML，但它们有一些重要的区别：

```jsx
// 1. className 而不是 class
// ❌ 错误
<div class="container">Content</div>

// ✅ 正确
<div className="container">Content</div>

// 2. camelCase 属性名
// ❌ 错误
<input type="text" onchange={handleChange} />

// ✅ 正确
<input type="text" onChange={handleChange} />

// 3. 自闭合标签
// ❌ 错误
<img src="image.jpg">
<br>

// ✅ 正确
<img src="image.jpg" />
<br />

// 4. JavaScript 表达式
const name = 'John';
const age = 30;

// ✅ 使用花括号包裹表达式
<div>
  <p>Name: {name}</p>
  <p>Age: {age}</p>
  <p>Next year: {age + 1}</p>
</div>
```

### 2. JSX 表达式

JSX 中的花括号 `{}` 可以包含任何 JavaScript 表达式：

```jsx
function App() {
  const name = 'John';
  const isLoggedIn = true;
  const items = ['Apple', 'Banana', 'Orange'];

  return (
    <div>
      {/* 变量 */}
      <h1>Hello, {name}!</h1>

      {/* 表达式计算 */}
      <p>2 + 2 = {2 + 2}</p>
      <p>Age: {30}</p>

      {/* 函数调用 */}
      <p>Uppercase: {name.toUpperCase()}</p>
      <p>Greeting: {getGreeting(name)}</p>

      {/* 三元表达式 */}
      <p>Status: {isLoggedIn ? 'Logged in' : 'Not logged in'}</p>

      {/* 逻辑与 */}
      {isLoggedIn && <p>Welcome back!</p>}

      {/* 数组方法 */}
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}

function getGreeting(name) {
  return `Hello, ${name}!`;
}
```

### 3. JSX 类型

JSX 可以表示 HTML 标签和 React 组件：

```jsx
// HTML 标签（小写开头）
const divElement = <div>This is a div</div>;
const buttonElement = <button>Click me</button>;

// React 组件（大写开头）
function Welcome() {
  return <h1>Welcome!</h1>;
}

const welcomeElement = <Welcome />;

// 带属性的组件
function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
}

const greetingElement = <Greeting name="John" />;
```

## 基本用法

### 1. 基本元素创建

```jsx
import React from 'react';

// 简单元素
const simpleElement = <h1>Hello, World!</h1>;

// 带属性的元素
const styledElement = (
  <div className="container" id="main">
    <p style={{ color: 'red', fontSize: '16px' }}>
      This is a styled paragraph
    </p>
  </div>
);

// 嵌套元素
const nestedElement = (
  <div className="card">
    <div className="card-header">
      <h2>Card Title</h2>
    </div>
    <div className="card-body">
      <p>Card content goes here</p>
      <button>Action</button>
    </div>
  </div>
);
```

### 2. 属性设置

```jsx
function AttributeExample() {
  const handleClick = () => console.log('Clicked');
  const inputValue = 'Default value';
  const imageUrl = 'https://example.com/image.jpg';

  return (
    <div>
      {/* 基本属性 */}
      <input type="text" placeholder="Enter text" />
      <button disabled={false}>Enabled Button</button>

      {/* 动态属性 */}
      <input value={inputValue} onChange={handleChange} />

      {/* 事件处理 */}
      <button onClick={handleClick}>Click me</button>

      {/* boolean 属性 */}
      <input type="checkbox" checked={true} />
      <input type="checkbox" defaultChecked={true} />

      {/* 自定义属性（data- 属性） */}
      <div data-id="123" data-user="john">
        Custom data attributes
      </div>

      {/* HTML 转义 */}
      <div dangerouslySetInnerHTML={{
        __html: '<strong>Bold text</strong>'
      }} />
    </div>
  );
}

function handleChange(event) {
  console.log(event.target.value);
}
```

### 3. 条件渲染

```jsx
function ConditionalRendering() {
  const [isLoggedIn, setIsLoggedIn] = React.useState(false);
  const [userRole, setUserRole] = React.useState('user');

  return (
    <div>
      {/* 1. 三元表达式 */}
      <div>
        {isLoggedIn ? <WelcomeUser /> : <LoginForm />}
      </div>

      {/* 2. 逻辑与操作符 */}
      <div>
        {isLoggedIn && <UserDashboard />}
      </div>

      {/* 3. 逻辑或操作符 */}
      <div>
        {userRole === 'admin' || <p>Access denied</p>}
      </div>

      {/* 4. 变量存储 */}
      <div>
        {(() => {
          if (userRole === 'admin') {
            return <AdminPanel />;
          } else if (userRole === 'user') {
            return <UserPanel />;
          } else {
            return <GuestPanel />;
          }
        })()}
      </div>

      {/* 5. 组件封装 */}
      <div>
        <ConditionalComponent
          condition={isLoggedIn}
          trueComponent={<WelcomeUser />}
          falseComponent={<LoginForm />}
        />
      </div>
    </div>
  );
}

function ConditionalComponent({ condition, trueComponent, falseComponent }) {
  return condition ? trueComponent : falseComponent;
}
```

### 4. 列表渲染

```jsx
function ListRendering() {
  const [items, setItems] = React.useState([
    { id: 1, name: 'Apple', price: 1.99 },
    { id: 2, name: 'Banana', price: 0.99 },
    { id: 3, name: 'Orange', price: 2.49 }
  ]);

  const [users, setUsers] = React.useState([
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' }
  ]);

  return (
    <div>
      {/* 基本列表 */}
      <h2>Fruits</h2>
      <ul>
        {items.map((item, index) => (
          <li key={item.id}>
            {item.name} - ${item.price}
          </li>
        ))}
      </ul>

      {/* 嵌套列表 */}
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <strong>{user.name}</strong>
            <ul>
              <li>Email: {user.email}</li>
              <li>ID: {user.id}</li>
            </ul>
          </li>
        ))}
      </ul>

      {/* 条件列表渲染 */}
      <h2>Expensive Items</h2>
      <ul>
        {items
          .filter(item => item.price > 1.5)
          .map(item => (
            <li key={item.id}>
              {item.name} - ${item.price}
            </li>
          ))}
      </ul>

      {/* 带索引的列表（仅在静态列表中使用） */}
      <h2>Simple List</h2>
      <ul>
        {['First', 'Second', 'Third'].map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 5. 表单处理

```jsx
function FormHandling() {
  const [formData, setFormData] = React.useState({
    username: '',
    email: '',
    password: '',
    gender: 'male',
    interests: []
  });

  const handleChange = (event) => {
    const { name, value, type, checked } = event.target;

    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox'
        ? checked
          ? [...prev.interests, value]
          : prev.interests.filter(item => item !== value)
        : value
    }));
  };

  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted:', formData);
  };

  return (
    <form onSubmit={handleSubmit}>
      {/* 文本输入 */}
      <div>
        <label htmlFor="username">Username:</label>
        <input
          type="text"
          id="username"
          name="username"
          value={formData.username}
          onChange={handleChange}
          placeholder="Enter username"
        />
      </div>

      {/* 邮箱输入 */}
      <div>
        <label htmlFor="email">Email:</label>
        <input
          type="email"
          id="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Enter email"
        />
      </div>

      {/* 密码输入 */}
      <div>
        <label htmlFor="password">Password:</label>
        <input
          type="password"
          id="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          placeholder="Enter password"
        />
      </div>

      {/* 单选按钮 */}
      <div>
        <label>Gender:</label>
        <label>
          <input
            type="radio"
            name="gender"
            value="male"
            checked={formData.gender === 'male'}
            onChange={handleChange}
          />
          Male
        </label>
        <label>
          <input
            type="radio"
            name="gender"
            value="female"
            checked={formData.gender === 'female'}
            onChange={handleChange}
          />
          Female
        </label>
      </div>

      {/* 复选框 */}
      <div>
        <label>Interests:</label>
        <label>
          <input
            type="checkbox"
            name="interests"
            value="coding"
            checked={formData.interests.includes('coding')}
            onChange={handleChange}
          />
          Coding
        </label>
        <label>
          <input
            type="checkbox"
            name="interests"
            value="design"
            checked={formData.interests.includes('design')}
            onChange={handleChange}
          />
          Design
        </label>
        <label>
          <input
            type="checkbox"
            name="interests"
            value="music"
            checked={formData.interests.includes('music')}
            onChange={handleChange}
          />
          Music
        </label>
      </div>

      {/* 下拉选择 */}
      <div>
        <label htmlFor="country">Country:</label>
        <select
          id="country"
          name="country"
          value={formData.country}
          onChange={handleChange}
        >
          <option value="">Select country</option>
          <option value="us">United States</option>
          <option value="uk">United Kingdom</option>
          <option value="cn">China</option>
        </select>
      </div>

      {/* 文本区域 */}
      <div>
        <label htmlFor="bio">Bio:</label>
        <textarea
          id="bio"
          name="bio"
          value={formData.bio}
          onChange={handleChange}
          rows="4"
          placeholder="Tell us about yourself"
        />
      </div>

      <button type="submit">Submit</button>
    </form>
  );
}
```

## 实际应用

### 1. 响应式导航栏

```jsx
function ResponsiveNavbar() {
  const [isOpen, setIsOpen] = React.useState(false);
  const [scrolled, setScrolled] = React.useState(false);

  React.useEffect(() => {
    const handleScroll = () => {
      setScrolled(window.scrollY > 50);
    };

    window.addEventListener('scroll', handleScroll);
    return () => window.removeEventListener('scroll', handleScroll);
  }, []);

  const navItems = [
    { name: 'Home', path: '/' },
    { name: 'About', path: '/about' },
    { name: 'Services', path: '/services' },
    { name: 'Contact', path: '/contact' }
  ];

  return (
    <nav className={`navbar ${scrolled ? 'scrolled' : ''}`}>
      <div className="nav-container">
        <div className="nav-brand">
          <a href="/">MyBrand</a>
        </div>

        {/* 桌面菜单 */}
        <ul className="nav-menu desktop">
          {navItems.map(item => (
            <li key={item.path}>
              <a href={item.path}>{item.name}</a>
            </li>
          ))}
        </ul>

        {/* 移动菜单按钮 */}
        <button
          className="nav-toggle"
          onClick={() => setIsOpen(!isOpen)}
          aria-label="Toggle menu"
        >
          <span className={`hamburger ${isOpen ? 'active' : ''}`}>
            <span></span>
            <span></span>
            <span></span>
          </span>
        </button>
      </div>

      {/* 移动菜单 */}
      <ul className={`nav-menu mobile ${isOpen ? 'active' : ''}`}>
        {navItems.map(item => (
          <li key={item.path}>
            <a
              href={item.path}
              onClick={() => setIsOpen(false)}
            >
              {item.name}
            </a>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

### 2. 产品卡片组件

```jsx
function ProductCard({ product }) {
  const [isInCart, setIsInCart] = React.useState(false);
  const [quantity, setQuantity] = React.useState(1);

  const handleAddToCart = () => {
    console.log(`Adding ${quantity} ${product.name}(s) to cart`);
    setIsInCart(true);
  };

  const handleQuantityChange = (newQuantity) => {
    if (newQuantity >= 1 && newQuantity <= product.maxQuantity) {
      setQuantity(newQuantity);
    }
  };

  return (
    <div className="product-card">
      {/* 产品图片 */}
      <div className="product-image">
        <img src={product.image} alt={product.name} />
        {product.discount && (
          <span className="discount-badge">
            {product.discount}% OFF
          </span>
        )}
      </div>

      {/* 产品信息 */}
      <div className="product-info">
        <h3 className="product-name">{product.name}</h3>
        <p className="product-description">
          {product.description}
        </p>

        {/* 价格 */}
        <div className="product-price">
          {product.discount ? (
            <>
              <span className="original-price">
                ${product.originalPrice.toFixed(2)}
              </span>
              <span className="discounted-price">
                ${product.price.toFixed(2)}
              </span>
            </>
          ) : (
            <span className="price">
              ${product.price.toFixed(2)}
            </span>
          )}
        </div>

        {/* 数量选择器 */}
        <div className="quantity-selector">
          <button
            onClick={() => handleQuantityChange(quantity - 1)}
            disabled={quantity <= 1}
          >
            -
          </button>
          <input
            type="number"
            value={quantity}
            onChange={(e) => handleQuantityChange(parseInt(e.target.value) || 1)}
            min="1"
            max={product.maxQuantity}
          />
          <button
            onClick={() => handleQuantityChange(quantity + 1)}
            disabled={quantity >= product.maxQuantity}
          >
            +
          </button>
        </div>

        {/* 购买按钮 */}
        <button
          className={`add-to-cart-btn ${isInCart ? 'in-cart' : ''}`}
          onClick={handleAddToCart}
          disabled={isInCart}
        >
          {isInCart ? 'Added to Cart' : 'Add to Cart'}
        </button>

        {/* 产品特性 */}
        {product.features && (
          <ul className="product-features">
            {product.features.map((feature, index) => (
              <li key={index}>
                <span className="feature-icon">✓</span>
                {feature}
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}

// 使用示例
function ProductList() {
  const products = [
    {
      id: 1,
      name: 'Premium Wireless Headphones',
      description: 'High-quality wireless headphones with noise cancellation',
      price: 199.99,
      originalPrice: 249.99,
      discount: 20,
      image: '/images/headphones.jpg',
      maxQuantity: 5,
      features: [
        'Noise cancellation',
        '30-hour battery life',
        'Bluetooth 5.0',
        'Comfortable fit'
      ]
    },
    {
      id: 2,
      name: 'Smart Watch Series X',
      description: 'Advanced smartwatch with health tracking',
      price: 299.99,
      image: '/images/smartwatch.jpg',
      maxQuantity: 3,
      features: [
        'Heart rate monitor',
        'GPS tracking',
        'Water resistant',
        '7-day battery'
      ]
    }
  ];

  return (
    <div className="product-list">
      {products.map(product => (
        <ProductCard key={product.id} product={product} />
      ))}
    </div>
  );
}
```

### 3. 数据表格组件

```jsx
function DataTable({ data, columns }) {
  const [sortConfig, setSortConfig] = React.useState({
    key: null,
    direction: 'ascending'
  });
  const [selectedRows, setSelectedRows] = React.useState(new Set());
  const [filter, setFilter] = React.useState('');

  // 排序处理
  const handleSort = (key) => {
    let direction = 'ascending';
    if (sortConfig.key === key && sortConfig.direction === 'ascending') {
      direction = 'descending';
    }
    setSortConfig({ key, direction });
  };

  // 过滤和排序数据
  const processedData = React.useMemo(() => {
    let filteredData = data;

    // 过滤
    if (filter) {
      filteredData = data.filter(item =>
        Object.values(item).some(value =>
          String(value).toLowerCase().includes(filter.toLowerCase())
        )
      );
    }

    // 排序
    if (sortConfig.key) {
      filteredData.sort((a, b) => {
        if (a[sortConfig.key] < b[sortConfig.key]) {
          return sortConfig.direction === 'ascending' ? -1 : 1;
        }
        if (a[sortConfig.key] > b[sortConfig.key]) {
          return sortConfig.direction === 'ascending' ? 1 : -1;
        }
        return 0;
      });
    }

    return filteredData;
  }, [data, filter, sortConfig]);

  // 行选择处理
  const handleRowSelect = (id) => {
    const newSelected = new Set(selectedRows);
    if (newSelected.has(id)) {
      newSelected.delete(id);
    } else {
      newSelected.add(id);
    }
    setSelectedRows(newSelected);
  };

  const handleSelectAll = () => {
    if (selectedRows.size === processedData.length) {
      setSelectedRows(new Set());
    } else {
      setSelectedRows(new Set(processedData.map(item => item.id)));
    }
  };

  return (
    <div className="data-table-container">
      {/* 搜索过滤 */}
      <div className="table-controls">
        <input
          type="text"
          placeholder="Search..."
          value={filter}
          onChange={(e) => setFilter(e.target.value)}
          className="search-input"
        />
        <div className="selection-info">
          {selectedRows.size > 0 && (
            <span>{selectedRows.size} selected</span>
          )}
        </div>
      </div>

      {/* 数据表格 */}
      <div className="table-wrapper">
        <table className="data-table">
          <thead>
            <tr>
              <th>
                <input
                  type="checkbox"
                  checked={selectedRows.size === processedData.length}
                  onChange={handleSelectAll}
                />
              </th>
              {columns.map(column => (
                <th
                  key={column.key}
                  onClick={() => handleSort(column.key)}
                  className={sortConfig.key === column.key ? 'sorted' : ''}
                >
                  {column.label}
                  {sortConfig.key === column.key && (
                    <span className="sort-indicator">
                      {sortConfig.direction === 'ascending' ? '↑' : '↓'}
                    </span>
                  )}
                </th>
              ))}
            </tr>
          </thead>
          <tbody>
            {processedData.map(row => (
              <tr
                key={row.id}
                className={selectedRows.has(row.id) ? 'selected' : ''}
              >
                <td>
                  <input
                    type="checkbox"
                    checked={selectedRows.has(row.id)}
                    onChange={() => handleRowSelect(row.id)}
                  />
                </td>
                {columns.map(column => (
                  <td key={column.key}>
                    {column.render
                      ? column.render(row[column.key], row)
                      : row[column.key]}
                  </td>
                ))}
              </tr>
            ))}
          </tbody>
        </table>
      </div>

      {/* 分页 */}
      <div className="table-pagination">
        <span>Total: {processedData.length} items</span>
      </div>
    </div>
  );
}

// 使用示例
function UserTable() {
  const users = [
    { id: 1, name: 'John Doe', email: 'john@example.com', role: 'Admin', status: 'Active' },
    { id: 2, name: 'Jane Smith', email: 'jane@example.com', role: 'User', status: 'Active' },
    { id: 3, name: 'Bob Johnson', email: 'bob@example.com', role: 'User', status: 'Inactive' },
    { id: 4, name: 'Alice Williams', email: 'alice@example.com', role: 'Editor', status: 'Active' },
  ];

  const columns = [
    { key: 'name', label: 'Name' },
    { key: 'email', label: 'Email' },
    {
      key: 'role',
      label: 'Role',
      render: (role) => (
        <span className={`role-badge ${role.toLowerCase()}`}>
          {role}
        </span>
      )
    },
    {
      key: 'status',
      label: 'Status',
      render: (status) => (
        <span className={`status-badge ${status.toLowerCase()}`}>
          {status}
        </span>
      )
    }
  ];

  return <DataTable data={users} columns={columns} />;
}
```

## 注意事项

### 1. JSX 语法规则

```jsx
// ✅ 正确：单个根元素
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// ✅ 正确：使用 Fragment
import { Fragment } from 'react';

function App() {
  return (
    <Fragment>
      <h1>Title</h1>
      <p>Content</p>
    </Fragment>
  );
}

// ✅ 正确：使用简写语法
function App() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// ❌ 错误：多个根元素
function App() {
  return (
    <h1>Title</h1>
    <p>Content</p>
  );
}
```

### 2. 属性命名约定

```jsx
// ✅ 正确：使用 camelCase
<div className="container" />
<input onChange={handleChange} />

// ❌ 错误：使用 kebab-case
<div class="container" />
<input on-change={handleChange} />

// ✅ 正确：特殊属性名
<label htmlFor="input" />
<iframe srcDoc="<p>Hello</p>" />

// ❌ 错误：使用 HTML 属性名
<label for="input" />
<iframe srcdoc="<p>Hello</p>" />
```

### 3. 表达式使用

```jsx
// ✅ 正确：使用花括号包裹表达式
const name = 'John';
<div>Hello, {name}!</div>

// ✅ 正确：对象样式
<div style={{ color: 'red', fontSize: '16px' }} />

// ❌ 错误：在属性中使用引号包裹表达式
<div class="{name}" />

// ❌ 错误：在 JSX 中使用注释
<div>
  <!-- This is a comment -->
</div>

// ✅ 正确：使用 JavaScript 注释
<div>
  {/* This is a comment */}
</div>
```

### 4. 条件渲染最佳实践

```jsx
// ✅ 好的做法：使用三元表达式
<div>
  {isLoggedIn ? <UserDashboard /> : <LoginForm />}
</div>

// ✅ 好的做法：使用逻辑与
<div>
  {showMessage && <Message />}
</div>

// ❌ 不好的做法：在表达式中使用 if 语句
<div>
  {if (isLoggedIn) {
    return <UserDashboard />;
  }}
</div>

// ✅ 好的做法：使用立即执行函数或变量
<div>
  {(() => {
    if (isLoggedIn) {
      return <UserDashboard />;
    }
    return <LoginForm />;
  })()}
</div>
```

### 5. 列表渲染和 Keys

```jsx
// ✅ 正确：使用稳定的、唯一的 key
{items.map(item => (
  <div key={item.id}>{item.name}</div>
))}

// ✅ 正确：对于静态列表使用索引
{['Apple', 'Banana'].map((fruit, index) => (
  <li key={index}>{fruit}</li>
))}

// ❌ 错误：使用不稳定的 key（如随机数）
{items.map(item => (
  <div key={Math.random()}>{item.name}</div>
))}

// ❌ 错误：在可能重新排序的列表中使用索引
{sortedItems.map((item, index) => (
  <div key={index}>{item.name}</div>
))}
```

## 最佳实践

### 1. 组件拆分和复用

```jsx
// ❌ 不好的做法：组件过于复杂
function ComplexComponent() {
  return (
    <div className="container">
      <header>
        <div className="logo">
          <img src="/logo.png" alt="Logo" />
        </div>
        <nav>
          <ul>
            <li><a href="/">Home</a></li>
            <li><a href="/about">About</a></li>
            <li><a href="/contact">Contact</a></li>
          </ul>
        </nav>
      </header>
      <main>
        <section className="hero">
          <h1>Welcome</h1>
          <p>Some description</p>
        </section>
      </main>
      <footer>
        <p>&copy; 2024 My App</p>
      </footer>
    </div>
  );
}

// ✅ 好的做法：拆分成小组件
function ComplexComponent() {
  return (
    <div className="container">
      <Header />
      <Main />
      <Footer />
    </div>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Navigation />
    </header>
  );
}

function Navigation() {
  const navItems = [
    { name: 'Home', path: '/' },
    { name: 'About', path: '/about' },
    { name: 'Contact', path: '/contact' }
  ];

  return (
    <nav>
      <ul>
        {navItems.map(item => (
          <li key={item.path}>
            <a href={item.path}>{item.name}</a>
          </li>
        ))}
      </ul>
    </nav>
  );
}
```

### 2. Props 默认值和类型检查

```jsx
import PropTypes from 'prop-types';

// ✅ 好的做法：设置默认值和类型检查
function Button({
  children,
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  className = ''
}) {
  const buttonClass = `btn btn-${variant} btn-${size} ${className}`;

  return (
    <button
      className={buttonClass}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

Button.propTypes = {
  children: PropTypes.node.isRequired,
  variant: PropTypes.oneOf(['primary', 'secondary', 'danger']),
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  disabled: PropTypes.bool,
  onClick: PropTypes.func,
  className: PropTypes.string
};

// 使用示例
function App() {
  return (
    <div>
      <Button onClick={() => console.log('Clicked')}>
        Click me
      </Button>
      <Button variant="secondary" size="large">
        Large Button
      </Button>
    </div>
  );
}
```

### 3. 条件渲染优化

```jsx
// ✅ 好的做法：提取条件渲染逻辑
function ConditionalComponent({ condition, children }) {
  if (!condition) return null;
  return <>{children}</>;
}

// 使用示例
function App() {
  const [user, setUser] = useState(null);

  return (
    <div>
      <ConditionalComponent condition={user}>
        <UserProfile user={user} />
        <UserSettings user={user} />
      </ConditionalComponent>

      <ConditionalComponent condition={!user}>
        <LoginForm onLogin={setUser} />
      </ConditionalComponent>
    </div>
  );
}

// ✅ 好的做法：使用渲染模式
function List({ items, emptyMessage }) {
  if (!items || items.length === 0) {
    return <div className="empty">{emptyMessage}</div>;
  }

  return (
    <ul className="list">
      {items.map(item => (
        <li key={item.id} className="list-item">
          {item.name}
        </li>
      ))}
    </ul>
  );
}
```

## 总结

JSX 是 React 开发的核心语法，它将 JavaScript 的强大功能与 HTML 的直观性结合在一起，为开发者提供了出色的开发体验。

### 关键要点

1. **语法规则**：掌握 JSX 的基本语法规则和与 HTML 的区别
2. **表达式使用**：熟练使用花括号嵌入 JavaScript 表达式
3. **条件渲染**：灵活运用各种条件渲染技术
4. **列表渲染**：正确使用 key 和列表渲染方法
5. **最佳实践**：遵循组件拆分、代码复用等最佳实践

### 学习建议

1. **实践为主**：通过编写实际组件来掌握 JSX 语法
2. **注意细节**：特别关注属性命名、表达式使用等细节
3. **性能优化**：理解 JSX 编译过程，编写高效的组件代码
4. **工具使用**：熟练使用 ESLint、Prettier 等工具提高代码质量
5. **持续学习**：关注 React 生态系统的最新发展

JSX 的学习曲线相对平缓，但要编写高质量的 JSX 代码需要持续的练习和对 React 深入的理解。通过不断的实践和总结，你将能够编写出优雅、高效的 JSX 代码。