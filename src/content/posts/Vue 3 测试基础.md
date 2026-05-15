---
title: Vue 3 测试基础
published: 2024-03-20
description: 'Vitest 测试框架的详细介绍和学习笔记'
image: ''
tags: ["Vue3","测试"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 单元测试是确保 Vue 3 应用质量和稳定性的重要手段。通过测试，我们可以：

- 验证组件行为和渲染结果
- 防止代码变更引入回归错误
- 提供组件使用文档
- 支持安全重构
- 提升代码质量

Vue 3 测试主要使用 Vitest 作为测试框架，Vue Test Utils Next 作为组件测试工具。

### Vitest vs Jest

Vitest 是专为 Vite 构建的测试框架，相比 Jest 有以下优势：

- 更快的启动速度（基于 Vite 的即时编译）
- 原生支持 ESM
- 与 Vite 共享配置
- 更好的 TypeScript 支持
- 兼容 Jest API

---

## 核心概念

### Vue Test Utils Next 核心概念

1. **mount() vs shallowMount()**：
   - `mount()`: 完全渲染组件及其子组件
   - `shallowMount()`: 浅渲染，不渲染子组件

2. **Wrapper API**：包装组件实例的测试工具

3. **Composition API 测试**：测试组合式函数和组件

4. **异步测试**：处理异步操作和生命周期

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

---

## 基本用法

### 1. 环境配置

```bash
# 安装依赖
npm install --save-dev vitest @vue/test-utils @vitejs/plugin-vue jsdom @testing-library/jest-dom
npm install --save-dev @vitest/ui @vitest/coverage-v8
```

```javascript
// vitest.config.js
import { defineConfig } from 'vitest/config';
import vue from '@vitejs/plugin-vue';
import { fileURLToPath } from 'node:url';

export default defineConfig({
  plugins: [vue()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: ['./tests/setup.js'],
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'tests/',
        '**/*.d.ts',
        '**/*.config.*',
        '**/mockData',
        'dist/',
      ]
    }
  },
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url))
    }
  }
});
```

```javascript
// tests/setup.js
import { vi } from 'vitest';
import '@testing-library/jest-dom';

// 全局 mock
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn()
  }))
});
```

```javascript
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:run": "vitest run"
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

<script setup>
import { computed } from 'vue';

const props = defineProps({
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
});

const emit = defineEmits(['click']);

const variantClass = computed(() => `btn-${props.variant}`);

const handleClick = (event) => {
  if (!props.disabled) {
    emit('click', event);
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
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import Button from '../Button.vue';

describe('Button 组件', () => {
  describe('基础渲染', () => {
    it('应该渲染按钮文本', () => {
      const wrapper = mount(Button, {
        props: {
          text: '点击我'
        }
      });

      expect(wrapper.text()).toContain('点击我');
    });

    it('应该渲染默认文本', () => {
      const wrapper = mount(Button);

      expect(wrapper.text()).toBe('');
    });

    it('应该渲染 slot 内容', () => {
      const wrapper = mount(Button, {
        slots: {
          default: '<span>Slot 内容</span>'
        }
      });

      expect(wrapper.html()).toContain('Slot 内容');
    });

    it('应该渲染按钮元素', () => {
      const wrapper = mount(Button);

      expect(wrapper.find('button').exists()).toBe(true);
    });
  });

  describe('属性测试', () => {
    it('应该应用正确的 variant 类名', () => {
      const wrapper = mount(Button, {
        props: {
          variant: 'primary'
        }
      });

      expect(wrapper.find('.btn-primary').exists()).toBe(true);
    });

    it('应该应用 secondary variant', () => {
      const wrapper = mount(Button, {
        props: {
          variant: 'secondary'
        }
      });

      expect(wrapper.find('.btn-secondary').exists()).toBe(true);
    });

    it('应该应用 danger variant', () => {
      const wrapper = mount(Button, {
        props: {
          variant: 'danger'
        }
      });

      expect(wrapper.find('.btn-danger').exists()).toBe(true);
    });

    it('disabled 属性应该正确设置', () => {
      const wrapper = mount(Button, {
        props: {
          disabled: true
        }
      });

      const button = wrapper.find('button');
      expect(button.attributes('disabled')).toBe('');
    });
  });

  describe('事件测试', () => {
    it('应该触发 click 事件', async () => {
      const wrapper = mount(Button);

      await wrapper.find('button').trigger('click');

      expect(wrapper.emitted('click')).toBeTruthy();
      expect(wrapper.emitted('click').length).toBe(1);
    });

    it('应该传递事件对象', async () => {
      const wrapper = mount(Button);

      await wrapper.find('button').trigger('click');

      const clickEvent = wrapper.emitted('click')[0][0];
      expect(clickEvent).toBeDefined();
      expect(clickEvent.type).toBe('click');
    });

    it('disabled 时不应该触发 click 事件', async () => {
      const wrapper = mount(Button, {
        props: {
          disabled: true
        }
      });

      await wrapper.find('button').trigger('click');

      expect(wrapper.emitted('click')).toBeFalsy();
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

<script setup>
import { computed, ref } from 'vue';

const props = defineProps({
  modelValue: {
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
});

const emit = defineEmits(['update:modelValue', 'input', 'blur']);

const id = ref(`input-${Math.random().toString(36).substr(2, 9)}`);

const internalValue = computed({
  get: () => props.modelValue,
  set: (value) => {
    emit('update:modelValue', value);
  }
});

const handleInput = (event) => {
  emit('input', event.target.value);
};

const handleBlur = (event) => {
  emit('blur', event);
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
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import FormInput from '../FormInput.vue';

describe('FormInput 组件', () => {
  describe('基础渲染', () => {
    it('应该渲染 label', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名'
        }
      });

      expect(wrapper.text()).toContain('用户名');
    });

    it('应该渲染 input 元素', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名'
        }
      });

      expect(wrapper.find('input').exists()).toBe(true);
    });

    it('应该设置正确的 input 类型', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '邮箱',
          type: 'email'
        }
      });

      expect(wrapper.find('input').attributes('type')).toBe('email');
    });

    it('应该设置 placeholder', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          placeholder: '请输入用户名'
        }
      });

      expect(wrapper.find('input').attributes('placeholder')).toBe('请输入用户名');
    });
  });

  describe('v-model 测试', () => {
    it('应该初始化输入值', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          modelValue: '张三'
        }
      });

      expect(wrapper.find('input').element.value).toBe('张三');
    });

    it('应该触发 input 事件', async () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名'
        }
      });

      const input = wrapper.find('input');
      await input.setValue('李四');

      expect(wrapper.emitted('input')).toBeTruthy();
      expect(wrapper.emitted('input')[0]).toEqual(['李四']);
    });

    it('应该触发 update:modelValue 事件', async () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名'
        }
      });

      const input = wrapper.find('input');
      await input.setValue('新值');

      expect(wrapper.emitted('update:modelValue')).toBeTruthy();
      expect(wrapper.emitted('update:modelValue')[0]).toEqual(['新值']);
    });

    it('应该双向绑定', async () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          modelValue: '初始值'
        }
      });

      const input = wrapper.find('input');
      await input.setValue('新值');

      expect(wrapper.vm.internalValue).toBe('新值');
    });
  });

  describe('错误处理', () => {
    it('应该显示错误信息', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          error: '用户名不能为空'
        }
      });

      expect(wrapper.find('.error-message').exists()).toBe(true);
      expect(wrapper.find('.error-message').text()).toBe('用户名不能为空');
    });

    it('没有错误时不应该显示错误信息', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          error: ''
        }
      });

      expect(wrapper.find('.error-message').exists()).toBe(false);
    });
  });

  describe('禁用状态', () => {
    it('应该设置 disabled 属性', () => {
      const wrapper = mount(FormInput, {
        props: {
          label: '用户名',
          disabled: true
        }
      });

      expect(wrapper.find('input').attributes('disabled')).toBe('');
    });

    it('禁用时不应该触发 input 事件', async () => {
      const wrapper = mount(FormInput, {
        props: {
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
    it('应该触发 blur 事件', async () => {
      const wrapper = mount(FormInput, {
        props: {
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

### 1. 组合式函数测试

```javascript
// composables/useCounter.js
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  const doubleCount = computed(() => count.value * 2);

  const increment = () => {
    count.value++;
  };

  const decrement = () => {
    count.value--;
  };

  const reset = () => {
    count.value = initialValue;
  };

  const setValue = (value) => {
    count.value = value;
  };

  return {
    count,
    doubleCount,
    increment,
    decrement,
    reset,
    setValue
  };
}
```

```javascript
// composables/__tests__/useCounter.spec.js
import { describe, it, expect } from 'vitest';
import { useCounter } from '../useCounter';

