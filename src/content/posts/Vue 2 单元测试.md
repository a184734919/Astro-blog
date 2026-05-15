---
title: Vue 2 单元测试
published: 2023-10-25
description: 'Jest + Vue Test Utils的详细介绍和学习笔记'
image: ''
tags: ["Vue2","测试"]
category: 'Vue2'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 2 单元测试是确保 Vue 2 应用质量和稳定性的重要手段。通过测试，我们可以：

- 验证组件行为和渲染结果
- 防止代码变更引入回归错误
- 提供组件使用文档
- 支持安全重构
- 提升代码质量

Vue 2 单元测试主要使用 Jest 作为测试框架，Vue Test Utils 作为组件测试工具。

---

## 核心概念

### Vue Test Utils 核心概念

1. **Mount vs ShallowMount**：
   - `mount()`: 完全渲染组件及其子组件
   - `shallowMount()`: 浅渲染，不渲染子组件

2. **Wrapper**：包装组件实例的测试工具

3. **选择器**：用于查找 DOM 元素

4. **测试工具**：触发事件、修改数据等

### 测试策略

- **单元测试**：测试独立的组件功能
- **集成测试**：测试组件之间的交互
- **快照测试**：测试组件渲染的稳定性

---

## 基本用法

### 1. 环境配置

```bash
# 安装依赖
npm install --save-dev jest @vue/test-utils vue-jest babel-jest @babel/preset-env
npm install --save-dev jest-serializer-vue @testing-library/jest-dom
```

```javascript
// jest.config.js
module.exports = {
  moduleFileExtensions: ['js', 'json', 'vue'],
  transform: {
    '^.+\\.vue$': 'vue-jest',
    '^.+\\.js$': 'babel-jest'
  },
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  snapshotSerializers: ['jest-serializer-vue'],
  testMatch: [
    '**/tests/unit/**/*.spec.(js|jsx|ts|tsx)|**/__tests__/*.(js|jsx|ts|tsx)',
    '**/src/**/__tests__/*.(js|jsx|ts|tsx)'
  ],
  collectCoverageFrom: [
    'src/**/*.{js,vue}',
    '!src/main.js',
    '!**/node_modules/**'
  ],
  coverageDirectory: '<rootDir>/coverage',
  coverageReporters: ['text-summary', 'lcov', 'html']
};
```

```javascript
// babel.config.js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      targets: {
        node: 'current'
      }
    }]
  ]
};
```

```javascript
// package.json
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest --config jest.config.js",
    "test:coverage": "jest --coverage",
    "test:watch": "jest --watch"
  }
}
```

### 2. 基础组件测试

```vue
<!-- components/Button.vue -->
<template>
  <button
    :class="['btn', variantClass]"
    :disabled="disabled"
    @click="handleClick"
    data-testid="button"
  >
    <slot>{{ text }}</slot>
  </button>
</template>

<script>
export default {
  name: 'Button',
  props: {
    text: {
      type: String,
      default: ''
    },
    variant: {
      type: String,
      default: 'primary',
      validator: (value) => ['primary', 'secondary', 'danger'].includes(value)
    },
    disabled: {
      type: Boolean,
      default: false
    }
  },
  computed: {
    variantClass() {
      return `btn-${this.variant}`;
    }
  },
  methods: {
    handleClick(event) {
      if (!this.disabled) {
        this.$emit('click', event);
      }
    }
  }
};
</script>

<style scoped>
.btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-primary {
  background-color: #007bff;
  color: white;
}

.btn-secondary {
  background-color: #6c757d;
  color: white;
}

.btn-danger {
  background-color: #dc3545;
  color: white;
}

.btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}
</style>
```

