---
title: Vue 3 模板 Refs
published: 2024-04-21
description: 'ref 绑定 DOM 元素的详细介绍和学习笔记'
image: ''
tags: ["Vue3","Refs"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

模板 Refs 是 Vue 中访问 DOM 元素或组件实例的重要机制。通过 ref 属性，我们可以在模板中标记元素或组件，然后在组件逻辑中通过 ref 对象访问它们。这对于需要直接操作 DOM 或调用组件方法的情况非常有用。

## 核心概念

### Ref 的作用

1. **访问 DOM 元素**: 直接操作 DOM 元素
2. **访问组件实例**: 调用子组件的方法或访问其数据
3. **第三方库集成**: 与需要 DOM 引用的第三方库集成
4. **表单操作**: 访问表单元素进行验证或操作

### Ref 的工作原理

Vue 使用 ref 属性在模板中标记元素，然后在组件中通过 ref 对象访问这些标记的元素或组件。Vue 会在组件挂载后自动将对应的 DOM 元素或组件实例赋值给 ref 对象。

## 基本用法

### 1. 访问 DOM 元素

```vue
<script setup>
import { ref, onMounted } from 'vue';

const inputRef = ref(null);
const buttonRef = ref(null);

onMounted(() => {
  // 访问 DOM 元素
  console.log('Input 元素:', inputRef.value);
  console.log('Button 元素:', buttonRef.value);
  
  // 聚焦输入框
  if (inputRef.value) {
    inputRef.value.focus();
  }
});

function focusInput() {
  if (inputRef.value) {
    inputRef.value.focus();
  }
}

function blurInput() {
  if (inputRef.value) {
    inputRef.value.blur();
  }
}
</script>

<template>
  <div>
    <input 
      ref="inputRef" 
      type="text" 
      placeholder="点击按钮聚焦"
    >
    <button ref="buttonRef" @click="focusInput">
      聚焦输入框
    </button>
    <button @click="blurInput">
      失去焦点
    </button>
  </div>
</template>
```

### 2. 访问组件实例

```vue
<!-- ChildComponent.vue -->
<script setup>
import { ref } from 'vue';

const count = ref(0);
const message = ref('Hello from child');

function increment() {
  count.value++;
}

function decrement() {
  count.value--;
}

function reset() {
  count.value = 0;
}

function getChildData() {
  return {
    count: count.value,
    message: message.value
  };
}

// 暴露方法给父组件
defineExpose({
  increment,
  decrement,
  reset,
  getChildData
});
</script>

<template>
  <div class="child-component">
    <h3>子组件</h3>
    <p>计数: {{ count }}</p>
    <p>消息: {{ message }}</p>
  </div>
</template>

<!-- ParentComponent.vue -->
<script setup>
import { ref } from 'vue';
import ChildComponent from './ChildComponent.vue';

const childRef = ref(null);

function callChildMethod() {
  if (childRef.value) {
    // 调用子组件方法
    childRef.value.increment();
  }
}

function getChildData() {
  if (childRef.value) {
    const data = childRef.value.getChildData();
    console.log('子组件数据:', data);
    alert(`子组件数据: ${JSON.stringify(data)}`);
  }
}

function resetChild() {
  if (childRef.value) {
    childRef.value.reset();
  }
}
</script>

<template>
  <div>
    <h2>父组件</h2>
    <ChildComponent ref="childRef" />
    
    <div class="controls">
      <button @click="callChildMethod">调用子组件方法</button>
      <button @click="getChildData">获取子组件数据</button>
      <button @click="resetChild">重置子组件</button>
    </div>
  </div>
</template>
```

### 3. 动态 Ref

```vue
<script setup>
import { ref } from 'vue';

const items = ref([
  { id: 1, name: '项目 1' },
  { id: 2, name: '项目 2' },
  { id: 3, name: '项目 3' }
]);

const itemRefs = ref([]);

function setItemRef(el) {
  if (el) {
    itemRefs.value.push(el);
  }
}

function highlightAll() {
  itemRefs.value.forEach((item, index) => {
    if (item) {
      item.style.backgroundColor = index % 2 === 0 ? '#f0f0f0' : '#e0e0e0';
    }
  });
}

function scrollToItem(index) {
  if (itemRefs.value[index]) {
    itemRefs.value[index].scrollIntoView({ behavior: 'smooth' });
  }
}
</script>

<template>
  <div>
    <h2>动态 Ref 示例</h2>
    
    <div class="controls">
      <button @click="highlightAll">高亮所有项目</button>
      <button @click="scrollToItem(0)">滚动到第一个</button>
      <button @click="scrollToItem(items.length - 1)">滚动到最后一个</button>
    </div>
    
    <div class="item-list">
      <div 
        v-for="(item, index) in items" 
        :key="item.id"
        :ref="setItemRef"
        class="item"
      >
        {{ item.name }}
      </div>
    </div>
  </div>
</template>

<style scoped>
.item-list {
  margin-top: 20px;
}

.item {
  padding: 15px;
  margin: 10px 0;
  border: 1px solid #ddd;
  border-radius: 4px;
  transition: background-color 0.3s;
}

.controls {
  margin-bottom: 20px;
}

.controls button {
  margin-right: 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

### 4. 函数式 Ref

```vue
<script setup>
import { ref } from 'vue';

const inputRefs = ref([]);

function setInputRef(el, index) {
  if (el) {
    inputRefs.value[index] = el;
  }
}

function focusInput(index) {
  if (inputRefs.value[index]) {
    inputRefs.value[index].focus();
  }
}

function focusAll() {
  inputRefs.value.forEach(input => {
    if (input) {
      input.focus();
      input.blur();
    }
  });
}

const inputs = ref(['输入框 1', '输入框 2', '输入框 3']);
</script>

<template>
  <div>
    <h2>函数式 Ref 示例</h2>
    
    <div 
      v-for="(input, index) in inputs" 
      :key="index"
      class="input-group"
    >
      <label>{{ input }}:</label>
      <input 
        :ref="(el) => setInputRef(el, index)"
        type="text"
        placeholder="输入内容"
      >
      <button @click="focusInput(index)">聚焦</button>
    </div>
    
    <button @click="focusAll" class="focus-all">
      聚焦所有输入框
    </button>
  </div>
</template>

<style scoped>
.input-group {
  display: flex;
  align-items: center;
  margin: 15px 0;
  gap: 10px;
}

.input-group label {
  min-width: 100px;
}

.input-group input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.input-group button {
  padding: 8px 16px;
  background: #2196F3;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.focus-all {
  margin-top: 20px;
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}
</style>
```

## 实际应用

### 1. 表单验证和操作

```vue
<script setup>
import { ref } from 'vue';

const formData = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

const formRefs = reactive({
  username: null,
  email: null,
  password: null,
  confirmPassword: null
});

const errors = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
});

