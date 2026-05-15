---
title: React 入门介绍
published: 2023-11-01
description: 'React 核心概念和特点的详细介绍和学习笔记'
image: ''
tags: ["React","基础"]
category: 'React'
draft: false
lang: 'zh-CN'
---

# React 入门介绍

## 概述

React 是由 Facebook 开发的一个用于构建用户界面的 JavaScript 库，特别擅长构建单页面应用（SPA）和复杂的交互式 Web 应用。React 通过组件化、声明式编程和虚拟 DOM 等概念，革命性地改变了现代前端开发的方式。它专注于视图层，可以轻松与其他库或框架配合使用，形成完整的前端解决方案。

### React 的核心优势

**声明式编程**：通过描述 UI 应该是什么样子，而不是如何通过操作 DOM 来实现，让代码更易读、易维护。

**组件化架构**：将复杂的 UI 拆分成独立、可复用的组件，每个组件关注自己的逻辑和渲染。

**虚拟 DOM**：通过在内存中维护一个虚拟 DOM 树，最小化实际 DOM 操作，提升性能。

**单向数据流**：数据从父组件流向子组件，使得数据流清晰可追踪，减少状态管理的复杂度。

**生态系统丰富**：拥有庞大的第三方库支持，覆盖路由、状态管理、UI 组件等各个方面。

## 核心概念

### 1. JSX

JSX 是 React 的语法扩展，允许在 JavaScript 中编写类似 HTML 的代码。它不是必需的，但极大地提升了开发体验。

```jsx
// JSX 基本语法
const element = <h1>Hello, React!</h1>;

// 在 JSX 中嵌入表达式
const name = 'World';
const greeting = <h1>Hello, {name}!</h1>;

// 使用函数调用
function formatName(user) {
  return user.firstName + ' ' + user.lastName;
}

const user = { firstName: 'John', lastName: 'Doe' };
const element = <h1>Hello, {formatName(user)}!</h1>;

// 属性指定
const element = <div className="container" id="main">Content</div>;

// 条件渲染
const isLoggedIn = true;
const element = (
  <div>
    {isLoggedIn ? <Welcome /> : <Login />}
  </div>
);
```

### 2. 组件

React 应用由组件构成，组件是独立的、可复用的代码块。

```jsx
// 函数组件（推荐）
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// 箭头函数组件
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};

// 简化的箭头函数组件
const Welcome = ({ name }) => <h1>Hello, {name}</h1>;

// 类组件（传统方式）
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// 组件组合
function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
      <Welcome name="Charlie" />
    </div>
  );
}
```

### 3. Props

Props 是组件间传递数据的方式，是只读的。

```jsx
// 父组件传递数据
function ParentComponent() {
  const user = {
    name: 'John',
    age: 30,
    email: 'john@example.com'
  };

  return (
    <ChildComponent
      user={user}
      title="用户信息"
      isActive={true}
      items={['item1', 'item2', 'item3']}
    />
  );
}

// 子组件接收和使用 props
function ChildComponent({ user, title, isActive, items }) {
  return (
    <div className={`card ${isActive ? 'active' : ''}`}>
      <h2>{title}</h2>
      <p>Name: {user.name}</p>
      <p>Age: {user.age}</p>
      <p>Email: {user.email}</p>
      <ul>
        {items.map((item, index) => (
          <li key={index}>{item}</li>
        ))}
      </ul>
    </div>
  );
}
```

### 4. State

State 是组件内部的状态，可以随时间变化。

```jsx
import { useState } from 'react';

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

// 对象状态
function UserProfile() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const updateName = (name) => {
    setUser(prev => ({ ...prev, name }));
  };

  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => updateName(e.target.value)}
        placeholder="Enter name"
      />
      <p>Name: {user.name}</p>
    </div>
  );
}
```

### 5. 事件处理

React 使用合成事件系统，提供跨浏览器的一致性体验。

```jsx
function ButtonExample() {
  // 基本事件处理
  const handleClick = () => {
    console.log('Button clicked!');
  };

  // 事件对象处理
  const handleSubmit = (event) => {
    event.preventDefault();
    console.log('Form submitted');
  };

  // 传递参数
  const handleDelete = (id) => {
    console.log('Deleting item:', id);
  };

  return (
    <div>
      <button onClick={handleClick}>
        Click me
      </button>

      <form onSubmit={handleSubmit}>
        <button type="submit">Submit</button>
      </form>

      <button onClick={() => handleDelete(123)}>
        Delete Item 123
      </button>
    </div>
  );
}
```

