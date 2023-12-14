假设我们有如下路由 controller:

```ts
@Controller()
export class AppController {
  contructor(private readonly appService: AppService) {}

  @Get()
  @UseGuards(AaaGuard)
  @UseInterceptors(AaaInterceptor)
  getHello(): string {
    return this.appService.getHello();
  }
}
```

我们在路由级别上加入了一个 guard 和 一个 interceptor
然后，我们向路由上加入一个 metadata:

```ts
class AppController {
  // ...

  @Get()
  @UseGuards(AaaGuard)
  @UseInterceptors(AaaInterceptor)
  @SetMetadata("roles", ["admin"])
  getHello(): string {
    return this.appService.getHello();
  }
}
```

我们可以通过以下方式在 guard 和 interceptor 中提取这个 metadata:

```ts
// 路由守卫中提取 metadata
@Injectable()
export class AaaGuard implements CanActivate {
  contructor(private reflector: Reflector) {}

  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    console.log("guard");
    console.log(this.reflector.get("roles", context.getHandler()));

    return true;
  }
}
```

```ts
// 拦截器中提取 metadata
@Injectable()
export class AaaInterceptor implements NestInterceptor {
  private reflector: Reflector;

  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log("interceptor");

    console.log(this.reflector.get("roles", context.getHandler()));

    return next.handle();
  }
}
```

当然，我们也可以在 class 上附加 metadata:

```ts
@Controller()
@SetMatadata("roles", ["user"])
export class AppController {
  contructor(private readonly appService: AppService) {}

  @Get()
  @UseGuards(AaaGuard)
  @UseInterceptors(AaaInterceptor)
  @SetMetadata("roles", ["admin"])
  getHello(): string {
    return this.appService.getHello();
  }
}

// this.reflector.get("role", context.getClass()) ['user']
```