```javascript
// components/__tests__/Button.spec.js
import { mount } from '@vue/test-utils';
import Button from '../Button';

describe('Button 组件', () => {
  describe('基础渲染', () => {
    test('应该渲染按钮文本', () => {
      const wrapper = mount(Button, {
        propsData: {
          text: '点击我'
        }
      });

      expect(wrapper.text()).toContain('点击我');
    });

    test('应该渲染默认文本', () => {
      const wrapper = mount(Button);

      expect(wrapper.text()).toBe('');
    });

    test('应该渲染 slot 内容', () => {
      const wrapper = mount(Button, {
        slots: {
          default: '<span>Slot 内容</span>'
        }
      });

      expect(wrapper.html()).toContain('Slot 内容');
    });

    test('应该渲染按钮元素', () => {
      const wrapper = mount(Button);

      expect(wrapper.find('button').exists()).toBe(true);
    });
  });

  describe('属性测试', () => {
    test('应该应用正确的 variant 类名', () => {
      const wrapper = mount(Button, {
        propsData: {
          variant: 'primary'
        }
      });

      expect(wrapper.find('.btn-primary').exists()).toBe(true);
    });

    test('应该应用 secondary variant', () => {
      const wrapper = mount(Button, {
        propsData: {
          variant: 'secondary'
        }
      });

      expect(wrapper.find('.btn-secondary').exists()).toBe(true);
    });

    test('应该应用 danger variant', () => {
      const wrapper = mount(Button, {
        propsData: {
          variant: 'danger'
        }
      });

      expect(wrapper.find('.btn-danger').exists()).toBe(true);
    });

    test('无效 variant 应该抛出警告', () => {
      const warn = jest.spyOn(console, 'warn');
      mount(Button, {
        propsData: {
          variant: 'invalid'
        }
      });

      expect(warn).toHaveBeenCalled();
      warn.mockRestore();
    });

    test('disabled 属性应该正确设置', () => {
      const wrapper = mount(Button, {
        propsData: {
          disabled: true
        }
      });

      const button = wrapper.find('button');
      expect(button.attributes('disabled')).toBe('disabled');
    });
  });

  describe('事件测试', () => {
    test('应该触发 click 事件', async () => {
      const wrapper = mount(Button);

      await wrapper.find('button').trigger('click');

      expect(wrapper.emitted('click')).toBeTruthy();
      expect(wrapper.emitted('click').length).toBe(1);
    });

    test('应该传递事件对象', async () => {
      const wrapper = mount(Button);

      await wrapper.find('button').trigger('click');

      const clickEvent = wrapper.emitted('click')[0][0];
      expect(clickEvent).toBeDefined();
      expect(clickEvent.type).toBe('click');
    });

    test('disabled 时不应该触发 click 事件', async () => {
      const wrapper = mount(Button, {
        propsData: {
          disabled: true
        }
      });

      await wrapper.find('button').trigger('click');

      expect(wrapper.emitted('click')).toBeFalsy();
    });
  });

  describe('用户交互', () => {
    test('点击按钮应该触发事件', async () => {
      const wrapper = mount(Button);

      const button = wrapper.find('button');
      await button.trigger('click');

      expect(wrapper.emitted('click')).toBeTruthy();
    });

    test('多次点击应该触发多次事件', async () => {
      const wrapper = mount(Button);

      const button = wrapper.find('button');
      await button.trigger('click');
      await button.trigger('click');
      await button.trigger('click');

      expect(wrapper.emitted('click').length).toBe(3);
    });
  });
});
```

### 3. 表单组件测试

```vue
<!-- components/FormInput.vue -->
<template>
  <div class="form-group">
    <label :for="id">{{ label }}</label>
    <input
      :id="id"
      v-model="internalValue"
      :type="type"
      :placeholder="placeholder"
      :disabled="disabled"
      @input="handleInput"
      @blur="handleBlur"
      data-testid="input"
    />
    <span v-if="error" class="error-message">{{ error }}</span>
  </div>
</template>

<script>
export default {
  name: 'FormInput',
  props: {
    value: {
      type: [String, Number],
      default: ''
    },
    label: {
      type: String,
      required: true
    },
    type: {
      type: String,
      default: 'text'
    },
    placeholder: {
      type: String,
      default: ''
    },
    disabled: {
      type: Boolean,
      default: false
    },
    error: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      id: `input-${Math.random().toString(36).substr(2, 9)}`
    };
  },
  computed: {
    internalValue: {
      get() {
        return this.value;
      },
      set(value) {
        this.$emit('input', value);
      }
    }
  },
  methods: {
    handleInput(event) {
      this.$emit('input', event.target.value);
    },
    handleBlur(event) {
      this.$emit('blur', event);
    }
  }
};
</script>

<style scoped>
.form-group {
  margin-bottom: 16px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error-message {
  color: red;
  font-size: 12px;
  margin-top: 4px;
}
</style>
```

