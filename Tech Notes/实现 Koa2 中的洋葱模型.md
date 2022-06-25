Koa2 中利用了 es6 的 `async` 和 `await` 特性，巧妙地实现了洋葱模型，使用的时候通过一个 `await next()` 来将控制权移交给下一个中间件，而当后面的中间件执行完毕之后，控制权又会回到原来的中间件中，执行剩余的代码。即每个中间件都有两次执行机会。

![](http://cdn.liwuhou.cn/tmp/20220625100514.png)

Koa 源码中 `application.js` 中的类 `Application`，提供了一些属性和方法

```javascript
'use strict'

module.exports = class Application extends Emitter {

  constructor(option) {
    super()
    option = option || {}
    this.middleware = [] // use 方法中传入的中间件
  }

  use(fn) {
    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!')

    this.middleware.push(fn)
    return this
  }
}


```

当我们实例化一个 Koa 实例的时候，通过 `use` 方法，将中间件方法 `push` 进实例的 `middleware` 中。

```javascript
const Koa = require('koa')

const app = new Koa()

app.use(async (ctx, next) => {
  console.log('middle 1 start')
  await next()
  console.log(ctx.state.someProp)
  console.log('middle 1 end')
})

app.use(async (ctx, next) => {
  console.log('middle 2 start')
  ctx.state.someProp = 'Get 2'
  await next()
  console.log('middle 2 end')
})

app.listen(3000, () => {
  console.log('server is running at 3000 port')
})
```

`app` 中的中间件打印顺序如下：

```text
middle 1 start
middle 2 start
middle 2 end
Get 2 
middle 1 end
```

让我们再看看 `listen` 方法

```javascript
module.exports = class Application extends Emitter {

  constructor(option) {
    super()
    option = option || {}
    this.middleware = []
  }

  use(){/*...*/}

  listen(...args) {
    const server = http.createServer(this.callback())
    return server.listen(...args)
  }


  callback() {
    const fn = compose(this.middleware) // 关键一步

    const handleRequest = (req, res) => {
      const ctx = this.createContext(req, res)
      return this.handleRequest(ctx, fn)
    }

    return handleRequest
  }
}
```

`listen` 这里通过调用了原生Node中 `http` 模块下的 `createServer` 方法，传入了 `this.callback()` 方法的回调。
`callback` 中对实例中所有的中间件进行了处理，关键的一步就是 `const fn = compose(this.middleware)`，让我们看看 `compose` 方法做了什么

```js
const compose(middleware) {
  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
  for (let fn of middleware) {
    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
  }

  return function (context, next) {
    let index = -1
    return dispatch(0)

    function dispatch(i) {
      if (i <= index) return Promise.reject(new Error('next() called multiple times!'))

      index = i
      let fn = middleware[i]
      let (i === midleware.length) fn = next
      if (!fn) return Promise.resolve()
      try {
        return Promise.resolve(fn(context, dispatch.bind(null, i + 1)))
      } catch(err) {
        return Promise.reject(err)
      }
    }
  }
}
```

`middleware` 是一系列中间件的方法，这里构造了每个函数都有一个 `next` 的方法去调用下一个中间件的方法，利用 `async/await` 特性，自然地实现了洋葱模型。