describe('useCounter', () => {
  it('应该初始化计数器', () => {
    const { count } = useCounter();

    expect(count.value).toBe(0);
  });

  it('应该使用自定义初始值', () => {
    const { count } = useCounter(10);

    expect(count.value).toBe(10);
  });

  it('应该增加计数', () => {
    const { count, increment } = useCounter();

    increment();
    increment();

    expect(count.value).toBe(2);
  });

  it('应该减少计数', () => {
    const { count, decrement } = useCounter(5);

    decrement();
    decrement();

    expect(count.value).toBe(3);
  });

  it('应该重置计数', () => {
    const { count, increment, reset } = useCounter();

    increment();
    increment();
    reset();

    expect(count.value).toBe(0);
  });

  it('应该重置到初始值', () => {
    const { count, increment, reset } = useCounter(10);

    increment();
    increment();
    reset();

    expect(count.value).toBe(10);
  });

  it('应该设置计数器值', () => {
    const { count, setValue } = useCounter();

    setValue(42);

    expect(count.value).toBe(42);
  });

  it('应该正确计算双倍值', () => {
    const { count, doubleCount, increment } = useCounter();

    expect(doubleCount.value).toBe(0);

    increment();
    expect(doubleCount.value).toBe(2);

    increment();
    expect(doubleCount.value).toBe(4);
  });
});
```

### 2. 异步组合式函数测试

```javascript
// composables/useFetch.js
import { ref } from 'vue';

