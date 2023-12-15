在 nest 中, post 请求的数据是通过 @body 装饰器取，并且需要一个 dto class 来接收：

```ts
export class UserDto {
  name: string;
  age: number;
  sex: boolean;
  hobbies: Array<string>;
}

@Post('create')
createUser(@Body obj: UserDto) {
  console.log(obj);
}
```

我们从这个 handler 可以拿到数据：
`{ name: 'luxiaxue', age: 26, hobbies: ['aaa', 'bbb', 'ccc']}`

但是我们对 age 的限制为 `number`, 意味着 age 可以是小数（小数也是 number）, 这个时候需要使用到 ValidationPipe

#### 使用 ValidationPipe

首先需要安装两个依赖包:
`npm install class-validator class-transformer`

然后在 @Body 里添加这个 pipe:

```ts
@Post('create')
createUser(@Body (new ValidationPipe()) obj: UserDto) {
  console.log(obj);
}
```

在 dto 里使用 class-validator 里的装饰器来标记限制:

```ts
import { IsInt } from "class-validator";

export class UserDto {
  name: string;
  @IsInt()
  age: number;
  sex: boolean;
  hobbies: Array<string>;
}
```

然后再发送小数 age 就会触发报错

#### class-validator 支持的验证方式

`import { Contains, IsDate, IsEmail, IsFQDN, IsInt, Length, Max, Min } from 'class-validator';`
其中：

- IsFQDN 是是否为域名的意思

```ts
export class ValidateCls {
  @Length(10, 20)
  title: string;

  @Contain("hello")
  text: string;

  @IsInt()
  @Min(0)
  @Max(10)
  rating: number;

  @IsEmail()
  email: string;

  @IsFQDN()
  site: string;
}

// 可以定制错误消息
export class CutomError {
  @Length(10, 20, {
    message({ targetName, property, value, constraints }) {
      return `${targetName} 类的 ${property}属性的值 ${value} 不满足约束: ${constraints}`;
    },
  })
  title: string;
}
```

我们填入 `{title: 'aa'}` 时, 返回的错误信息为 CustomError 类的 title 属性值 aa 不满足约束: 10, 20
