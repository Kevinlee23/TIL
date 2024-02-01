### Observable

> Subscribing to an Observable is analogous to calling a Function.

Observable have 4 core concerns (process):

- creating
- subscribing
- executing
- disposing

```javascript
import { Observable } from "rxjs";

// <creating 过程>
const stream = new Observable((observer) => {
  /**
   * Observable are producer of data, who push them(data) to Observers(comsumers)
   * observer are producer, 它拥有方法: next, error, complete, 分别作用: 推送数据、报错和终止
   */

  try {
    // 同步方式
    observer.next(50);
    observer.next(100);
    observer.next(200);

    // 异步方式
    setTimeout(() => {
      observer.next(300);
    }, 1000);
  } catch (err) {
    observer.error(err);
  }

  observer.complete();
});

// 观察数据 <subscribing 过程>
stream.subscribe((x) => {
  console.log(x);
});

// 停止观察
setTimeout(() => {
  // <disposing 过程>
  stream.unSubscribe();
}, 500);

// result: 50 100 200
```

### Observer

观察数据时， 我们可以通过传入一个对象 { next / error / complete } 在适当时候通知<消费者>

```javascript
// <executing 过程>
const observer = {
  next: (x) => console.log("Observer got a next value: " + x),
  error: (err) => console.error("Observer got an error: " + err),
  complete: () => console.log("Observer got a complete notification"),
};

stream.subscribe(observer);
```
