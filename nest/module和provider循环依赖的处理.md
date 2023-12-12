首先我们有两个模块 Aaa 和 Bbb, 并且相互引用

```ts
@Module({
  imports: [BbbModule],
})
export class AaaModule {}
```

```ts
@Module({
  imports: [AaaModule],
})
export class BbbModule {}
```

我们直接启动项目时，会报错，两个 Module 循环依赖了
我们可以单独创建两个 Module, 再将它们关联起来，也就是使用 forwardRef 的方式:

```ts
import { forwardRef } from "@nestjs/core";
@Module({
  imports: [forwardRef(() => BbbModule)],
})
export class AaaModule {}
```

```ts
import { forwardRef } from "@nestjs/core";
@Module({
  imports: [forwardRef(() => AaaModule)],
})
export class BbbModule {}
```

同样的，两个 Service 也可以使用这种方式相互引用:

```ts
@Injectable()
export class CccService {
  constructor(
    @Inject(forwardRef(() => DddService)) private dddService: DddService
  ) {}

  ccc() {
    return "ccc";
  }

  eee() {
    return this.dddService.ddd() + "eee";
  }
}
```

```ts
@Injectable()
export class DddService {
  constructor(
    @Inject(forwardRef(() => CccService)) private cccService: CccService
  ) {}

  ddd() {
    return this.cccService.ccc() + "ddd";
  }
}
```
