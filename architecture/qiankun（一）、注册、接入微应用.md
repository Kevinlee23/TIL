#### 微前端是什么？

微前端是借鉴了后端微服务的架构模式，将 Web 应用由单一的单体应用转变成多个小型前端应用，聚合为一的应用

#### 原则

【独立运行、独立部署、独立开发】

#### 基座的模式

1. 通用中心路由基座：只有公共功能的主应用
2. 特定中心路由基座：一个包含业务代码的项目

#### 注册微应用的方式 ———— 自动模式（推荐）

使用 registerMicroApps + start, 路由变化加载微应用（一旦浏览器的 url 发生变化，便会自动触发 qiankun 的匹配）

```javascript
// 主应用（基座）安装
yarn add qiankun
```

```javascript
// 主应用 main.js(ts)
import { registerMicroApps, start } from "qiankun";

// 1. 微应用配置
const MICRO_CONFIG = [
  {
    // 基础配置
    name: "appName", // 应用名字
    entry: "http://localhost:host", // 加载这个html 解析里面的js 动态的执行 (! 子应用必须支持跨域)fetch
    container: "#containerId", // 挂载具体容器ID
    activeRule: "appActiveRoute", // 根据路由匹配, 激活子应用
    // 下发给子应用的状态, 方法
    props: {
      globalState: ...,
      utils: ...
    },
  },
  // 其他微应用
  ...
];

// 2. 注册微应用
registerMicroApps(MICRO_CONFIG)

// 3. 启动微服务
start()
```

#### 微应用挂载场景 ———— 根 DOM 中与主应用同级挂载，显示当前应用

```vue
<template>
  <div id="app">
    <div
      v-show="location.hash.startsWith('#/app1')"
      id="sub-app1-container"
    ></div>
    <div
      v-show="location.hash.startsWith('#/app2')"
      id="sub-app2-container"
    ></div>
    <div
      v-show="location.hash.startsWith('#/app3')"
      id="sub-app3-container"
    ></div>
  </div>
</template>
```

#### 微应用接入

##### 1. 微应用的入口文件修改

```javascript
// 修改 webpack_public_path
// 微应用/src/const/public-path.js

if (window.__POWERED_BY_QIANKUN__) {
  __webpack_public_path__ = window.__INJECTED_PUBLIC_PATH_BY_QIANKUN__;
}
```

##### 2. 微应用 webpack 新增配置

```javascript
module.exports = {
  devServer: {
    port: 8081, // 要与主应用中配置的一致
    disableHostCheck: true, // 关闭主机检查, 使微应用可以被 fetch
    headers: {
      "Access-Control-Allow-Origin": "*", // 子应用必须允许跨域
    },
    configureWebpack: {
      outptu: {
        library: `${name}-[name]`, // 微应用的包名, 需要与主应用中注册的一致
        libraryTarget: "umd",
        jsonpFunction: `webpackJsonp_${name}`,
      },
    },
  },
};
```

##### 3. 微应用添加声明周期（vue3）

```typescript
import { App, createApp } from "vue";
import root from "./App.vue";
import store from "./store";
import router from "./router";

import "./const/public-path.js";

// 使用ts需要定义接口
declare global {
  interface Window {
    __POWERED_BY_QIANKUN__?: string;
  }
}

let instance: App<Element> | null;

// 1. 注册方法
function render(props: { container: Element | string; utils: any }) {
  const { container, utils } = props;

  instance = createApp(root);
  instance.use(router).use(store);

  instance.config.globalProperties.parentUtils = utils;

  // 判断是从 qiankun 启动还是独立运行
  const el =
    container instanceof Element
      ? (container.querySelector("#app") as Element)
      : (container as String);
  instance.mount(el);
}

// 独立运行时
if (!window.__POWERED_BY_QIANKUN__) {
  render({ container: "#app", utils: {} });
}

// 2. 导出的生命周期
/**
 * bootstrap只会调用一次, 通常在这里做一些全局变量的初始化
 */
export async function bootstrap() {
  // empty
}

/**
 * 应用每次进入都会调用的方法, 通常在这里触发应用的渲染方法
 */
export async function mount(props) {
  render(props);
}

/**
 * 应用每次卸载都会调用的方法, 通常在这里卸载微应用的实例
 */
export async function unmount() {
  instance!.unmount();
  instance!._container!.innerHTML = "";
  instance = null;
}
```
