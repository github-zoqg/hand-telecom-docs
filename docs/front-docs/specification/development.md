---
sidebar_position: 2
sidebar_label: "开发规范"
---

# 开发规范

开发工具及代码风格、模块划分和代码开发规范

## 开发工具

- 统一使用 `VSCode` 编辑器
- 安装 `Prettier` 插件，统一代码格式
- 安装 `ESLint` 插件，统一代码规范

![分支流转流程](./img/vscode.png)
![分支流转流程](./img/prettier.png)
![分支流转流程](./img/eslint.png)

## 命名规范

- **文件夹**统一使用`-`连接，例：`home-page`
- **js/ts/css 文件**统一使用驼峰命名，例：`homePage`
- **组件**以帕斯卡规则命名，例：`HomePage`
- **样式类**统一使用`-`或`_`连接，例：`xxxx-xxxx`, `xxxx_xxx`
- **变量**以小驼峰规则命名，例：`xxxXxxx`
- 非常长的变量名需要加注释

## 模块划分

- 页面入口模块结构需要与路由层级保持一致，入口文件统一命名为 index.vue
- 业务组件统一放到 index.vue 入口文件的同级 `components` 目录下
- 如果需要使用组合式函数，统一放至 `hooks` 目录下，文件名以驼峰规则命名

```
my-page/
│
├── components/
│   ├── XxxxXxxx.vue
│   └── YyyyYyyy.vue
│
├── hooks/
│   ├── useXxxxXxxx.ts
│   └── useYyyyYyyy.ts
└── index.vue
```

## 注释规范

- 业务上的变量名称必须添加注释
```javascript
// 采购订单头信息
const orderHeaderInfo = ref([]);
```
- 工具函数必须添加注释
```typescript
/**
 * 格式化采购订单时间
 * @param orderTime - 原始订单时间（时间戳/字符串）
 * @param format - 时间格式（默认：YYYY-MM-DD HH:mm:ss）
 * @returns 格式化后的时间字符串
 * @example formatOrderTime(1718000000000) // '2024-06-10 12:00:00'
 */
export function formatOrderTime(
    orderTime: number | string,
    format = 'YYYY-MM-DD HH:mm:ss'
): string {
    if (!orderTime) return '-';
    return dayjs(orderTime).format(format);
}
```

- 具有复杂业务逻辑的函数必须按照步骤，添加注释
```javascript
function doSomething() {
    // 步骤1：xxxx
    
    // 步骤2：xxxx
    
    // 步骤3：xxxx
}
```
- vue组件在顶部添加注释，说明组件的职责
```vue
<!--
 * @Author: taohaoyuan50050 haoyuan.tao@hand-china.com
 * @Date: 2025-10-29 09:21:13
 * @LastEditors: taohaoyuan50050 haoyuan.tao@hand-china.com
 * @LastEditTime: 2025-10-29 09:30:13
 * @FilePath: \xcerp-app-poc-front\src\components\hand-lov\index.vue
 * @Description: lov组件，一个有放大镜的下拉框，用于适配特定的后端接口
-->
<template>
  xxx
</template>
```
