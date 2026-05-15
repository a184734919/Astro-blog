---
title: React Props
published: 2023-07-09
description: '组件属性传递的详细介绍和学习笔记'
image: ''
tags: ["React","Props"]
category: 'React'
draft: false
lang: 'zh-CN'
---

# React Props

## 概述

Props（Properties）是 React 组件间传递数据的主要方式，它实现了组件之间的数据通信。Props 是只读的，从父组件传递到子组件，形成单向数据流。理解 Props 的使用对于构建可复用、可维护的 React 组件至关重要。

### Props 的核心特性

**单向数据流**：数据从父组件流向子组件，子组件不能直接修改 props

**只读性**：Props 是只读的，子组件不能修改接收到的 props

**类型检查**：可以使用 PropTypes 进行类型检查，提高代码质量

**默认值**：可以为 props 设置默认值，提高组件的健壮性

**解构使用**：可以使用解构赋值简化 props 的使用

## 核心概念

### 1. Props 的基本使用

```jsx
// 父组件传递 props
function ParentComponent() {
  const user = {
    name: 'John Doe',
    age: 30,
    email: 'john@example.com'
  };

  return (
    <div>
      <ChildComponent
        name={user.name}
        age={user.age}
        email={user.email}
        isActive={true}
      />
    </div>
  );
}

// 子组件接收 props
function ChildComponent(props) {
  return (
    <div>
      <h2>User Profile</h2>
      <p>Name: {props.name}</p>
      <p>Age: {props.age}</p>
      <p>Email: {props.email}</p>
      <p>Status: {props.isActive ? 'Active' : 'Inactive'}</p>
    </div>
  );
}

// 使用解构简化 props
function ChildComponent({ name, age, email, isActive }) {
  return (
    <div>
      <h2>User Profile</h2>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
      <p>Status: {isActive ? 'Active' : 'Inactive'}</p>
    </div>
  );
}
```

### 2. Props 的类型

```jsx
function PropsExample() {
  return (
    <div>
      {/* 字符串 */}
      <Component name="John" />

      {/* 数字 */}
      <Component age={30} />

      {/* 布尔值 */}
      <Component isActive={true} />
      <Component isActive /> {/* 等同于 isActive={true} */}
      <Component isActive={false} />

      {/* 数组 */}
      <Component items={['Apple', 'Banana', 'Orange']} />

      {/* 对象 */}
      <Component user={{ name: 'John', age: 30 }} />

      {/* 函数 */}
      <Component onClick={handleClick} />

      {/* JSX 元素 */}
      <Component header={<h1>Title</h1>} />

      {/* null 和 undefined */}
      <Component data={null} />
      <Component data={undefined} />

      {/* 混合类型 */}
      <Component
        name="John"
        age={30}
        isActive={true}
        items={['item1', 'item2']}
        onClick={handleClick}
      />
    </div>
  );
}
```

### 3. Props 传递方式

```jsx
function PropsPassing() {
  // 1. 逐个传递 props
  return (
    <ChildComponent
      name="John"
      age={30}
      email="john@example.com"
    />
  );

  // 2. 使用展开运算符传递对象
  const user = {
    name: 'John',
    age: 30,
    email: 'john@example.com'
  };

  return <ChildComponent {...user} />;

  // 3. 部分解构 + 展开运算符
  const { name, ...rest } = user;
  return (
    <ChildComponent
      name={name}
      {...rest}
    />
  );

  // 4. 动态计算 props
  const isActive = user.age >= 18;
  return (
    <ChildComponent
      {...user}
      isActive={isActive}
      isAdult={user.age >= 18}
    />
  );
}
```

### 4. Props 验证