function validateField(fieldName) {
  const value = formData[fieldName];
  const ref = formRefs[fieldName];
  
  errors[fieldName] = '';
  
  if (!value) {
    errors[fieldName] = '此字段不能为空';
    return false;
  }
  
  switch (fieldName) {
    case 'username':
      if (value.length < 3) {
        errors[fieldName] = '用户名至少3个字符';
        return false;
      }
      break;
      
    case 'email':
      const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
      if (!emailRegex.test(value)) {
        errors[fieldName] = '邮箱格式不正确';
        return false;
      }
      break;
      
    case 'password':
      if (value.length < 6) {
        errors[fieldName] = '密码至少6个字符';
        return false;
      }
      break;
      
    case 'confirmPassword':
      if (value !== formData.password) {
        errors[fieldName] = '两次密码不一致';
        return false;
      }
      break;
  }
  
  return true;
}

function focusFirstError() {
  for (const fieldName in errors) {
    if (errors[fieldName] && formRefs[fieldName]) {
      formRefs[fieldName].focus();
      return;
    }
  }
}

function validateForm() {
  let isValid = true;
  
  for (const fieldName in formData) {
    if (!validateField(fieldName)) {
      isValid = false;
    }
  }
  
  if (!isValid) {
    focusFirstError();
  }
  
  return isValid;
}

