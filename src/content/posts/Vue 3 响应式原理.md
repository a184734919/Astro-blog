---
title: Vue 3 响应式原理
published: 2024-05-03
description: 'Proxy 响应式实现的详细介绍和学习笔记'
image: ''
tags: ["Vue3","原理"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## 概述

Vue 3 的响应式系统是基于 ES6 的 Proxy 对象实现的，相比 Vue 2 的 Object.defineProperty 方案，Proxy 提供了更强大的拦截能力和更好的性能。理解 Vue 3 的响应式原理对于深入掌握 Vue 框架和优化应用性能至关重要。

## 核心概念

### Proxy 基础

Proxy 是 ES6 新增的构造函数，用于创建一个对象的代理，从而实现对该对象的基本操作拦截和自定义。

```javascript
const target = {
  name: 'Vue 3',
  version: '3.0'
};

const handler = {
  get(target, property, receiver) {
    console.log(`获取属性: ${property}`);
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    console.log(`设置属性: ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  }
};

const proxy = new Proxy(target, handler);

proxy.name; // 输出: 获取属性: name
proxy.version = '3.1'; // 输出: 设置属性: version = 3.1
```

### Vue 3 响应式系统组成

Vue 3 的响应式系统由以下核心部分组成：

1. **Proxy**: 拦截对象操作
2. **Track**: 依赖收集
3. **Trigger**: 触发更新
4. **Effect**: 副作用函数
5. **WeakMap**: 存储响应式对象的依赖关系

## 基本用法

### 1. ref 响应式

ref 用于创建包装式的响应式对象，主要处理基本数据类型。

```javascript
import { ref } from 'vue';

// 创建 ref
const count = ref(0);
const name = ref('Vue 3');

// 访问和修改
console.log(count.value); // 0
count.value = 10;

// 在模板中自动解包
// <template>{{ count }}</template> // 10
```

### 2. reactive 响应式

reactive 用于创建深度响应式对象。

```javascript
import { reactive } from 'vue';

// 创建 reactive
const user = reactive({
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    street: '长安街'
  }
});

// 访问和修改
console.log(user.name); // 张三
user.name = '李四';
user.address.city = '上海';
```

### 3. computed 计算属性

computed 创建基于其他响应式数据的计算属性。

```javascript
import { ref, computed } from 'vue';

const count = ref(0);
const doubleCount = computed(() => count.value * 2);

console.log(doubleCount.value); // 0
count.value = 5;
console.log(doubleCount.value); // 10

// 可写的计算属性
const firstName = ref('张');
const lastName = ref('三');
const fullName = computed({
  get() {
    return firstName.value + lastName.value;
  },
  set(newValue) {
    [firstName.value, lastName.value] = newValue.split('');
  }
});

fullName.value = '李四'; // firstName.value = '李', lastName.value = '四'
```

### 4. watch 侦听器

watch 用于侦听响应式数据的变化。

```javascript
import { ref, reactive, watch } from 'vue';

const count = ref(0);
const user = reactive({ name: '张三' });

// 侦听 ref
watch(count, (newVal, oldVal) => {
  console.log(`count 变化: ${oldVal} -> ${newVal}`);
});

// 侦听 reactive 对象的属性
watch(() => user.name, (newVal, oldVal) => {
  console.log(`user.name 变化: ${oldVal} -> ${newVal}`);
});

// 侦听多个源
watch([count, () => user.name], ([newCount, newName], [oldCount, oldName]) => {
  console.log(`多个值变化: ${oldCount} -> ${newCount}, ${oldName} -> ${newName}`);
});

// 立即执行
watch(count, (newVal) => {
  console.log('立即执行:', newVal);
}, { immediate: true });

// 深度侦听
const deepObj = reactive({ nested: { value: 1 } });
watch(deepObj, (newVal) => {
  console.log('深度侦听触发');
}, { deep: true });
```

### 5. watchEffect

watchEffect 自动追踪依赖并在依赖变化时重新执行。

```javascript
import { ref, watchEffect } from 'vue';

const count = ref(0);
const doubled = ref(0);

watchEffect(() => {
  doubled.value = count.value * 2;
  console.log(`watchEffect 执行: count = ${count.value}, doubled = ${doubled.value}`);
});

count.value = 5; // 自动触发 watchEffect
```

## 实际应用

### 1. 手动实现简易响应式系统

```javascript
// 简易的响应式系统实现
let activeEffect = null;

class Dep {
  constructor() {
    this.subscribers = new Set();
  }
  
  depend() {
    if (activeEffect) {
      this.subscribers.add(activeEffect);
    }
  }
  
  notify() {
    this.subscribers.forEach(effect => effect());
  }
}

const targetMap = new WeakMap();

function track(target, key) {
  let depsMap = targetMap.get(target);
  if (!depsMap) {
    targetMap.set(target, (depsMap = new Map()));
  }
  
  let dep = depsMap.get(key);
  if (!dep) {
    depsMap.set(key, (dep = new Dep()));
  }
  
  dep.depend();
}

function trigger(target, key) {
  const depsMap = targetMap.get(target);
  if (!depsMap) return;
  
  const dep = depsMap.get(key);
  if (dep) {
    dep.notify();
  }
}

function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      const result = Reflect.get(target, key, receiver);
      track(target, key);
      if (typeof result === 'object' && result !== null) {
        return reactive(result);
      }
      return result;
    },
    
    set(target, key, value, receiver) {
      const oldValue = target[key];
      const result = Reflect.set(target, key, value, receiver);
      if (oldValue !== value) {
        trigger(target, key);
      }
      return result;
    }
  });
}

function effect(fn) {
  activeEffect = fn;
  fn();
  activeEffect = null;
}