```jsx
import PropTypes from 'prop-types';

function ValidatedComponent({
  name,
  age,
  email,
  isActive,
  items,
  user,
  handleClick
}) {
  return (
    <div>
      {/* 组件内容 */}
    </div>
  );
}

// PropTypes 定义
ValidatedComponent.propTypes = {
  // 基本类型
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string,
  isActive: PropTypes.bool,

  // 数组类型
  items: PropTypes.arrayOf(PropTypes.string),

  // 对象类型
  user: PropTypes.shape({
    name: PropTypes.string.isRequired,
    age: PropTypes.number,
    email: PropTypes.string
  }),

  // 函数类型
  handleClick: PropTypes.func,

  // 元素类型
  header: PropTypes.element,

  // 节点类型（可以是数字、字符串、元素等）
  children: PropTypes.node,

  // 实例类型
  component: PropTypes.instanceOf(MyComponent),

  // 枚举类型
  status: PropTypes.oneOf(['active', 'inactive', 'pending']),

  // 联合类型
  value: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number
  ]),

  // 对象数组类型
  users: PropTypes.arrayOf(
    PropTypes.shape({
      id: PropTypes.number.isRequired,
      name: PropTypes.string.isRequired
    })
  ),

  // 自定义验证函数
  customProp: function(props, propName, componentName) {
    if (!/test/.test(props[propName])) {
      return new Error(
        `Invalid prop \`${propName}\` supplied to` +
        ` \`${componentName}\`. Validation failed.`
      );
    }
  }
};

// 默认值
ValidatedComponent.defaultProps = {
  age: 0,
  email: '',
  isActive: false,
  items: [],
  user: {},
  handleClick: () => {}
};
```

## 基本用法

### 1. 组件组合和 Children

```jsx
// 1. children prop 的使用
function Card({ children, title }) {
  return (
    <div className="card">
      {title && <h2 className="card-title">{title}</h2>}
      <div className="card-content">
        {children}
      </div>
    </div>
  );
}

// 使用示例
function App() {
  return (
    <div>
      <Card title="Welcome">
        <p>This is the card content.</p>
        <button>Action</button>
      </Card>

      <Card>
        <p>Card without title</p>
      </Card>
    </div>
  );
}

// 2. 命名插槽
function Layout({ header, sidebar, content, footer }) {
  return (
    <div className="layout">
      {header && <header className="layout-header">{header}</header>}
      <div className="layout-body">
        {sidebar && <aside className="layout-sidebar">{sidebar}</aside>}
        {content && <main className="layout-content">{content}</main>}
      </div>
      {footer && <footer className="layout-footer">{footer}</footer>}
    </div>
  );
}

// 使用示例
function App() {
  return (
    <Layout
      header={<Header />}
      sidebar={<Sidebar />}
      content={<MainContent />}
      footer={<Footer />}
    />
  );
}

// 3. 函数作为 children
function List({ items, renderItem }) {
  return (
    <ul className="list">
      {items.map((item, index) => (
        <li key={index}>
          {renderItem ? renderItem(item, index) : item}
        </li>
      ))}
    </ul>
  );
}

// 使用示例
function App() {
  const users = [
    { id: 1, name: 'John', email: 'john@example.com' },
    { id: 2, name: 'Jane', email: 'jane@example.com' }
  ];

  return (
    <List
      items={users}
      renderItem={(user, index) => (
        <div>
          <strong>{user.name}</strong>
          <p>{user.email}</p>
        </div>
      )}
    />
  );
}
```

### 2. Props 向下传递（组件嵌套）

```jsx
// 多层组件嵌套传递 props
function GrandParent() {
  const theme = {
    colors: {
      primary: '#007bff',
      secondary: '#6c757d'
    },
    fonts: {
      primary: 'Arial, sans-serif'
    }
  };

  const user = {
    name: 'John',
    preferences: {
      language: 'en',
      theme: 'dark'
    }
  };

  return (
    <Parent theme={theme} user={user} />
  );
}

function Parent({ theme, user }) {
  return (
    <div style={{ fontFamily: theme.fonts.primary }}>
      <Child theme={theme} user={user} />
    </div>
  );
}

function Child({ theme, user }) {
  return (
    <div style={{ color: theme.colors.primary }}>
      <GrandChild
        theme={theme}
        user={user}
        userName={user.name}
        userLanguage={user.preferences.language}
      />
    </div>
  );
}

