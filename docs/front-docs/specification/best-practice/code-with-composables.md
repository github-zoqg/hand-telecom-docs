---
sidebar_position: 2
sidebar_label: "组合式函数Composables"
---

# 使用组合式函数编写业务代码

## 核心原则
对于无需复用的业务场景，新页面开发建议直接使用 Pinia 进行状态管理，这是一个更简单的开发方案

## 什么是组合式函数？
组合式函数是 Vue3 提供的一种逻辑复用方案，本质是一个**封装了特定业务逻辑的函数**，通过组合 Vue 的响应式 API（如 `ref`、`reactive`、`watch` 等），抽离分散在组件中的重复逻辑，最终返回可供组件使用的响应式数据和操作方法。

👉 详细定义可参考 [Vue3 官方文档 - 组合式函数](https://cn.vuejs.org/guide/reusability/composables.html#composables)

## 组合式函数的核心优势
1. **逻辑复用**：抽离同类业务的重复逻辑，避免复制粘贴，降低维护成本
2. **数据与 UI 解耦**：将数据处理、接口请求等逻辑与 UI 渲染分离，组件更专注于视图展示
3. **代码结构化**：复杂组件的逻辑可按功能拆分到不同组合式函数，便于阅读和维护
4. **响应式兼容**：天然适配 Vue 响应式系统，返回的数据可直接在模板中使用，自动响应状态变化

## 基础使用示例

### 定义组合式函数（`useXxx.ts`）
```typescript
/**
 * 封装 XXX 表格数据的获取与操作逻辑
 * 命名规范：使用 useXXX 作为函数名，遵循 Vue 官方约定
 */
export function useXxxData() {
  // 响应式查询条件（与查询栏组件双向绑定）
  const query = reactive({
    keyword: '',
    status: '',
    startTime: '',
    endTime: ''
  })

  // 响应式表格数据（与表格组件联动）
  const xxxData = ref<XxxDataType[]>([])
  // 加载状态（控制表格加载动画）
  const loading = ref(false)

  /**
   * 加载表格数据的核心方法
   * 封装接口请求、数据处理、加载状态控制逻辑
   */
  async function loadXxxData() {
    try {
      loading.value = true
      // 调用接口获取数据（可结合 axios/request 封装）
      const res = await getXxxList(query)
      xxxData.value = res.data.list
    } catch (error) {
      console.error('加载 XXX 数据失败：', error)
      xxxData.value = []
    } finally {
      loading.value = false
    }
  }

  // 返回组件所需的响应式数据和方法
  return {
    query,
    xxxData,
    loading,
    loadXxxData
  }
}
```

### 在组件中使用组合式函数

```vue
<template>
  <!-- 查询栏：双向绑定查询条件，查询事件触发数据加载 -->
  <QueryBar 
    v-model:query="query" 
    @query="loadXxxData"
  />

  <!-- 表格：绑定组合式函数返回的响应式数据 -->
  <XxxTable 
    :table-data="xxxData" 
    :loading="loading"
    @refresh="loadXxxData"
  />
</template>

<script setup lang="ts">
// 导入组合式函数
import { useXxxData } from './xxx.ts'
// 解构获取响应式数据和方法（直接在模板中使用）
const { query, xxxData, loading, loadXxxData } = useXxxData()

// 页面初始化时加载数据
onMounted(() => {
  loadXxxData()
})
</script>
```

## 最佳实践
1. **命名规范**：组合式函数统一使用 useXXX 命名（如 useTableData、useFormValidator），便于识别和导入
2. **职责单一**：一个组合式函数聚焦单一功能（如仅处理表格数据、仅封装表单验证），避免功能臃肿
3. **传递依赖**：若需依赖外部参数（如接口地址、默认参数），通过函数参数传入，增强灵活性
```typescript
// 示例：通过参数配置接口地址
export function useXxxData(apiUrl: string) {
 async function loadXxxData() {
   const res = await request.get(apiUrl, { params: query })
   // ...
 }
}
```