```javascript
// components/__tests__/FormInput.spec.js
import { mount } from '@vue/test-utils';
import FormInput from '../FormInput';

describe('FormInput 组件', () => {
  describe('基础渲染', () => {
    test('应该渲染 label', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名'
        }
      });

      expect(wrapper.text()).toContain('用户名');
    });

    test('应该渲染 input 元素', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名'
        }
      });

      expect(wrapper.find('input').exists()).toBe(true);
    });

    test('应该设置正确的 input 类型', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '邮箱',
          type: 'email'
        }
      });

      expect(wrapper.find('input').attributes('type')).toBe('email');
    });

    test('应该设置 placeholder', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          placeholder: '请输入用户名'
        }
      });

      expect(wrapper.find('input').attributes('placeholder')).toBe('请输入用户名');
    });
  });

  describe('v-model 测试', () => {
    test('应该初始化输入值', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          value: '张三'
        }
      });

      expect(wrapper.find('input').element.value).toBe('张三');
    });

    test('应该触发 input 事件', async () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名'
        }
      });

      const input = wrapper.find('input');
      await input.setValue('李四');

      expect(wrapper.emitted('input')).toBeTruthy();
      expect(wrapper.emitted('input')[0]).toEqual(['李四']);
    });

    test('应该双向绑定', async () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          value: '初始值'
        }
      });

      const input = wrapper.find('input');
      await input.setValue('新值');

      expect(wrapper.vm.internalValue).toBe('新值');
    });
  });

  describe('错误处理', () => {
    test('应该显示错误信息', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          error: '用户名不能为空'
        }
      });

      expect(wrapper.find('.error-message').exists()).toBe(true);
      expect(wrapper.find('.error-message').text()).toBe('用户名不能为空');
    });

    test('没有错误时不应该显示错误信息', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          error: ''
        }
      });

      expect(wrapper.find('.error-message').exists()).toBe(false);
    });
  });

  describe('禁用状态', () => {
    test('应该设置 disabled 属性', () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          disabled: true
        }
      });

      expect(wrapper.find('input').attributes('disabled')).toBe('disabled');
    });

    test('禁用时不应该触发 input 事件', async () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名',
          disabled: true
        }
      });

      const input = wrapper.find('input');
      await input.setValue('新值');

      expect(wrapper.emitted('input')).toBeFalsy();
    });
  });

  describe('事件触发', () => {
    test('应该触发 blur 事件', async () => {
      const wrapper = mount(FormInput, {
        propsData: {
          label: '用户名'
        }
      });

      const input = wrapper.find('input');
      await input.trigger('blur');

      expect(wrapper.emitted('blur')).toBeTruthy();
    });
  });
});
```

---

## 实际应用

### 1. 计算属性测试

```vue
<!-- components/UserCard.vue -->
<template>
  <div class="user-card" data-testid="user-card">
    <h3 data-testid="user-name">{{ fullName }}</h3>
    <p data-testid="user-email">{{ user.email }}</p>
    <p data-testid="user-status">{{ statusText }}</p>
    <button @click="toggleStatus">切换状态</button>
  </div>
</template>

<script>
export default {
  name: 'UserCard',
  props: {
    user: {
      type: Object,
      required: true,
      validator: (user) => {
        return user.firstName && user.lastName && user.email;
      }
    }
  },
  data() {
    return {
      isActive: true
    };
  },
  computed: {
    fullName() {
      return `${this.user.firstName} ${this.user.lastName}`;
    },
    statusText() {
      return this.isActive ? '活跃' : '不活跃';
    }
  },
  methods: {
    toggleStatus() {
      this.isActive = !this.isActive;
      this.$emit('status-change', this.isActive);
    }
  }
};
</script>
```

