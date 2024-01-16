### compression-webpack-plugins 配置

打包时使用插件后，会在打包出的资源处额外增加同一文件的 \*.gz 格式。

将资源放到服务器，并且服务器配置支持 gzip 后，客户端请求资源的 Request Headers 中若有 Accept-Encoding: gzip, deflate 则会请求 gzip 文件，服务端的 Response Headers 中则会有 Content-Encoding: gzip 。

如果客户端和服务端任意一方不支持 gzip，则会请求正常的 html/css/js 文件。

#### 安装

`npm install compression-webpack-plugin -save-dev`

#### 配置中使用

```js
const CompressionPlugin = require("compression-webpack-plugin");

module.exports = {
  plugins: [new CompressionPlugin()],
};
```

#### vue-cli 项目中使用

```js
module.exports = {
  // 基于环境有条件地配置行为, 或者想要直接修改配置, 使用函数的方式
  configureWebpack: (config) => {
    if (isProd) {
      // 启用 gzip 压缩插件
      config.plugins.push(
        new CompressionWebpackPlugin({
          test: /\.js$|\.html$|\.css$/u,
          algorithm: "gzip", // 压缩算法
          filename: "[path][base].gz",
          threshold: 4096, // 只处理比这个值大的资源。按字节计算
          minRatio: 0.8, // 只有压缩率比这个值小的资源才会被处理
        })
      );
    }
  },
};
```

或者

```js
module.exports = {
  chainWebpack(config) {
    // 压缩gzip
    config.plugin("CompressionPlugin").use("compression-webpack-plugin", [
      {
        test: /\.js$|\.css$|\.html$/,
        algorithm: "gzip",
        filename: "[path][base].gz",
        threshold: 10240, // 只处理比这个值大的资源。按字节计算
        minRatio: 0.8, // 只有压缩率比这个值小的资源才会被处理
      },
    ]);
  },
};
```