## 基本用法

### 创建 React 应用

```bash
# 使用 Create React App（传统方式）
npx create-react-app my-app
cd my-app
npm start

# 使用 Vite（现代推荐）
npm create vite@latest my-app -- --template react
cd my-app
npm install
npm run dev

# 使用 Next.js（全栈框架）
npx create-next-app@latest my-app
cd my-app
npm run dev
```

### 项目结构

```
my-app/
├── public/
│   ├── index.html
│   └── favicon.ico
├── src/
│   ├── components/
│   │   ├── Header.jsx
│   │   ├── Footer.jsx
│   │   └── Button.jsx
│   ├── pages/
│   │   ├── Home.jsx
│   │   └── About.jsx
│   ├── App.jsx
│   ├── App.css
│   └── index.js
├── package.json
└── vite.config.js
```

### 组件开发模板

```jsx
// Button.jsx - 可复用按钮组件
import React from 'react';
import './Button.css';

const Button = ({
  children,
  type = 'button',
  variant = 'primary',
  size = 'medium',
  disabled = false,
  onClick,
  className = '',
  ...props
}) => {
  const buttonClass = `btn btn-${variant} btn-${size} ${className}`;

  return (
    <button
      type={type}
      className={buttonClass}
      disabled={disabled}
      onClick={onClick}
      {...props}
    >
      {children}
    </button>
  );
};

export default Button;

// 使用示例
function App() {
  const handleClick = () => {
    console.log('Button clicked');
  };

  return (
    <div>
      <Button onClick={handleClick}>
        Primary Button
      </Button>
      <Button variant="secondary" size="large">
        Secondary Large Button
      </Button>
      <Button variant="danger" disabled>
        Disabled Button
      </Button>
    </div>
  );
}
```

## 实际应用

### 待办事项列表

```jsx
import { useState } from 'react';

function TodoApp() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'Learn React', completed: false },
    { id: 2, text: 'Build a project', completed: false }
  ]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      const newTodo = {
        id: Date.now(),
        text: inputValue,
        completed: false
      };
      setTodos([...todos, newTodo]);
      setInputValue('');
    }
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
    <div style={{ maxWidth: '600px', margin: '0 auto', padding: '20px' }}>
      <h1>Todo List</h1>

      <div style={{ marginBottom: '20px' }}>
        <input
          type="text"
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="Add a new todo"
          onKeyPress={(e) => e.key === 'Enter' && addTodo()}
          style={{
            padding: '10px',
            marginRight: '10px',
            width: 'calc(100% - 100px)'
          }}
        />
        <button
          onClick={addTodo}
          style={{
            padding: '10px 20px',
            cursor: 'pointer'
          }}
        >
          Add
        </button>
      </div>

      <ul style={{ listStyle: 'none', padding: 0 }}>
        {todos.map(todo => (
          <li
            key={todo.id}
            style={{
              display: 'flex',
              alignItems: 'center',
              marginBottom: '10px',
              padding: '10px',
              backgroundColor: '#f5f5f5',
              borderRadius: '4px'
            }}
          >
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => toggleTodo(todo.id)}
              style={{ marginRight: '10px' }}
            />
            <span
              style={{
                flex: 1,
                textDecoration: todo.completed ? 'line-through' : 'none'
              }}
            >
              {todo.text}
            </span>
            <button
              onClick={() => deleteTodo(todo.id)}
              style={{
                padding: '5px 10px',
                backgroundColor: '#ff4444',
                color: 'white',
                border: 'none',
                borderRadius: '4px',
                cursor: 'pointer'
              }}
            >
              Delete
            </button>
          </li>
        ))}
      </ul>

      <p>Total: {todos.length} | Completed: {todos.filter(t => t.completed).length}</p>
    </div>
  );
}

export default TodoApp;
```

### 表单处理