function GrandChild({ theme, userName, userLanguage }) {
  return (
    <div>
      <h2 style={{ color: theme.colors.secondary }}>
        Hello, {userName}!
      </h2>
      <p>Language: {userLanguage}</p>
    </div>
  );
}
```

### 3. 条件传递 Props

```jsx
function ConditionalProps({ user, showMessage }) {
  const isPremium = user && user.subscription === 'premium';

  return (
    <div>
      {/* 条件传递不同的组件 */}
      {isPremium ? (
        <PremiumComponent user={user} />
      ) : (
        <StandardComponent user={user} />
      )}

      {/* 条件传递 props */}
      <Button
        onClick={handleClick}
        {...(showMessage && { message: 'Click me!' })}
        {...(isPremium && { variant: 'primary' })}
      />

      {/* 基于条件传递不同的 props 值 */}
      <Card
        title={isPremium ? 'Premium Card' : 'Standard Card'}
        className={isPremium ? 'premium-card' : 'standard-card'}
        {...(isPremium && {
          features: ['Feature 1', 'Feature 2', 'Feature 3']
        })}
      />

      {/* 复杂条件逻辑 */}
      <UserProfile
        user={user}
        {...(() => {
          const props = {};
          if (user) {
            props.name = user.name;
            props.email = user.email;
            if (isPremium) {
              props.badge = 'Premium';
              props.features = true;
            }
          }
          return props;
        })()}
      />
    </div>
  );
}
```

### 4. Props 映射和转换

```jsx
// 1. Props 映射组件
function PropsMapper({ sourceProps, mapping }) {
  const mappedProps = {};

  Object.keys(mapping).forEach(targetKey => {
    const sourceKey = mapping[targetKey];
    if (sourceKey in sourceProps) {
      mappedProps[targetKey] = sourceProps[sourceKey];
    }
  });

  return <TargetComponent {...mappedProps} />;
}

// 使用示例
function App() {
  const userData = {
    fullName: 'John Doe',
    userEmail: 'john@example.com',
    userAge: 30
  };

  const fieldMapping = {
    name: 'fullName',
    email: 'userEmail',
    age: 'userAge'
  };

  return (
    <PropsMapper
      sourceProps={userData}
      mapping={fieldMapping}
    />
  );
}

// 2. Props 转换组件
function PropsTransformer({ data, transform }) {
  const transformedProps = transform ? transform(data) : data;
  return <Component {...transformedProps} />;
}

// 使用示例
function App() {
  const rawData = {
    first_name: 'John',
    last_name: 'Doe',
    age: '30'
  };

  const transformData = (data) => ({
    name: `${data.first_name} ${data.last_name}`,
    age: parseInt(data.age, 10)
  });

  return (
    <PropsTransformer
      data={rawData}
      transform={transformData}
    />
  );
}

// 3. Props 过滤组件
function PropsFilter({ children, allowedProps, ...props }) {
  const filteredProps = {};

  Object.keys(props).forEach(key => {
    if (allowedProps.includes(key)) {
      filteredProps[key] = props[key];
    }
  });

  return <div {...filteredProps}>{children}</div>;
}

// 使用示例
function App() {
  return (
    <PropsFilter
      allowedProps={['className', 'id', 'style']}
      className="container"
      id="main"
      data-value="123"
      onClick={handleClick}
    >
      Content
    </PropsFilter>
  );
}
```

## 实际应用

### 1. 通用按钮组件

```jsx
import PropTypes from 'prop-types';

function Button({
  children,
  type = 'button',
  variant = 'primary',
  size = 'medium',
  disabled = false,
  loading = false,
  block = false,
  icon,
  iconPosition = 'left',
  onClick,
  className = '',
  ...props
}) {
  const handleClick = (event) => {
    if (!disabled && !loading && onClick) {
      onClick(event);
    }
  };

  const buttonClasses = [
    'btn',
    `btn-${variant}`,
    `btn-${size}`,
    block ? 'btn-block' : '',
    disabled ? 'btn-disabled' : '',
    loading ? 'btn-loading' : '',
    className
  ].filter(Boolean).join(' ');

  return (
    <button
      type={type}
      className={buttonClasses}
      disabled={disabled || loading}
      onClick={handleClick}
      {...props}
    >
      {loading && <span className="btn-spinner">Loading...</span>}
      {icon && iconPosition === 'left' && (
        <span className="btn-icon btn-icon-left">{icon}</span>
      )}
      <span className="btn-text">{children}</span>
      {icon && iconPosition === 'right' && (
        <span className="btn-icon btn-icon-right">{icon}</span>
      )}
    </button>
  );
}

