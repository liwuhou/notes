AOP 的好处是可以把一些通用的逻辑分享到切面中，保持业务逻辑的纯粹性，并且还可以很方便的复用切面逻辑。
在 Nestjs 中，实现 AOP 的形式总共有五种：Middleware、Guard、Pipe、Interceptor、ExceptionFilter。

### 中间件 Middleware
Nest 的底层是 Express，而 Express 的中间件的洋葱模型也是一种 AOP，因此 Nest 自然可以使用中间件的形式实现 AOP。Nest 的中间件可以细分为全局中间件和路由中间件：

全局中间件就是 Express 那种中间件，在请求之前和之后加入一些操作，每个请求都会走到这里：

```ts
// main.ts
import { AppModule } from './app.module'
import Logger from 'logger'

const app = await NestFactory.create(AppModule)

app.use(Logger) // 使用方式
await app.listen(3000)
```

而路由中间件只是针对某个路由，范围会更小一点，所有经过该路由的请求都会经过中间件处理，实现方式是对应的 Module 要继承  NestModule 然后要实现 configure 方法，在参数 `consumer` 中注入中间件：

```ts
// app.module.ts
import { Module, NestModule } from '@nestjs/common'
import { LoggerMiddleware } from 'Logger'

export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // 注入中间件
      .forRoutes('person') // 为这个路由
  }
}
```

### 路由守卫 Guard

![](http://cdn.liwuhou.cn/tmp/20230601081537.png)

Guard 会在调用某个 Controller 之前判断权限，返回 `true` 或者 `false` 来决定是否放行。
声明一个路由守卫需要实现 CanActivate 接口，实现 `canActivate` 方法，可以从 `context` 拿到请求的信息，然后做一些权限验证等处理，方法返回一个 `boolean` 值。

```ts
import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common'

@Injectable()
export class RolesGuard implements CanActivate {
  canActivate(
    context: ExecutionContext
  ): boolean | Promise<boolean> | Observable<boolean> {
    return true
  }
}
```

然后通过 UseGuards 加入到 IOC 容器中，**就可以在某个 Controller 中启用了**：

```ts
import { Controller, UseGuards } from '@nestjs/common'

@Controller('roles')
// @UseGuards(RolesGuard) 等价，Nest 内部会将类初始化
@UseGuards(new RolesGuard())
export class RolesController {}
```

也可以像全局中间件一样，加到全局。

```ts
const app = await NestFactory.create(AppModule)
app.useGlobalGuards(new RolesGuard())
```

> 注： Guard 可以抽离路由访问 Controller 的控制逻辑，但是并不能对请求、响应做修改，如果需要对某个请求做权限判断并修改请求和返回的话，可以使用 Middleware 或 Interceptor


### 拦截器 Interceptor

![](http://cdn.liwuhou.cn/tmp/20230601081604.png)

Interceptor 可以在目标 Controller 方法前后加入一些逻辑，创建 Interceptor 的方式是这样的：

```ts
@Injectable()
export class LogginInterceptor implements NestInterceptor {
  intercept(context: ExexutionContext, next: CallHandler): Observable<any> {
    console.log('Before ...')
    // 调用目标 Controller 服务 之前的逻辑

    return next.handle()
      .pipe(
        tap(() => {
          console.log('Affter')
          // 调用之后的逻辑

      )
  }
}
```

Controller 之前之后的处理逻辑可能是异步的，Nest 中通过 rxjs 来组织它们，所以可以使用 rxjs 的各种 operator

Interceptor 支持每个路由单独启用，只作用于某个 Controller，也能全局启用，作用于全局：

```ts
// 作用于某个 Controller
// app.controll.ts
@Controller()
@UseInterceptors(new LoggingInterceptor())
export AppController {

  // 作用于某个单独的路由
  @Get(['bar', 'foo'])
  @UseInterceptors(new LogginInterceptor())
  getBar() {
    return 'bar'
  }
  
}

// 也能作用于全局
// main.ts
async function bootstrap() {
  const app = await NestFactory.create<NestExpressApplication>(AppModule)
  
  app.useGlobalInterceptors(new LogginInterceptor())
}
```

### Pipe
日常开发中，除了对路由权限的控制，目标 Controller 之前之后的处理外，也有一些对参数的处理逻辑，比如把前端传来的小驼峰转为下划线，或者是对参数进行一些其它的转换。

![](http://cdn.liwuhou.cn/tmp/20230608064343.png)

创建 Pipe 要实现 PipeTransform 接口，并实现 `transform` 方法：

```ts
@Injectable()
export class Camel2Underline implements PipeTransform {
  transform(value: any, metadata: ArgumentMetadata) {
    return value
  }
}
```

`transform` 方法可以对参数做一些验证，比如格式、类型是否正确，如果不正确就抛出异常。也可以做一些转换，返回转换后的参数。

Nest 中内置有 9 个 Pipe，从命名中也能看出它们的意思

*   ValidationPipe
*   ParseIntPipe
*   ParseBoolPipe
*   ParseArrayPipe
*   ParseUUIDPipe
*   DefaultValuePipe
*   ParseEnumPipe
*   ParseFloatPipe
*   ParseFilePipe

Pipe 可以只对参数生效，某个路由生效，也可以对每个路由生效：

```ts
// 全局生效
// main.ts
async function bootrap() {
  const app = await NestFactory.create(AppModule)
  app.useGlobalPipes(new ValidationPipe())
}

// 对某个路由
@Controller()
export class AppController {
  @Get()
  @UsePipes(new ValidationPipe()) // 对某个路由
  hello(@Param('name',  new ValidationPipe()) name: string) { // 对某个参数
    return `hello ${name}`
  }
}
```

### ExceptionFilter
不管是 Pipe、Guard 还是 Interceptor，抑或是最终调用的 Controller，过程中都会抛出一些异常，而 ExceptionFIlter 就是一个能对异常做出响应的映射。

![](http://cdn.liwuhou.cn/tmp/20230608065730.png)

创建一个 ExceptionFilter 需要先实现 ExceptionFilter 接口，实现 catch 方法：

```ts

@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp()
    const response = ctx.getResponse<Response>()
    const request = ctx.getRequest<Request>()
    const status = exception.getStatus()

    response
      .status(status)
      .json({
        statusCode: status,
        timestamp: new Data().toISOString(),
        path: request.url
      })
  }
}

```

这样就能进行异常拦截了，对拦截的异常做一些处理，可以返回对应的响应，给用户更友好的提示。
同样的，ExceptionFilter 也能选择全局开启，或是只作用于单独路由

```ts
// 全局 main.ts
async function bootstrap() {
  const app = await NestFactory.create(AppModule)
  app.useGlobalFilters(new HttpExceptionFilter())
}

// 某个路由 app.controller.ts
@Post()
@UseFilters(new HttpExceptionFilter()) 
async setAge(@param('age', new ParseIntPipe() age: number)) {
  return this.appService.changeAge(age)
}

```

几种 AOP 机制的顺序