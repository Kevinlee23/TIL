### What's the p-limit?

p-limit 是用来处理并发控制的库，常见的并行操作如下:

- `await Promise.all([promise1, promise2])`;
- `await Promise.race([promise1, promise2])`

### review

#### steps

#### 1.传入并发数量，返回一个添加并发任务的函数:

```js
const pLimit = (concurrency) => {
  const generator = (fn, ...args) => {
    new Promise((resolve) => {
      // next
    });
  };
};
```

#### 2.添加的并发任务需要排队，准备一个 queue, 以及入队函数 enqueue, 记录当前正在进行中的异步任务

```js
const pLimit = (concurrency) => {
  const queue = [];
  let activeCount = 0;

  const enqueue = (fn, resolve, ...args) => {
    queue.push(run.bind(null, fn, resolve, ...args));

    if (activeCount < concurrency && queue.length > 0) {
      queue.shift()();
    }
  };

  const generator = (fn, ...args) => {
    new Promise((resolve) => {
      enqueue(fn, resolve, ...args);
    });
  };

  return generator;
};
```

#### 3.具体的执行逻辑

```js
const pLimit = (concurrency) => {
  // definition before

  // 活跃任务数 -1, 执行下一个任务
  const next = () => {
    activeCount--;

    if (queue.length > 0) {
      queue.shift()();
    }
  };

  // 执行逻辑
  const run = async (fn, resolve, ...args) => {
    // 活跃的任务数 +1
    activeCount++;

    const result = (async () => fn(...args))();

    resolve(result);

    try {
      await result;
    } catch {}

    // 下一步处理
    next();
  };
};
```

#### Summary

实现过程就是：创建一个队列来保存任务，开始时一次性执行最大并发任务，然后每执行完一个启动一个新的任务。

### trick

#### 传入并发数量的校验

```js
if (
  !(
    (Number.isInteger(concurrency) || concurrency === Infinity) &&
    concurrency > 0
  )
) {
  throw new TypeError("Expected `concurrency` to be a number from 1 and up");
}
```

#### 并发数量的准确控制

以下部分代码:

```js
const enqueue = (fn, resolve, ...args) => {
  queue.push(run.bind(null, fn, resolve, ...args));

  if (activeCount < concurrency && queue.length > 0) {
    queue.shift()();
  }
};
```

在并发数量判断时并不准确，因为 activeCount-- 是在任务执行完毕后才执行的。
万一任务没有执行完，这里是不准确的。
所以为了保证并发数量能控制准确，要等全部的微任务执行完再拿 activeCount。

所以，增加一个微任务:

```js
const enqueue = (fn, resolve, ...args) => {
  queue.push(run.bind(null, fn, resolve, ...args));

  (async () => {
    await Promise.resolve();

    if (activeCount < concurrency && queue.length > 0) {
      queue.shift()();
    }
  })();
};
```

#### 通过 Object.defineProperties 将函数内私有变量暴露出去, 并提供额外方法

```js
const pLimit = (concurrency) => {
  // definition before

  const generator = (fn, ...args) => {
    new Promise((resolve) => {
      // next
    });
  };

  Obejct.defineProperties(generator, {
    // activeCount, pendingCount 只定义 get 函数, 这样就是只读的
    // 提供一个方法 clearQueue 来清空任务队列
    activeCount: {
      get: () => activeCoutn,
    },
    pendingCount: {
      get: () => queue.length,
    },
    clearQueue: {
      value: () => {
        queue.length = 0;
      },
    },
  });
};
```