```javascript
// components/__tests__/UserCard.spec.js
import { mount } from '@vue/test-utils';
import UserCard from '../UserCard';

describe('UserCard 组件', () => {
  const mockUser = {
    firstName: '张',
    lastName: '三',
    email: 'zhangsan@example.com'
  };

  describe('计算属性', () => {
    test('应该正确计算全名', () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      expect(wrapper.vm.fullName).toBe('张 三');
      expect(wrapper.find('[data-testid="user-name"]').text()).toBe('张 三');
    });

    test('应该正确显示状态文本', () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      expect(wrapper.vm.statusText).toBe('活跃');
      expect(wrapper.find('[data-testid="user-status"]').text()).toBe('活跃');
    });

    test('应该响应数据变化', async () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      expect(wrapper.vm.statusText).toBe('活跃');

      wrapper.setData({ isActive: false });
      await wrapper.vm.$nextTick();

      expect(wrapper.vm.statusText).toBe('不活跃');
    });
  });

  describe('用户信息显示', () => {
    test('应该显示用户邮箱', () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      expect(wrapper.find('[data-testid="user-email"]').text()).toBe('zhangsan@example.com');
    });
  });

  describe('方法测试', () => {
    test('toggleStatus 应该切换状态', async () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      expect(wrapper.vm.isActive).toBe(true);

      wrapper.vm.toggleStatus();
      await wrapper.vm.$nextTick();

      expect(wrapper.vm.isActive).toBe(false);
    });

    test('toggleStatus 应该触发事件', async () => {
      const wrapper = mount(UserCard, {
        propsData: {
          user: mockUser
        }
      });

      await wrapper.find('button').trigger('click');

      expect(wrapper.emitted('status-change')).toBeTruthy();
      expect(wrapper.emitted('status-change')[0]).toEqual([false]);
    });
  });
});
```

### 2. 列表组件测试

```vue
<!-- components/TodoList.vue -->
<template>
  <div class="todo-list">
    <input
      v-model="newTodo"
      @keyup.enter="addTodo"
      placeholder="输入新待办事项"
      data-testid="new-todo-input"
    />
    <button @click="addTodo" data-testid="add-button">添加</button>

    <ul data-testid="todo-list">
      <li
        v-for="(todo, index) in todos"
        :key="todo.id"
        :class="{ completed: todo.completed }"
        data-testid="todo-item"
      >
        <span>{{ todo.text }}</span>
        <button @click="toggleTodo(index)">切换</button>
        <button @click="removeTodo(index)">删除</button>
      </li>
    </ul>

    <p v-if="todos.length === 0">暂无待办事项</p>
  </div>
</template>

<script>
export default {
  name: 'TodoList',
  data() {
    return {
      newTodo: '',
      todos: []
    };
  },
  methods: {
    addTodo() {
      if (this.newTodo.trim()) {
        this.todos.push({
          id: Date.now(),
          text: this.newTodo.trim(),
          completed: false
        });
        this.newTodo = '';
      }
    },
    toggleTodo(index) {
      this.todos[index].completed = !this.todos[index].completed;
    },
    removeTodo(index) {
      this.todos.splice(index, 1);
    }
  }
};
</script>

<style scoped>
.todo-list {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

input {
  padding: 8px;
  margin-right: 8px;
}

button {
  padding: 8px 16px;
  margin-right: 8px;
}

.completed {
  text-decoration: line-through;
  color: #999;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 10px;
  margin: 8px 0;
  background-color: #f5f5f5;
}
</style>
```

```javascript
// components/__tests__/TodoList.spec.js
import { mount } from '@vue/test-utils';
import TodoList from '../TodoList';

describe('TodoList 组件', () => {
  describe('初始状态', () => {
    test('应该渲染空的待办列表', () => {
      const wrapper = mount(TodoList);

      expect(wrapper.find('[data-testid="todo-list"]').exists()).toBe(true);
      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(0);
    });

    test('应该显示空状态提示', () => {
      const wrapper = mount(TodoList);

      expect(wrapper.text()).toContain('暂无待办事项');
    });
  });

  describe('添加待办事项', () => {
    test('应该能够添加待办事项', async () => {
      const wrapper = mount(TodoList);

      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('学习 Vue.js');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      const todoItems = wrapper.findAll('[data-testid="todo-item"]');
      expect(todoItems).toHaveLength(1);
      expect(todoItems[0].text()).toContain('学习 Vue.js');
    });

    test('应该能够通过回车键添加', async () => {
      const wrapper = mount(TodoList);

      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('学习测试');

      await input.trigger('keyup.enter');
      await wrapper.vm.$nextTick();

      const todoItems = wrapper.findAll('[data-testid="todo-item"]');
      expect(todoItems).toHaveLength(1);
    });

    test('不应该添加空待办事项', async () => {
      const wrapper = mount(TodoList);

      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('   ');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      const todoItems = wrapper.findAll('[data-testid="todo-item"]');
      expect(todoItems).toHaveLength(0);
    });

    test('添加后应该清空输入框', async () => {
      const wrapper = mount(TodoList);

      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('新任务');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      expect(input.element.value).toBe('');
    });
  });

  describe('待办事项操作', () => {
    beforeEach(async () => {
      const wrapper = mount(TodoList);
      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();
    });

    test('应该能够切换完成状态', async () => {
      const wrapper = mount(TodoList);
      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      const toggleButton = wrapper.findAll('button')[1]; // 第二个按钮是切换按钮
      await toggleButton.trigger('click');
      await wrapper.vm.$nextTick();

      const todoItem = wrapper.find('[data-testid="todo-item"]');
      expect(todoItem.classes()).toContain('completed');
    });

    test('应该能够删除待办事项', async () => {
      const wrapper = mount(TodoList);
      const input = wrapper.find('[data-testid="new-todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(1);

      const deleteButton = wrapper.findAll('button')[2]; // 第三个按钮是删除按钮
      await deleteButton.trigger('click');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(0);
    });
  });
});
```

