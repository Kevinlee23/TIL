### basic

尽管 Observable 是 rxjs 的基础，但是其最重要、有用的概念是 operators （操作符）
操作符能够使你用声明的方式进行异步编程

### category

#### 有两大类操作符函数:

- pipe 管道型
- creation 创建型

```js
import { of, map } from "rxjs";

of(1, 2, 3)
  .pipe(map(x) => x * x)
  .subscribe((v) => console.log(`value: ${v}`))

// Logs:
// value: 1
// value: 2
// value: 3
```

of 就是一个创建型的操作符, 它接受三个数据
通过管道函数 .pipe, Observable 通过了一个叫做 map 的操作符，将数据 x => x \* x

.pipe 管道操作符可以同时接收多个 op, 并合并这些操作符的结果:
`obs.pipe(op1(), op2(), op3())`

#### 高阶 Observable

高阶 Observable 就是处理 Observable 的 Observable
ex:

```js
// http.get 为每个单独的 URL 返回一个 Observable
const fileObservable = urlObservable.pipe(map((url) => http.get(url)));

// 使用操作符 concatAll 将 高阶 Observable 转换为普通 Observable
const fileObservable = urlObservable.pipe(
  map((url) => http.get(url)),
  concatAll()
);
```

相关的三个 operator:

- concatAll
- mergeAll
- switchAll

concatAll 会将 Observables 生产出的数据按每个 Observable 的传入顺序输出

```txt
a - - - b
- c d - -
- - e - -

Result:
a c de - b
```

concatAll equal to mergeAll(1)

mergetAll 类似于 concatAll, 但是它通过第一个传入参数来限制并发数

switchAll 会在每次下一个 Observable 开始终止当前的 Observable 的输出

```txt
a - - b - c - - - d
- - - - e - f - - g

Result:
a - - b e - f - - g
```

#### 具体的分类有:

- Creation 创建
- Join 合并
- Transformation 转换
- Filtering 过滤
- Multicasting 广播
- Error Handling 异常处理
- Conditional 条件
- Utility 通用处理
- Mathmatical-Aggregate 数学和聚合`reduce`
