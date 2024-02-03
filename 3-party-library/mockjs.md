### mock.js

mock.js 是一个生成随机请求的库

#### 构造语法

数值模板定义 DTD:

`属性名 生成规则 属性值`
`'name | rule': value`

mock 的生成规则共有 7 种:

- `'name|min-max': value`
- `'name|count': value`
- `'name|min-max.dmin-dmax': value`
- `'name|min-max.dcount': value`
- `'name|count.dmin-dmax': value`
- `'name|count.dcount': value`
- `'name|+step': value`

tips:

- 生成规则的含义需要依赖属性值的类型才能确定
- 属性值还可以包含 `@占位符`
- 属性值还指定了最终值的初始值和类型

#### mock 方法

```js
import Mock from "mockjs";

Mock.mock(template); // 根据数据模板生成模拟数据
// 拦截到匹配 rurl 的请求时, 将根据数据模板 template 生成模拟数据
Mock.mock(rurl, template);
/**
 * 拦截匹配 rurl 的请求, 将回调函数的执行结果作为相应数据返回
 * options: url, type, body
 */
Mock.mock(rurl, `function(options)`);
Mock.mock(rurl, rtype, template); // ...
Mock.mock(rurl, rtype, `function(options)`); // ...
```

#### setup 方法

配合拦截 Ajax 请求的行为, 设置模拟接口的响应时间:
`Mock.setup({ timeout: 400 })`

#### Random 方法

调用占位符:

```js
import Mock from "mockjs";

var Random = Mock.Random;
Random.email();
// => "n.clark@miller.io"
Mock.mock("@email");
// => "y.lee@lewis.org"
Mock.mock({ email: "@email" });
// => { email: "v.lewis@hall.gov" }
```

扩展占位符:

```js
Random.extend({
  constellation: function (date) {
    var constellations = [
      "白羊座",
      "金牛座",
      "双子座",
      "巨蟹座",
      "狮子座",
      "处女座",
      "天秤座",
      "天蝎座",
      "射手座",
      "摩羯座",
      "水瓶座",
      "双鱼座",
    ];
    return this.pick(constellations);
  },
});
Random.constellation();
// => "水瓶座"
Mock.mock("@CONSTELLATION");
// => "天蝎座"
Mock.mock({
  constellation: "@CONSTELLATION",
});
// => { constellation: "射手座" }
```
