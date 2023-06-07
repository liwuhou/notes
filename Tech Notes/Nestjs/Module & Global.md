Nest 中一个模块B的引用另一个模块B的服务比较麻烦。首先，被引入的模块A，其服务要在 `exports` 字段声明导出，然后在目标模块B中的 `imports` 字段声明引入模块A。

```ts
// Module A
@Module({
  controllers: [AController],
  providers: [AService],
  exports: [AService], // 导出模块
})


// Module B
@Module({
  controllers: [BController],
  providers: [BService],
  imports: [AController]
})

// Service B
@Injectable()
class BService{
  constructor(private readonly aService: AService /* 直接声明 */) {}

  findAll() {
    return aService.findAll()
  }
}

```

如果有多个模块要引入模块 A，那就要在每个模块中都引入一遍吗？那也太麻烦了，这里，就可以考虑使用一下把模块 A 声明为全局模块：

```ts
// Module A
@Global()
@Module({
  controllers: [AController],
  providers: [AService],
  exports: [AService]
})

// Module B
@Module({
  controllers: [BController],
  providers: [BService],
  imports: [] // 没有在些引用
})

// Service B 没有改动
```

这这是全局模块，项目中最好通过项目架构来标识全局模块的位置，防止注入太多 Provider，但却很难找到来源，降低代码的可维护性。