Button.propTypes = {
  children: PropTypes.node.isRequired,
  type: PropTypes.oneOf(['button', 'submit', 'reset']),
  variant: PropTypes.oneOf(['primary', 'secondary', 'success', 'danger', 'warning', 'info', 'light', 'dark']),
  size: PropTypes.oneOf(['small', 'medium', 'large']),
  disabled: PropTypes.bool,
  loading: PropTypes.bool,
  block: PropTypes.bool,
  icon: PropTypes.node,
  iconPosition: PropTypes.oneOf(['left', 'right']),
  onClick: PropTypes.func,
  className: PropTypes.string
};

// 使用示例
function ButtonExamples() {
  return (
    <div className="button-examples">
      <h2>Button Examples</h2>

      <div className="example-group">
        <h3>Basic Buttons</h3>
        <Button>Primary Button</Button>
        <Button variant="secondary">Secondary Button</Button>
        <Button variant="danger">Danger Button</Button>
      </div>

      <div className="example-group">
        <h3>Sizes</h3>
        <Button size="small">Small Button</Button>
        <Button size="medium">Medium Button</Button>
        <Button size="large">Large Button</Button>
      </div>

      <div className="example-group">
        <h3>States</h3>
        <Button disabled>Disabled Button</Button>
        <Button loading>Loading Button</Button>
      </div>

      <div className="example-group">
        <h3>With Icons</h3>
        <Button icon={<span>🔍</span>}>
          Search
        </Button>
        <Button
          icon={<span>📤</span>}
          iconPosition="right"
        >
          Export
        </Button>
      </div>

      <div className="example-group">
        <h3>Block Button</h3>
        <Button block>Full Width Button</Button>
      </div>

      <div className="example-group">
        <h3>Interactive</h3>
        <Button onClick={() => console.log('Clicked!')}>
          Click Me
        </Button>
        <Button
          onClick={handleSubmit}
          loading={isLoading}
        >
          Submit
        </Button>
      </div>
    </div>
  );
}
```

### 2. 表单输入组件

```jsx
import PropTypes from 'prop-types';

function Input({
  label,
  name,
  type = 'text',
  value,
  onChange,
  placeholder = '',
  disabled = false,
  required = false,
  error = '',
  helpText = '',
  maxLength,
  minLength,
  pattern,
  autoComplete,
  autoFocus = false,
  className = '',
  ...props
}) {
  const inputId = `input-${name}`;
  const errorId = error ? `error-${name}` : undefined;
  const helpId = helpText ? `help-${name}` : undefined;

  const handleChange = (event) => {
    if (onChange && !disabled) {
      onChange(event.target.value, event);
    }
  };

  const inputClasses = [
    'input-field',
    error ? 'input-field-error' : '',
    disabled ? 'input-field-disabled' : '',
    className
  ].filter(Boolean).join(' ');

  return (
    <div className="input-group">
      {label && (
        <label
          htmlFor={inputId}
          className="input-label"
        >
          {label}
          {required && <span className="input-required">*</span>}
        </label>
      )}

      <input
        id={inputId}
        name={name}
        type={type}
        value={value}
        onChange={handleChange}
        placeholder={placeholder}
        disabled={disabled}
        required={required}
        maxLength={maxLength}
        minLength={minLength}
        pattern={pattern}
        autoComplete={autoComplete}
        autoFocus={autoFocus}
        className={inputClasses}
        aria-invalid={!!error}
        aria-describedby={[errorId, helpId].filter(Boolean).join(' ')}
        {...props}
      />

      {error && (
        <div id={errorId} className="input-error">
          {error}
        </div>
      )}

      {helpText && !error && (
        <div id={helpId} className="input-help">
          {helpText}
        </div>
      )}
    </div>
  );
}

