---
title: React useReducer
published: 2023-08-06
description: '深入理解 React useReducer Hook：复杂状态管理的最佳实践'
image: ''
tags: ["React","Hooks"]
category: 'React'
draft: false
lang: 'zh-CN'
---

## 概述

useReducer 是 React 中用于管理复杂状态的 Hook。它是 useState 的替代方案，特别适合处理具有多个子值的复杂状态逻辑，或者下一个状态依赖于前一个状态的情况。useReducer 的灵感来源于 Redux，它让你能够用 action 来描述状态的变化，使状态逻辑更加可预测和可维护。

### 为什么需要 useReducer

当状态逻辑变得复杂时，useState 可能会导致以下问题：

1. **状态更新逻辑分散**：多个相关的状态更新分散在不同的函数中
2. **难以追踪状态变化**：状态变化的因果关系不明显
3. **测试困难**：状态逻辑与组件逻辑耦合，难以单独测试
4. **代码重复**：相似的状态更新逻辑重复出现

useReducer 通过将状态更新逻辑集中到 reducer 函数中解决了这些问题。

## 核心概念

### useReducer 的基本结构

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

- **reducer**：接受当前状态和 action，返回新状态的函数
- **initialState**：状态的初始值
- **state**：当前状态值
- **dispatch**：用于发送 action 的函数

### Reducer 函数

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'SET_VALUE':
      return { count: action.payload };
    default:
      return state;
  }
}
```

### Action 的结构

```jsx
// 标准 action 格式
{
  type: 'ACTION_TYPE',
  payload: 'action data'
}

// 简化的 action
{ type: 'INCREMENT' } // 不需要 payload
```

## 基本用法

### 计数器示例

```jsx
import { useReducer } from 'react';

// 1. 定义 reducer 函数
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 };
    case 'DECREMENT':
      return { count: state.count - 1 };
    case 'SET':
      return { count: action.payload };
    case 'RESET':
      return { count: 0 };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// 2. 定义初始状态
const initialState = { count: 0 };

// 3. 在组件中使用
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  return (
    <div>
      <p>当前计数: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        增加 (+1)
      </button>
      <button onClick={() => dispatch({ type: 'DECREMENT' })}>
        减少 (-1)
      </button>
      <button onClick={() => dispatch({ type: 'SET', payload: 10 })}>
        设置为 10
      </button>
      <button onClick={() => dispatch({ type: 'RESET' })}>
        重置
      </button>
    </div>
  );
}
```

### 惰性初始化

```jsx
function init(initialCount) {
  return { count: initialCount, message: '欢迎使用计数器' };
}

function reducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return { ...state, count: state.count + 1 };
    case 'DECREMENT':
      return { ...state, count: state.count - 1 };
    default:
      return state;
  }
}

function CounterWithInit({ initialCount = 0 }) {
  // 使用惰性初始化函数
  const [state, dispatch] = useReducer(reducer, initialCount, init);
  
  return (
    <div>
      <p>{state.message}</p>
      <p>当前计数: {state.count}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>
        增加
      </button>
    </div>
  );
}
```

### 复杂状态管理

```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return {
        ...state,
        todos: [
          ...state.todos,
          {
            id: Date.now(),
            text: action.payload.text,
            completed: false
          }
        ]
      };
    
    case 'TOGGLE_TODO':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload.id
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    
    case 'DELETE_TODO':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload.id)
      };
    
    case 'SET_FILTER':
      return {
        ...state,
        filter: action.payload.filter
      };
    
    case 'SET_LOADING':
      return {
        ...state,
        loading: action.payload.loading
      };
    
    default:
      return state;
  }
}

