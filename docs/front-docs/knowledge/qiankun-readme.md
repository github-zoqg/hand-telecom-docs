---
sidebar_position: 1
sidebar_label: "微前端-qiankun"
---

# qiankun 主子应用使用手册

## 主应用

1. main.ts 入口文件

   使用`qiankun`的`registerMicroApps`方法注册子应用，`start`方法启动。

```js
import { registerMicroApps, start } from "qiankun";
registerMicroApps([
  {
    name: "appPocFront",
    entry: "//localhost:8081",
    container: "#sub-app",
    activeRule: "/appPocFront",
  },
  // ...其他子应用
]);
start({
  sandbox: {
    experimentalStyleIsolation: true,
  },
});
```

2. router.ts 路由配置

   为子应用建立单独的路由，避免主应用和子应用路由冲突。

```js
import { createRouter, createWebHistory } from "vue-router";
const router = createRouter({
  history: createWebHistory(),
  routes: [
    // 子应用路由
    {
      path: "/appPocFront/:pathMatch(.*)*",
      name: "xcerpPoc",
      component: Layout,
    },
  ],
});
```

3. 给子应用添加挂载 dom 节点

   在指定区域为子应用添加挂载节点 `<div id="sub-app"></div>`，`id`需要与应用注册时的`container`保持一致。

## 子应用

1. vite.config.ts 配置

   需要引入`vite-plugin-qiankun`插件

```js
export default ({ mode }: ConfigEnv): UserConfig => {
  return {
    plugins: [
      qiankun("appPocFront", {
        // 微应用名称，与主应用注册时一致
        useDevMode: true,
      }),
      // ...
    ],
    // ...其他配置
  };
};
```

2. main.ts 入口文件

   与普通项目启动稍有差别，需要判断是否使用`renderWithQiankun`方法启动应用

```js
import { renderWithQiankun, qiankunWindow, QiankunProps } from 'vite-plugin-qiankun/dist/helper'

async function render(props: any = {}) {
  const { container } = props as any
  instance = createApp(Root)
  // ...something
  instance.mount(container ? container.querySelector('#xcerp-container') : '#xcerp-container')
}

renderWithQiankun({
  bootstrap: () => {
    console.log('[vue3] vue app bootstraped')
    return Promise.resolve()
  },
  mount: props => {
    try {
      console.log('moduleB is work')
      // 作为子应用启动
      render(props)
    } catch (error) {
      console.log('渲染出错', error)
    }
  },
  unmount: props => {
    console.log('[vue3] vue app unmount')
  },
  update: (props: QiankunProps): void | Promise<void> => {
    throw new Error('Function not implemented.')
  },
})

// 独立运行时，直接渲染
if (!qiankunWindow.__POWERED_BY_QIANKUN__) {
// 挂载子应用本身的dom
  render()
}
```

3. router 配置

   作为子应用时需要配置 base 为'子应用路由前缀'

```js
import { qiankunWindow } from "vite-plugin-qiankun/dist/helper";

const base = qiankunWindow.__POWERED_BY_QIANKUN__
  ? import.meta.env.VITE_APP_BASE
  : "/";
const router = createRouter({
  routes: [...staticRoutes],
  history: createWebHistory(base),
});

export default router;
```

4. 打包部署

   需要为子应用配置 nginx 代理，允许主应用进行跨域访问。
