vue2的时候看过源码，知道vue是通过Object.defineProperties 这个属性来收集依赖和触发依赖的。

在vue3中的 ref 和 effect 是这样用的

```js
import { ref, effect } from '@vue/reactivity'

const a = ref(0)
let b = 0

effect(() => {
  b = a.value + 10
  console.log(b)
})

a.value = 20
```

ref 与 effect 之间的配合就跟崔大画的这个流程图一样。

![](http://cdn.liwuhou.cn/tmp/20221109235939.png)

再加上vue3的响应式用了Proxy重写了，我马上就想到了第一版的实现。


```javascript

const ref = (value) => {
  const wrap = {} // 构造一个对象，用来盛放value，并且被 Proxy 代理
  const deps = [] // 用来收集依赖

  return new Proxy(wrap, {
    get(target, prop) {
      if (prop === 'value') {
        // TODO: 访问了属性的时候，收集依赖
        return target[prop]
      }
    },
    set(target, prop, value) {
      if (prop === 'value') {
        target[prop] = value
        // TODO: 触发了依赖
      }
    }
  })
}

```

`ref` 方法返回一个代理对象，通过 get 访问属性的时候收集依赖，set 改变属性的时候触发收集的依赖方法。

而我们的使用 effect 的时候，传入第一个函数会在 ref 包裹后的对象被改变的时候，触发。有个小细节是，最开始的时候，传进去的函数是会先被触发一次的。

那么effect的方法大概要做什么工作就有思路了

```javascript
let tmpfn = null // 用来暂存依赖

const effect = (fn ) => {
  tmpFn = fn
  fn() // 执行一遍
}


```

此时执行了fn的方法之后，fn的方法里有访问经 ref 方法包裹的值，就会触发对应的 get 方法，那我们可以在 get 方法中收集依赖，怎么收集呢？就是在effect执行的时候，用一个全局的变量，缓存依赖，在接下来的 Proxied  对象的get 方法里收集起来。自然，后续对 proxied 对象的set 就是触发闭包中收集的依赖了。

```javascript
const ref = (value) => {
  const wrap = {}
  const deps = []
  return new Proxy(wrap, {
    get(target, prop) {
      if (prop === 'value') {
        if (typeof tmpFn === 'function') { // 此时tmpFn全局上有值的时候，说明就是刚刚effect主动触发的，方便这里收集依赖，在set里使用。
          deps.push(tmpFn)
          tmpFn = null // **
        }
      }
      return target[prop]
    },
    set(target, prop, vlaue) {
      if (prop === 'value') {
        target[prop] = value
        deps.forEach((dep) => dep?.()) // set之后，循环执行收集的依赖
        return true
      }
      return false
    }
  })
}

```

实现完毕之后，引入我实现的方法，跟着例子跑一遍，牛逼效果一样一样的，说明思路是对的，代码至少是能完成要求的。

### 改进

第一版写得肯定很粗糙，其实还有很多地方改进，看了崔大视频的后面部分，发现了几点。
1. deps 是用了 Set 的格式来避免重复收集依赖
2. 然后将 deps 单独解耦抽象为一个类。
3. 然后我打了 `**` 标记的那行代码被移到effect方法里。

其实也是，执行effect传入的fn时，fn内并不一定只有一个Proxied 对象被访问到，如果按我第一版这么设计的话，后面的 Proxied 对象就收集不到依赖了。
改后的代码

```javascript
let currentEffect

class Deps {
  constructor(value) {
    this.deps = new Set()
  }

  // 依赖收集
  depend() {
    if (currentEffect) {
      this.deps.add(currentEffect)
    }
  }

  // 触发依赖
  notify() {
    this.deps.forEach((dep) => dep?.())
  }
}

const effect = (fn) => {
  (tmpFn = fn)()
  tmpFn = null
}

const ref = (value) => {
  const deps = new Deps()

  return new Proxy({value}, {
    get(target, prop) {
      if (prop === 'value') {
        deps.depend();
        return target[prop]
      }
    },
    set(target, prop, value) {
      if (prop === 'value') {
        deps.notify()
      }
    }
  })

}
```

打完收工