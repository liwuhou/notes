装饰器是设计模式中常用的一种设计模式，一般用来装饰类或者类中的静态方法。达到不更改被装饰的实体但却能改变实体的行为的一种效果。

而在 ES 2017 标准中，正式引入了装饰器语法，这其实是一种语法糖，但却让开发的代码看起来更简捷更优雅。

```ts
@Component()
export default class App {}

class A {

  @Logged
  Login() {}
}
```

而在 Nest 中，装饰器语法使用的场景更多，诸如 Controller、Guard、Interceptor 等，都是可装饰类或者各种类方法，而 Nest 更是靠着 Get、Post、Param、Body 等装饰器实现 RestFull Api。同时整个框架中的 IOC 设计思想和实现依赖注入的思路，都依托于装饰器。

```ts
@Controller()
export class AppController {
  @Inject(AppService)
  private readonly appService: AppService
}

```

### Nestjs 中自定义装饰器

使用 nest 的 generate 命令快速创建自定义装饰器模板

```bash
nest g decorator xxx 
```


对原有的装饰器进行一层封装

```ts
import { SetMetadata } from '@nestjs/common'

export const Xxx = (...args: string[]) => Setmetadata('xxx', args)
```

使用的时候就可以这样使用

```ts
// controller.ts

@Controller()
export class AppController {
  @Get('/foo')
  @Xxx('aaa')
  @SetMetadata('xxx', 'aaa') // 等价于
  foo() {
    return 'foo'
  }
}
```

也可以将多个装饰器合并为一个

```ts
import { applyDecorators, Get, SetMetadata, UseGuards } from '@nestjs/common'
import { XxxGuard } from './xxx.guard'

export const Xxx = (path, role) => applyDecorators(
  Get(path),
  UseGuards(XxxGuard),
  SetMetadata('role', role)
)
```

使用的时候就可以略过很多装饰器

```ts
// controller.ts

@Controller()
export class AppController {
  @Get('/foo')
  @UseGuards(XxxGuard)
  @SetMetadata('role', 'aaa') // 等价于
  foo() {
    return 'foo'
  }


  // 上面可以直接这么套一个装饰器
  @Xxx('/foo', 'aaa')
  foo2() {
    return 'foo'
  }
  
}
```

自定义参数装饰器

Nestjs 中有一类用于装饰参数的装饰器

```ts

@Controller()
export class AppController {

  @Get('foo')
  getFoo(@Param() b) { // Param 是 Nest 提供的，用来装饰参数的装饰器
    return b
  }
}

```

这类装饰器也可以自定义

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common'

export const Ccc = createParamDecorator(
  (data: string, ctx: ExecutionContext) => {
    return 'ccc'
  }
)
```

这个 data 就是我们传入装饰器中的参数，而 ExecutionContext 可以用来取出 `request`和 `resonse` 对象。利用这点，我们可以很轻易地实现 Nest 中内置的 @Param, @Query, @Ip, @Headers。

```ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common'
import { Request } from 'express'

export MyHeaders = createParamDecorator(
  (key: string, ctx: ExecutionContext) => {
    const request = ctx.switchToHttp().getRequest() // 取出 request 对象
    return key ? request.headers[key] : request.headers
  }
)
```


### 总结

组合多个装饰器可以使用 `applyDecorators` 方法，参数装饰器可以通过 createParamDecorator 来创建，然后通过 ExecutionContext 来获取到 request 和 response，进而实例诸如 @Query、@Headers 等内置装饰器。