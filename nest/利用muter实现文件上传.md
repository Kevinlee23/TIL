### nest 文件上传

#### nest 服务开启静态文件支持

```ts
const app = await NestFactory.create<NestExpressApplication>(AppModule);
app.useStaticAssets(join(__dirname, "..", "public"), {
  prefix: "/static",
});
```

或者单独跑一个 http-server 来支持静态服务

```ts
const app = await NestFactory.create(AppModule, {
  cors: true,
});
```

```cmd
npx http-server
```

#### 单字段单文件

```ts
@Post('aaa')
  @UseInterceptors(
    FileInterceptor('file', {
      dest: 'uploads',
    }),
  )
// 注意提前装好 @types/multer 依赖用于解析类型
uploadFile(@UploadedFile() file: Express.Multer.File, @Body() body) {
    console.log('body', body);
    console.log('file', file);
};
```

#### 单字段多文件

```ts
@Post('bbb')
// FilesInterceptor 单字段多文件
@UseInterceptors(FilesInterceptor('bbb', 3, {
    dest: 'uploads'
}))
// 多文件使用UploadedFiles
uploadFiles(@UploadedFiles() files: Array<Express.Multer.File>, @Body() body) {
    console.log('body', body);
    console.log('files', files);
};
```

#### 多字段多文件

```ts
@Post('ccc')
// FileFieldsInterceptor 多字段多文件
@UseInterceptors(FileFieldsInterceptor([
    { name: 'aaa', maxCount: 2 },
    { name: 'bbb', maxCount: 3 },
], {
    dest: 'uploads'
}))
uploadFileFields(@UploadedFiles() files: { aaa?: Express.Multer.File[], bbb?: Express.Multer.File[] }, @Body() body) {
    console.log('body', body);
    console.log('files', files);
};
```

#### 任意字段多文件

```ts
@Post('ddd')
// AnyFilesInterceptor 任意字段多文件
@UseInterceptors(AnyFilesInterceptor({
    dest: 'uploads'
}))
uploadAnyFiles(@UploadedFiles() files: Array<Express.Multer.File>, @Body() body) {
    console.log('body', body);
    console.log('files', files);
};
```

#### 指定 storage

```js
// storage.js

import * as multer from "multer";
import * as fs from "fs";
import * as path from "path";

const storage = multer.diskStorage({
  destination: function (req, file, cb) {
    try {
      fs.mkdirSync(path.join(process.cwd(), "uploads"));
    } catch (e) {}

    cb(null, path.join(process.cwd(), "uploads"));
  },
  filename: function (req, file, cb) {
    const uniqueSuffix =
      Date.now() +
      "-" +
      Math.round(Math.random() * 1e9) +
      "-" +
      file.originalname;
    // 重新定义文件名
    cb(null, file.fieldname + "-" + uniqueSuffix);
  },
});

export { storage };
```

```ts
@Post('ddd')
// AnyFilesInterceptor 任意字段多文件
@UseInterceptors(AnyFilesInterceptor({
  storage
}))
uploadAnyFiles(@UploadedFiles() files: Array<Express.Multer.File>, @Body() body) {
    console.log('body', body);
    console.log('files', files);
};
```

#### 使用 ParseFilePipe 来对文件大小、类型进行校验

`ParseFilePipe` 的作用是调用传入的 validator 来对文件进行校验，常见的有：

- `MaxFileSizeValidator` 校验文件大小
- `FileTypeValidator` 校验文件类型

```ts
@Post('fff')
@UseInterceptors(FileInterceptor('aaa', {
  dest: 'uploads'
}))
uploadFile3(@UploadedFile(new ParseFilePipe({
  exceptionFactory: err => {
    throw new HttpExceotion('xxx' + err, 400)
  },
  validators: [
    new MaxFileSizeValidator({ maxSize: 1000 }),
    new FileTypeValidator({ fileType: 'image/jpeg' })
  ],
})) file: Express.Multer.File, @Body() body) {
    console.log('body', body);
    console.log('files', files);
};
```

也可以自定义 validator:

```ts
import { FileValidator } from "@nestjs/common";

// 继承 FileValidator, 实现 isValid, buildErrorMessage 两个方法
export class MyFileValidator extends FileValidator {
  constructor(options) {
    super(options);
  }

  isValid(file: Express.Multer.File): boolean | Promise<boolean> {
    if (file.size > 10000) {
      return false;
    }
    return true;
  }
  buildErrorMessage(file: Express.Multer.File): string {
    return `文件 ${file.originalname} 大小超出 10k`;
  }
}
```