export function useFetch(url) {
  const data = ref(null);
  const error = ref(null);
  const loading = ref(false);

  const fetchData = async () => {
    loading.value = true;
    error.value = null;

    try {
      const response = await fetch(url);
      const json = await response.json();
      data.value = json;
    } catch (err) {
      error.value = err.message;
    } finally {
      loading.value = false;
    }
  };

  return {
    data,
    error,
    loading,
    fetchData
  };
}
```

```javascript
// composables/__tests__/useFetch.spec.js
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { useFetch } from '../useFetch';

// Mock fetch
global.fetch = vi.fn();

describe('useFetch', () => {
  beforeEach(() => {
    fetch.mockClear();
  });

  it('应该初始化状态', () => {
    const { data, error, loading } = useFetch('/api/data');

    expect(data.value).toBeNull();
    expect(error.value).toBeNull();
    expect(loading.value).toBe(false);
  });

  it('应该成功获取数据', async () => {
    const mockData = { id: 1, name: '测试数据' };
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => mockData
    });

    const { data, loading, fetchData } = useFetch('/api/data');

    fetchData();

    expect(loading.value).toBe(true);

    // 等待异步操作完成
    await new Promise(resolve => setTimeout(resolve, 0));

    expect(loading.value).toBe(false);
    expect(data.value).toEqual(mockData);
    expect(error.value).toBeNull();
  });

  it('应该处理错误', async () => {
    const errorMessage = 'Network error';
    fetch.mockRejectedValueOnce(new Error(errorMessage));

    const { data, error, loading, fetchData } = useFetch('/api/data');

    fetchData();

    expect(loading.value).toBe(true);

    await new Promise(resolve => setTimeout(resolve, 0));

    expect(loading.value).toBe(false);
    expect(data.value).toBeNull();
    expect(error.value).toBe(errorMessage);
  });

  it('应该调用正确的 API', async () => {
    fetch.mockResolvedValueOnce({
      ok: true,
      json: async () => ({})
    });

    const { fetchData } = useFetch('/api/data');

    fetchData();

    await new Promise(resolve => setTimeout(resolve, 0));

    expect(fetch).toHaveBeenCalledTimes(1);
    expect(fetch).toHaveBeenCalledWith('/api/data');
  });
});
```

### 3. 列表组件测试

```vue
<!-- components/UserList.vue -->
<template>
  <div class="user-list" data-testid="user-list">
    <div v-if="loading" data-testid="loading">加载中...</div>
    <div v-else-if="error" data-testid="error">{{ error }}</div>
    <div v-else-if="users.length === 0" data-testid="empty">暂无用户</div>
    <ul v-else>
      <li
        v-for="user in users"
        :key="user.id"
        :class="{ active: user.isActive }"
        data-testid="user-item"
      >
        <span data-testid="user-name">{{ user.name }}</span>
        <span data-testid="user-email">{{ user.email }}</span>
        <button @click="handleEdit(user)" data-testid="edit-button">编辑</button>
        <button @click="handleDelete(user.id)" data-testid="delete-button">删除</button>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const props = defineProps({
  fetchUsers: {
    type: Function,
    required: true
  }
});

