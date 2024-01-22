### 类型里的逻辑运算

#### extend: 条件运算

```ts
type StringOrNumber<T> = T extends string ? "string" : "number";
```

#### infer: 提取操作

```ts
// 提取数组内任意位置的值
type ExtractStartAndEnd<T extend any[]> = T extends [infer A, ...any[], infer B] ? [A, B] : T;
type Swap<T extends any[]> = T extends [infer A, infer B] ? [B, A] : [T]

type ArrayItemType<T> = T extends Array<infer ElementType> ? ElementType : never;

// 作用于对象
type PropType<T, K extends keyof T> = T extends { [Key in K]: infer R } ? R : never;

// 作用于 Promise
type PromiseValue<T> = T extends Promise<infer V> ? V : T;
type PromiseValue2<T> = T extends Promise<infer V> ? promiseValue2<V> : T;
```
