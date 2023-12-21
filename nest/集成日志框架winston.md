服务端打印日志不能简单通过 console.log 实现，因为：

- console.log 打印完就没了，而服务端的日志经常要用来排查问题，需要搜索、分析日志内容，因此需要写入到文件或者数据库里
- 打印的日志需要分级别，有的是错误日志，有的只是普通入职，需要能够过滤不同级别的日志
- 打印的日志需要带上时间戳和代码位置信息

所以，日志功能一般会集成专门的日志框架来完成，比如 本文介绍的 winston

#### node 里安装使用

`yarn add winston`

```js
import winston from "winston";

const logger = winston.createLogger({
  level: "debug",
  format: winston.format.simple(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({
      dirname: "log",
      filename: "test.log",
    }),
  ],
});
```

`level` 是打印的日志级别，包含 6 个级别：

```json
{
  "error": 0,
  "warn": 1,
  "info": 2,
  "http": 3,
  "verbose": 4,
  "debug": 5,
  "silly": 6
}
```

从上到下，重要程度依次降低
比如当你指定 level 是 info 时，那 info、warn、error 的日志会输出，而 http、debug 这些不会。

`format` 是日志的格式

简单使用：

```ts
const logger = winston.createLogger({
  level: "debug",
  format: winston.format.simple(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({
      dirname: "log",
      filename: "test.log",
    }),
  ],
});

logger.info("123"); // info: 123
```

组合使用：

```ts
const logger = winston.createLogger({
  level: "debug",
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.json()
  ),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({
      dirname: "log",
      filename: "test.log",
    }),
  ],
});

logger.info("123"); // { "level": "info", "message": "123", "timestamp": "2023-12-20..." }
```

不同的 transport 指定不同的格式：

```ts
const logger = winston.createLogger({
  level: "debug",
  transports: [
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(), // 上色插件
        winston.format.simple()
      ),
    }),
    new winston.transports.File({
      dirname: "log3",
      filename: "test.log",
      format: winston.format.json(),
    }),
  ],
});
```

也可以创建多个 Logger 实例，这样就可以指定不同的日志用不同的方式输出

`transport` 是日志的传输方式, winston 的 [文档](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fwinstonjs%2Fwinston%2Fblob%2FHEAD%2Fdocs%2Ftransports.md%23winston-core) 里可以看到有很多 transport （有的是官方提供的，也有社区提供，需要安装的）比如：

- `File` Built-in to winston (官方库提供), 输入日志到文件
- `Http` Built-in to winston, 向 http 接口输出日志
- `Console` Built-in to winston, 向控制台输出日志
- `MongoDB` Maintained by winston contributors (社区提供), 输入日志到数据库 mongodb 中
- ...

#### 集成到 nest 项目中

安装依赖：

```cmd
yarn add winston
yarn add dayjs
yarn add chalk@4
```

`myLoggerService.ts`

```ts
import { Injectable, LoggerService } from "@nestjs/common";
import * as chalk from "chalk"; // 上色插件
import * as dayjs from "dayjs"; // 日期格式化插件
import { createLogger, format, Logger, transports } from "winston";

@Injectable()
export class MyLoggerService implements LoggerService {
  private logger: Logger;

  constructor() {
    this.logger = createLogger({
      level: "debug",
      transports: [
        // 控制台打印配置
        new transports.Console({
          format: format.combine(
            format.colorize(),
            format.printf(({ context, level, message, time }) => {
              const appStr = chalk.green(`[NEST]`);
              const contextStr = chalk.yellow(`[${context}]`);

              return `${appStr} ${time} ${level} ${contextStr} ${message} `;
            })
          ),
        }),
        // 文件日志配置
        new transports.File({
          format: format.combine(format.timestamp(), format.json()),
          filename: "file_upload_log.log",
          dirname: "log",
        }),
      ],
    });
  }

  log(message: string, context: string) {
    const time = dayjs(Date.now()).format("YYYY-MM-DD HH:mm:ss");

    this.logger.log("info", message, { context, time });
  }

  error(message: string, context: string) {
    const time = dayjs(Date.now()).format("YYYY-MM-DD HH:mm:ss");

    this.logger.log("info", message, { context, time });
  }

  warn(message: string, context: string) {
    const time = dayjs(Date.now()).format("YYYY-MM-DD HH:mm:ss");

    this.logger.log("info", message, { context, time });
  }
}
```

`main.ts`

```ts
async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule);

  // 注册 logger
  app.useLogger(new MyLoggerService());

  await app.listen(3000);

  if (module.hot) {
    module.hot.accept();
    module.hot.dispose(() => app.close());
  }
}
```

然后就可以在 controller, service, filter 或者 guard 中注入这个 logger 并使用了：

`app.controller.ts`

```ts
// module 中 注入
// providers: [AppService, MyLoggerService],

@Controller("blog")
export class AppController {
  private logger = new Logger();
  constructor(private readonly appService: AppService) {}

  @Get("aaa")
  aaa() {
    this.logger.log("aaa", AppController.name);
    return "aaa";
  }
}
```