const emit = defineEmits(['edit', 'delete']);

const users = ref([]);
const loading = ref(false);
const error = ref(null);

const loadUsers = async () => {
  loading.value = true;
  error.value = null;

  try {
    users.value = await props.fetchUsers();
  } catch (err) {
    error.value = '加载用户列表失败';
  } finally {
    loading.value = false;
  }
};

const handleEdit = (user) => {
  emit('edit', user);
};

const handleDelete = (userId) => {
  emit('delete', userId);
};

onMounted(() => {
  loadUsers();
});
</script>

<style scoped>
.user-list {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 10px;
  margin: 8px 0;
  background-color: #f5f5f5;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

li.active {
  background-color: #d4edda;
}

button {
  margin-left: 8px;
}
</style>
```

```javascript
// components/__tests__/UserList.spec.js
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { mount } from '@vue/test-utils';
import UserList from '../UserList.vue';

describe('UserList 组件', () => {
  const mockUsers = [
    { id: 1, name: '张三', email: 'zhangsan@example.com', isActive: true },
    { id: 2, name: '李四', email: 'lisi@example.com', isActive: false }
  ];

  const mockFetchUsers = vi.fn();
  const mockEdit = vi.fn();
  const mockDelete = vi.fn();

  const createWrapper = (fetchUsers = mockFetchUsers) => {
    return mount(UserList, {
      props: {
        fetchUsers
      },
      attrs: {
        onEdit: mockEdit,
        onDelete: mockDelete
      }
    });
  };

  beforeEach(() => {
    mockFetchUsers.mockClear();
    mockEdit.mockClear();
    mockDelete.mockClear();
  });

  describe('加载状态', () => {
    it('应该显示加载状态', async () => {
      mockFetchUsers.mockImplementation(() => new Promise(() => {}));

      const wrapper = createWrapper();
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="loading"]').exists()).toBe(true);
    });

    it('加载时不应该渲染用户列表', async () => {
      mockFetchUsers.mockImplementation(() => new Promise(() => {}));

      const wrapper = createWrapper();
      await wrapper.vm.$nextTick();

      expect(wrapper.find('ul').exists()).toBe(false);
    });
  });

  describe('成功加载', () => {
    it('应该渲染用户列表', async () => {
      mockFetchUsers.mockResolvedValue(mockUsers);

      const wrapper = createWrapper();

      // 等待异步操作完成
      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      const userItems = wrapper.findAll('[data-testid="user-item"]');
      expect(userItems).toHaveLength(2);
      expect(wrapper.text()).toContain('张三');
      expect(wrapper.text()).toContain('李四');
    });

    it('应该调用 fetchUsers 函数', async () => {
      mockFetchUsers.mockResolvedValue(mockUsers);

      createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));

      expect(mockFetchUsers).toHaveBeenCalledTimes(1);
    });
  });

  describe('错误处理', () => {
    it('应该显示错误信息', async () => {
      mockFetchUsers.mockRejectedValue(new Error('Network error'));

      const wrapper = createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="error"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="error"]').text()).toContain('加载用户列表失败');
    });

    it('空列表时应该显示空状态', async () => {
      mockFetchUsers.mockResolvedValue([]);

      const wrapper = createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="empty"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="empty"]').text()).toContain('暂无用户');
    });
  });

  describe('用户操作', () => {
    it('应该触发编辑事件', async () => {
      mockFetchUsers.mockResolvedValue(mockUsers);

      const wrapper = createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      const editButton = wrapper.findAll('[data-testid="edit-button"]')[0];
      await editButton.trigger('click');

      expect(mockEdit).toHaveBeenCalledWith(mockUsers[0]);
    });

    it('应该触发删除事件', async () => {
      mockFetchUsers.mockResolvedValue(mockUsers);

      const wrapper = createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      const deleteButton = wrapper.findAll('[data-testid="delete-button"]')[0];
      await deleteButton.trigger('click');

      expect(mockDelete).toHaveBeenCalledWith(mockUsers[0].id);
    });
  });

  describe('样式和状态', () => {
    it('应该应用 active 类名', async () => {
      mockFetchUsers.mockResolvedValue(mockUsers);

      const wrapper = createWrapper();

      await new Promise(resolve => setTimeout(resolve, 0));
      await wrapper.vm.$nextTick();

      const userItems = wrapper.findAll('[data-testid="user-item"]');
      expect(userItems[0].classes()).toContain('active');
      expect(userItems[1].classes()).not.toContain('active');
    });
  });
});
```

### 4. Pinia Store 测试

```javascript
// stores/user.js
import { defineStore } from 'pinia';

