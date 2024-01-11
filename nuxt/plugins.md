### Plugins

Nuxts 的插件系统, 可以在创建 Vue 应用程序时使用 vue 插件/指令或其他功能

#### 注册插件

只有顶层文件才会被自动注册

```txt
-| plugins/
---| foo.ts      // 被扫描
---| bar/
-----| baz.ts    // 不被扫描
-----| foz.vue   // 不被扫描
-----| index.ts  // 目前被扫描，但已弃用
```

子目录中的文件需要在 nuxt.config.ts 的 plugins 选项中注册:

```ts
export default defineNuxtConfig({
  plugins: ["~/plugins/bar/baz", "~/plugins/bar/foz"],
});
```

#### 运行时钩子

```ts
export default defineNuxtPlugin((nuxtApp) => {
  // setup

  nuxtApp.hook("app:created", () => {
    // 创建初始实例时调用
  });
  nuxtApp.hook("page:start", () => {
    // 页面开始时调用
  });
  nuxtApp.hook("page:finish", () => {
    // 页面完成时调用
  });
});
```

#### 对象语法

```ts
export default defineNuxtPlugin({
  name: "my-plugin",
  enforce: "pre", // 或 'post'
  async setup(nuxtApp) {
    // 这相当于一个普通的功能性插件
  },
  hooks: {
    // 这里可以直接注册Nuxt应用运行时钩子
    "app:created"() {
      const nuxtApp = useNuxtApp();
      //
    },
  },
  env: {
    // 如果不希望插件在仅渲染服务器端或孤立组件时运行，请将此值设置为`false`。
    islands: true,
  },
});
```

#### 注册顺序

按照文件名字母顺序加载，如果你希望自己定义执行顺序，在文件名前使用`字母排序`

#### 使用组合式 api

```ts
export default defineNuxtPlugin((nuxtApp) => {
  const foo = useFoo();
});
```

限制:

- 如果一个组合式依赖于稍后注册的另一个插件，可能无法正常工作。
- 如果一个组合式依赖于 Vue.js 的生命周期，它将无法正常工作。

#### 用法

- 注册 vue 插件
- 注册 vue 指令
- 编写功能性插件 - 对象语法中展示

注册插件和指令是靠 nuxtApp (nuxt 运行时上下文) 下的 vueApp (全局 vue.js 实例) 下的：

- use()
- directive()
  完成

`注册插件`

```ts
import VueGtag, { trackRouter } from "vue-gtag-next";

export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.use(VueGtag, {
    property: {
      id: "GA_MEASUREMENT_ID",
    },
  });
  trackRouter(useRouter());
});
```

`注册指令`

```ts
export default defineNuxtPlugin((nuxtApp) => {
  nuxtApp.vueApp.directive("focus", {
    mounted(el) {
      el.focus();
    },
    getSSRProps(binding, vnode) {
      // 你可以在这里提供SSR特定的props
      return {};
    },
  });

  // or
  nuxtApp.vueApp.directive("directiveName");
});
```
