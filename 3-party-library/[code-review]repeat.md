实现一个 repeat 方法，要求如下:

```js
function repeat(func, times, wait) {} // 实现repat方法, 使func方法重复执行times次, 每次间隔wait秒
```

实现:

```js
const sleep = (ms) => new Promise((res) => setTimeout(res, ms));

const repeat = (func, times, wait) => {
  return async (...args) => {
    for (let i = 0; i < times; i++) {
      func(...args);
      await sleep(wait);
    }
  };
};
```

实现 2:

```js
const repeat = (func, times, wait) => {
  let num = 0;
  return (args) => {
    function run() {
      func(args);
      num++;
      if (num < times) {
        setTimeout(run, wait);
      }
    }

    run();
  };
};
```