export const useUserStore = defineStore('user', {
  state: () => ({
    users: [],
    currentUser: null,
    loading: false,
    error: null
  }),

  getters: {
    userById: (state) => (id) => {
      return state.users.find(user => user.id === id);
    },
    activeUsers: (state) => {
      return state.users.filter(user => user.isActive);
    },
    userCount: (state) => state.users.length
  },

  actions: {
    async fetchUsers() {
      this.loading = true;
      this.error = null;

      try {
        const response = await fetch('/api/users');
        const users = await response.json();
        this.users = users;
      } catch (error) {
        this.error = '获取用户列表失败';
        throw error;
      } finally {
        this.loading = false;
      }
    },

    async fetchUserById(userId) {
      this.loading = true;
      this.error = null;

      try {
        const response = await fetch(`/api/users/${userId}`);
        const user = await response.json();
        this.currentUser = user;
      } catch (error) {
        this.error = '获取用户信息失败';
        throw error;
      } finally {
        this.loading = false;
      }
    },

    addUser(user) {
      this.users.push(user);
    },

    updateUser(userId, updates) {
      const index = this.users.findIndex(user => user.id === userId);
      if (index !== -1) {
        this.users[index] = { ...this.users[index], ...updates };
      }
    },

    deleteUser(userId) {
      this.users = this.users.filter(user => user.id !== userId);
    },

    clearError() {
      this.error = null;
    }
  }
});
```

```javascript
// stores/__tests__/user.spec.js
import { describe, it, expect, vi, beforeEach, setActivePinia, createPinia } from 'vitest';
import { useUserStore } from '../user';

