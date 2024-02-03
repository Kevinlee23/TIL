### postcss-pxtorem 插件配置

```js
module.exports = {
  postcss: {
    plugins: {
      "postcss-pxtorem": {
        rootValue: 75, // 750设计标准, 如果是 375 则 设置为 37.5
        unitPrecision: 5, // 转换成的rem后，保留小数点后几位
        propList: ["*", "!letter-spacing"], // 转化为rem的属性列表
        selectorBlackList: [".rem-"], // 不会被转换的class选择器名，支持正则
        replace: true, // 是否直接更换属性值而不添加备用属性
        mediaQuery: false, // 允许在媒体查询中转换`px`
        minPixelValue: 1, // 小于1px的将不会被转换
        exclude: function (filename) {}, // 忽略的文件忽略某些文件夹下的文件或特定文件
      },
    },
  },
};
```
