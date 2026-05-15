---
title: CSS-in-JS 方案
published: 2024-07-01
description: 'Styled-components 使用的详细介绍和学习笔记'
image: ''
tags: ["CSS","样式"]
category: '前端样式'
draft: false
lang: 'zh-CN'
---

## 概述

CSS-in-JS 是一种将 CSS 样式写在 JavaScript 代码中的技术方案。它允许开发者使用 JavaScript 的全部能力来编写样式，实现了样式与组件的紧密耦合，解决了传统 CSS 的一些痛点。

## 核心概念

- **组件化样式**：每个组件拥有自己的样式，避免样式冲突
- **动态样式**：使用 JavaScript 变量和逻辑生成动态样式
- **样式隔离**：自动生成唯一的类名，避免全局污染
- **主题支持**：内置主题系统，轻松实现主题切换

## 基本用法

### Styled Components

```javascript
import styled from 'styled-components';

// 创建样式组件
const Button = styled.button`
  background-color: ${props => props.primary ? '#3498db' : '#fff'};
  color: ${props => props.primary ? '#fff' : '#333'};
  padding: 12px 24px;
  border: 2px solid #3498db;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;

  &:hover {
    background-color: ${props => props.primary ? '#2980b9' : '#f0f0f0'};
  }

  &:disabled {
    opacity: 0.5;
    cursor: not-allowed;
  }
`;

// 使用
function App() {
  return (
    <div>
      <Button>普通按钮</Button>
      <Button primary>主要按钮</Button>
    </div>
  );
}
```

### Emotion

```javascript
import { css } from '@emotion/react';

// 定义样式
const buttonStyle = css`
  background-color: #3498db;
  color: white;
  padding: 12px 24px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background-color: #2980b9;
  }
`;

// 使用
function App() {
  return (
    <button css={buttonStyle}>按钮</button>
  );
}
```

### JSS

```javascript
import { createUseStyles } from 'react-jss';

// 定义样式
const useStyles = createUseStyles({
  button: {
    backgroundColor: '#3498db',
    color: 'white',
    padding: '12px 24px',
    border: 'none',
    borderRadius: '4px',
    cursor: 'pointer',
    '&:hover': {
      backgroundColor: '#2980b9',
    },
  },
});

// 使用
function App() {
  const classes = useStyles();
  return <button className={classes.button}>按钮</button>;
}
```

## 实际应用

### 主题系统

```javascript
import styled, { ThemeProvider } from 'styled-components';

// 定义主题
const lightTheme = {
  colors: {
    primary: '#3498db',
    secondary: '#2ecc71',
    background: '#ffffff',
    text: '#333333',
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px',
  },
};

const darkTheme = {
  colors: {
    primary: '#5dade2',
    secondary: '#27ae60',
    background: '#1a1a1a',
    text: '#ffffff',
  },
  spacing: {
    small: '8px',
    medium: '16px',
    large: '24px',
  },
};

// 使用主题
const Card = styled.div`
  background-color: ${props => props.theme.colors.background};
  color: ${props => props.theme.colors.text};
  padding: ${props => props.theme.spacing.large};
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
`;

// 应用
function App() {
  const [theme, setTheme] = useState(lightTheme);

  return (
    <ThemeProvider theme={theme}>
      <Card>
        <h1>主题切换</h1>
        <button onClick={() => setTheme(darkTheme)}>切换到深色主题</button>
        <button onClick={() => setTheme(lightTheme)}>切换到浅色主题</button>
      </Card>
    </ThemeProvider>
  );
}
```

### 响应式组件

```javascript
import styled, { css } from 'styled-components';

// 断点工具
const breakpoints = {
  mobile: '480px',
  tablet: '768px',
  desktop: '1024px',
};

// 媒体查询帮助函数
const media = Object.keys(breakpoints).reduce((acc, label) => {
  acc[label] = (...args) => css`
    @media (min-width: ${breakpoints[label]}) {
      ${css(...args)}
    }
  `;
  return acc;
}, {});

// 响应式组件
const Container = styled.div`
  width: 100%;
  padding: 16px;

  ${media.tablet`
    padding: 24px;
  `}

  ${media.desktop`
    max-width: 1200px;
    margin: 0 auto;
    padding: 32px;
  `}
`;

function App() {
  return <Container>响应式容器</Container>;
}
```

### 复杂动画

```javascript
import styled, { keyframes } from 'styled-components';

// 定义动画
const fadeIn = keyframes`
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
`;

// 带动画的组件
const Modal = styled.div`
  animation: ${fadeIn} 0.3s ease-out;
  background: white;
  padding: 32px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
`;
```

### 动态样式

```javascript
import styled from 'styled-components';

const ProgressBar = styled.div`
  width: ${props => props.width}%;
  height: 8px;
  background: linear-gradient(
    to right,
    #3498db,
    ${props => props.progress > 80 ? '#2ecc71' : '#3498db'}
  );
  border-radius: 4px;
  transition: width 0.3s ease;
`;

function App() {
  const [progress, setProgress] = useState(0);

  return (
    <div>
      <ProgressBar width={progress} progress={progress} />
      <button onClick={() => setProgress(prev => Math.min(prev + 10, 100))}>
        增加
      </button>
    </div>
  );
}
```

## 注意事项

1. **性能考虑**：
   - 使用 `memo` 避免不必要的重渲染
   - 避免在渲染函数中创建样式
   - 使用 `createGlobalStyle` 设置全局样式

2. **样式组织**：
   - 将样式组件和逻辑组件分开
   - 使用命名约定提高可读性
   - 避免过度嵌套

3. **类型安全**：
   ```typescript
   import styled from 'styled-components';

   interface ButtonProps {
     primary?: boolean;
     disabled?: boolean;
   }

   const Button = styled.button<ButtonProps>`
     background: ${props => props.primary ? '#3498db' : '#fff'};
   `;
   ```

4. **最佳实践**：
   - 优先使用 props 而不是继承样式
   - 复用样式使用 `css` 辅助函数
   - 合理使用主题系统

5. **常见问题**：
   - 服务端渲染需要特殊处理
   - 动态类名生成可能影响性能
   - 与现有 CSS 框架的集成

## 总结

CSS-in-JS 提供了强大的样式管理能力，特别适合组件化开发。通过合理使用 CSS-in-JS，可以实现样式隔离、主题切换、动态样式等功能，提高开发效率和代码可维护性。但也要注意性能影响，合理选择使用场景。