describe('User Store', () => {
  beforeEach(() => {
    setActivePinia(createPinia());
    global.fetch = vi.fn();
  });

  describe('State', () => {
    it('应该初始化空状态', () => {
      const store = useUserStore();

      expect(store.users).toEqual([]);
      expect(store.currentUser).toBeNull();
      expect(store.loading).toBe(false);
      expect(store.error).toBeNull();
    });
  });

  describe('Getters', () => {
    it('userById 应该返回正确的用户', () => {
      const store = useUserStore();
      store.users = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      const user = store.userById(1);

      expect(user).toEqual({ id: 1, name: '张三' });
    });

    it('activeUsers 应该返回活跃用户', () => {
      const store = useUserStore();
      store.users = [
        { id: 1, name: '张三', isActive: true },
        { id: 2, name: '李四', isActive: false },
        { id: 3, name: '王五', isActive: true }
      ];

      const activeUsers = store.activeUsers;

      expect(activeUsers).toHaveLength(2);
      expect(activeUsers.every(user => user.isActive)).toBe(true);
    });

    it('userCount 应该返回用户数量', () => {
      const store = useUserStore();
      store.users = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' },
        { id: 3, name: '王五' }
      ];

      expect(store.userCount).toBe(3);
    });
  });

  describe('Actions', () => {
    it('fetchUsers 应该获取用户列表', async () => {
      const mockUsers = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      global.fetch.mockResolvedValueOnce({
        ok: true,
        json: async () => mockUsers
      });

      const store = useUserStore();
      await store.fetchUsers();

      expect(store.users).toEqual(mockUsers);
      expect(store.loading).toBe(false);
      expect(store.error).toBeNull();
    });

    it('fetchUsers 应该处理错误', async () => {
      global.fetch.mockRejectedValueOnce(new Error('Network error'));

      const store = useUserStore();
      await expect(store.fetchUsers()).rejects.toThrow('Network error');

      expect(store.error).toBe('获取用户列表失败');
      expect(store.loading).toBe(false);
    });

    it('addUser 应该添加用户', () => {
      const store = useUserStore();
      const newUser = { id: 1, name: '新用户' };

      store.addUser(newUser);

      expect(store.users).toContain(newUser);
      expect(store.users).toHaveLength(1);
    });

    it('updateUser 应该更新用户', () => {
      const store = useUserStore();
      store.users = [
        { id: 1, name: '张三', email: 'zhangsan@example.com' }
      ];

      store.updateUser(1, { name: '张三更新', email: 'new@example.com' });

      expect(store.users[0]).toEqual({
        id: 1,
        name: '张三更新',
        email: 'new@example.com'
      });
    });

    it('deleteUser 应该删除用户', () => {
      const store = useUserStore();
      store.users = [
        { id: 1, name: '张三' },
        { id: 2, name: '李四' }
      ];

      store.deleteUser(1);

      expect(store.users).toHaveLength(1);
      expect(store.users[0].id).toBe(2);
    });

    it('clearError 应该清除错误', () => {
      const store = useUserStore();
      store.error = '测试错误';

      store.clearError();

      expect(store.error).toBeNull();
    });
  });
});
```

### 5. 组件集成测试

```vue
<!-- components/TodoApp.vue -->
<template>
  <div class="todo-app" data-testid="todo-app">
    <h1>待办事项</h1>

    <form @submit.prevent="addTodo" data-testid="todo-form">
      <input
        v-model="newTodo"
        placeholder="输入新待办事项"
        data-testid="todo-input"
      />
      <button type="submit" data-testid="add-button">添加</button>
    </form>

    <div data-testid="todo-stats">
      <span>总计: {{ todos.length }}</span>
      <span>已完成: {{ completedCount }}</span>
      <span>未完成: {{ incompleteCount }}</span>
    </div>

    <ul data-testid="todo-list">
      <li
        v-for="todo in filteredTodos"
        :key="todo.id"
        :class="{ completed: todo.completed }"
        data-testid="todo-item"
      >
        <input
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo.id)"
          data-testid="todo-checkbox"
        />
        <span>{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)" data-testid="remove-button">删除</button>
      </li>
    </ul>

    <div v-if="todos.length === 0" data-testid="empty-state">暂无待办事项</div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

const todos = ref([]);
const newTodo = ref('');
const filter = ref('all');

