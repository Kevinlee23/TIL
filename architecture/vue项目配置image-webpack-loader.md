### image-webpack-loader 配置

vue.config.js 配置

```js
module.exports = {
  // ... 其他的一些配置
  // 关于 chainWebpack 的使用，请看 Vue Cli 文档
  // https://cli.vuejs.org/zh/guide/webpack.html#%E9%93%BE%E5%BC%8F%E6%93%8D%E4%BD%9C-%E9%AB%98%E7%BA%A7
  chainWebpack: (config) => {
    config.module
      .rule("image")
      .test(/\.(png|jpe?g|gif)(\?.*)?$/)
      .use("image-webpack-loader")
      .loader("image-webpack-loader")
      .options({
        // 此处为ture的时候不会启用压缩处理,目的是为了开发模式下调试速度更快
        disable: process.env.NODE_ENV == "development" ? true : false,
      })
      .end();
  },
};
```
