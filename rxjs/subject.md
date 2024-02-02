### Subject

#### multicast

subject 是 rxjs 中一类特殊的 Observeable, 它支持将数据广播给 Observers:

```js
import { Subject } from "rxjs";

var subject = new Subject();

subject.subscribe({
  next: (v) => console.log("observerA: " + v),
});
subject.subscribe({
  next: (v) => console.log("observerB: " + v),
});

subject.next(1);
subject.next(2);

/**
 * Logs
 * observerA: 1
 * observerB: 1
 * observerA: 2
 * observerB: 2
 */
```

#### provide to subscribe

```js
import { Subject, from } from "rxjs";
var subject = new Subject();

subject.subscribe({
  next: (v) => console.log("observerA: " + v),
});
subject.subscribe({
  next: (v) => console.log("observerB: " + v),
});

var observable = from([1, 2, 3]);

observable.subscribe(subject); // You can subscribe providing a Subject

/**
 * Logs
 * observerA: 1
 * observerB: 1
 * observerA: 2
 * observerB: 2
 * observerA: 3
 * observerB: 3
 */
```

#### BehaviorSubject

这是一个变体 Subject, 它有一个当前值的概念:

```js
import { BehaviorSubject } from "rxjs";
const subject = new BehaviorSubject(0); // 0 is the initial value

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(3);

// Logs
// observerA: 0
// observerA: 1
// observerA: 2
// observerB: 2
// observerA: 3
// observerB: 3
```

#### ReplaySubject

类似于 BehaviorSubject, 它会缓存部分的观察结果:

```js
import { ReplaySubject } from "rxjs";
const subject = new ReplaySubject(3); // buffer 3 values for new subscribers

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);

// Logs:
// observerA: 1
// observerA: 2
// observerA: 3
// observerA: 4
// observerB: 2
// observerB: 3
// observerB: 4
// observerA: 5
// observerB: 5
```

ReplaySubject() 的第二个参数为时间参数，表示记录多久之前的数据

#### AsyncSubject

AsyncSubject 也是 Subject 的一种变体，它只将最后一个记录发给观察者，并只在 complete 时发送:

```js
import { AsyncSubject } from "rxjs";
const subject = new AsyncSubject();

subject.subscribe({
  next: (v) => console.log(`observerA: ${v}`),
});

subject.next(1);
subject.next(2);
subject.next(3);
subject.next(4);

subject.subscribe({
  next: (v) => console.log(`observerB: ${v}`),
});

subject.next(5);
subject.complete();

// Logs:
// observerA: 5
// observerB: 5
```