Input.propTypes = {
  label: PropTypes.string,
  name: PropTypes.string.isRequired,
  type: PropTypes.oneOf(['text', 'password', 'email', 'number', 'tel', 'url', 'search']),
  value: PropTypes.oneOfType([PropTypes.string, PropTypes.number]),
  onChange: PropTypes.func,
  placeholder: PropTypes.string,
  disabled: PropTypes.bool,
  required: PropTypes.bool,
  error: PropTypes.string,
  helpText: PropTypes.string,
  maxLength: PropTypes.number,
  minLength: PropTypes.number,
  pattern: PropTypes.string,
  autoComplete: PropTypes.string,
  autoFocus: PropTypes.bool,
  className: PropTypes.string
};

// 使用示例
function FormExample() {
  const [formData, setFormData] = React.useState({
    username: '',
    email: '',
    password: '',
    age: ''
  });
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});

  const handleChange = (name) => (value) => {
    setFormData(prev => ({ ...prev, [name]: value }));
    setTouched(prev => ({ ...prev, [name]: true }));

    // 清除错误
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const validateForm = () => {
    const newErrors = {};

    if (!formData.username.trim()) {
      newErrors.username = 'Username is required';
    } else if (formData.username.length < 3) {
      newErrors.username = 'Username must be at least 3 characters';
    }

    if (!formData.email.trim()) {
      newErrors.email = 'Email is required';
    } else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Invalid email format';
    }

    if (!formData.password) {
      newErrors.password = 'Password is required';
    } else if (formData.password.length < 8) {
      newErrors.password = 'Password must be at least 8 characters';
    }

    if (formData.age && (formData.age < 18 || formData.age > 120)) {
      newErrors.age = 'Age must be between 18 and 120';
    }

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  const handleSubmit = (event) => {
    event.preventDefault();

    if (validateForm()) {
      console.log('Form submitted:', formData);
      // 提交逻辑
    }
  };

  return (
    <form onSubmit={handleSubmit} className="form">
      <h2>Registration Form</h2>

      <Input
        label="Username"
        name="username"
        type="text"
        value={formData.username}
        onChange={handleChange('username')}
        placeholder="Enter your username"
        required
        error={touched.username && errors.username}
        helpText="Choose a unique username (min 3 characters)"
        minLength={3}
        maxLength={20}
        autoComplete="username"
        autoFocus
      />

      <Input
        label="Email"
        name="email"
        type="email"
        value={formData.email}
        onChange={handleChange('email')}
        placeholder="Enter your email"
        required
        error={touched.email && errors.email}
        helpText="We'll never share your email with anyone else"
        autoComplete="email"
      />

      <Input
        label="Password"
        name="password"
        type="password"
        value={formData.password}
        onChange={handleChange('password')}
        placeholder="Enter your password"
        required
        error={touched.password && errors.password}
        helpText="Use a strong password (min 8 characters)"
        minLength={8}
        autoComplete="new-password"
      />

      <Input
        label="Age"
        name="age"
        type="number"
        value={formData.age}
        onChange={handleChange('age')}
        placeholder="Enter your age"
        error={touched.age && errors.age}
        helpText="Must be between 18 and 120"
        min="18"
        max="120"
      />

      <button type="submit" className="btn btn-primary">
        Register
      </button>
    </form>
  );
}
```

### 3. 模态框组件

```jsx
import PropTypes from 'prop-types';
import { createPortal } from 'react-dom';

