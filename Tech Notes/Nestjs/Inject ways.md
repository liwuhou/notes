使用 Injectable 装饰过的 class 即为 Provider，而 Provider 在 Module 中使用 `providers` 引入。

```ts
@Module({
  imports: [],
  controllers: [AppController],
  providers: [AppService], // 就这
})
```

然而，这其实是一种简写。完整的写法，一个 Provider 要有一个 token 和一个具体注入的对象，Nest 会自动将其实例化再注入。

```ts
@Module({
  providers: [{
    provide: AppService,
    useClass: AppService
  }]
})
```

`provide` 字段为该注入对象的 token，可以使用字符串，也可以使用这个对象的类指向。在 Controller 中，使用传入的 token 来指向注入的对象。

```ts
@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  foo(): string {
    return this.appService.bar()
  }
}
```

不想用构造器注入，也可以使用属性注入，通过 @Inject 指定注入的 provider 的 token 就行

```ts
@Controller()
class AppController {
  @Inject(AppService) private readonly appService: AppService
}
```

然后这个 token 也可以是一个字符串，也可以给这个注入起一个 token 之外的别名。后续 Controller 层注入也可以使用这个别名。

```ts
@Module({
  imports: [],
  contollers: [AppController],
  providers: [{
    provide: 'app_service',
    useClass: AppService,
    useExisting: 'AppService'
  }]
})
```

相应地，注入的时候就要使用 @Inject 手动指定注入的对象了：

```ts
@Controller()
export class AppController {
  constructor(
    @Inject('app_service') private readonly appService: Appservice,
    @Inject('AppService') private readonly _appService: Appservice,
  ) {}
}
```

其实相比之下，使用 class 注入，可以省去 @Inject 装饰的步骤，会更简便一些。

注入的对象除了指定 class 外，还可以直接指定一个值，让 IOC 窗口注入

![](http://cdn.liwuhou.cn/tmp/20230605063524.png)

然后在对象里注入它：

```ts
@Controller()
export class AppController {
  constructor(
    @Inject('app_service') private readonly appService: AppService,
    @Inject('person') private readonly person: { name: string, age: number }
  )

  getPersonName(): string {
    return this.person.name
  }
}
```

当然，provider 的值也支持动态生成，使用 `useFactory` 字段。

![](http://cdn.liwuhou.cn/tmp/20230605071809.png)

也可以使用 `useValue` 更简单直接的注入一个值，

```ts
const providers = [
  {
    provide: 'uuid',
    useValue: generateUUId()
  }
]
```

### 总结
声明 Provider 的方式有多种，最常用的就是 useClass，并且一般会简写，也就是直接传入 class。

声明 Provider 的方式
- 直接传入 @Injectable 装饰的类，默认的 token 就是 class，在 @Inject 的时候不需要指定注入的 token
- `provide` 服务的 token，用来被 @Inject 时，传入的用来标识的服务的标识。
- `useExisting` 只是用来起别名，基本用于兼容一些不同版本等场景
- `useClass` 传入类的方式，由 IOC 窗口负责实例化。
- `useFactory` 可以动态传入参数然后动态生成数据，支持异步
- `useValue` 简单直接传入数据

注入 Provider 的方式
使用 `useClass` ，更多的是使用简写的形式，较为常用

```ts
@Controller()
export class AppController {
  // 构造器注入
  constructor(
    private readonly appService: AppService, // useClass，简写形式，最为常用
    @Inject('app_service') private readonly appService2: AppService // 字符 token
  )

  // 属性注入
  @Inject(AppService)
  private readonly appService: AppService // useClass

  @Inject('app_service')
  private readonly appService2: AppService

}

```