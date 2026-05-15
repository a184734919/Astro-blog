---
title: Vue 3 快速上手
published: 2024-01-01
description: 'Vue 3 新特性和项目创建的详细介绍和学习笔记'
image: ''
tags: ["Vue3","基础"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 是 Vue.js 框架的全新版本，于 2020 年正式发布。相比 Vue 2，Vue 3 在性能、类型支持、组合式 API 等方面都有显著改进。它采用全新的响应式系统，大幅提升了应用的性能和开发体验。

## 核心概念

### 主要特性

- **组合式 API**: 更灵活的代码组织方式
- **更好的 TypeScript 支持**: 完整的类型推断
- **性能提升**: 更小的打包体积和更快的运行速度
- **Tree-shaking 支持**: 按需引入，减少打包体积
- **多个根节点**: 组件支持多个根元素
- **Teleport**: 将组件渲染到 DOM 的任意位置
- **Fragments**: 支持 Fragment 组件
- **Suspense**: 异步组件的改进支持

### 核心改变

- 使用 Proxy 替代 Object.defineProperty 实现响应式
- 全新的组合式 API
- 重新设计的虚拟 DOM 和编译器优化
- 全局 API 的模块化

## 基本用法

### 项目创建

#### 使用 Vite 创建项目

```bash
# 创建新的 Vue 3 项目
npm create vue@latest my-vue-app

# 进入项目目录
cd my-vue-app

# 安装依赖
npm install

# 启动开发服务器
npm run dev
```

#### 使用 Vue CLI 创建项目

```bash
# 如果需要使用 Vue CLI
npm install -g @vue/cli
vue create my-vue-app
```

### 项目结构

```text
my-vue-app/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   ├── components/
│   ├── App.vue
│   ├── main.js
│   └── ...
├── .gitignore
├── index.html
├── package.json
├── README.md
└── vite.config.js
```

### 入门示例

#### 基础组件

```vue
<template>
  <div class="counter">
    <h2>计数器示例</h2>
    <p>当前计数: {{ count }}</p>
    <button @click="increment">增加</button>
    <button @click="decrement">减少</button>
    <button @click="reset">重置</button>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue';

// 响应式状态
const count = ref(0);

// 方法
function increment() {
  count.value++;
}

function decrement() {
  count.value--;
}

function reset() {
  count.value = 0;
}
</script>

<style scoped>
.counter {
  text-align: center;
  padding: 20px;
}

button {
  margin: 0 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background: #45a049;
}
</style>
```

#### 响应式数据

```vue
<script setup>
import { ref, reactive, computed } from 'vue';

// ref 用于基本类型
const message = ref('Hello Vue 3');

// reactive 用于对象类型
const user = reactive({
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com'
});

// 计算属性
const userInfo = computed(() => {
  return `${user.name}，${user.age}岁`;
});

// 方法
function updateUser() {
  user.name = '李四';
  user.age = 30;
}

function resetUser() {
  user.name = '张三';
  user.age = 25;
}
</script>

<template>
  <div>
    <h2>{{ message }}</h2>
    <p>{{ userInfo }}</p>
    <p>Email: {{ user.email }}</p>
    <button @click="updateUser">更新用户</button>
    <button @click="resetUser">重置用户</button>
  </div>
</template>
```

### 列表渲染

```vue
<script setup>
import { ref } from 'vue';

const items = ref([
  { id: 1, name: '学习 Vue 3', completed: false },
  { id: 2, name: '编写代码', completed: true },
  { id: 3, name: '调试程序', completed: false }
]);

const newItem = ref('');

function addItem() {
  if (newItem.value.trim()) {
    items.value.push({
      id: Date.now(),
      name: newItem.value,
      completed: false
    });
    newItem.value = '';
  }
}

function removeItem(index) {
  items.value.splice(index, 1);
}

function toggleComplete(index) {
  items.value[index].completed = !items.value[index].completed;
}
</script>

<template>
  <div>
    <h2>待办事项</h2>
    <input 
      v-model="newItem" 
      @keyup.enter="addItem" 
      placeholder="添加新任务"
    >
    <button @click="addItem">添加</button>

    <ul>
      <li v-for="(item, index) in items" :key="item.id">
        <input 
          type="checkbox" 
          v-model="item.completed"
          @change="toggleComplete(index)"
        >
        <span :style="{ textDecoration: item.completed ? 'line-through' : 'none' }">
          {{ item.name }}
        </span>
        <button @click="removeItem(index)">删除</button>
      </li>
    </ul>
  </div>
</template>
```

### 条件渲染

```vue
<script setup>
import { ref } from 'vue';

const isLoggedIn = ref(false);
const userName = ref('');
const showDetails = ref(false);

function login() {
  isLoggedIn.value = true;
  userName.value = '用户';
}

function logout() {
  isLoggedIn.value = false;
  userName.value = '';
}
</script>

<template>
  <div>
    <!-- v-if 条件渲染 -->
    <div v-if="isLoggedIn">
      <p>欢迎回来，{{ userName }}！</p>
      <button @click="logout">退出登录</button>
      
      <!-- v-show 切换显示状态 -->
      <button @click="showDetails = !showDetails">
        {{ showDetails ? '隐藏' : '显示' }}详细信息
      </button>
      <div v-show="showDetails">
        <p>这是用户的详细信息</p>
      </div>
    </div>

    <div v-else>
      <p>请登录</p>
      <button @click="login">登录</button>
    </div>

    <!-- 多个条件 -->
    <div v-if="isLoggedIn && userName === '用户'">
      <p>管理员权限</p>
    </div>
  </div>
</template>
```

### 事件处理

```vue
<script setup>
import { ref } from 'vue';

const count = ref(0);
const inputText = ref('');
const selectedOption = ref('option1');

function handleClick(event) {
  console.log('点击事件', event);
  count.value++;
}

function handleInput(event) {
  console.log('输入值:', event.target.value);
}

function handleKeyup(event) {
  if (event.key === 'Enter') {
    console.log('回车键按下');
  }
}

function submitForm(event) {
  event.preventDefault(); // 阻止表单默认提交
  console.log('表单提交:', { inputText: inputText.value, selectedOption: selectedOption.value });
}
</script>

<template>
  <div>
    <h2>事件处理示例</h2>
    
    <p>计数: {{ count }}</p>
    <button @click="handleClick">点击我</button>
    <button @click.right="console.log('右键点击')">右键点击</button>
    
    <input 
      v-model="inputText" 
      @input="handleInput"
      @keyup="handleKeyup"
      placeholder="输入文本"
    >
    
    <form @submit="submitForm">
      <select v-model="selectedOption">
        <option value="option1">选项1</option>
        <option value="option2">选项2</option>
        <option value="option3">选项3</option>
      </select>
      <button type="submit">提交</button>
    </form>
  </div>
</template>
```

## 实际应用

### 表单验证

```vue
<script setup>
import { ref, computed } from 'vue';

const form = reactive({
  username: '',
  email: '',
  password: ''
});

const errors = reactive({
  username: '',
  email: '',
  password: ''
});

const isFormValid = computed(() => {
  return form.username && form.email && form.password &&
         !errors.username && !errors.email && !errors.password;
});

function validateUsername() {
  if (!form.username) {
    errors.username = '用户名不能为空';
  } else if (form.username.length < 3) {
    errors.username = '用户名至少3个字符';
  } else {
    errors.username = '';
  }
}

function validateEmail() {
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  if (!form.email) {
    errors.email = '邮箱不能为空';
  } else if (!emailRegex.test(form.email)) {
    errors.email = '邮箱格式不正确';
  } else {
    errors.email = '';
  }
}

function validatePassword() {
  if (!form.password) {
    errors.password = '密码不能为空';
  } else if (form.password.length < 6) {
    errors.password = '密码至少6个字符';
  } else {
    errors.password = '';
  }
}

function handleSubmit() {
  validateUsername();
  validateEmail();
  validatePassword();
  
  if (isFormValid.value) {
    console.log('表单提交成功:', form);
    // 这里可以添加提交逻辑
  }
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="login-form">
    <h2>用户注册</h2>
    
    <div>
      <label>用户名:</label>
      <input 
        v-model="form.username" 
        @blur="validateUsername"
        placeholder="输入用户名"
      >
      <span class="error">{{ errors.username }}</span>
    </div>
    
    <div>
      <label>邮箱:</label>
      <input 
        v-model="form.email" 
        @blur="validateEmail"
        placeholder="输入邮箱"
      >
      <span class="error">{{ errors.email }}</span>
    </div>
    
    <div>
      <label>密码:</label>
      <input 
        v-model="form.password" 
        type="password"
        @blur="validatePassword"
        placeholder="输入密码"
      >
      <span class="error">{{ errors.password }}</span>
    </div>
    
    <button type="submit" :disabled="!isFormValid">注册</button>
  </form>
</template>

<style scoped>
.login-form {
  max-width: 400px;
  margin: 0 auto;
  padding: 20px;
  border: 1px solid #ddd;
  border-radius: 8px;
}

.login-form div {
  margin-bottom: 15px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
}

input {
  width: 100%;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.error {
  color: red;
  font-size: 12px;
}

button {
  width: 100%;
  padding: 10px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:disabled {
  background: #cccccc;
  cursor: not-allowed;
}
</style>
```

### 组件通信

```vue
<!-- ParentComponent.vue -->
<script setup>
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const parentMessage = ref('来自父组件的消息');
const receivedData = ref('');

function handleChildEvent(data) {
  receivedData.value = data;
}
</script>

<template>
  <div>
    <h2>父组件</h2>
    <p>父组件消息: {{ parentMessage }}</p>
    <p>子组件返回: {{ receivedData }}</p>
    
    <ChildComponent 
      :message="parentMessage" 
      @child-event="handleChildEvent"
    />
  </div>
</template>

<!-- ChildComponent.vue -->
<script setup>
import { ref } from 'vue';

const props = defineProps({
  message: {
    type: String,
    required: true
  }
});

const emit = defineEmits(['child-event']);

const childData = ref('');

function sendToParent() {
  emit('child-event', childData.value);
}
</script>

<template>
  <div class="child">
    <h3>子组件</h3>
    <p>接收到的消息: {{ message }}</p>
    <input v-model="childData" placeholder="输入要发送给父组件的数据">
    <button @click="sendToParent">发送给父组件</button>
  </div>
</template>

<style scoped>
.child {
  border: 1px solid #ddd;
  padding: 15px;
  margin-top: 20px;
  border-radius: 8px;
}
</style>
```

## 注意事项

### 开发建议

1. **使用组合式 API**: 优先使用 `<script setup>` 语法糖
2. **类型安全**: 在 TypeScript 项目中充分利用类型推断
3. **性能优化**: 合理使用计算属性和侦听器
4. **组件设计**: 保持组件的单一职责原则
5. **状态管理**: 简单应用使用响应式 API，复杂应用考虑 Pinia

### 常见错误

1. **忘记 `.value`**: 在脚本中使用 ref 时记得添加 `.value`
2. **响应式丢失**: 直接解构 reactive 对象会丢失响应性
3. **过度使用侦听器**: 能用计算属性解决的就不要用侦听器
4. **组件命名**: 避免使用 HTML 保留字作为组件名

### 性能优化

1. **合理使用 `v-if` 和 `v-show`**: 频繁切换用 `v-show`，条件很少变化用 `v-if`
2. **列表渲染 key**: 为 `v-for` 提供唯一的 key
3. **计算属性缓存**: 利用计算属性的缓存特性
4. **按需引入**: 使用 Tree-shaking 减少打包体积

## 总结

Vue 3 带来了很多改进和新特性，让开发更加高效和愉快。通过学习本文的内容，你应该能够：

1. 快速搭建 Vue 3 开发环境
2. 理解 Vue 3 的核心概念和特性
3. 掌握基本的响应式数据和事件处理
4. 实现常见的前端功能如表单、列表等
5. 理解组件通信和状态管理

接下来可以深入学习组合式 API、生命周期钩子、路由管理、状态管理等高级主题，构建更加复杂和强大的应用。

推荐的学习路径：
- Vue 3 基础语法和概念
- 组合式 API 详解
- Vue Router 路由管理
- Pinia 状态管理
- TypeScript 集成
- 性能优化最佳实践
- 测试和部署

Vue 3 的学习曲线相对平缓，但功能强大，是现代前端开发的优秀选择。