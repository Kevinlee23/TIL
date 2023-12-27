#### 前端分片处理

```js
const chunkSize = 20 * 1024; // 20kb

const file = fileInput.files[0]; // fileInput 为 input 标签的 ref

const chunks = [];
let startPos = 0;
while (startPos < file.size) {
  chunks.push(file.slice(startPos, startPos + chunkSize));
  startPos += chunkSize;
}

const randomStr = Math.random().toString().slice(2, 8); // 随机名称

const tasks = [];
chunks.map((chunk, index) => {
  const data = new FormData();
  data.set("name", randomStr + "_" + file.name + "-" + index);
  data.append("files", chunk);

  // 将所有的分片上传任务保存为一个队列
  tasks.push(axios.post("http://localhost:3000/upload", data));
});

await Promise.all(tasks);

// 结束分片上传后, 调用文件合并的接口
axios.get("http://localhost:3000?merge?name=" + randomStr + "_" + file.name);
```

#### 服务端接收

```ts

// 在 uploads 下创建 chunks_文件名 的目录，把文件复制过去，然后删掉原始文件。
@Post('upload')
@UseInterceptors(FilesInterceptor('files', 20, {
  dest: 'uploads'
}))
uploadFiles(@UploadedFiles() files: Array<Express.Multer.File>, @Body() body: { name: string }) {
  const fileName = body.name.match(/(.+)\-\d+$/)[1];
  const chunkDir = 'uploads/chunks_'+ fileName;

  if(!fs.existsSync(chunkDir)){
    fs.mkdirSync(chunkDir);
  }
  // 将文件复制到临时目录
  fs.cpSync(files[0].path, chunkDir + '/' + body.name);
  // 在 upload 目录下上传的文件删除
  fs.rmSync(files[0].path);
}

// 合并接口
@Get('merge')
merge(@Query('name') name: string) {
  const chunkDir = 'uploads/chunks_'+ name;

  const files = fs.readdirSync(chunkDir);

  let count = 0
  let startPos = 0;
  files.map(file => {
    const filePath = chunkDir + '/' + file;
    const stream = fs.createReadStream(filePath);
    stream.pipe(fs.createWriteStream('uploads/' + name, {
      start: startPos
    })).on('finish', () => {
      count++;

      // 合并完成后将 chunks 目录删掉
      if (count === files.length) {
        fs.rm(chunkDir, {
          recursive: true
        }, () => {});
      }
    })

    startPos += fs.statSync(filePath).size;
  });
}
```
