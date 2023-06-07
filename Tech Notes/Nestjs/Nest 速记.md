Nest 在玩一种算不上很新，但在前端领域上来说很新的东西 —— IoC（Invert of Control），即控制反转。
常规在服务器开发中，使用 MVC 等架构时， Controller 会依赖很多模块，这些模块又可能会依赖更多的其它的模块。到头来时序性和依赖可能会错综复杂，开发起来很不方便。而 Nestjs 通过利用装饰器和 TS，让 Controller 使用这些依赖变得非常无脑。

### Module

使用 `@Injectable` 来声明（就是装饰）某个 service 类可被注入或可注入的特性，然后使用 `@Controller` 来声明和赋予其 Controller 特性，随后在 module 层声明。这些被 `@Injectable` 装饰的类称为 `Provider`

```ts
import { Injectable, Controller, Module, Get } from '@nestjs/common'
// service
@Injectable()
class AppService() {
  hello() { return 'hello'}
}

// controller
@Controller()
class AppController {
  constructor(
    private readonly appService: AppService, // 注意看这行神奇的 TS 注解
  ) {}

  @Get('/' /* 默认就是 / */) {
    return this.appService.hello() // 使用 service 的功能
  }
}

@Module({
  imports: [], // 导入其它 Modules，然后其 Service 和 Controller 能使用该 Module 的 exports 字段暴露出来的 Service
  controllers: [AppController], // 等待被注入 Services 的 Controllers
  prividers: [AppService], // 可供 Controllers 或者其它 Services 注入的 Services
  exports: [] // 对外暴露的 Services，可被其它 Modules 引入并使用
})
class AppModule {}

```

### DI

需要注入（Dependency Injected）的依赖在 module 层的 `providers` 字段传入，然后就可以在 module 层下的 controller 和其它 services 中注入了。

module 层下的其它实体引入注入也极其的简单，甚至有点无脑了。

```ts
@Controller()
class Controller {
  constructor(
    private readonly service: AService,
    private readonly service: BService
  ) {}
}
```

是的，只管在构造函数里引入，Nest 会自动通过引入的 TS 类型，去注入相应的 Service，这点也是开发者最爽的。

### 全局依赖
当一个模块需要被很多地方引用的时候，就要在很多模块下使用 `imports` 字段去引入，Nest 提供了一种全局模块的方式，让我们可以将一个模块声明为全局模块，让各处的模块都能引入并使用。

Nest 提供了一个 `Global` 的装饰器，可以将 `Module` 装饰为全局模块，其它模块无需在 `Module` 声明 `imports` 字段，即可使用全局模块在 `exports` 导出的 `Service` 等功能。

```ts
import { Global, Module, Injectable } from '@nestjs/common'
import { GlobalController } from 'src/global/global.controller'
import { GlobalService } from 'src/global/global.service'

@Global()
@Module({
  controllers: [GlobalController],
  providers: [GlobalService],
  exports: [GlobalService]
})


// 使用的地方
@Injectable()
class OtherService() {
  constructor(private readonly globalService: GlobalService) {}

  otherService() {
    return this.globalService.globalMethon()
  }
}

```


### AOP

![](http://cdn.liwuhou.cn/tmp/20230531074730.png)

在后端的 MVC 架构中，处理请求到了 Controller 层时，可能还会经过 Service 层或是 Repository，但如果我们需要在调用链路里加一些诸如日志记录、权限控制、异常处理的通用逻辑的话，直接改造 Controller 层代码就显得不够优雅，Controller 中也会被业务无关的逻辑入侵。这个时候就可以使用面向切面编程，好处是可以将一些与业务逻辑无关的通用逻辑分离到切面中，保持业务逻辑的纯粹性，这样不仅可以复用切面逻辑，也可以动态的增删。


### Get

构建一个 RESTful API 服务器，需要用到各类请求方法，在 Nestjs 中已经帮我们封装了各种装饰器，可以很方便的开发各类服务，比如 `get` 请求等

```ts
import { Controller, Get, Post } from '@nestjs/common'

class Controller {
  @Get()
  returnIndex() {
    return 'indexPage'
  }

  @Post('/post')
  @Get('post')
  postMessage(data) {
    // TODO:
  }
}
```