function Modal({
  isOpen = false,
  onClose,
  title,
  children,
  size = 'medium',
  closeOnOverlay = true,
  closeOnEsc = true,
  showCloseButton = true,
  footer,
  className = '',
  ...props
}) {
  const modalRef = React.useRef(null);
  const previousActiveElement = React.useRef(null);

  // 处理 ESC 键关闭
  React.useEffect(() => {
    if (!isOpen || !closeOnEsc) return;

    const handleEsc = (event) => {
      if (event.key === 'Escape') {
        onClose();
      }
    };

    document.addEventListener('keydown', handleEsc);
    return () => document.removeEventListener('keydown', handleEsc);
  }, [isOpen, closeOnEsc, onClose]);

  // 处理焦点陷阱
  React.useEffect(() => {
    if (!isOpen) return;

    // 保存当前聚焦元素
    previousActiveElement.current = document.activeElement;

    // 聚焦到模态框
    if (modalRef.current) {
      modalRef.current.focus();
    }

    // 禁用背景滚动
    document.body.style.overflow = 'hidden';

    return () => {
      // 恢复焦点
      if (previousActiveElement.current) {
        previousActiveElement.current.focus();
      }

      // 恢复滚动
      document.body.style.overflow = '';
    };
  }, [isOpen]);

  const handleOverlayClick = (event) => {
    if (closeOnOverlay && event.target === event.currentTarget) {
      onClose();
    }
  };

  const modalClasses = [
    'modal',
    `modal-${size}`,
    className
  ].filter(Boolean).join(' ');

  if (!isOpen) return null;

  return createPortal(
    <div
      className="modal-overlay"
      onClick={handleOverlayClick}
      role="dialog"
      aria-modal="true"
      aria-labelledby={title ? 'modal-title' : undefined}
    >
      <div
        ref={modalRef}
        className={modalClasses}
        tabIndex={-1}
        {...props}
      >
        {(title || showCloseButton) && (
          <div className="modal-header">
            {title && (
              <h2 id="modal-title" className="modal-title">
                {title}
              </h2>
            )}
            {showCloseButton && (
              <button
                className="modal-close"
                onClick={onClose}
                aria-label="Close modal"
              >
                ×
              </button>
            )}
          </div>
        )}

        <div className="modal-body">
          {children}
        </div>

        {footer && (
          <div className="modal-footer">
            {footer}
          </div>
        )}
      </div>
    </div>,
    document.body
  );
}

Modal.propTypes = {
  isOpen: PropTypes.bool,
  onClose: PropTypes.func.isRequired,
  title: PropTypes.string,
  children: PropTypes.node,
  size: PropTypes.oneOf(['small', 'medium', 'large', 'fullscreen']),
  closeOnOverlay: PropTypes.bool,
  closeOnEsc: PropTypes.bool,
  showCloseButton: PropTypes.bool,
  footer: PropTypes.node,
  className: PropTypes.string
};

// 使用示例
function ModalExample() {
  const [isModalOpen, setIsModalOpen] = React.useState(false);
  const [modalSize, setModalSize] = React.useState('medium');

  const openModal = (size = 'medium') => {
    setModalSize(size);
    setIsModalOpen(true);
  };

  const closeModal = () => {
    setIsModalOpen(false);
  };

  return (
    <div className="modal-example">
      <h2>Modal Examples</h2>

      <div className="button-group">
        <button onClick={() => openModal('small')}>
          Small Modal
        </button>
        <button onClick={() => openModal('medium')}>
          Medium Modal
        </button>
        <button onClick={() => openModal('large')}>
          Large Modal
        </button>
        <button onClick={() => openModal('fullscreen')}>
          Fullscreen Modal
        </button>
      </div>

      <Modal
        isOpen={isModalOpen}
        onClose={closeModal}
        size={modalSize}
        title="Example Modal"
        footer={
          <>
            <button onClick={closeModal} className="btn btn-secondary">
              Cancel
            </button>
            <button onClick={closeModal} className="btn btn-primary">
              Confirm
            </button>
          </>
        }
      >
        <p>This is a modal dialog.</p>
        <p>You can put any content here, including forms, images, or other components.</p>

        <h3>Features:</h3>
        <ul>
          <li>Close by clicking the overlay</li>
          <li>Close by pressing ESC key</li>
          <li>Focus trapping</li>
          <li>Background scroll prevention</li>
          <li>Multiple size options</li>
        </ul>
      </Modal>

      {/* 无标题的确认对话框 */}
      <Modal
        isOpen={false}
        onClose={() => {}}
        showCloseButton={false}
        footer={
          <>
            <button className="btn btn-secondary">Cancel</button>
            <button className="btn btn-danger">Delete</button>
          </>
        }
      >
        <p>Are you sure you want to delete this item? This action cannot be undone.</p>
      </Modal>
    </div>
  );
}
```

## 注意事项

### 1. Props 不可变性

```jsx
// ❌ 错误：直接修改 props
function BadComponent(props) {
  props.user.name = 'Modified';
  props.items.push('new item');
  return <div>{props.user.name}</div>;
}