const initialState = {
  todos: [],
  filter: 'all', // 'all' | 'active' | 'completed'
  loading: false
};

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [inputValue, setInputValue] = useState('');
  
  const handleAddTodo = () => {
    if (inputValue.trim()) {
      dispatch({ type: 'ADD_TODO', payload: { text: inputValue } });
      setInputValue('');
    }
  };
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <div>
      <div>
        <input
          value={inputValue}
          onChange={(e) => setInputValue(e.target.value)}
          placeholder="添加待办事项"
        />
        <button onClick={handleAddTodo}>添加</button>
      </div>
      
      <div>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'all' }) }}>
          全部
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'active' }) }}>
          未完成
        </button>
        <button onClick={() => dispatch({ type: 'SET_FILTER', payload: { filter: 'completed' }) }}>
          已完成
        </button>
      </div>
      
      <ul>
        {filteredTodos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.completed}
              onChange={() => dispatch({ type: 'TOGGLE_TODO', payload: { id: todo.id } })}
            />
            <span style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => dispatch({ type: 'DELETE_TODO', payload: { id: todo.id } })}>
              删除
            </button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## 实际应用

### 表单状态管理

```jsx
function formReducer(state, action) {
  switch (action.type) {
    case 'SET_FIELD':
      return {
        ...state,
        values: {
          ...state.values,
          [action.payload.field]: action.payload.value
        },
        errors: {
          ...state.errors,
          [action.payload.field]: '' // 清除该字段的错误
        }
      };
    
    case 'SET_ERROR':
      return {
        ...state,
        errors: {
          ...state.errors,
          [action.payload.field]: action.payload.error
        }
      };
    
    case 'SET_TOUCHED':
      return {
        ...state,
        touched: {
          ...state.touched,
          [action.payload.field]: true
        }
      };
    
    case 'RESET':
      return initialState;
    
    case 'SUBMIT_START':
      return {
        ...state,
        submitting: true
      };
    
    case 'SUBMIT_SUCCESS':
      return {
        ...state,
        submitting: false,
        submitSuccess: true
      };
    
    case 'SUBMIT_ERROR':
      return {
        ...state,
        submitting: false,
        submitError: action.payload.error
      };
    
    default:
      return state;
  }
}

const initialState = {
  values: {
    username: '',
    email: '',
    password: ''
  },
  errors: {
    username: '',
    email: '',
    password: ''
  },
  touched: {
    username: false,
    email: false,
    password: false
  },
  submitting: false,
  submitSuccess: false,
  submitError: null
};

function RegistrationForm() {
  const [state, dispatch] = useReducer(formReducer, initialState);
  
  const validateField = (field, value) => {
    switch (field) {
      case 'username':
        return value.length < 3 ? '用户名至少需要3个字符' : '';
      case 'email':
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return !emailRegex.test(value) ? '邮箱格式不正确' : '';
      case 'password':
        return value.length < 6 ? '密码至少需要6个字符' : '';
      default:
        return '';
    }
  };
  
  const handleChange = (field) => (e) => {
    const value = e.target.value;
    dispatch({ type: 'SET_FIELD', payload: { field, value } });
    
    // 实时验证
    const error = validateField(field, value);
    if (error) {
      dispatch({ type: 'SET_ERROR', payload: { field, error } });
    }
  };
  
  const handleBlur = (field) => () => {
    dispatch({ type: 'SET_TOUCHED', payload: { field } });
    
    // 失去焦点时验证
    const error = validateField(field, state.values[field]);
    dispatch({ type: 'SET_ERROR', payload: { field, error } });
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    
    // 验证所有字段
    const errors = {};
    Object.keys(state.values).forEach(field => {
      const error = validateField(field, state.values[field]);
      if (error) {
        errors[field] = error;
        dispatch({ type: 'SET_ERROR', payload: { field, error } });
      }
    });
    
    if (Object.keys(errors).length > 0) {
      return;
    }
    
    dispatch({ type: 'SUBMIT_START' });
    
    try {
      const response = await fetch('/api/register', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(state.values)
      });
      
      if (!response.ok) {
        throw new Error('注册失败');
      }
      
      dispatch({ type: 'SUBMIT_SUCCESS' });
      
      // 重置表单
      setTimeout(() => {
        dispatch({ type: 'RESET' });
      }, 2000);
      
    } catch (error) {
      dispatch({ type: 'SUBMIT_ERROR', payload: { error: error.message } });
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>用户名:</label>
        <input
          value={state.values.username}
          onChange={handleChange('username')}
          onBlur={handleBlur('username')}
        />
        {state.touched.username && state.errors.username && (
          <span style={{ color: 'red' }}>{state.errors.username}</span>
        )}
      </div>
      
      <div>
        <label>邮箱:</label>
        <input
          type="email"
          value={state.values.email}
          onChange={handleChange('email')}
          onBlur={handleBlur('email')}
        />
        {state.touched.email && state.errors.email && (
          <span style={{ color: 'red' }}>{state.errors.email}</span>
        )}
      </div>
      
      <div>
        <label>密码:</label>
        <input
          type="password"
          value={state.values.password}
          onChange={handleChange('password')}
          onBlur={handleBlur('password')}
        />
        {state.touched.password && state.errors.password && (
          <span style={{ color: 'red' }}>{state.errors.password}</span>
        )}
      </div>
      
      <button type="submit" disabled={state.submitting}>
        {state.submitting ? '注册中...' : '注册'}
      </button>
      
      {state.submitSuccess && (
        <div style={{ color: 'green' }}>注册成功！</div>
      )}
      
      {state.submitError && (
        <div style={{ color: 'red' }}>{state.submitError}</div>
      )}
    </form>
  );
}
```

