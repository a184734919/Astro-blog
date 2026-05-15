---
title: Vue 3 组合式最佳实践
published: 2024-06-30
description: '组合式函数设计原则的详细介绍和学习笔记'
image: ''
tags: ["Vue3","最佳实践"]
category: 'Vue3'
draft: false
lang: 'zh-CN'
---

## Vue 3 组合式 API

组合式 API 是 Vue 3 的核心特性，它提供了更灵活的代码组织方式。

## setup 函数

```javascript
import { ref, reactive, computed, onMounted } from 'vue';

export default {
  setup() {
    // 响应式数据
    const count = ref(0);
    const state = reactive({
      name: 'Vue 3',
      version: '3.0'
    });

    // 计算属性
    const doubleCount = computed(() => count.value * 2);

    // 方法
    function increment() {
      count.value++;
    }

    // 生命周期
    onMounted(() => {
      console.log('组件已挂载');
    });

    // 返回给模板使用
    return {
      count,
      state,
      doubleCount,
      increment
    };
  }
};
```

## Script Setup 语法糖

```vue
<script setup>
import { ref, computed } from 'vue';

// 直接定义响应式数据
const count = ref(0);
const double = computed(() => count.value * 2);

// 直接定义方法
function increment() {
  count.value++;
}
</script>

<template>
  <button @click="increment">{{ count }}</button>
  <p>双倍: {{ double }}</p>
</template>
```

## 组合式函数

```javascript
// useCounter.js
import { ref, computed } from 'vue';

export function useCounter(initialValue = 0) {
  const count = ref(initialValue);
  const double = computed(() => count.value * 2);
  
  function increment() {
    count.value++;
  }
  
  function decrement() {
    count.value--;
  }
  
  return {
    count,
    double,
    increment,
    decrement
  };
}

// 使用
import { useCounter } from './useCounter';

const { count, increment } = useCounter(10);
```

## 总结

组合式 API 让代码更易复用、更易测试、更易理解。