// ✅ 正确：使用本地状态或创建副本
function GoodComponent({ user, items }) {
  const [localUser, setLocalUser] = React.useState(user);

  const handleUpdateUser = () => {
    setLocalUser({ ...localUser, name: 'Modified' });
  };

  return (
    <div>
      <p>{localUser.name}</p>
      <button onClick={handleUpdateUser}>Update</button>
    </div>
  );
}

// ✅ 正确：需要修改数组时创建新数组
function GoodList({ items }) {
  const handleAddItem = (newItem) => {
    const updatedItems = [...items, newItem];
    // 使用更新后的数组
  };

  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{item}</li>
      ))}
    </ul>
  );
}
```

### 2. Props 验证的重要性

```jsx
// ❌ 不好的做法：没有类型检查
function UnvalidatedComponent({ name, age, email }) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

// ✅ 好的做法：添加 PropTypes 验证
function ValidatedComponent({ name, age, email }) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}

ValidatedComponent.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number,
  email: PropTypes.string
};

// ✅ 更好的做法：使用 TypeScript
interface ValidatedComponentProps {
  name: string;
  age?: number;
  email?: string;
}

function ValidatedComponent({ name, age, email }: ValidatedComponentProps) {
  return (
    <div>
      <p>Name: {name}</p>
      <p>Age: {age}</p>
      <p>Email: {email}</p>
    </div>
  );
}
```

### 3. 避免 Props 污染

```jsx
// ❌ 不好的做法：传递所有 props
function BadComponent(props) {
  return <ChildComponent {...props} />;
}

// ✅ 好的做法：明确传递需要的 props
function GoodComponent({ title, content, onClick }) {
  return (
    <ChildComponent
      title={title}
      content={content}
      onClick={onClick}
    />
  );
}

// ✅ 好的做法：过滤 props
function FilteredComponent(props) {
  const { className, ...rest } = props;
  return (
    <div className={className}>
      <ChildComponent {...rest} />
    </div>
  );
}
```

### 4. Props 命名规范

```jsx
// ✅ 正确：使用描述性的 prop 名称
<UserCard
  userName="John"
  userEmail="john@example.com"
  userAge={30}
  isActive={true}
/>

// ❌ 错误：使用不清晰的名称
<UserCard
  n="John"
  e="john@example.com"
  a={30}
  act={true}
/>

// ✅ 正确：使用布尔值 prop 名称
<Button disabled />
<Input required />
<Modal isOpen />

// ❌ 错误：使用动词形式的布尔值
<Button isDisabled />
<Input isRequired />
<Modal shouldOpen />
```

## 最佳实践

### 1. 组件设计原则

```jsx
// ✅ 好的做法：单一职责原则
function UserAvatar({ src, alt, size }) {
  return (
    <img
      src={src}
      alt={alt}
      className={`avatar avatar-${size}`}
    />
  );
}

function UserInfo({ name, email, bio }) {
  return (
    <div className="user-info">
      <h3>{name}</h3>
      <p>{email}</p>
      <p>{bio}</p>
    </div>
  );
}

function UserCard({ user }) {
  return (
    <div className="user-card">
      <UserAvatar src={user.avatar} alt={user.name} size="medium" />
      <UserInfo
        name={user.name}
        email={user.email}
        bio={user.bio}
      />
    </div>
  );
}

// ❌ 不好的做法：组件职责过多
function BadUserCard({ user, onClick, onEdit, onDelete, ...props }) {
  // 包含了头像、信息、按钮等多种功能
  return (
    <div className="user-card" {...props}>
      <img src={user.avatar} alt={user.name} />
      <div>
        <h3>{user.name}</h3>
        <p>{user.email}</p>
      </div>
      <div className="actions">
        <button onClick={onClick}>View</button>
        <button onClick={onEdit}>Edit</button>
        <button onClick={onDelete}>Delete</button>
      </div>
    </div>
  );
}
```

### 2. Props 组合和模式

```jsx
// 1. 容器/展示组件模式
function UserListContainer() {
  const [users, setUsers] = React.useState([]);
  const [loading, setLoading] = React.useState(true);

  React.useEffect(() => {
    fetchUsers().then(data => {
      setUsers(data);
      setLoading(false);
    });
  }, []);

  if (loading) return <LoadingSpinner />;
  if (users.length === 0) return <EmptyState />;

  return <UserList users={users} />;
}