### 3. 异步组件测试

```vue
<!-- components/UserProfile.vue -->
<template>
  <div class="user-profile" data-testid="user-profile">
    <div v-if="loading" data-testid="loading">加载中...</div>
    <div v-else-if="error" data-testid="error">{{ error }}</div>
    <div v-else-if="user" data-testid="user-info">
      <h2>{{ user.name }}</h2>
      <p>邮箱: {{ user.email }}</p>
      <p>电话: {{ user.phone }}</p>
    </div>
    <div v-else data-testid="not-found">用户不存在</div>
  </div>
</template>

<script>
export default {
  name: 'UserProfile',
  props: {
    userId: {
      type: String,
      required: true
    }
  },
  data() {
    return {
      loading: false,
      user: null,
      error: null
    };
  },
  watch: {
    userId: {
      immediate: true,
      handler(newUserId) {
        this.fetchUser(newUserId);
      }
    }
  },
  methods: {
    async fetchUser(userId) {
      this.loading = true;
      this.error = null;

      try {
        // 模拟 API 调用
        const response = await this.$http.get(`/api/users/${userId}`);
        this.user = response.data;
      } catch (error) {
        this.error = '加载用户信息失败';
      } finally {
        this.loading = false;
      }
    }
  }
};
</script>
```

```javascript
// components/__tests__/UserProfile.spec.js
import { mount } from '@vue/test-utils';
import UserProfile from '../UserProfile';

// Mock HTTP 客户端
const mockHttp = {
  get: jest.fn()
};

describe('UserProfile 组件', () => {
  let wrapper;

  beforeEach(() => {
    mockHttp.get.mockClear();
  });

  const createWrapper = (userId) => {
    return mount(UserProfile, {
      propsData: { userId },
      mocks: {
        $http: mockHttp
      }
    });
  };

  describe('加载状态', () => {
    test('应该显示加载状态', async () => {
      mockHttp.get.mockImplementation(() => new Promise(() => {}));

      wrapper = createWrapper('1');
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="loading"]').exists()).toBe(true);
    });

    test('加载时不应该显示用户信息', async () => {
      mockHttp.get.mockImplementation(() => new Promise(() => {}));

      wrapper = createWrapper('1');
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="user-info"]').exists()).toBe(false);
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

      mockHttp.get.mockResolvedValue({ data: mockUser });

      wrapper = createWrapper('1');
      await wrapper.vm.$nextTick();

      // 等待异步操作完成
      await new Promise(resolve => setTimeout(resolve, 0));

      expect(wrapper.find('[data-testid="user-info"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="user-info"]').text()).toContain('张三');
    });

    test('应该调用正确的 API', async () => {
      mockHttp.get.mockResolvedValue({ data: { id: '1', name: '张三' } });

      wrapper = createWrapper('1');

      await new Promise(resolve => setTimeout(resolve, 0));

      expect(mockHttp.get).toHaveBeenCalledTimes(1);
      expect(mockHttp.get).toHaveBeenCalledWith('/api/users/1');
    });
  });

  describe('错误处理', () => {
    test('应该显示错误信息', async () => {
      mockHttp.get.mockRejectedValue(new Error('Network error'));

      wrapper = createWrapper('1');
      await wrapper.vm.$nextTick();

      // 等待异步操作完成
      await new Promise(resolve => setTimeout(resolve, 0));

      expect(wrapper.find('[data-testid="error"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="error"]').text()).toContain('加载用户信息失败');
    });

    test('用户不存在时应该显示提示', async () => {
      mockHttp.get.mockResolvedValue({ data: null });

      wrapper = createWrapper('1');
      await wrapper.vm.$nextTick();

      // 等待异步操作完成
      await new Promise(resolve => setTimeout(resolve, 0));

      expect(wrapper.find('[data-testid="not-found"]').exists()).toBe(true);
    });
  });

  describe('重新加载', () => {
    test('userId 变化时应该重新加载', async () => {
      mockHttp.get
        .mockResolvedValueOnce({ data: { id: '1', name: '张三' } })
        .mockResolvedValueOnce({ data: { id: '2', name: '李四' } });

      wrapper = createWrapper('1');

      await new Promise(resolve => setTimeout(resolve, 0));
      expect(wrapper.text()).toContain('张三');

      await wrapper.setProps({ userId: '2' });

      await new Promise(resolve => setTimeout(resolve, 0));
      expect(wrapper.text()).toContain('李四');

      expect(mockHttp.get).toHaveBeenCalledTimes(2);
    });
  });
});
```