### 购物车管理

```jsx
function cartReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      const existingItem = state.items.find(
        item => item.id === action.payload.product.id
      );
      
      if (existingItem) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.product.id
              ? { ...item, quantity: item.quantity + action.payload.quantity }
              : item
          )
        };
      } else {
        return {
          ...state,
          items: [
            ...state.items,
            {
              ...action.payload.product,
              quantity: action.payload.quantity
            }
          ]
        };
      }
    
    case 'REMOVE_ITEM':
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload.id)
      };
    
    case 'UPDATE_QUANTITY':
      return {
        ...state,
        items: state.items.map(item =>
          item.id === action.payload.id
            ? { ...item, quantity: action.payload.quantity }
            : item
        )
      };
    
    case 'CLEAR_CART':
      return {
        ...state,
        items: []
      };
    
    case 'APPLY_COUPON':
      return {
        ...state,
        coupon: action.payload.coupon,
        discount: action.payload.discount
      };
    
    case 'REMOVE_COUPON':
      return {
        ...state,
        coupon: null,
        discount: 0
      };
    
    default:
      return state;
  }
}

const initialState = {
  items: [],
  coupon: null,
  discount: 0
};

function ShoppingCart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  const totalItems = state.items.reduce((sum, item) => sum + item.quantity, 0);
  const subtotal = state.items.reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );
  const discountAmount = subtotal * (state.discount / 100);
  const total = subtotal - discountAmount;
  
  const addToCart = (product, quantity = 1) => {
    dispatch({ type: 'ADD_ITEM', payload: { product, quantity } });
  };
  
  const updateQuantity = (id, quantity) => {
    if (quantity <= 0) {
      dispatch({ type: 'REMOVE_ITEM', payload: { id } });
    } else {
      dispatch({ type: 'UPDATE_QUANTITY', payload: { id, quantity } });
    }
  };
  
  const applyCoupon = (code) => {
    // 模拟优惠券验证
    const coupons = {
      'SAVE10': { discount: 10, name: '10% Off' },
      'SAVE20': { discount: 20, name: '20% Off' },
      'SUMMER': { discount: 15, name: 'Summer Sale' }
    };
    
    const coupon = coupons[code];
    if (coupon) {
      dispatch({ type: 'APPLY_COUPON', payload: coupon });
    } else {
      alert('无效的优惠券代码');
    }
  };
  
  return (
    <div>
      <h2>购物车</h2>
      
      {/* 模拟添加商品 */}
      <div>
        <h3>商品列表</h3>
        <button onClick={() => addToCart({ id: 1, name: '产品 A', price: 100 })}>
          添加产品 A ($100)
        </button>
        <button onClick={() => addToCart({ id: 2, name: '产品 B', price: 200 })}>
          添加产品 B ($200)
        </button>
        <button onClick={() => addToCart({ id: 3, name: '产品 C', price: 150 })}>
          添加产品 C ($150)
        </button>
      </div>
      
      {/* 购物车项目 */}
      <div>
        <h3>购物车 ({totalItems} 件)</h3>
        {state.items.length === 0 ? (
          <p>购物车是空的</p>
        ) : (
          <ul>
            {state.items.map(item => (
              <li key={item.id}>
                {item.name} - ${item.price}
                <input
                  type="number"
                  min="1"
                  value={item.quantity}
                  onChange={(e) => updateQuantity(item.id, parseInt(e.target.value))}
                />
                <button onClick={() => dispatch({ type: 'REMOVE_ITEM', payload: { id: item.id } })}>
                  移除
                </button>
              </li>
            ))}
          </ul>
        )}
      </div>
      
      {/* 优惠券 */}
      <div>
        <h3>优惠券</h3>
        {state.coupon ? (
          <div>
            已应用: {state.coupon.name} ({state.discount}% Off)
            <button onClick={() => dispatch({ type: 'REMOVE_COUPON' })}>
              移除
            </button>
          </div>
        ) : (
          <div>
            <input
              type="text"
              placeholder="输入优惠券代码"
              id="coupon-input"
            />
            <button onClick={() => {
              const input = document.getElementById('coupon-input');
              applyCoupon(input.value);
            }}>
              应用
            </button>
          </div>
        )}
      </div>
      
      {/* 订单摘要 */}
      <div>
        <h3>订单摘要</h3>
        <p>小计: ${subtotal.toFixed(2)}</p>
        {state.discount > 0 && (
          <p>折扣: -${discountAmount.toFixed(2)}</p>
        )}
        <p>总计: ${total.toFixed(2)}</p>
        
        {state.items.length > 0 && (
          <button onClick={() => alert('结账功能未实现')}>
            结账 (${total.toFixed(2)})
          </button>
        )}
      </div>
    </div>
  );
}
```

