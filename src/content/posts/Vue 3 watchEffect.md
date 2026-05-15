---
title: Vue 3 watchEffect
published: 2024-01-10
description: 'watchEffect 自动追踪的详细介绍和学习笔记'
image: ''
tags: ["Vue3","watch"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 的 `watchEffect` 是一个自动追踪依赖的响应式 API，它会立即执行传入的函数，并自动追踪函数内部使用的响应式数据。当这些数据发生变化时，函数会重新执行。

`watchEffect` 的核心特点：
- **自动依赖收集**：无需显式指定监听源，自动追踪函数内使用的响应式数据
- **立即执行**：在创建时立即执行一次，不像 watch 需要等待数据变化
- **更简洁的语法**：对于简单的副作用，语法比 watch 更简洁

## 核心概念

### 与 watch 的区别

| 特性 | watch | watchEffect |
|------|-------|-------------|
| 依赖追踪 | 手动指定 | 自动追踪 |
| 执行时机 | 数据变化时 | 立即执行 + 数据变化时 |
| 访问新旧值 | 支持 | 不支持 |
| 使用场景 | 明确监听源 | 自动依赖收集 |
| 性能 | 更精确 | 可能执行多余计算 |

### 工作原理

```javascript
watchEffect(() => {
  // 这个函数会立即执行
  // 并自动追踪里面使用的响应式数据
  console.log(count.value) // 自动追踪 count
  console.log(user.value.name) // 自动追踪 user.name
  
  // 当 count 或 user.name 变化时，函数会重新执行
})
```

### 执行时机

```javascript
// 默认在组件更新前执行
watchEffect(() => {
  console.log('pre flush')
})

// 在组件更新后执行
watchEffect(() => {
  console.log('post flush')
}, { flush: 'post' })

// 同步执行
watchEffect(() => {
  console.log('sync flush')
}, { flush: 'sync' })
```

## 基本用法

### 简单示例

```vue
<template>
  <div>
    <h2>watchEffect 基础示例</h2>
    
    <div class="counter-demo">
      <p>计数器: {{ count }}</p>
      <button @click="increment">增加</button>
      <button @click="decrement">减少</button>
    </div>
    
    <div class="log-output">
      <h3>执行日志:</h3>
      <div v-for="(log, index) in logs" :key="index" class="log-entry">
        {{ log }}
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue'

const count = ref(0)
const logs = ref([])

// 使用 watchEffect
watchEffect(() => {
  console.log('当前计数:', count.value)
  logs.value.unshift(`当前计数: ${count.value}`)
  
  // 限制日志数量
  if (logs.value.length > 10) {
    logs.value.pop()
  }
})

const increment = () => count.value++
const decrement = () => count.value--
</script>

<style scoped>
.counter-demo {
  margin: 20px 0;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
  text-align: center;
}

.counter-demo button {
  margin: 0 10px;
  padding: 8px 16px;
  background: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.log-output {
  margin-top: 20px;
  padding: 20px;
  background: white;
  border-radius: 8px;
  border: 1px solid #e0e0e0;
}

.log-output h3 {
  margin-top: 0;
  color: #333;
}

.log-entry {
  padding: 8px 12px;
  margin: 5px 0;
  background: #f5f5f5;
  border-radius: 4px;
  font-family: monospace;
  font-size: 14px;
}
</style>
```

### 计算属性副作用

```vue
<template>
  <div>
    <h2>计算属性副作用</h2>
    
    <div class="calculator">
      <div class="input-group">
        <label>宽度:</label>
        <input type="number" v-model.number="width" />
      </div>
      
      <div class="input-group">
        <label>高度:</label>
        <input type="number" v-model.number="height" />
      </div>
      
      <div class="result">
        <h3>计算结果:</h3>
        <p>面积: {{ area }}</p>
        <p>周长: {{ perimeter }}</p>
      </div>
      
      <div class="history">
        <h4>历史记录:</h4>
        <div v-for="(record, index) in history" :key="index" class="history-item">
          {{ record.width }} × {{ record.height }} = 面积: {{ record.area }}, 周长: {{ record.perimeter }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'

const width = ref(10)
const height = ref(5)

const area = computed(() => width.value * height.value)
const perimeter = computed(() => 2 * (width.value + height.value))

const history = ref([])

// 使用 watchEffect 记录计算历史
watchEffect(() => {
  if (width.value > 0 && height.value > 0) {
    history.value.unshift({
      width: width.value,
      height: height.value,
      area: area.value,
      perimeter: perimeter.value,
      timestamp: new Date().toLocaleTimeString()
    })
    
    // 限制历史记录数量
    if (history.value.length > 5) {
      history.value.pop()
    }
  }
})
</script>

<style scoped>
.calculator {
  max-width: 500px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.input-group {
  margin-bottom: 20px;
}

.input-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #333;
}

.input-group input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
}

.result {
  margin: 30px 0;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.result h3 {
  margin-top: 0;
  color: #333;
}

.result p {
  font-size: 18px;
  color: #666;
  margin: 10px 0;
}

.history {
  margin-top: 30px;
}

.history h4 {
  margin-bottom: 15px;
  color: #333;
}

.history-item {
  padding: 12px;
  margin: 8px 0;
  background: #f5f5f5;
  border-radius: 4px;
  font-size: 14px;
  color: #666;
  border-left: 3px solid #4CAF50;
}
</style>
```

### DOM 操作

```vue
<template>
  <div>
    <h2>DOM 操作示例</h2>
    
    <div class="dom-demo">
      <div class="controls">
        <input 
          v-model="backgroundColor" 
          placeholder="输入背景颜色"
          class="color-input"
        />
        <input 
          v-model="fontSize" 
          type="number" 
          min="12" 
          max="32"
          placeholder="字体大小"
          class="size-input"
        />
        <input 
          v-model="borderRadius" 
          type="number" 
          min="0" 
          max="50"
          placeholder="圆角大小"
          class="radius-input"
        />
      </div>
      
      <div ref="styledElement" class="styled-element">
        <p>这是一段可以动态设置样式的文本</p>
        <p>当前设置: 背景色 {{ backgroundColor }}, 字体 {{ fontSize }}px, 圆角 {{ borderRadius }}px</p>
      </div>
      
      <div class="code-display">
        <h4>生成的 CSS:</h4>
        <pre>{{ generatedCSS }}</pre>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect } from 'vue'

const backgroundColor = ref('#4CAF50')
const fontSize = ref(16)
const borderRadius = ref(8)
const styledElement = ref(null)
const generatedCSS = ref('')

// 使用 watchEffect 操作 DOM
watchEffect(() => {
  if (styledElement.value) {
    // 直接操作 DOM 元素样式
    styledElement.value.style.backgroundColor = backgroundColor.value
    styledElement.value.style.fontSize = `${fontSize.value}px`
    styledElement.value.style.borderRadius = `${borderRadius.value}px`
    
    // 生成 CSS 代码
    generatedCSS.value = `
background-color: ${backgroundColor.value};
font-size: ${fontSize.value}px;
border-radius: ${borderRadius.value}px;
    `.trim()
  }
})
</script>

<style scoped>
.dom-demo {
  max-width: 600px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.controls {
  display: grid;
  grid-template-columns: 1fr 1fr 1fr;
  gap: 15px;
  margin-bottom: 30px;
}

.controls input {
  width: 100%;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 14px;
}

.styled-element {
  padding: 30px;
  margin: 30px 0;
  color: white;
  text-align: center;
  transition: all 0.3s ease;
}

.styled-element p {
  margin: 10px 0;
  line-height: 1.6;
}

.code-display {
  margin-top: 30px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.code-display h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.code-display pre {
  background: white;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
  font-family: 'Courier New', monospace;
  font-size: 14px;
  color: #333;
  margin: 0;
}
</style>
```

## 实际应用

### 自动保存

```vue
<template>
  <div>
    <h2>自动保存文档</h2>
    
    <div class="editor-container">
      <div class="editor-toolbar">
        <div class="save-status" :class="saveStatus">
          <span v-if="saveStatus === 'saving'">💾 保存中...</span>
          <span v-else-if="saveStatus === 'saved'">✓ 已保存</span>
          <span v-else-if="saveStatus === 'error'">✗ 保存失败</span>
          <span v-else>⏳ 等待编辑</span>
        </div>
        
        <div class="save-info">
          <span>上次保存: {{ lastSaveTime }}</span>
          <span>自动保存: {{ autoSave ? '开启' : '关闭' }}</span>
        </div>
      </div>
      
      <div class="editor-input">
        <input 
          v-model="document.title" 
          placeholder="文档标题"
          class="title-input"
        />
        
        <textarea
          v-model="document.content"
          placeholder="开始输入内容..."
          class="content-input"
          rows="15"
        ></textarea>
      </div>
      
      <div class="editor-footer">
        <button @click="toggleAutoSave" class="toggle-btn">
          {{ autoSave ? '关闭自动保存' : '开启自动保存' }}
        </button>
        <button @click="manualSave" class="save-btn">
          立即保存
        </button>
        <button @click="clearDocument" class="clear-btn">
          清空文档
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, watchEffect, onMounted } from 'vue'

const STORAGE_KEY = 'auto-save-document'

const document = ref({
  title: '',
  content: ''
})

const saveStatus = ref('idle') // idle, saving, saved, error
const autoSave = ref(true)
const lastSaveTime = ref('')
const saveTimeout = ref(null)

// 加载保存的文档
onMounted(() => {
  const savedDoc = localStorage.getItem(STORAGE_KEY)
  if (savedDoc) {
    try {
      document.value = JSON.parse(savedDoc)
      console.log('文档已恢复')
    } catch (error) {
      console.error('恢复文档失败:', error)
    }
  }
})

// 保存文档
const saveDocument = async () => {
  if (!autoSave.value) return
  
  saveStatus.value = 'saving'
  
  try {
    // 模拟异步保存操作
    await new Promise(resolve => setTimeout(resolve, 500))
    
    localStorage.setItem(STORAGE_KEY, JSON.stringify(document.value))
    saveStatus.value = 'saved'
    lastSaveTime.value = new Date().toLocaleTimeString()
    
    // 2秒后重置状态
    setTimeout(() => {
      saveStatus.value = 'idle'
    }, 2000)
  } catch (error) {
    saveStatus.value = 'error'
    console.error('保存失败:', error)
  }
}

// 使用 watchEffect 实现自动保存
watchEffect((onCleanup) => {
  // 如果自动保存关闭，不执行保存逻辑
  if (!autoSave.value) return
  
  // 清除之前的定时器
  if (saveTimeout.value) {
    clearTimeout(saveTimeout.value)
  }
  
  // 检查文档是否有内容
  if (document.value.title || document.value.content) {
    // 设置新的定时器，2秒后自动保存
    saveTimeout.value = setTimeout(() => {
      saveDocument()
    }, 2000)
  }
  
  // 清理函数
  onCleanup(() => {
    if (saveTimeout.value) {
      clearTimeout(saveTimeout.value)
    }
  })
})

const toggleAutoSave = () => {
  autoSave.value = !autoSave.value
}

const manualSave = async () => {
  await saveDocument()
}

const clearDocument = () => {
  if (confirm('确定要清空文档吗？')) {
    document.value = {
      title: '',
      content: ''
    }
    localStorage.removeItem(STORAGE_KEY)
    saveStatus.value = 'idle'
    lastSaveTime.value = ''
  }
}
</script>

<style scoped>
.editor-container {
  max-width: 800px;
  margin: 20px auto;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 15px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

.editor-toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background: #f8f9fa;
  border-bottom: 1px solid #e0e0e0;
}

.save-status {
  padding: 5px 12px;
  border-radius: 4px;
  font-size: 14px;
  font-weight: 500;
}

.save-status.saving {
  background: #e3f2fd;
  color: #1976d2;
}

.save-status.saved {
  background: #e8f5e9;
  color: #4CAF50;
}

.save-status.error {
  background: #ffebee;
  color: #f44336;
}

.save-info {
  display: flex;
  gap: 20px;
  font-size: 12px;
  color: #666;
}

.editor-input {
  padding: 20px;
}

.title-input {
  width: 100%;
  padding: 12px;
  font-size: 18px;
  font-weight: bold;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  box-sizing: border-box;
  margin-bottom: 15px;
}

.title-input:focus,
.content-input:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

.content-input {
  width: 100%;
  padding: 12px;
  font-size: 14px;
  border: 1px solid #e0e0e0;
  border-radius: 4px;
  box-sizing: border-box;
  font-family: inherit;
  line-height: 1.6;
  resize: vertical;
}

.editor-footer {
  display: flex;
  gap: 10px;
  padding: 15px 20px;
  background: #f8f9fa;
  border-top: 1px solid #e0e0e0;
}

.editor-footer button {
  padding: 10px 20px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
  font-weight: 500;
  transition: background 0.2s;
}

.toggle-btn {
  background: #6c757d;
  color: white;
}

.save-btn {
  background: #4CAF50;
  color: white;
}

.clear-btn {
  background: #f44336;
  color: white;
}

.editor-footer button:hover {
  opacity: 0.9;
}
</style>
```

### 响应式布局

```vue
<template>
  <div>
    <h2>响应式布局示例</h2>
    
    <div class="layout-info">
      <div class="info-item">
        <span>窗口宽度:</span>
        <strong>{{ windowSize.width }}px</strong>
      </div>
      <div class="info-item">
        <span>窗口高度:</span>
        <strong>{{ windowSize.height }}px</strong>
      </div>
      <div class="info-item">
        <span>设备类型:</span>
        <strong>{{ deviceType }}</strong>
      </div>
      <div class="info-item">
        <span>当前断点:</span>
        <strong>{{ currentBreakpoint }}</strong>
      </div>
    </div>
    
    <div :class="['responsive-grid', currentBreakpoint]">
      <div v-for="i in 6" :key="i" class="grid-item">
        <h3>项目 {{ i }}</h3>
        <p>响应式布局演示</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watchEffect, onMounted, onUnmounted } from 'vue'

const windowSize = ref({
  width: window.innerWidth,
  height: window.innerHeight
})

// 断点定义
const breakpoints = {
  mobile: 0,
  tablet: 768,
  desktop: 1024,
  large: 1440
}

// 计算当前设备类型
const deviceType = computed(() => {
  if (windowSize.value.width < breakpoints.tablet) {
    return '移动设备'
  } else if (windowSize.value.width < breakpoints.desktop) {
    return '平板设备'
  } else {
    return '桌面设备'
  }
})

// 计算当前断点
const currentBreakpoint = computed(() => {
  const width = windowSize.value.width
  
  if (width < breakpoints.tablet) return 'mobile'
  if (width < breakpoints.desktop) return 'tablet'
  if (width < breakpoints.large) return 'desktop'
  return 'large'
})

// 监听窗口大小变化
const handleResize = () => {
  windowSize.value = {
    width: window.innerWidth,
    height: window.innerHeight
  }
}

// 使用 watchEffect 处理响应式逻辑
watchEffect(() => {
  console.log(`窗口大小变化: ${windowSize.value.width}x${windowSize.value.height}`)
  console.log(`设备类型: ${deviceType.value}`)
  console.log(`当前断点: ${currentBreakpoint.value}`)
  
  // 根据设备类型调整字体大小
  const root = document.documentElement
  if (windowSize.value.width < breakpoints.tablet) {
    root.style.setProperty('--base-font-size', '14px')
  } else if (windowSize.value.width < breakpoints.desktop) {
    root.style.setProperty('--base-font-size', '16px')
  } else {
    root.style.setProperty('--base-font-size', '18px')
  }
})

onMounted(() => {
  window.addEventListener('resize', handleResize)
})

onUnmounted(() => {
  window.removeEventListener('resize', handleResize)
})
</script>

<style scoped>
.layout-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
  margin: 20px 0;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 12px;
  background: white;
  border-radius: 4px;
  border: 1px solid #e0e0e0;
}

.info-item strong {
  color: #4CAF50;
}

.responsive-grid {
  display: grid;
  gap: 20px;
  padding: 20px;
  margin-top: 20px;
  transition: all 0.3s ease;
}

/* 移动设备布局 */
.responsive-grid.mobile {
  grid-template-columns: 1fr;
}

/* 平板设备布局 */
.responsive-grid.tablet {
  grid-template-columns: repeat(2, 1fr);
}

/* 桌面设备布局 */
.responsive-grid.desktop {
  grid-template-columns: repeat(3, 1fr);
}

/* 大屏幕布局 */
.responsive-grid.large {
  grid-template-columns: repeat(3, 1fr);
  max-width: 1200px;
  margin: 20px auto;
}

.grid-item {
  padding: 25px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  color: white;
  text-align: center;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  transition: transform 0.2s;
}

.grid-item:hover {
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.grid-item h3 {
  margin-top: 0;
  margin-bottom: 10px;
  font-size: 1.2em;
}

.grid-item p {
  margin: 0;
  opacity: 0.9;
  font-size: 0.9em;
}
</style>
```

### 数据验证

```vue
<template>
  <div>
    <h2>实时数据验证</h2>
    
    <form class="validation-form" @submit.prevent="handleSubmit">
      <div class="form-group">
        <label>用户名:</label>
        <input 
          v-model="formData.username" 
          placeholder="3-20个字符，只能包含字母、数字和下划线"
          :class="{ 'error': errors.username }"
        />
        <span v-if="errors.username" class="error-message">{{ errors.username }}</span>
      </div>
      
      <div class="form-group">
        <label>邮箱:</label>
        <input 
          v-model="formData.email" 
          type="email"
          placeholder="请输入有效的邮箱地址"
          :class="{ 'error': errors.email }"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>
      
      <div class="form-group">
        <label>密码:</label>
        <input 
          v-model="formData.password" 
          type="password"
          placeholder="至少8个字符，包含字母和数字"
          :class="{ 'error': errors.password }"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
        
        <div class="password-strength">
          <div 
            v-for="(level, index) in 4" 
            :key="index"
            class="strength-bar"
            :class="{ 'active': index < passwordStrength }"
          ></div>
          <span class="strength-text">{{ passwordStrengthText }}</span>
        </div>
      </div>
      
      <div class="form-group">
        <label>确认密码:</label>
        <input 
          v-model="formData.confirmPassword" 
          type="password"
          placeholder="请再次输入密码"
          :class="{ 'error': errors.confirmPassword }"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>
      
      <div class="form-actions">
        <button 
          type="submit" 
          :disabled="!isFormValid"
          class="submit-btn"
        >
          注册
        </button>
        <button type="button" @click="resetForm" class="reset-btn">
          重置
        </button>
      </div>
      
      <div class="validation-summary">
        <h4>验证状态:</h4>
        <div class="status-item" :class="{ 'valid': !errors.username }">
          <span>用户名:</span>
          <span>{{ errors.username ? '❌ 无效' : '✓ 有效' }}</span>
        </div>
        <div class="status-item" :class="{ 'valid': !errors.email }">
          <span>邮箱:</span>
          <span>{{ errors.email ? '❌ 无效' : '✓ 有效' }}</span>
        </div>
        <div class="status-item" :class="{ 'valid': !errors.password }">
          <span>密码:</span>
          <span>{{ errors.password ? '❌ 无效' : '✓ 有效' }}</span>
        </div>
        <div class="status-item" :class="{ 'valid': !errors.confirmPassword }">
          <span>确认密码:</span>
          <span>{{ errors.confirmPassword ? '❌ 无效' : '✓ 有效' }}</span>
        </div>
      </div>
    </form>
  </div>
</template>

<script setup>
import { ref, computed, watchEffect } from 'vue'

const formData = ref({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const errors = ref({
  username: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// 验证规则
const validators = {
  username: (value) => {
    if (!value) return '用户名不能为空'
    if (value.length < 3) return '用户名至少需要3个字符'
    if (value.length > 20) return '用户名不能超过20个字符'
    if (!/^[a-zA-Z0-9_]+$/.test(value)) return '用户名只能包含字母、数字和下划线'
    return ''
  },
  
  email: (value) => {
    if (!value) return '邮箱不能为空'
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
    if (!emailRegex.test(value)) return '请输入有效的邮箱地址'
    return ''
  },
  
  password: (value) => {
    if (!value) return '密码不能为空'
    if (value.length < 8) return '密码至少需要8个字符'
    if (!/[a-zA-Z]/.test(value)) return '密码必须包含字母'
    if (!/[0-9]/.test(value)) return '密码必须包含数字'
    return ''
  },
  
  confirmPassword: (value, allValues) => {
    if (!value) return '请确认密码'
    if (value !== allValues.password) return '两次输入的密码不一致'
    return ''
  }
}

// 计算密码强度
const passwordStrength = computed(() => {
  const password = formData.value.password
  if (!password) return 0
  
  let strength = 0
  if (password.length >= 8) strength++
  if (/[a-z]/.test(password) && /[A-Z]/.test(password)) strength++
  if (/[0-9]/.test(password)) strength++
  if (/[^a-zA-Z0-9]/.test(password)) strength++
  
  return strength
})

const passwordStrengthText = computed(() => {
  const texts = ['很弱', '弱', '中等', '强', '很强']
  return texts[passwordStrength.value] || ''
})

// 表单是否有效
const isFormValid = computed(() => {
  return Object.values(errors.value).every(error => !error) &&
         Object.values(formData.value).every(value => value !== '')
})

// 使用 watchEffect 进行实时验证
watchEffect(() => {
  // 验证用户名
  errors.value.username = validators.username(formData.value.username)
  
  // 验证邮箱
  errors.value.email = validators.email(formData.value.email)
  
  // 验证密码
  errors.value.password = validators.password(formData.value.password)
  
  // 验证确认密码
  errors.value.confirmPassword = validators.confirmPassword(
    formData.value.confirmPassword, 
    formData.value
  )
})

const handleSubmit = () => {
  if (isFormValid.value) {
    console.log('表单提交:', formData.value)
    alert('注册成功！')
  }
}

const resetForm = () => {
  formData.value = {
    username: '',
    email: '',
    password: '',
    confirmPassword: ''
  }
}
</script>

<style scoped>
.validation-form {
  max-width: 500px;
  margin: 20px auto;
  padding: 30px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-group label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #333;
}

.form-group input {
  width: 100%;
  padding: 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-sizing: border-box;
  font-size: 16px;
  transition: border-color 0.3s;
}

.form-group input:focus {
  outline: none;
  border-color: #4CAF50;
  box-shadow: 0 0 0 2px rgba(76, 175, 80, 0.2);
}

.form-group input.error {
  border-color: #f44336;
}

.error-message {
  display: block;
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
}

.password-strength {
  display: flex;
  align-items: center;
  gap: 5px;
  margin-top: 10px;
}

.strength-bar {
  flex: 1;
  height: 4px;
  background: #e0e0e0;
  border-radius: 2px;
  transition: background 0.3s;
}

.strength-bar.active:nth-child(1) { background: #f44336; }
.strength-bar.active:nth-child(2) { background: #ff9800; }
.strength-bar.active:nth-child(3) { background: #4CAF50; }
.strength-bar.active:nth-child(4) { background: #2196F3; }

.strength-text {
  font-size: 12px;
  color: #666;
  margin-left: 5px;
}

.form-actions {
  display: flex;
  gap: 10px;
  margin-top: 30px;
}

.form-actions button {
  flex: 1;
  padding: 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: 500;
  transition: opacity 0.2s;
}

.submit-btn {
  background: #4CAF50;
  color: white;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.reset-btn {
  background: #f44336;
  color: white;
}

.form-actions button:hover:not(:disabled) {
  opacity: 0.9;
}

.validation-summary {
  margin-top: 30px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.validation-summary h4 {
  margin-top: 0;
  margin-bottom: 15px;
  color: #333;
}

.status-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  border-bottom: 1px solid #e0e0e0;
}

.status-item:last-child {
  border-bottom: none;
}

.status-item.valid {
  color: #4CAF50;
}
</style>
```

## 高级特性

### 清理副作用

```javascript
watchEffect((onCleanup) => {
  const controller = new AbortController()
  
  fetchData(source.value, { signal: controller.signal })
  
  // 清理函数：在副作用重新执行或停止时调用
  onCleanup(() => {
    controller.abort()
  })
})
```

### 停止 watchEffect

```javascript
const stop = watchEffect(() => {
  console.log('执行副作用')
})

// 在需要时停止
stop()
```

### 刷新时机控制

```javascript
// 默认：pre（组件更新前）
watchEffect(() => {
  console.log('pre flush')
})

// post：组件更新后
watchEffect(() => {
  console.log('post flush')
}, { flush: 'post' })

// sync：同步执行
watchEffect(() => {
  console.log('sync flush')
}, { flush: 'sync' })
```

### 调试功能

```javascript
watchEffect(
  () => {
    console.log('副作用执行')
  },
  {
    onTrack(e) {
      console.log('追踪依赖:', e)
    },
    onTrigger(e) {
      console.log('触发更新:', e)
    }
  }
)
```

## 最佳实践

### 1. 选择合适的 API

```javascript
// 明确知道要监听哪些数据时，使用 watch
watch([ref1, ref2], ([newVal1, newVal2]) => {
  console.log('明确的监听')
})

// 需要自动依赖收集时，使用 watchEffect
watchEffect(() => {
  console.log(ref1.value, ref2.value, ref3.value)
})
```

### 2. 避免无限循环

```javascript
// 好的做法：使用条件判断
watchEffect(() => {
  if (someCondition.value) {
    data.value = processData(data.value)
  }
})

// 避免：无条件修改可能导致循环
watchEffect(() => {
  data.value++ // 可能导致无限循环
})
```

### 3. 及时清理副作用

```javascript
watchEffect((onCleanup) => {
  const subscription = source.subscribe(data => {
    console.log('收到数据:', data)
  })
  
  onCleanup(() => {
    subscription.unsubscribe()
  })
})
```

### 4. 性能优化

```javascript
// 使用 throttle 防止频繁执行
let throttleTimer = null

watchEffect(() => {
  if (throttleTimer) return
  
  throttleTimer = setTimeout(() => {
    console.log('节流执行')
    throttleTimer = null
  }, 100)
})
```

## 注意事项

1. **依赖自动收集**：watchEffect 会自动收集依赖，但也可能导致意外执行。
2. **无法访问旧值**：与 watch 不同，watchEffect 无法访问新旧值。
3. **立即执行**：watchEffect 会立即执行一次，需要注意初始状态。
4. **性能考虑**：复杂计算或大量数据时要注意性能影响。
5. **清理副作用**：忘记清理可能导致内存泄漏。
6. **调试困难**：自动依赖收集使得调试相对困难。
7. **类型推断**：在 TypeScript 中可能需要额外的类型提示。

## 常见问题

### Q: watchEffect 和 watch 应该选择哪个？

A: 如果需要明确监听特定数据或访问新旧值，使用 watch；如果需要自动依赖收集和简洁语法，使用 watchEffect。

### Q: watchEffect 会影响性能吗？

A: 合理使用不会明显影响性能，但复杂的副作用可能需要优化。

### Q: 如何在 watchEffect 中访问旧值？

A: watchEffect 不支持访问旧值，如需此功能应使用 watch。

### Q: watchEffect 会在何时停止？

A: 在组件卸载时自动停止，也可以手动调用停止函数。

## 总结

Vue 3 的 watchEffect 提供了一种简洁而强大的响应式副作用处理方式：

1. **自动依赖收集**：无需手动指定监听源，代码更简洁
2. **立即执行**：在创建时立即执行，适合初始化逻辑
3. **灵活的清理机制**：通过 onCleanup 回调清理副作用
4. **可控的执行时机**：支持 pre、post、sync 三种刷新时机
5. **适用于多种场景**：DOM 操作、自动保存、数据验证等

合理使用 watchEffect 可以让代码更加简洁和优雅，但也要注意其特性和限制，在合适的场景下选择最合适的 API。在实际项目中，watchEffect 和 watch 各有优势，应根据具体需求灵活选择。