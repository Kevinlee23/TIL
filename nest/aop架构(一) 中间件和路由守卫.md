#### 什么是 aop 架构

后端框架基本都是 MVC 架构，即 model view controller 层的简称
MVC 架构下，请求会先发送给 controller, 由它调用 model 层的 service 来完成业务逻辑，然后返回相应的 view

nest 在这个流程中，还提供了 aop(aspect oriented programming) 的能力，即面向切面编程
前面我们提到，一个请求进来会经过 controller, service, repository(数据库访问), 如果我们需要在这个调用链路中加入一些通用逻辑，
直接在 controller 里加入不太优雅，因为这些通用的逻辑侵入到了业务逻辑里面

所以我们可以在调用 controller 前后加入一个执行通用逻辑的阶段，这个横向扩展点就叫做切面，这种透明的加入一些切面逻辑的编程方式就叫做 aop

#### nest 中实现 aop 的方式

共有五种：middleware(中间件), guard, pipe, interceptor, exceptionFilter

##### middleware 中间件

全局中间件, 在 main.ts 里通过 app.use 使用:

```ts
async function bootstrap() {
  const app = await NestFactory.create("AppModule");

  app.use(function (req: Request, res: Response, next: NextFunction) {
    console.log("before", req.url);
    next();
    console.log("after");
  });

  await app.listen(3000);
}
```

路由中间件:
`nest g middleware log --no-spec` cli 快捷创建中间件的指令

```ts
@Injectable()
export class LogMiddleware implements NestMiddleware {
  use(req: Request, res: Response, next: () => void) {
    console.log("before", req.url);

    next();

    console.log("after");
  }
}

// 在 appMoudle 里启用
@Module({
  import: [],
  controllers: [AppController],
  providers: [AppService],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    // 配置哪些 logMiddleware 在哪些路由生效
    consumer.apply(LogMiddleware).forRoutes("...");
  }
}
```

##### guard 路由守卫

路由守卫，可以用于在调用某个 controller 之前判断权限，返回 true 或者 false 来决定是否放行
`nest g guard login --no-spec` cli 快捷创建路由守卫的指令

```ts
@Injectable()
export class LoginGuard implements CanActivate {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    console.log("login check");
    return false;
  }
}

// 在 AppController 里启用:
@Get('aaa')
@UserGuards(LoginGuard)
aaa(): string {
  console.log('aaa...')
  return 'aaa'
}
```

当访问 localhost:3000/aaa 时会返回 403 没有权限

全局启用（在 main.ts 里声明）：

```ts
app.useGlobalGuards(new LoginGuard());
```

在 AppModule 里进行全局声明:

```ts
import { APP_GUARD } from '@nestjs/core'

@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: APP_GUARD,
      useClass: LoginGuard
    }
  ]
})
```

两种全局声明的方式，区别在于，当你需要在路由守卫里注入其他服务时，需要使用第二种方式

```ts
@Injectable()
export class LoginGuard implements CanActivate {
  @Inject(AppService)
  private appService: AppService;

  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    console.log("login check", this.appService.getHello());
    return false;
  }
}
```