### 游戏状态管理

```jsx
function gameReducer(state, action) {
  switch (action.type) {
    case 'START_GAME':
      return {
        ...state,
        status: 'playing',
        score: 0,
        timeRemaining: action.payload.duration || 60,
        questions: generateQuestions(10),
        currentQuestionIndex: 0
      };
    
    case 'ANSWER_QUESTION':
      const { questionId, answer, timeTaken } = action.payload;
      const question = state.questions.find(q => q.id === questionId);
      const isCorrect = question.correctAnswer === answer;
      
      const points = calculatePoints(isCorrect, timeTaken);
      
      return {
        ...state,
        score: state.score + points,
        answers: [
          ...state.answers,
          {
            questionId,
            answer,
            isCorrect,
            points,
            timeTaken
          }
        ],
        currentQuestionIndex: state.currentQuestionIndex + 1
      };
    
    case 'UPDATE_TIME':
      const newTimeRemaining = state.timeRemaining - 1;
      
      if (newTimeRemaining <= 0) {
        return {
          ...state,
          status: 'gameOver',
          timeRemaining: 0
        };
      }
      
      return {
        ...state,
        timeRemaining: newTimeRemaining
      };
    
    case 'END_GAME':
      return {
        ...state,
        status: 'gameOver'
      };
    
    case 'RESET_GAME':
      return initialState;
    
    default:
      return state;
  }
}

function generateQuestions(count) {
  // 生成简单的数学问题
  return Array.from({ length: count }, (_, i) => {
    const num1 = Math.floor(Math.random() * 10) + 1;
    const num2 = Math.floor(Math.random() * 10) + 1;
    const operations = ['+', '-', '*'];
    const operation = operations[Math.floor(Math.random() * operations.length)];
    
    let correctAnswer;
    switch (operation) {
      case '+':
        correctAnswer = num1 + num2;
        break;
      case '-':
        correctAnswer = num1 - num2;
        break;
      case '*':
        correctAnswer = num1 * num2;
        break;
    }
    
    return {
      id: i + 1,
      question: `${num1} ${operation} ${num2} = ?`,
      correctAnswer,
      options: generateOptions(correctAnswer)
    };
  });
}

function generateOptions(correctAnswer) {
  const options = [correctAnswer];
  
  while (options.length < 4) {
    const wrongAnswer = correctAnswer + Math.floor(Math.random() * 10) - 5;
    if (!options.includes(wrongAnswer) && wrongAnswer >= 0) {
      options.push(wrongAnswer);
    }
  }
  
  return options.sort(() => Math.random() - 0.5);
}

function calculatePoints(isCorrect, timeTaken) {
  if (!isCorrect) return 0;
  // 基于时间的积分系统
  const basePoints = 100;
  const timeBonus = Math.max(0, 10 - timeTaken) * 10;
  return basePoints + timeBonus;
}

const initialState = {
  status: 'idle', // 'idle' | 'playing' | 'gameOver'
  score: 0,
  timeRemaining: 0,
  questions: [],
  currentQuestionIndex: 0,
  answers: []
};

function MathGame() {
  const [state, dispatch] = useReducer(gameReducer, initialState);
  const [currentQuestionStartTime, setCurrentQuestionStartTime] = useState(0);
  
  // 游戏计时器
  useEffect(() => {
    let timer;
    
    if (state.status === 'playing') {
      timer = setInterval(() => {
        dispatch({ type: 'UPDATE_TIME' });
      }, 1000);
    }
    
    return () => clearInterval(timer);
  }, [state.status]);
  
  const startGame = () => {
    dispatch({ type: 'START_GAME', payload: { duration: 60 } });
  };
  
  const handleAnswer = (answer) => {
    const timeTaken = (Date.now() - currentQuestionStartTime) / 1000;
    const currentQuestion = state.questions[state.currentQuestionIndex];
    
    dispatch({
      type: 'ANSWER_QUESTION',
      payload: {
        questionId: currentQuestion.id,
        answer,
        timeTaken
      }
    });
    
    setCurrentQuestionStartTime(Date.now());
  };
  
  const resetGame = () => {
    dispatch({ type: 'RESET_GAME' });
  };
  
  const currentQuestion = state.questions[state.currentQuestionIndex];
  const isGameOver = state.status === 'gameOver' || state.currentQuestionIndex >= state.questions.length;
  
  return (
    <div>
      <h1>数学游戏</h1>
      
      {state.status === 'idle' && (
        <div>
          <p>在 60 秒内回答尽可能多的数学问题！</p>
          <button onClick={startGame}>开始游戏</button>
        </div>
      )}
      
      {state.status === 'playing' && currentQuestion && (
        <div>
          <div>
            <p>分数: {state.score}</p>
            <p>时间: {state.timeRemaining} 秒</p>
            <p>进度: {state.currentQuestionIndex + 1} / {state.questions.length}</p>
          </div>
          
          <div>
            <h2>{currentQuestion.question}</h2>
            <div>
              {currentQuestion.options.map((option, index) => (
                <button
                  key={index}
                  onClick={() => handleAnswer(option)}
                  style={{ margin: '10px', padding: '20px' }}
                >
                  {option}
                </button>
              ))}
            </div>
          </div>
        </div>
      )}
      
      {isGameOver && (
        <div>
          <h2>游戏结束！</h2>
          <p>最终分数: {state.score}</p>
          <p>正确答案: {state.answers.filter(a => a.isCorrect).length} / {state.answers.length}</p>
          
          <h3>答案详情:</h3>
          <ul>
            {state.answers.map((answer, index) => (
              <li key={index}>
                问题 {index + 1}: {answer.isCorrect ? '正确' : '错误'} (+{answer.points}分)
              </li>
            ))}
          </ul>
          
          <button onClick={resetGame}>再玩一次</button>
        </div>
      )}
    </div>
  );
}
```