const filteredTodos = computed(() => {
  switch (filter.value) {
    case 'completed':
      return todos.value.filter(todo => todo.completed);
    case 'incomplete':
      return todos.value.filter(todo => !todo.completed);
    default:
      return todos.value;
  }
});

const completedCount = computed(() => todos.value.filter(todo => todo.completed).length);
const incompleteCount = computed(() => todos.value.filter(todo => !todo.completed).length);

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: Date.now(),
      text: newTodo.value.trim(),
      completed: false
    });
    newTodo.value = '';
  }
};

const toggleTodo = (id) => {
  const todo = todos.value.find(t => t.id === id);
  if (todo) {
    todo.completed = !todo.completed;
  }
};

const removeTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id);
};
</script>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

form {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

input {
  flex: 1;
  padding: 8px;
}

button {
  padding: 8px 16px;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 10px;
  margin: 8px 0;
  background-color: #f5f5f5;
  display: flex;
  align-items: center;
  gap: 10px;
}

li.completed {
  text-decoration: line-through;
  color: #999;
}

.todo-stats {
  margin-bottom: 20px;
  display: flex;
  gap: 20px;
}
</style>
```

```javascript
// components/__tests__/TodoApp.spec.js
import { describe, it, expect, vi } from 'vitest';
import { mount } from '@vue/test-utils';
import TodoApp from '../TodoApp.vue';

describe('TodoApp 组件', () => {
  describe('初始状态', () => {
    it('应该渲染空的应用', () => {
      const wrapper = mount(TodoApp);

      expect(wrapper.find('[data-testid="todo-app"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="todo-list"]').exists()).toBe(true);
      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(0);
    });

    it('应该显示空状态', () => {
      const wrapper = mount(TodoApp);

      expect(wrapper.find('[data-testid="empty-state"]').exists()).toBe(true);
      expect(wrapper.find('[data-testid="empty-state"]').text()).toContain('暂无待办事项');
    });

    it('应该显示正确的统计信息', () => {
      const wrapper = mount(TodoApp);

      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('总计: 0');
      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('已完成: 0');
      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('未完成: 0');
    });
  });

  describe('添加待办事项', () => {
    it('应该能够添加待办事项', async () => {
      const wrapper = mount(TodoApp);

      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('学习 Vue 3');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      const todoItems = wrapper.findAll('[data-testid="todo-item"]');
      expect(todoItems).toHaveLength(1);
      expect(todoItems[0].text()).toContain('学习 Vue 3');
    });

    it('应该能够通过表单提交添加', async () => {
      const wrapper = mount(TodoApp);

      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('学习测试');

      await wrapper.find('[data-testid="todo-form"]').trigger('submit');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(1);
    });

    it('不应该添加空待办事项', async () => {
      const wrapper = mount(TodoApp);

      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('   ');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(0);
    });

    it('添加后应该清空输入框', async () => {
      const wrapper = mount(TodoApp);

      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('新任务');

      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      expect(input.element.value).toBe('');
    });
  });

  describe('待办事项操作', () => {
    beforeEach(async () => {
      const wrapper = mount(TodoApp);
      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();
    });

    it('应该能够切换完成状态', async () => {
      const wrapper = mount(TodoApp);
      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      const checkbox = wrapper.find('[data-testid="todo-checkbox"]');
      await checkbox.setChecked(true);
      await wrapper.vm.$nextTick();

      const todoItem = wrapper.find('[data-testid="todo-item"]');
      expect(todoItem.classes()).toContain('completed');
      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('已完成: 1');
    });

    it('应该能够删除待办事项', async () => {
      const wrapper = mount(TodoApp);
      const input = wrapper.find('[data-testid="todo-input"]');
      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(1);

      const deleteButton = wrapper.find('[data-testid="remove-button"]');
      await deleteButton.trigger('click');
      await wrapper.vm.$nextTick();

      expect(wrapper.findAll('[data-testid="todo-item"]')).toHaveLength(0);
    });
  });

  describe('统计信息', () => {
    it('应该正确计算统计信息', async () => {
      const wrapper = mount(TodoApp);

      // 添加三个待办事项
      const input = wrapper.find('[data-testid="todo-input"]');

      await input.setValue('任务1');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      await input.setValue('任务2');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      await input.setValue('任务3');
      await wrapper.find('[data-testid="add-button"]').trigger('click');
      await wrapper.vm.$nextTick();

      // 完成一个任务
      const checkbox = wrapper.findAll('[data-testid="todo-checkbox"]')[0];
      await checkbox.setChecked(true);
      await wrapper.vm.$nextTick();

      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('总计: 3');
      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('已完成: 1');
      expect(wrapper.find('[data-testid="todo-stats"]').text()).toContain('未完成: 2');
    });
  });
});
```

---

## 注意事项

### 1. 测试 Composition API

```javascript
// ✅ 直接测试组合式函数
describe('useCounter', () => {
  it('应该增加计数', () => {
    const { count, increment } = useCounter();
    increment();
    expect(count.value).toBe(1);
  });
});

// ✅ 在组件中测试组合式函数
describe('Counter 组件', () => {
  it('应该使用 useCounter', () => {
    const wrapper = mount(Counter);
    expect(wrapper.text()).toContain('Count: 0');
  });
});
```

### 2. 正确处理响应式数据

```javascript
// ✅ 使用 nextTick 等待 DOM 更新
it('应该更新视图', async () => {
  const wrapper = mount(Component);
  wrapper.setData({ count: 1 });
  await wrapper.vm.$nextTick();
  expect(wrapper.text()).toContain('1');
});

// ✅ 使用 waitFor 等待条件
import { waitFor } from '@testing-library/vue';

it('应该等待异步更新', async () => {
  const wrapper = mount(Component);
  await waitFor(() => {
    expect(wrapper.text()).toContain('完成');
  });
});
```

### 3. 正确使用 Vitest 功能

```javascript
// ✅ 使用 describe 和 it 组织测试
describe('组件测试', () => {
  it('应该渲染组件', () => {
    // 测试代码
  });
});

// ✅ 使用 beforeEach 和 afterEach
describe('组件测试', () => {
  beforeEach(() => {
    // 准备工作
  });

  afterEach(() => {
    // 清理工作
  });
});

// ✅ 使用 describe 进行分组
describe('按钮组件', () => {
  describe('基础渲染', () => {
    // 渲染测试
  });

  describe('事件处理', () => {
    // 事件测试
  });
});
```

### 4. 避免测试实现细节

```javascript
// ✅ 测试用户行为
it('点击按钮应该显示消息', async () => {
  const wrapper = mount(Button);
  await wrapper.find('button').trigger('click');
  expect(wrapper.text()).toContain('消息已显示');
});

// ❌ 测试实现细节
it('应该调用 showMessage 方法', () => {
  const wrapper = mount(Button);
  wrapper.vm.showMessage();
  expect(wrapper.text()).toContain('消息已显示');
});
```

---

## 总结

Vue 3 单元测试是保证应用质量的重要手段：

### 核心要点

1. **Vitest**：专为 Vite 构建的测试框架
2. **Vue Test Utils Next**：Vue 3 官方测试工具
3. **Composition API**：测试组合式函数和组件
4. **响应式测试**：正确处理响应式数据和 DOM 更新

### 最佳实践

- 优先测试用户行为而非实现细节
- 使用语义化选择器和 data-testid
- 正确处理异步操作和生命周期
- 保持测试简洁和可维护
- 适当的 mock 和 spy

### 常见场景

- 组件渲染测试
- 事件处理测试
- 表单验证测试
- 异步操作测试
- Pinia 状态管理测试
- 组合式函数测试

### Vitest 优势

- 更快的启动速度
- 原生支持 ESM
- 与 Vite 共享配置
- 更好的 TypeScript 支持
- 兼容 Jest API

通过掌握 Vue 3 单元测试和 Vitest，你可以构建更可靠、更易维护的 Vue 3 应用。记住测试不仅仅是验证功能，更是设计和文档的重要组成部分。