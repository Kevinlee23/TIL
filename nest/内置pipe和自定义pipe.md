#### 内置 pipe

nest 内置的 pipe 如下：

- ValidationPipe
- DefaultValuePipe 设置参数默认值
- ParseIntPipe 转换成整数
- ParseFloatPipe 转换成 float 类型
- ParseBoolPipe 转换成 boolean 类型
- ParseArrayPipe 转换成数组类型，需要安装 class-validator 和 class-transformer 包
- ParseUUIDPipe 校验参数是否时 UUID
- ParseEnumPipe 提取枚举类型
- ParseFilePipe

内置 pipe 的报错是可以定制的：

修改 statusCode

```ts
@Controller()
export class AppController {
  constrctor() {}

  @Get()
  getAll(
    @Query(
      "aa",
      new ParseIntPipe({
        errorHttpStatusCode: HttpStatus.xxx,
      })
    )
    aa: string
  ): string {
    return aa;
  }
}
```

自己抛出异常

```ts
@Controller()
export class AppController {
  constrctor() {}

  @Get()
  getAll(
    @Query(
      "aa",
      new ParseIntPipe({
        exceptionFactory: (msg) => {
          console.log(msg);
          throw new HttpException("xxx" + msg, HttpStatus.xxx);
        },
      })
    )
    aa: string
  ): string {
    return aa;
  }
}
```

或者通过 @UseFilters 来使用自己的 exception filter 进行处理

#### 自定义 pipe

```ts
@Injectable()
export class ValidatePipe implements PipeTransform {
  // 可以实现自定义入参, 实现功能
  contructor() {}
  transform(value: any, metadata: ArgumentMetadata) {
    /**
     * value 为 query, param 的值
     * 而 metadata 里面包含 type、metatype、data
     * type 就是 @Query, @Param, @Body 装饰器, 或者自定义装饰器
     * metaType 是 参数的 ts 类型
     * data 是传给 @Query, @Param, @Body 等装饰器的参数
     * 有了这些内容, 做一下验证, 抛出异常给 exception filter 处理, 或者对 value 进行转换都是很简单的事情
     */
    if (Number.isNaN(parseInt(value))) {
      throw new BadRequestException(`参数${metadata.data}只能是字符串或者数字`);
    }

    return typeof value === "number" ? value * 10 : parseInt(value) * 10;
  }
}
```