## 注意事项

### 1. Reducer 必须是纯函数

```jsx
// ❌ 错误：修改原始状态
function badReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      state.items.push(action.payload); // 直接修改状态
      return state;
    default:
      return state;
  }
}

// ✅ 正确：返回新状态
function goodReducer(state, action) {
  switch (action.type) {
    case 'ADD_ITEM':
      return {
        ...state,
        items: [...state.items, action.payload] // 创建新数组
      };
    default:
      return state;
  }
}
```

### 2. 处理复杂的状态更新

```jsx
// ❌ 难以维护的嵌套更新
function complexUpdate(state, action) {
  return {
    ...state,
    data: {
      ...state.data,
      users: {
        ...state.data.users,
        items: state.data.users.items.map(user =>
          user.id === action.payload.id
            ? { ...user, profile: { ...user.profile, age: action.payload.age } }
            : user
        )
      }
    }
  };
}

// ✅ 更清晰的更新逻辑
function complexUpdate(state, action) {
  const users = state.data.users.items.map(user =>
    user.id === action.payload.id
      ? { ...user, profile: { ...user.profile, age: action.payload.age } }
      : user
  );
  
  return {
    ...state,
    data: {
      ...state.data,
      users: {
        ...state.data.users,
        items: users
      }
    }
  };
}
```

