### rxjs 中的创建类操作符

#### 同步类型

`单值创建`:

- of
- empty
- never
- throw
- create \*\*

`多值创建`:

- range
- repeat

#### 异步类型

异步类型有:

- interval/timer
- from (fromPromise, fromEvent) \*\*
- ajax \*\*
- defer

#### example

```js
import { from, fromEvent, of, interval, merge } from "rxjs";
const ajax = () => fetch(url).then((v) => v.json());

merge(
  from([1, 2, 3]), // 从数组多值创建
  interval(3000), // 定时器轮训创建
  fromEvent(document.getElementById("btn"), "click"), // 从点击事件创建
  of({ hello: "world" }), // 单值创建
  from(ajax()) // 从接口请求创建
);
```
