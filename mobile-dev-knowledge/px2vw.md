### postcss-px-to-viewport 插件配置

```js
module.exports = {
  postcss: {
    plugins: {
      "postcss-px-to-viewport": {
        unitToConvert: "px", // 需要转换的单位
        viewportWidth: 750, // 视口宽度，等同于设计稿宽度
        unitPrecision: 5, // 精确到小数点后几位
        propList: ["*"], // 将会被转换的css属性列表, 例如: ['*position*'], ['!letter-spacing'], ['!font*']
        viewportUnit: "vw", // 需要转换成为的单位
        fontViewportUnit: "vw", // 需要转换称为的字体单位
        selectorBlackList: [], // 需要忽略的选择器，即这些选择器对应的属性值不做单位转换
        minPixelValue: 1, // 最小的像素单位值
        mediaQuery: false, // 是否转换媒体查询中设置的属性值
        replace: true, // 是否直接更换属性值而不添加备用属性
        exclude: [], // 忽略的文件忽略某些文件夹下的文件或特定文件
        landscape: false, // 是否添加根据landscapeWidth生成的媒体查询条件 @media (orientation: landscape)
        landscapeUnit: "vw", // 横屏单位
        landscapeWidth: 1334, // 横屏宽度
      },
    },
  },
};
```