```jsx
import { useState } from 'react';

function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    subject: '',
    message: ''
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: value
    }));
    // 清除对应字段的错误
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.name.trim()) {
      newErrors.name = 'Name is required';
    }

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!formData.subject.trim()) {
      newErrors.subject = 'Subject is required';
    }

    if (!formData.message.trim()) {
      newErrors.message = 'Message is required';
    }

    return newErrors;
  };

  const handleSubmit = async (e) => {
    e.preventDefault();

    const validationErrors = validateForm();
    if (Object.keys(validationErrors).length > 0) {
      setErrors(validationErrors);
      return;
    }

    setIsSubmitting(true);

    try {
      // 模拟 API 调用
      await new Promise(resolve => setTimeout(resolve, 2000));
      console.log('Form submitted:', formData);
      alert('Message sent successfully!');
      setFormData({ name: '', email: '', subject: '', message: '' });
    } catch (error) {
      console.error('Error submitting form:', error);
      alert('Error sending message. Please try again.');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <div style={{ maxWidth: '500px', margin: '0 auto', padding: '20px' }}>
      <h2>Contact Us</h2>

      <form onSubmit={handleSubmit}>
        <div style={{ marginBottom: '15px' }}>
          <label htmlFor="name" style={{ display: 'block', marginBottom: '5px' }}>
            Name:
          </label>
          <input
            type="text"
            id="name"
            name="name"
            value={formData.name}
            onChange={handleChange}
            style={{
              width: '100%',
              padding: '10px',
              border: errors.name ? '1px solid red' : '1px solid #ccc',
              borderRadius: '4px'
            }}
          />
          {errors.name && <p style={{ color: 'red', fontSize: '12px' }}>{errors.name}</p>}
        </div>

        <div style={{ marginBottom: '15px' }}>
          <label htmlFor="email" style={{ display: 'block', marginBottom: '5px' }}>
            Email:
          </label>
          <input
            type="email"
            id="email"
            name="email"
            value={formData.email}
            onChange={handleChange}
            style={{
              width: '100%',
              padding: '10px',
              border: errors.email ? '1px solid red' : '1px solid #ccc',
              borderRadius: '4px'
            }}
          />
          {errors.email && <p style={{ color: 'red', fontSize: '12px' }}>{errors.email}</p>}
        </div>

        <div style={{ marginBottom: '15px' }}>
          <label htmlFor="subject" style={{ display: 'block', marginBottom: '5px' }}>
            Subject:
          </label>
          <input
            type="text"
            id="subject"
            name="subject"
            value={formData.subject}
            onChange={handleChange}
            style={{
              width: '100%',
              padding: '10px',
              border: errors.subject ? '1px solid red' : '1px solid #ccc',
              borderRadius: '4px'
            }}
          />
          {errors.subject && <p style={{ color: 'red', fontSize: '12px' }}>{errors.subject}</p>}
        </div>

        <div style={{ marginBottom: '15px' }}>
          <label htmlFor="message" style={{ display: 'block', marginBottom: '5px' }}>
            Message:
          </label>
          <textarea
            id="message"
            name="message"
            value={formData.message}
            onChange={handleChange}
            rows="5"
            style={{
              width: '100%',
              padding: '10px',
              border: errors.message ? '1px solid red' : '1px solid #ccc',
              borderRadius: '4px',
              resize: 'vertical'
            }}
          />
          {errors.message && <p style={{ color: 'red', fontSize: '12px' }}>{errors.message}</p>}
        </div>

        <button
          type="submit"
          disabled={isSubmitting}
          style={{
            width: '100%',
            padding: '12px',
            backgroundColor: isSubmitting ? '#ccc' : '#007bff',
            color: 'white',
            border: 'none',
            borderRadius: '4px',
            cursor: isSubmitting ? 'not-allowed' : 'pointer',
            fontSize: '16px'
          }}
        >
          {isSubmitting ? 'Sending...' : 'Send Message'}
        </button>
      </form>
    </div>
  );
}

export default ContactForm;
```

## 注意事项

### 1. 组件命名规范

```jsx
// ✅ 正确：使用大驼峰命名法
function UserProfile() {}
const UserProfile = () => {};

// ❌ 错误：使用小驼峰或下划线
function userProfile() {}
const user_profile = () => {};
```

### 2. JSX 规则

```jsx
// ✅ 正确：总是返回单个根元素
function App() {
  return (
    <div>
      <h1>Title</h1>
      <p>Content</p>
    </div>
  );
}

// ✅ 正确：使用 Fragment 避免额外的 div
import { Fragment } from 'react';

function App() {
  return (
    <Fragment>
      <h1>Title</h1>
      <p>Content</p>
    </Fragment>
  );
}

// ✅ 正确：简写语法
function App() {
  return (
    <>
      <h1>Title</h1>
      <p>Content</p>
    </>
  );
}

// ❌ 错误：返回多个根元素
function App() {
  return (
    <h1>Title</h1>
    <p>Content</p>
  );
}
```

### 3. 事件处理注意事项