// 使用示例
const state = reactive({
  count: 0,
  doubled: 0
});

effect(() => {
  state.doubled = state.count * 2;
  console.log(`Effect 执行: count = ${state.count}, doubled = ${state.doubled}`);
});

state.count = 5; // 触发 effect
```

### 2. 响应式工具函数

```javascript
import { ref, reactive, computed, toRef, toRefs } from 'vue';

// toRef - 为 reactive 对象的属性创建 ref
const user = reactive({
  name: '张三',
  age: 25
});

const userName = toRef(user, 'name');
console.log(userName.value); // 张三
userName.value = '李四';
console.log(user.name); // 李四

// toRefs - 将 reactive 对象转换为普通对象，每个属性都是 ref
const userRefs = toRefs(user);
console.log(userRefs.name.value); // 李四

// 在组件中使用
<script setup>
import { reactive, toRefs } from 'vue';

const props = defineProps(['user']);

// 将 props 转换为 ref
const { name, age } = toRefs(props);
</script>
```

### 3. 自定义响应式工具

```javascript
import { ref, watch, computed } from 'vue';

// 创建具有本地存储的响应式数据
export function useLocalStorage(key, defaultValue) {
  const storedValue = localStorage.getItem(key);
  const value = ref(storedValue ? JSON.parse(storedValue) : defaultValue);
  
  watch(value, (newValue) => {
    localStorage.setItem(key, JSON.stringify(newValue));
  }, { deep: true });
  
  return value;
}

// 使用
const theme = useLocalStorage('theme', 'light');

// 创建防抖的响应式数据
export function useDebouncedRef(value, delay = 300) {
  const timeout = ref(null);
  const debouncedValue = ref(value);
  
  watch(value, (newValue) => {
    if (timeout.value) {
      clearTimeout(timeout.value);
    }
    
    timeout.value = setTimeout(() => {
      debouncedValue.value = newValue;
    }, delay);
  });
  
  return debouncedValue;
}

// 使用
const searchQuery = ref('');
const debouncedQuery = useDebouncedRef(searchQuery, 500);

// 创建可重置的响应式数据
export function useResettableRef(initialValue) {
  const value = ref(initialValue);
  
  function reset() {
    value.value = initialValue;
  }
  
  return {
    value,
    reset
  };
}

// 使用
const { value: count, reset } = useResettableRef(0);
```

### 4. 响应式性能优化

```javascript
import { ref, shallowRef, triggerRef, readonly, shallowReactive } from 'vue';

// shallowRef - 只有.value 访问是响应式的
const shallowData = shallowRef({
  nested: {
    value: 1
  }
});

shallowData.value.nested.value = 2; // 不会触发更新
triggerRef(shallowData); // 手动触发更新

// readonly - 创建只读的响应式对象
const original = reactive({ count: 0 });
const copy = readonly(original);

// copy.count = 1; // 警告：不能修改只读对象

// shallowReactive - 只有第一层属性是响应式的
const state = shallowReactive({
  nested: {
    value: 1
  }
});

state.nested.value = 2; // 不会触发更新
state.nested = { value: 2 }; // 会触发更新
```

## 注意事项

### 响应式限制

1. **Proxy 限制**: 不支持所有类型的对象
2. **数组索引**: 直接通过索引设置数组项可能不响应
3. **添加属性**: 对响应式对象添加新属性需要特殊处理
4. **解构**: 解构 reactive 对象会失去响应性

### 常见错误

```javascript
// 错误 1: 解构失去响应性
const user = reactive({ name: '张三', age: 25 });
const { name } = user; // name 不是响应式的

// 正确做法
const { name } = toRefs(user); // name 是响应式的

// 错误 2: 直接替换 reactive 对象
const state = reactive({ count: 0 });
state = reactive({ count: 1 }); // 错误

// 正确做法
Object.assign(state, { count: 1 });

// 错误 3: 在模板中忘记 .value
<script setup>
const count = ref(0);
</script>

<template>
  <!-- 错误: {{ count.value }} -->
  <!-- 正确: {{ count }} -->
</template>

// 错误 4: 嵌套过深的响应式对象导致性能问题
const deepObject = reactive({
  level1: {
    level2: {
      level3: {
        // 太深
      }
    }
  }
});

// 正确做法: 使用 shallowRef 或拆分对象
```

### 性能优化建议

1. **合理使用 ref 和 reactive**: 基本类型用 ref，对象用 reactive
2. **避免不必要的响应式**: 静态数据不需要响应式
3. **使用 shallowRef/shallowReactive**: 对于大型数据结构
4. **计算属性缓存**: 充分利用计算属性的缓存
5. **watch 优化**: 合理设置侦听选项

## 总结

Vue 3 的响应式系统是一个强大而高效的解决方案，具有以下特点：

### 核心优势

1. **更强大的拦截**: Proxy 提供更全面的拦截能力
2. **更好的性能**: 相比 Vue 2 的响应式系统性能更优
3. **更小的体积**: 模块化设计支持 Tree-shaking
4. **更好的 TypeScript 支持**: 完整的类型推断

### 关键概念

1. **Proxy**: 响应式的基础
2. **Track/Trigger**: 依赖收集和触发机制
3. **Effect**: 副作用函数
4. **WeakMap**: 高效的依赖存储

### 实践建议

1. **理解原理**: 深入理解响应式原理有助于编写更好的代码
2. **合理选择**: 根据场景选择合适的响应式 API
3. **性能意识**: 注意响应式带来的性能开销
4. **最佳实践**: 遵循 Vue 3 的响应式最佳实践

Vue 3 的响应式系统是框架的核心，掌握它将帮助你更好地理解和使用 Vue 框架，构建高性能的应用。