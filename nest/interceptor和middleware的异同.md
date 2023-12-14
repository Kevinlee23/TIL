Nest 中的 middleware 和 interceptor 都是在请求前后加入一些逻辑的，
这两者有两点区别：

- interceptor 里能够从 context 上拿到目标 class 和 handler, 进而通过 reflector 拿到它的 matedata 等信息，而 middware 不可以

```ts
@Injectable()
export class AaaInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log(context.getClass(), context.getHandler);
  }
}
```

- interceptor 里是可以通过 rxjs 的操作符来组织响应处理流程

```ts
@Injectable()
export class AaaInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    return next.handle().pipe(
      timeout(5000),
      catchError((err) => {
        if (err instanceof TimeoutError) {
          return throwError(() => new RequestTimeoutExcetion());
        }

        return throwError(() => err);
      })
    );
  }
}
```

总结：
它们两者都是 Nest AOP 思想的体现，但是 interceptor 更适合处理与具体业务相关的逻辑，而 middleware 适合更通用的处理逻辑