### 4. Vuex 集成测试

```javascript
// store/user.js
export default {
  namespaced: true,
  state: {
    users: [],
    currentUser: null,
    loading: false
  },
  mutations: {
    SET_USERS(state, users) {
      state.users = users;
    },
    SET_CURRENT_USER(state, user) {
      state.currentUser = user;
    },
    SET_LOADING(state, loading) {
      state.loading = loading;
    }
  },
  actions: {
    async fetchUsers({ commit }) {
      commit('SET_LOADING', true);
      try {
        const response = await fetch('/api/users');
        const users = await response.json();
        commit('SET_USERS', users);
      } catch (error) {
        console.error('获取用户列表失败:', error);
      } finally {
        commit('SET_LOADING', false);
      }
    },
    async fetchUserById({ commit }, userId) {
      commit('SET_LOADING', true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const user = await response.json();
        commit('SET_CURRENT_USER', user);
      } catch (error) {
        console.error('获取用户信息失败:', error);
      } finally {
        commit('SET_LOADING', false);
      }
    }
  },
  getters: {
    userById: (state) => (id) => {
      return state.users.find(user => user.id === id);
    },
    activeUsers: (state) => {
      return state.users.filter(user => user.isActive);
    }
  }
};
```

```javascript
// store/__tests__/user.spec.js
import user from '../user';

describe('User Store', () => {
  let store;

  beforeEach(() => {
    store = {
      state: {
        users: [],
        currentUser: null,
        loading: false
      },
      getters: {}
    };

    // 创建 getters
    Object.keys(user.getters).forEach(getterName => {
      store.getters[getterName] = user.getters[getterName](store.state, store.getters, store.state, rootGetters => ({}));
    });
  });

  describe('Mutations', () => {
    test('SET_USERS 应该设置用户列表', () => {
      const mockUsers = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      user.mutations.SET_USERS(store.state, mockUsers);

      expect(store.state.users).toEqual(mockUsers);
    });

    test('SET_CURRENT_USER 应该设置当前用户', () => {
      const mockUser = { id: 1, name: '张三' };

      user.mutations.SET_CURRENT_USER(store.state, mockUser);

      expect(store.state.currentUser).toEqual(mockUser);
    });

    test('SET_LOADING 应该设置加载状态', () => {
      user.mutations.SET_LOADING(store.state, true);

      expect(store.state.loading).toBe(true);
    });
  });

  describe('Actions', () => {
    // Mock fetch
    global.fetch = jest.fn();

    afterEach(() => {
      fetch.mockClear();
    });

    test('fetchUsers 应该获取用户列表', async () => {
      const mockUsers = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      fetch.mockResolvedValueOnce({
        ok: true,
        json: async () => mockUsers
      });

      const commit = jest.fn();
      await user.actions.fetchUsers({ commit });

      expect(commit).toHaveBeenCalledWith('SET_LOADING', true);
      expect(commit).toHaveBeenCalledWith('SET_USERS', mockUsers);
      expect(commit).toHaveBeenCalledWith('SET_LOADING', false);
    });

    test('fetchUsers 应该处理错误', async () => {
      fetch.mockRejectedValueOnce(new Error('Network error'));

      const commit = jest.fn();
      const consoleSpy = jest.spyOn(console, 'error').mockImplementation(() => {});

      await user.actions.fetchUsers({ commit });

      expect(commit).toHaveBeenCalledWith('SET_LOADING', true);
      expect(commit).toHaveBeenCalledWith('SET_LOADING', false);
      expect(consoleSpy).toHaveBeenCalled();

      consoleSpy.mockRestore();
    });
  });

  describe('Getters', () => {
    test('userById 应该返回正确的用户', () => {
      store.state.users = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      const user = user.getters.userById(store.state)(1);

      expect(user).toEqual({ id: 1, name: '张三' });
    });

    test('activeUsers 应该返回活跃用户', () => {
      store.state.users = [
        { id: 1, name: '张三', isActive: true },
        { id: 2, name: '李四', isActive: false },
        { id: 3, name: '王五', isActive: true }
      ];

      const activeUsers = user.getters.activeUsers(store.state);

      expect(activeUsers).toHaveLength(2);
      expect(activeUsers.every(user => user.isActive)).toBe(true);
    });
  });
});
```