### 3. Action 的类型安全

```jsx
// 使用常量定义 action 类型
const ACTION_TYPES = {
  INCREMENT: 'INCREMENT',
  DECREMENT: 'DECREMENT',
  SET_VALUE: 'SET_VALUE',
  RESET: 'RESET'
};

function reducer(state, action) {
  switch (action.type) {
    case ACTION_TYPES.INCREMENT:
      return { count: state.count + 1 };
    // ...
    default:
      return state;
  }
}

// 或者使用 TypeScript 接口
interface Action {
  type: string;
  payload?: any;
}
```

### 4. useReducer vs useState

```jsx
// useState: 简单的状态
function SimpleCounter() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>增加</button>
    </div>
  );
}

// useReducer: 复杂的状态逻辑
function ComplexCounter() {
  const [state, dispatch] = useReducer(counterReducer, {
    count: 0,
    history: [],
    maxValue: 0
  });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <p>Max: {state.maxValue}</p>
      <button onClick={() => dispatch({ type: 'INCREMENT' })}>增加</button>
    </div>
  );
}
```

### 5. 性能优化

```jsx
// 使用 useMemo 缓存计算值
function Cart() {
  const [state, dispatch] = useReducer(cartReducer, initialState);
  
  const cartTotal = useMemo(() => {
    return state.items.reduce(
      (sum, item) => sum + item.price * item.quantity,
      0
    );
  }, [state.items]);
  
  return (
    <div>
      <p>总计: ${cartTotal.toFixed(2)}</p>
      {/* 其他渲染内容 */}
    </div>
  );
}
```

## 总结

useReducer 是 React 中处理复杂状态的强大工具：

### 核心要点

1. **集中状态逻辑**：将所有状态更新逻辑集中在一个地方
2. **可预测性**：通过 action 和 reducer 使状态变化可预测
3. **可测试性**：reducer 是纯函数，易于单独测试
4. **可维护性**：清晰的状态管理和更新模式

### 使用场景

1. **复杂状态逻辑**：多个子值的相关状态
2. **状态依赖**：下一个状态依赖于前一个状态
3. **可测试性要求**：需要单独测试状态逻辑
4. **类 Redux 需求**：需要类似 Redux 的状态管理模式

### 最佳实践

1. **纯函数 Reducer**：保持 reducer 的纯净性
2. **Action 类型**：使用常量定义 action 类型
3. **不可变性**：始终返回新的状态对象
4. **代码组织**：将 reducer 和 action 类型分离到独立文件
5. **性能优化**：合理使用 useMemo 和 useCallback

### useReducer vs Redux

**useReducer 适合：**
- 中小型应用的状态管理
- 组件级别的复杂状态
- 不需要时间旅行调试
- 不需要中间件支持

**Redux 适合：**
- 大型应用的全局状态管理
- 需要时间旅行调试
- 需要中间件支持
- 需要强大的开发者工具

useReducer 提供了一种优雅的方式来管理 React 组件中的复杂状态。它是连接简单 useState 和完整 Redux 桥梁，为开发者提供了灵活的状态管理选择。掌握 useReducer 能够帮助你构建更可维护、更可测试的 React 应用。