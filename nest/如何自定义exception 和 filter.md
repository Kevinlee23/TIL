`nest g filter errorCapture --flat --no-spec` // 创建一个自定义的 filter

```ts
import { Response } from "express";

@Catch(HttpException)
export class ErrorCaptureFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const responese = http.getResponse<Response>;

    const statusCode = exception.getStatus();

    response.status(statusCode).json({
      code: statusCode,
      message: exception.message,
      error: "Bad Request",
      xxx: 111,
    });
  }
}
```

@Catch() 里面指定要捕获的异常，这里指定的是最顶层的通用类 HttpException,
按照上述代码，抛异常时返回的响应就是自定义的了，
所有的 HttpException 都会被处理，但是我们使用 ValidationPipe 的时候不会被处理，需要进行以下修改：

```ts
@Catch(HttpException)
export class ErrorCaptureFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const response = http.getResponse<Response>();

    const statusCode = exception.getStatus();
    const res = exception.getResponse() as { message: string[] };

    debugger;

    response.status(statusCode).json({
      code: statusCode,
      message: res?.message?.join ? res?.message?.join(",") : exception.message,
      error: "Bad Request",
      xxx: 111,
    });
  }
}
```

如果 response.message 是个数组，就返回 join 的结果，否则还是返回 exception.message
这样就可以兼容处理 ValidationPipe 触发的报错

#### 注入服务

如果我们想往 Filter 里注入 service, 需要修改注册的方式:

```ts
@Module({
  import: [],
  controller: [AppControll],
  providers: [
    AppService,
    {
      provide: APP_FILTER,
      useClass: ErrorCaptureFilter,
    },
  ],
})
// ErrorCaptureFilter
@Catch(HttpException)
export class ErrorCaptureFilter implements ExceptionFilter {
  @Inject(AppService)
  catch(exception: HttpException, host: ArgumentsHost) {
    const http = host.switchToHttp();
    const response = http.getResponse<Response>();

    const statusCode = exception.getStatus();
    const res = exception.getResponse() as { message: string[] };

    debugger;

    response.status(statusCode).json({
      code: statusCode,
      message: res?.message?.join ? res?.message?.join(",") : exception.message,
      error: "Bad Request",
      xxx: 111,
    });
  }
}
```

#### 自定义 Exception

```ts
export class UnLoginException {
  message: string;

  contructor(message?) {
    this.message = message;
  }
}

@Catch(UnLoginException)
export class UnloginFilter implements ExceptionFilter {
  catch(exception: UnLoginException, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse<Response>();

    response
      .status(HttpStatus.UNAUTHORIZED)
      .json({
        code: HttpStatus.UNAUTHORIZED,
        message: "fail",
        data: exception.message || "用户未登录",
      })
      .end();
  }
}
```

定义了 Exception 的 class 之后，就能在 handler 里面抛出这个异常，并且在 Filter 里去捕获
