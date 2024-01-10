### 中间件 (Middleware)

Nuxt 提供了中间件来在导航到特定路由之前（或全局导航）运行代码

有三种类型：

- 匿名中间件，直接在页面内定义
- 命名路由中间件，放在 middleware/ 目录下，并在页面上使用时通过异步自动加载
- 全局路由中间件，放在 middleware/ 目录下，并以 .global 后缀结尾，例如: `menu-middleware.global.ts`，每次路由更改时运行

命名中间件定义：

```ts
import {
  defineNuxtRouteMiddleware,
  navigateTo,
  abortNavigation,
} from "#imports";

export default defineNuxtRouteMiddleware((to, from) => {
  if (to.params.id === "1") {
    return abortNavigation();
  }
  // 在实际应用中，你可能不会将每个路由重定向到 `/`
  // 但是在重定向之前检查 `to.path` 是很重要的，否则可能会导致无限重定向循环
  if (to.path !== "/") {
    return navigateTo("/");
  }
});
```

提供了两种全局可用的辅助函数:

- navigateTo: 重定向
- abortNavigation: 终止导航，并可选择提供错误信息

因此根据不同的返回值来确定中间件的处理结果：

- `return navigateTo('/')` 重定向到给定的路径，并在服务器端发生重定向时设置重定向代码为 302
- `return navigateTo('/', { redirectCode: 301 })` 重定向到给定的路径，并在服务器端发生重定向时设置重定向代码为 301
- `return abortNavigation()` 停止当前的导航
- `return abortNavigation(error)` 拒绝当前的导航并提供错误信息

#### 挂载中间件

页面挂载命名中间件:

```vue
<script setup lang="ts">
definePageMeta({
  middleware: "auth",
});
</script>

<template>
  <h1>欢迎来到您的仪表盘</h1>
</template>
```

页面挂载自定义内联中间件:

```vue
<script setup lang="ts">
definePageMeta({
  middleware: [
    function (to, from) {
      // 自定义内联中间件
    },
    "auth",
  ],
});
</script>
```

#### 全局中间件执行顺序

全局中间件是通过文件名排序的先后来执行的，如果你希望自己定义执行顺序，在文件名前使用`字母排序`:
01, 02, 03, ..., 10, 11, ...
