### splitChunksPlugin 配置

webpack 会基于如下原则自动分割代码：

- 可以被共享的代码块，或来自 node_modules 文件夹的模块。

- 打包出来的代码块大小超过 20k（在 min + gz 之前）。

- 当按需加载块时，并行请求的最大数量希望小于或等于 30 的时候。

- 在页面初始加载时并行请求的最大数量希望小于或等于 30 的时候。

#### vue.config.js 中配置

```js
module.exports = {
  configureWebpack: {
    optimization: {
      splitChunks: {
        // ...
      },
    },
  },
};
```

或

```js
module.exports = {
  chainWebpack(config) {
    config.optimization.splitChunks({
      chunks: "all", // async, initial, all; 代码满足条件后是否进行拆分
      minSize: 30000, // 满足尺寸才发生拆分
      minChunks: 1, // 最少被引用的 chunk 个数
      maxAsyncRequests: 5, // 异步代码最多可拆分次数
      maxInitialRequests: 3, // 同上

      // 自定义拆分组
      cacheGroups: {
        libs: {
          name: "chunk-libs",
          test: /[\\/]node_modules[\\/]/,
          priority: 2, // 缓存组打包的先后优先级, 如果两个组引用了同一个模块，优先级高的组可以获得此模块
          reuseExistingChunk: true, //  如果当前代码块包含的模块已经有了，就不在产生一个新的代码块
          chunks: "initial", // only package third parties that are initially dependent
        },

        default: {
          minChunks: 2,
          priority: -20,

          // 是否复用其他chunk内已拥有的模块
          // 默认为false，关闭表示拆分出复用部分的模块，给双方引用
          reuseExistingChunk: true,
        },
      },
    });
  },
};
```

#### 引用

详细的`各项配置的注释`和`示例情况`可以查看 `https://drylint.com/Webpack/SplitChunksPlugin.html#%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5`