function handleSubmit() {
  if (validateForm()) {
    console.log('表单提交:', formData);
    alert('表单提交成功！');
  }
}

function resetForm() {
  Object.assign(formData, {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
  
  Object.assign(errors, {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  });
}
</script>

<template>
  <form @submit.prevent="handleSubmit" class="registration-form">
    <h2>用户注册</h2>
    
    <div class="form-group">
      <label for="username">用户名:</label>
      <input 
        id="username"
        ref="el => formRefs.username = el"
        v-model="formData.username"
        type="text"
        :class="{ error: errors.username }"
        @blur="validateField('username')"
        placeholder="输入用户名"
      >
      <span v-if="errors.username" class="error-message">{{ errors.username }}</span>
    </div>
    
    <div class="form-group">
      <label for="email">邮箱:</label>
      <input 
        id="email"
        ref="el => formRefs.email = el"
        v-model="formData.email"
        type="email"
        :class="{ error: errors.email }"
        @blur="validateField('email')"
        placeholder="输入邮箱"
      >
      <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
    </div>
    
    <div class="form-group">
      <label for="password">密码:</label>
      <input 
        id="password"
        ref="el => formRefs.password = el"
        v-model="formData.password"
        type="password"
        :class="{ error: errors.password }"
        @blur="validateField('password')"
        placeholder="输入密码"
      >
      <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
    </div>
    
    <div class="form-group">
      <label for="confirmPassword">确认密码:</label>
      <input 
        id="confirmPassword"
        ref="el => formRefs.confirmPassword = el"
        v-model="formData.confirmPassword"
        type="password"
        :class="{ error: errors.confirmPassword }"
        @blur="validateField('confirmPassword')"
        placeholder="确认密码"
      >
      <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
    </div>
    
    <div class="form-actions">
      <button type="submit" class="submit-btn">注册</button>
      <button type="button" @click="resetForm" class="reset-btn">重置</button>
    </div>
  </form>
</template>

<style scoped>
.registration-form {
  max-width: 500px;
  margin: 0 auto;
  padding: 30px;
  border: 1px solid #ddd;
  border-radius: 8px;
  background: white;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.registration-form h2 {
  text-align: center;
  margin-top: 0;
  color: #333;
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: bold;
  color: #555;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
  box-sizing: border-box;
}

.form-group input:focus {
  outline: none;
  border-color: #4CAF50;
}

.form-group input.error {
  border-color: #f44336;
}

.error-message {
  color: #f44336;
  font-size: 12px;
  margin-top: 5px;
  display: block;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 30px;
}

.submit-btn, .reset-btn {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: background-color 0.3s;
}

.submit-btn {
  background: #4CAF50;
  color: white;
}

.submit-btn:hover {
  background: #45a049;
}

.reset-btn {
  background: #f44336;
  color: white;
}

.reset-btn:hover {
  background: #da190b;
}
</style>
```

### 2. 可滚动容器和元素操作

```vue
<script setup>
import { ref, onMounted } from 'vue';

const scrollContainerRef = ref(null);
const scrollToTopBtnRef = ref(null);
const sections = ref([]);

const sectionRefs = ref([]);

function setSectionRef(el, index) {
  if (el) {
    sectionRefs.value[index] = el;
  }
}

function scrollToTop() {
  if (scrollContainerRef.value) {
    scrollContainerRef.value.scrollTo({
      top: 0,
      behavior: 'smooth'
    });
  }
}

function scrollToSection(index) {
  if (sectionRefs.value[index]) {
    sectionRefs.value[index].scrollIntoView({
      behavior: 'smooth',
      block: 'start'
    });
  }
}

function scrollToBottom() {
  if (scrollContainerRef.value) {
    scrollContainerRef.value.scrollTo({
      top: scrollContainerRef.value.scrollHeight,
      behavior: 'smooth'
    });
  }
}

function getScrollPosition() {
  if (scrollContainerRef.value) {
    const scrollTop = scrollContainerRef.value.scrollTop;
    const scrollHeight = scrollContainerRef.value.scrollHeight;
    const clientHeight = scrollContainerRef.value.clientHeight;
    
    console.log('滚动位置:', {
      scrollTop,
      scrollHeight,
      clientHeight,
      scrollPercentage: (scrollTop / (scrollHeight - clientHeight)) * 100
    });
  }
}

function highlightSection(index) {
  sectionRefs.value.forEach((section, i) => {
    if (section) {
      section.style.backgroundColor = i === index ? '#fff3cd' : 'white';
    }
  });
}

onMounted(() => {
  // 初始化章节数据
  sections.value = [
    { id: 1, title: '第一章', content: '这是第一章的内容...' },
    { id: 2, title: '第二章', content: '这是第二章的内容...' },
    { id: 3, title: '第三章', content: '这是第三章的内容...' },
    { id: 4, title: '第四章', content: '这是第四章的内容...' },
    { id: 5, title: '第五章', content: '这是第五章的内容...' }
  ];
  
  // 自动滚动到第一个章节
  setTimeout(() => {
    scrollToSection(0);
  }, 500);
});
</script>

<template>
  <div class="scroll-demo">
    <h2>滚动容器示例</h2>
    
    <div class="controls">
      <button @click="scrollToTop">回到顶部</button>
      <button @click="scrollToBottom">滚动到底部</button>
      <button @click="getScrollPosition">获取滚动位置</button>
      
      <div class="section-nav">
        <button 
          v-for="(section, index) in sections" 
          :key="section.id"
          @click="() => { scrollToSection(index); highlightSection(index); }"
        >
          {{ section.title }}
        </button>
      </div>
    </div>
    
    <div ref="scrollContainerRef" class="scroll-container">
      <div 
        v-for="(section, index) in sections" 
        :key="section.id"
        :ref="el => setSectionRef(el, index)"
        class="section"
        @click="highlightSection(index)"
      >
        <h3>{{ section.title }}</h3>
        <p>{{ section.content }}</p>
        <p>更多内容...</p>
        <p>继续添加内容以增加滚动高度...</p>
        <p>这里是更多的示例文本。</p>
      </div>
    </div>
  </div>
</template>

<style scoped>
.scroll-demo {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.controls {
  margin-bottom: 20px;
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  align-items: center;
}

.controls button {
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background-color 0.3s;
}

.controls button:hover {
  background: #45a049;
}

.section-nav {
  display: flex;
  gap: 5px;
  margin-left: auto;
}

.section-nav button {
  padding: 6px 12px;
  background: #2196F3;
}

.section-nav button:hover {
  background: #0b7dda;
}

.scroll-container {
  height: 400px;
  overflow-y: auto;
  border: 2px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  background: #fafafa;
  scroll-behavior: smooth;
}

.section {
  padding: 20px;
  margin-bottom: 20px;
  border: 1px solid #eee;
  border-radius: 8px;
  background: white;
  transition: background-color 0.3s;
  cursor: pointer;
}

.section:hover {
  background: #f5f5f5;
}

.section h3 {
  margin-top: 0;
  color: #333;
}

.section p {
  color: #666;
  line-height: 1.6;
}
</style>
```

### 3. 第三方库集成

```vue
<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue';

const canvasRef = ref(null);
let chartInstance = null;

// 模拟图表库（实际项目中使用 ECharts、Chart.js 等）
class SimpleChart {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.data = [];
    this.colors = ['#4CAF50', '#2196F3', '#FF9800', '#F44336', '#9C27B0'];
  }
  
  setData(data) {
    this.data = data;
    this.draw();
  }
  
  draw() {
    const ctx = this.ctx;
    const width = this.canvas.width;
    const height = this.canvas.height;
    
    ctx.clearRect(0, 0, width, height);
    
    if (this.data.length === 0) return;
    
    const maxValue = Math.max(...this.data.map(d => d.value));
    const barWidth = (width - 40) / this.data.length - 10;
    
    this.data.forEach((item, index) => {
      const barHeight = (item.value / maxValue) * (height - 40);
      const x = 20 + index * (barWidth + 10);
      const y = height - 20 - barHeight;
      
      // 绘制柱状图
      ctx.fillStyle = this.colors[index % this.colors.length];
      ctx.fillRect(x, y, barWidth, barHeight);
      
      // 绘制标签
      ctx.fillStyle = '#333';
      ctx.font = '12px Arial';
      ctx.textAlign = 'center';
      ctx.fillText(item.label, x + barWidth / 2, height - 5);
      
      // 绘制数值
      ctx.fillStyle = '#666';
      ctx.fillText(item.value, x + barWidth / 2, y - 5);
    });
  }
  
  destroy() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
  }
}

const chartData = ref([
  { label: '一月', value: 65 },
  { label: '二月', value: 59 },
  { label: '三月', value: 80 },
  { label: '四月', value: 81 },
  { label: '五月', value: 56 },
  { label: '六月', value: 55 },
  { label: '七月', value: 40 }
]);

function updateChart() {
  if (chartInstance) {
    // 随机更新数据
    chartData.value = chartData.value.map(item => ({
      ...item,
      value: Math.floor(Math.random() * 100)
    }));
    
    chartInstance.setData(chartData.value);
  }
}

function addData() {
  const months = ['八月', '九月', '十月', '十一月', '十二月'];
  const nextMonth = months[chartData.value.length - 7] || `月份 ${chartData.value.length + 1}`;
  
  chartData.value.push({
    label: nextMonth,
    value: Math.floor(Math.random() * 100)
  });
  
  if (chartInstance) {
    chartInstance.setData(chartData.value);
  }
}

function removeData() {
  if (chartData.value.length > 1) {
    chartData.value.pop();
    
    if (chartInstance) {
      chartInstance.setData(chartData.value);
    }
  }
}

onMounted(() => {
  if (canvasRef.value) {
    chartInstance = new SimpleChart(canvasRef.value);
    chartInstance.setData(chartData.value);
  }
});

onBeforeUnmount(() => {
  if (chartInstance) {
    chartInstance.destroy();
    chartInstance = null;
  }
});
</script>

<template>
  <div class="chart-container">
    <h2>第三方库集成示例</h2>
    
    <div class="chart-controls">
      <button @click="updateChart">更新数据</button>
      <button @click="addData">添加数据</button>
      <button @click="removeData">移除数据</button>
    </div>
    
    <div class="canvas-wrapper">
      <canvas 
        ref="canvasRef" 
        width="600" 
        height="400"
        style="border: 1px solid #ddd; border-radius: 8px;"
      ></canvas>
    </div>
    
    <div class="data-display">
      <h3>当前数据</h3>
      <ul>
        <li v-for="(item, index) in chartData" :key="index">
          {{ item.label }}: {{ item.value }}
        </li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.chart-container {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
  text-align: center;
}

.chart-controls {
  margin-bottom: 20px;
  display: flex;
  justify-content: center;
  gap: 10px;
}

.chart-controls button {
  padding: 10px 20px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  transition: background-color 0.3s;
}

.chart-controls button:hover {
  background: #45a049;
}

.canvas-wrapper {
  display: flex;
  justify-content: center;
  margin-bottom: 20px;
}

.data-display {
  text-align: left;
  max-width: 400px;
  margin: 0 auto;
}

.data-display h3 {
  text-align: center;
  color: #333;
}

.data-display ul {
  list-style: none;
  padding: 0;
  border: 1px solid #ddd;
  border-radius: 8px;
  overflow: hidden;
}

.data-display li {
  padding: 10px 15px;
  border-bottom: 1px solid #eee;
  color: #555;
}

.data-display li:last-child {
  border-bottom: none;
}

.data-display li:nth-child(even) {
  background: #f9f9f9;
}
</style>
```

### 4. 动画和交互效果

```vue
<script setup>
import { ref, onMounted } from 'vue';

const animatedElement = ref(null);
const dragContainer = ref(null);
const resizeHandle = ref(null);

let isDragging = false;
let dragOffset = { x: 0, y: 0 };
let isResizing = false;

function startAnimation() {
  if (animatedElement.value) {
    animatedElement.value.style.transition = 'transform 2s ease-in-out';
    animatedElement.value.style.transform = 'rotate(360deg) scale(1.5)';
    
    setTimeout(() => {
      if (animatedElement.value) {
        animatedElement.value.style.transform = 'rotate(0deg) scale(1)';
      }
    }, 2000);
  }
}

function changeColor() {
  if (animatedElement.value) {
    const colors = ['#4CAF50', '#2196F3', '#FF9800', '#F44336', '#9C27B0'];
    const randomColor = colors[Math.floor(Math.random() * colors.length)];
    animatedElement.value.style.backgroundColor = randomColor;
  }
}

function startDrag(event) {
  if (!dragContainer.value) return;
  
  isDragging = true;
  const rect = dragContainer.value.getBoundingClientRect();
  dragOffset.x = event.clientX - rect.left;
  dragOffset.y = event.clientY - rect.top;
  
  dragContainer.value.style.cursor = 'grabbing';
}

function onDrag(event) {
  if (!isDragging || !dragContainer.value) return;
  
  const x = event.clientX - dragOffset.x;
  const y = event.clientY - dragOffset.y;
  
  dragContainer.value.style.left = `${x}px`;
  dragContainer.value.style.top = `${y}px`;
}

function stopDrag() {
  isDragging = false;
  if (dragContainer.value) {
    dragContainer.value.style.cursor = 'grab';
  }
}

function startResize(event) {
  isResizing = true;
  event.preventDefault();
}

function onResize(event) {
  if (!isResizing || !dragContainer.value) return;
  
  const newWidth = event.clientX - dragContainer.value.getBoundingClientRect().left;
  const newHeight = event.clientY - dragContainer.value.getBoundingClientRect().top;
  
  if (newWidth > 100) {
    dragContainer.value.style.width = `${newWidth}px`;
  }
  
  if (newHeight > 100) {
    dragContainer.value.style.height = `${newHeight}px`;
  }
}

function stopResize() {
  isResizing = false;
}

onMounted(() => {
  // 添加全局事件监听器
  document.addEventListener('mousemove', onDrag);
  document.addEventListener('mouseup', stopDrag);
  document.addEventListener('mousemove', onResize);
  document.addEventListener('mouseup', stopResize);
});
</script>

<template>
  <div class="animation-demo">
    <h2>动画和交互效果示例</h2>
    
    <div class="controls">
      <button @click="startAnimation">开始旋转动画</button>
      <button @click="changeColor">改变颜色</button>
    </div>
    
    <div class="demo-area">
      <!-- 动画元素 -->
      <div 
        ref="animatedElement" 
        class="animated-element"
      >
        动画元素
      </div>
      
      <!-- 可拖拽和调整大小的容器 -->
      <div 
        ref="dragContainer"
        class="drag-container"
        @mousedown="startDrag"
      >
        <h3>可拖拽容器</h3>
        <p>拖拽我移动位置</p>
        <div 
          ref="resizeHandle"
          class="resize-handle"
          @mousedown="startResize"
        >
          ↘
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.animation-demo {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.controls {
  margin-bottom: 30px;
  display: flex;
  gap: 10px;
  justify-content: center;
}

.controls button {
  padding: 12px 24px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background-color 0.3s, transform 0.2s;
}

.controls button:hover {
  background: #45a049;
  transform: translateY(-2px);
}

.controls button:active {
  transform: translateY(0);
}

.demo-area {
  position: relative;
  height: 400px;
  border: 2px dashed #ddd;
  border-radius: 8px;
  padding: 20px;
  background: #fafafa;
}

.animated-element {
  width: 100px;
  height: 100px;
  background: #4CAF50;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 8px;
  font-weight: bold;
  cursor: pointer;
  user-select: none;
  margin-bottom: 20px;
}

.drag-container {
  position: absolute;
  width: 200px;
  height: 150px;
  background: white;
  border: 2px solid #2196F3;
  border-radius: 8px;
  padding: 15px;
  cursor: grab;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  left: 250px;
  top: 100px;
}

.drag-container:active {
  cursor: grabbing;
}

.drag-container h3 {
  margin-top: 0;
  color: #333;
}

.drag-container p {
  margin: 10px 0;
  color: #666;
}

.resize-handle {
  position: absolute;
  bottom: 5px;
  right: 5px;
  width: 20px;
  height: 20px;
  background: #2196F3;
  color: white;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 4px;
  cursor: se-resize;
  font-size: 12px;
  font-weight: bold;
}
</style>
```

## 注意事项

### 最佳实践

1. **避免过度使用**: 只有在确实需要时才使用 ref
2. **安全访问**: 始终检查 ref.value 是否存在
3. **及时清理**: 在组件卸载时清理 ref 引用
4. **类型安全**: 在 TypeScript 中正确声明 ref 类型
5. **命名规范**: 使用有意义的 ref 名称

### 常见错误

```javascript
// 错误 1: 在模板中直接访问 ref.value
<template>
  <div>{{ inputRef.value }}</div> <!-- 错误 -->
</template>

// 正确做法
<template>
  <div ref="inputRef">{{ inputRef }}</div> <!-- 在模板中自动解包 -->
</template>

// 错误 2: 在组件挂载前访问 ref
onBeforeMount(() => {
  console.log(inputRef.value); // null
});

// 正确做法
onMounted(() => {
  console.log(inputRef.value); // DOM 元素
});

// 错误 3: 忘记检查 null
function focusInput() {
  inputRef.value.focus(); // 可能为 null
}

// 正确做法
function focusInput() {
  if (inputRef.value) {
    inputRef.value.focus();
  }
}

// 错误 4: 在 v-for 中使用相同 ref 名称
<template>
  <div v-for="item in items" :key="item.id">
    <input ref="inputRef"> <!-- 所有 input 都会引用同一个 ref -->
  </div>
</template>

// 正确做法：使用函数式 ref 或数组
<template>
  <div v-for="(item, index) in items" :key="item.id">
    <input :ref="(el) => setInputRef(el, index)">
  </div>
</template>
```

### TypeScript 支持

```typescript
import { ref } from 'vue';

// DOM 元素 ref
const inputRef = ref<HTMLInputElement | null>(null);

// 组件实例 ref
const childComponentRef = ref<InstanceType<typeof ChildComponent> | null>(null);

// 函数式 ref
function setElementRef(el: HTMLElement | null) {
  // 处理 ref
}
```

## 总结

模板 Refs 是 Vue 中访问 DOM 元素和组件实例的重要机制，掌握其使用方法对于构建交互丰富的应用至关重要：

### 核心要点

1. **基本用法**: 使用 ref 属性标记元素或组件
2. **访问时机**: ref 只在组件挂载后可用
3. **类型安全**: 在 TypeScript 中正确声明类型
4. **动态 Ref**: 使用函数式 ref 处理动态列表
5. **清理工作**: 在组件卸载时清理引用

### 使用场景

1. **DOM 操作**: 直接操作 DOM 元素
2. **组件通信**: 访问子组件的方法和数据
3. **第三方集成**: 与需要 DOM 引用的库集成
4. **表单操作**: 访问和操作表单元素
5. **动画效果**: 实现复杂的动画和交互

### 最佳实践

1. **优先使用声明式**: 大部分情况下应该使用 Vue 的声明式语法
2. **谨慎使用 ref**: 只在确实需要时使用 ref
3. **安全访问**: 始终检查 ref 是否存在
4. **性能考虑**: 避免频繁的 DOM 操作
5. **类型安全**: 充分利用 TypeScript 类型系统

模板 Refs 提供了直接访问 DOM 和组件实例的能力，是 Vue 强大功能的重要组成部分。合理使用 Refs 将帮助你构建更加丰富和交互性强的应用。