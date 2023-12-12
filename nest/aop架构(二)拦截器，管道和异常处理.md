#### Interceptor 拦截器

`nest g interceptor time --no-spec` cli 快速创建 interceptor 指令

controller 前后的逻辑可能是异步的，nest 里通过 rxjs 来组织他们

```ts
import {
  CallHandler,
  ExecutionContext,
  Injectable,
  NestInterceptor,
} from "@nestjs/common";
import { Observable, tap } from "rxjs";

@Injectable()
export class TimeInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    const startTime = Date.now();

    return next.handle().pipe(
      tap(() => {
        console.log("time: ", Date.now() - startTime);
      })
    );
  }
}
```

interceptor 和 middleware 的区别在于:
intercepetor 可以拿到调用的 controller 和 handler, 可以在它们上加一些 metadata

interceptor 支持每个路由单独启用，只作用于某个 handler:

```ts
@Get('bbb')
@UseInterceptors(TimeInterceptor)
bbb(): string {
  consoole.log('bbbb')
  return 'bbb'
}
```

也可以在 controller 级别启动，作用于下面的全部 handler:

```ts
@Controller()
@UseInterceptors(TimeInterceptor)
export class AppController {
  constrctor(...) {
    ...
  }
}
```

也支持全局启用：
`app.useGlobalInterceptors(new TimeInterceptor)`

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [
    AppService,
    {
      provide: APP_INTERCEPTOR,
      useClass: TimeInterceptor
    }
  ]
})
```

#### Pipe 管道

管道的作用是来对参数做一些检验和转换

`nest g pipe validate --no-spec` cli 快速创建管道的指令

```ts
import {
  ArgumentMetadata,
  BadRequestException,
  Injectable,
  PipeTransform,
} from "@nestjs/common";

@Injectable()
export class ValidatePipe implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    if (Number.isNaN(parseInt(value))) {
      throw new BadRequestException(`参数${metadata.data}只能是字符串或者数字`);
    }

    return typeof value === "number" ? value * 10 : parseInt(value) * 10;
  }
}
```

pipe 可以只对某个参数生效，或者对整个 Controller 都生效：

```ts
@Get('ccc')
ccc(@Query('num', ValidatePipe) num: number) {
  return num+1;
}

// or controller
@Controller()
@UsePipes(ValidatePipe)
export class AppController {
  constructor(...) {
    ...
  }
}
```

或者对全局生效：
`app.useGlobalPipes(new ValidatePipe())`

```ts
...
providers: [
  provide: APP_PIPE,
  useClass: ValidatePipe
],
...
```

#### ExceptionFilter 异常处理

ExceptionFilter 可以对抛出的异常做处理，返回对应的响应

`nest g filter test --no-spec` cli 快速创建 filter 的指令

```ts
import {
  ArgumentsHost,
  BadRequestException,
  Catch,
  ExceptionFilter,
} from "@nestjs/common";
import { Response } from "express";

@Catch(BadRequestException)
export class TestFilter implements ExceptionFilter {
  catch(exception: BadRequestException, host: ArgumentsHost) {
    const response: Response = host.switchToHttp().getResponse();

    response.status(400).json({
      statusCode: 400,
      message: "test: " + exception.message,
    });
  }
}

// 使用
@Get('ccc')
@UseFilters(TestFilter)
ccc(@Query('num', ValidatePipe) num: number) {
  return num+1;
}

// 异常返回的响应
{
  "statusCode": 400,
  "message": "test: 参数num只能是字符串或者数字"
}
```

filter 还可以对整个 controller 生效：

```ts
@Controller()
@UseFilter(TestFilter)
export class AppController {
  ...
}
```

也可以全局使用：
`app.useGlobalFilter(new TestFilter())`

```ts
...
providers: [
  provide: APP_FILTER,
  useClass: TestFilter
],
...
```

#### aop 机制的调用顺序

进入路由时，会先调用 Guard, 判断是否有权限
如果没有权限，会抛出异常，异常会被 ExceptionFilter 处理
如果有权限，就会调用拦截器 interceptor, 拦截器组织了一个链条，一个个的调用，最后会调用的 controller 的方法
调用 controller 方法前，会使用 pipe 对参数进行处理
middleware 是 express 中的概念，nest 只是继承了这个概念，会在最外层被调用

把这些调用顺序理清楚，就知道什么逻辑放在什么切面里了。