---

## 注意事项

### 1. 选择正确的渲染方法

```javascript
// ✅ 使用 shallowMount 测试组件行为
test('应该触发 click 事件', () => {
  const wrapper = shallowMount(Button);
  wrapper.trigger('click');
  expect(wrapper.emitted('click')).toBeTruthy();
});

// ✅ 使用 mount 测试组件集成
test('应该正确渲染子组件', () => {
  const wrapper = mount(ParentComponent);
  expect(wrapper.find(ChildComponent).exists()).toBe(true);
});

// ❌ 不必要地使用 mount
test('应该触发 click 事件', () => {
  const wrapper = mount(Button); // 不必要渲染所有子组件
  wrapper.trigger('click');
});
```

### 2. 正确处理异步操作

```javascript
// ✅ 正确的异步测试
test('应该异步获取数据', async () => {
  const wrapper = mount(Component);
  await wrapper.vm.fetchData();
  expect(wrapper.vm.data).toBeDefined();
});

// ✅ 使用 waitFor
test('应该等待 DOM 更新', async () => {
  const wrapper = mount(Component);
  wrapper.setData({ loading: true });
  await wrapper.vm.$nextTick();
  expect(wrapper.find('.loading').exists()).toBe(true);
});

// ❌ 没有等待异步完成
test('应该异步获取数据', () => {
  const wrapper = mount(Component);
  wrapper.vm.fetchData(); // 没有等待
  expect(wrapper.vm.data).toBeDefined(); // 可能失败
});
```

### 3. 避免测试实现细节

```javascript
// ✅ 测试用户行为
test('点击按钮应该显示消息', async () => {
  const wrapper = mount(Button);
  await wrapper.find('button').trigger('click');
  expect(wrapper.text()).toContain('消息已显示');
});

// ❌ 测试实现细节
test('应该调用 showMessage 方法', () => {
  const showMessage = jest.fn();
  const wrapper = mount(Button, {
    methods: { showMessage }
  });
  wrapper.vm.showMessage();
  expect(showMessage).toHaveBeenCalled();
});
```

### 4. 正确使用选择器

```javascript
// ✅ 使用语义化选择器
expect(wrapper.find('button').exists()).toBe(true);
expect(wrapper.find('[data-testid="submit-button"]').exists()).toBe(true);
expect(wrapper.find('.submit-button').exists()).toBe(true);

// ⚠️ 谨慎使用类名选择器（可能因样式改变而失效）
expect(wrapper.find('.btn-primary').exists()).toBe(true);

// ❌ 避免使用复杂的选择器
expect(wrapper.find('div > div > button').exists()).toBe(true);
```

---

## 总结

Vue 2 单元测试是保证应用质量的重要手段：

### 核心要点

1. **Vue Test Utils**：官方提供的测试工具库
2. **渲染方法**：mount 和 shallowMount 的正确使用
3. **测试策略**：单元测试、集成测试、快照测试
4. **异步处理**：正确处理异步操作和 DOM 更新

### 最佳实践

- 优先测试用户行为而非实现细节
- 使用语义化选择器
- 正确处理异步操作
- 保持测试简洁和可维护
- 适当的 mock 和 spy

### 常见场景

- 组件渲染测试
- 事件处理测试
- 表单验证测试
- 异步操作测试
- Vuex 状态管理测试

通过掌握 Vue 2 单元测试，你可以构建更可靠、更易维护的 Vue 2 应用。记住测试不仅仅是验证功能，更是设计和文档的重要组成部分。