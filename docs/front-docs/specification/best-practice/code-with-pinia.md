---
sidebar_position: 3
sidebar_label: "状态管理pinia"
---

# 使用 Pinia 编写业务代码

Pinia 是 Vue3 官方推荐的状态管理库，相比 Vuex 更简洁、更贴合 Composition API 语法，适用于组件间状态共享、全局状态管理场景。本文将介绍 Pinia 的核心使用方式、规范及最佳实践。

## 一、定义 Store（状态容器）

Store 是 Pinia 的核心概念，用于封装**响应式状态**和**状态操作方法**，本质是通过 `defineStore` 函数定义的独立模块。

### 核心原则
- 仅存放响应式状态（`ref`/`reactive` 包裹）和操作状态的方法
- 职责单一：不包含复杂业务逻辑，仅负责状态的存储与修改
- 禁止嵌套定义，按业务模块拆分独立 Store

### 示例 1：基础类型状态（ref 包裹）
```typescript
import { defineStore } from 'pinia'
import { ref } from 'vue'


export const useNumberStore = defineStore('number', () => {
    // 响应式状态：数字计数器
    const numberRef = ref<number>(0)
    
    // 状态操作方法：修改计数器（仅负责状态赋值，无复杂逻辑）
    function increase() {
        numberRef.value++
    }
    
    // 仅暴露响应式状态和操作方法
    return {
        numberRef,
        increase
    }
})
```

### 示例 2：复杂类型状态（reactive 包裹）
```typescript
import { defineStore } from 'pinia'
import { reactive } from 'vue'

// 定义状态接口，避免 any 类型，提升类型安全性
interface SomeInfo {
    value1: string
    value2: string
    value3: string
    value4: string
}

/**
* 复杂信息存储 Store
  */
export const useSomeInfoStore = defineStore('someInfo', () => {
    // 响应式状态：复杂对象类型
    const someInfo = reactive<SomeInfo>({
      value1: 'qaq',
      value2: '',
      value3: '',
      value4: '',
    })
    
    // 批量设置状态
    function setSomeInfo(info: Partial<SomeInfo>) {
        Object.assign(someInfo, info)
    }
    
    // 清空所有状态
    function clearSomeInfo() {
    Object.keys(someInfo).forEach(key => {
            someInfo[key as keyof SomeInfo] = ''
        })
    }
    
    // 单独设置 value1
    function setValue1(value: string) {
        someInfo.value1 = value
    }
    
    return {
        someInfo,
        setSomeInfo,
        clearSomeInfo,
        setValue1
    }
})
```

## 二、使用 Store（组件中调用）

在 Vue 单文件组件中，直接导入 Store 即可使用其状态和方法。需注意响应式状态的解构方式：
- `ref` 类型状态：需通过 `storeToRefs` 解构，保持响应式
- `reactive` 类型状态：直接解构即可，天然保留响应式

### 完整使用示例
```vue
<template>
  <div class="store-demo">
    <h3>数字计数器 Store 示例</h3>
    <div class="demo-item">当前值：{{ numberRef }}</div>
    <a-button @click="increase" type="primary">数字 +1</a-button>

    <h3>复杂信息 Store 示例</h3>
    <div class="demo-item">value1：{{ someInfo.value1 }}</div>
    <a-button @click="setValue1Randomly" type="default">随机设置 value1</a-button>
    <a-button @click="clearSomeInfo" type="danger" style="margin-left: 8px;">清空所有信息</a>
  </div>
</template>

<script setup lang="ts">
import { useNumberStore } from './useNumberStore.ts'
import { useSomeInfoStore } from './useSomeInfoStore.ts'
import { storeToRefs } from 'pinia'
import { Button as AButton } from 'ant-design-vue'

// 组件命名
defineOptions({
  name: 'PiniaUsageDemo'
})

// 1. 使用 基础类型 Store（ref 状态需通过 storeToRefs 解构）
const numberStore = useNumberStore()
const { increase } = numberStore // 方法直接解构
const { numberRef } = storeToRefs(numberStore) // ref 状态通过 storeToRefs 保持响应式

// 2. 使用 复杂类型 Store（reactive 状态直接解构）
const { someInfo, setValue1, clearSomeInfo } = useSomeInfoStore()

// 组件内业务逻辑：生成随机字符串设置 value1
function setValue1Randomly() {
  const randomStr = Math.random().toString(36).substring(2)
  setValue1(randomStr)
}
</script>
```

## 三、Pinia 核心优势
1. **组件解耦**：组件间无需通过 props/emit 传递状态，直接导入 Store 即可获取，简化组件通信
2. **响应式天然支持**：Store 状态默认响应式，修改后组件自动更新，无需额外处理
3. **语法简洁**：摒弃 Vuex 的 mutations/actions/modules 冗余设计，仅保留 Store 核心概念
4. **TypeScript 友好**：天然支持类型推导，无需额外类型声明，开发体验更优
5. **全局统一管理**：状态集中存储在 Store，便于调试和维护，避免状态分散在多个组件中

## 四、Store 开发规范
1. **禁止嵌套定义**：Store 需独立拆分，按业务模块命名（如 `useUserStore`、`useCartStore`），不可嵌套在其他 Store 中
2. **暴露内容限制**：Store 返回值仅允许是响应式状态（`ref`/`reactive`）或操作方法，禁止暴露普通变量
3. **方法调用限制**：Store 内部方法不可调用其他 Store 的方法，避免形成依赖耦合
4. **业务逻辑边界**：Store 仅负责状态的赋值与修改，复杂业务逻辑（如接口请求、数据处理）需放在组件或组合式函数中
5. **命名规范**：
    - Store 函数名：`use + 业务名称 + Store`（如 `useOrderStore`）
    - Store 唯一标识：与函数名一致（小写），如 `order`
    - 状态变量：使用 `XXXRef`（`ref` 类型）或直接命名（`reactive` 类型），明确区分响应式类型