function UserList({ users }) {
  return (
    <div className="user-list">
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// 2. 受控/非受控组件模式
function ControlledInput({ value, onChange, ...props }) {
  return (
    <input
      value={value}
      onChange={e => onChange(e.target.value)}
      {...props}
    />
  );
}

function UncontrolledInput({ defaultValue, ...props }) {
  const [value, setValue] = React.useState(defaultValue);

  return (
    <input
      value={value}
      onChange={e => setValue(e.target.value)}
      {...props}
    />
  );
}

// 3. 组合组件模式
function Tabs({ children, defaultIndex = 0 }) {
  const [activeIndex, setActiveIndex] = React.useState(defaultIndex);

  return (
    <div className="tabs">
      {React.Children.map(children, (child, index) => {
        if (React.isValidElement(child)) {
          return React.cloneElement(child, {
            isActive: index === activeIndex,
            onClick: () => setActiveIndex(index)
          });
        }
        return child;
      })}
    </div>
  );
}

function Tab({ children, isActive, onClick }) {
  return (
    <button
      className={`tab ${isActive ? 'tab-active' : ''}`}
      onClick={onClick}
    >
      {children}
    </button>
  );
}

// 使用示例
function App() {
  return (
    <Tabs>
      <Tab>Tab 1</Tab>
      <Tab>Tab 2</Tab>
      <Tab>Tab 3</Tab>
    </Tabs>
  );
}
```

### 3. 性能优化

```jsx
import { memo, useCallback, useMemo } from 'react';

// 1. 使用 memo 避免不必要的重新渲染
const ExpensiveComponent = memo(function ExpensiveComponent({ data }) {
  return (
    <div>
      {/* 复杂的渲染逻辑 */}
    </div>
  );
});

// 2. 使用 useCallback 缓存回调函数
function ParentComponent({ items }) {
  const handleClick = useCallback((id) => {
    console.log('Item clicked:', id);
  }, []);

  return (
    <div>
      {items.map(item => (
        <ChildComponent
          key={item.id}
          item={item}
          onClick={handleClick}
        />
      ))}
    </div>
  );
}

// 3. 使用 useMemo 缓存计算结果
function ProcessedList({ items, filter }) {
  const processedItems = useMemo(() => {
    console.log('Processing items...');
    return items
      .filter(item => item.category === filter)
      .map(item => ({
        ...item,
        processed: true
      }));
  }, [items, filter]);

  return (
    <ul>
      {processedItems.map(item => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  );
}

// 4. Props 拆分优化
function OptimizedComponent({ data, uiProps, eventProps }) {
  return (
    <div className={uiProps.className}>
      <div style={uiProps.style}>
        {/* 渲染数据 */}
      </div>
      <button onClick={eventProps.onClick}>
        {uiProps.buttonText}
      </button>
    </div>
  );
}
```

## 总结

Props 是 React 组件通信的核心机制，正确理解和使用 Props 对于构建高质量的 React 应用至关重要。

### 关键要点

1. **单向数据流**：Props 从父组件流向子组件，子组件不能修改
2. **类型安全**：使用 PropTypes 或 TypeScript 进行类型检查
3. **组件设计**：遵循单一职责原则，设计可复用的组件
4. **性能优化**：合理使用 memo、useCallback、useMemo 等优化技术
5. **最佳实践**：遵循命名规范，避免 props 污染，确保组件的可维护性

### 学习建议

1. **实践为主**：通过实际项目练习 Props 的各种使用场景
2. **类型检查**：养成使用 PropTypes 或 TypeScript 的习惯
3. **组件拆分**：学习如何设计合理组件的 props 接口
4. **性能意识**：理解 props 变化对组件重新渲染的影响
5. **模式学习**：掌握常见的组件设计模式和 props 使用模式

Props 的使用看似简单，但要真正掌握需要深入理解 React 的渲染机制和组件设计原则。通过持续的实践和学习，你将能够设计出优雅、高效、易维护的 React 组件。