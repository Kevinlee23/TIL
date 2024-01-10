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
- 编写功能性插件
