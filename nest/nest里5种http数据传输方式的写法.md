### 1. url param

`http://guang.zxg/person/1111`

```ts
@Controller("api/person")
export class PersonController {
  @Get(":id")
  urlParam(@Param("id") id: string) {
    return `received: id = ${id}`;
  }
}
```

### 2. query

`http://guang.zxg/person?name=guang&age=20`

```ts
@Controller("api/person")
export class PersonController {
  @Get("find")
  query(@Query("name") name: string, @Query("age") age: number) {
    return `received: name=${name},age=${age}`;
  }
}
```

### 3. form-urlencoded

直接用 form 表单提交, 指定 content-type 为 `application/x-www-form-urlencoded`

```ts
export class CreatePersonDto {
  name: string;
  age: number;
}

// ------------------------------

import { CreatePersonDto } from "./dto/create-person.dto";

@Controller("api/person")
export class PersonController {
  @Post()
  body(@Body() createPersonDto: CreatePersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```

### 4. json

使用 axios 的 post 方法 content-type 默认为: `application/json`
跟 #3 一样处理

```ts
@Controller("api/person")
export class PersonController {
  @Post()
  body(@Body() createPersonDto: CreatePersonDto) {
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```

### 5. form-data

指定 content-type 为 `multipart/form-data`, 这种方式适合传输多个文件
nest 解析 form data 使用 FilesInterceptor 拦截器 （启用 @UseInterceptors 装饰器）,
文件内容 通过 @UploadedFiles 获取，非文件内容，通过 @body 获取

```ts
import { AnyFilesInterceptor } from "@nestjs/platform-express";
import { CreatePersonDto } from "./dto/create-person.dto";

// 这里需要安装 @types/multer 来引入相关类型声明
@Controller("api/person")
export class PersonController {
  @Post("file")
  @UseInterceptors(
    AnyFilesInterceptor({
      dest: "uploads/", // 文件存储位置
    })
  )
  body2(
    @Body() createPersonDto: CreatePersonDto,
    @UploadedFiles() files: Array<Express.Multer.File>
  ) {
    console.log(files);
    return `received: ${JSON.stringify(createPersonDto)}`;
  }
}
```