```jsx
// ✅ 正确：传递函数引用
<button onClick={handleClick}>Click</button>

// ❌ 错误：立即调用函数
<button onClick={handleClick()}>Click</button>

// ✅ 正确：需要传递参数时使用箭头函数
<button onClick={() => handleDelete(id)}>Delete</button>

// ❌ 错误：直接调用函数
<button onClick={handleDelete(id)}>Delete</button>

// ✅ 正确：使用 bind 传递参数（传统方式）
<button onClick={handleDelete.bind(null, id)}>Delete</button>
```

### 4. 状态更新注意事项

```jsx
// ✅ 正确：使用函数式更新（推荐）
setCount(prevCount => prevCount + 1);
setUser(prevUser => ({ ...prevUser, name: newName }));

// ❌ 错误：直接修改状态
count++;
user.name = newName;

// ✅ 正确：创建新的对象/数组
setTodos([...todos, newTodo]);
setUsers(users.map(user =>
  user.id === id ? { ...user, active: true } : user
));

// ❌ 错误：使用 push、splice 等方法修改原数组
todos.push(newTodo);
users.splice(index, 1);
```

### 5. Keys 的重要性

```jsx
// ✅ 正确：使用稳定的、唯一的 key
{items.map(item => (
  <Item key={item.id} item={item} />
))}

// ❌ 错误：使用索引作为 key（当列表可能重新排序时）
{items.map((item, index) => (
  <Item key={index} item={item} />
))}

// ✅ 正确：对于静态列表，索引可以作为 key
{['Apple', 'Banana', 'Orange'].map((fruit, index) => (
  <li key={index}>{fruit}</li>
))}
```

## 最佳实践

### 1. 组件拆分原则

```jsx
// ❌ 不好的做法：组件过于复杂
function App() {
  const [user, setUser] = useState(null);
  const [todos, setTodos] = useState([]);
  const [loading, setLoading] = useState(false);

  // 大量的逻辑代码...

  return (
    <div>
      {/* 复杂的 JSX 结构... */}
    </div>
  );
}

// ✅ 好的做法：拆分成多个小组件
function App() {
  return (
    <div>
      <Header />
      <UserSection />
      <TodoSection />
      <Footer />
    </div>
  );
}

// 每个组件职责单一
function UserSection() {
  const [user, setUser] = useState(null);
  // 用户相关逻辑
  return <UserProfile user={user} />;
}

function TodoSection() {
  const [todos, setTodos] = useState([]);
  // 待办事项相关逻辑
  return <TodoList todos={todos} />;
}
```

### 2. 使用自定义 Hooks 复用逻辑

```jsx
// 自定义 Hook：数据获取
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url);
        const data = await response.json();
        setData(data);
        setError(null);
      } catch (err) {
        setError(err.message);
        setData(null);
      } finally {
        setLoading(false);
      }
    }

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// 使用自定义 Hook
function UserList() {
  const { data: users, loading, error } = useFetch('/api/users');

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error}</div>;

  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

### 3. 性能优化

```jsx
import { memo, useMemo, useCallback } from 'react';

// 使用 memo 避免不必要的重新渲染
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  return <div>{/* 复杂的渲染逻辑 */}</div>;
});

// 使用 useMemo 缓存计算结果
function Calculator({ numbers }) {
  const sum = useMemo(() => {
    console.log('Calculating sum...');
    return numbers.reduce((a, b) => a + b, 0);
  }, [numbers]);

  return <div>Sum: {sum}</div>;
}

// 使用 useCallback 缓存回调函数
function ParentComponent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log('Button clicked');
  }, []);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>
        Count: {count}
      </button>
      <ChildComponent onClick={handleClick} />
    </div>
  );
}
```

## 总结

React 通过其声明式的编程模型和组件化的架构，为现代前端开发提供了强大的工具。掌握 React 的核心概念——组件、Props、State、事件处理等，是构建高效、可维护的 React 应用的基础。

### 关键要点

1. **组件化思维**：将 UI 拆分成独立、可复用的组件
2. **状态管理**：合理使用 Props 和 State 管理数据流
3. **事件处理**：理解合成事件系统，正确处理用户交互
4. **性能优化**：合理使用 memo、useMemo、useCallback 等优化工具
5. **最佳实践**：遵循代码规范，保持组件职责单一

### 学习路径建议

1. 掌握基础概念：组件、Props、State
2. 学习 Hooks：useState、useEffect 等常用 Hooks
3. 理解生命周期和副作用处理
4. 掌握路由和状态管理
5. 学习性能优化和测试

React 的学习曲线相对平缓，但要深入掌握需要持续的实践和学习。建议通过实际项目来巩固所学知识，逐步提高对 React 生态系统的理解。