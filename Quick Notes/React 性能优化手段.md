### 发现性能问题的方式
1. React Dev Tools
2. Profile 组件 时间
3. Chrome Dev Tools —— Performance

### 优化的一种理念——把变化的部分中变化的部分中抽离

```jsx
import React, { useState } from 'react'
import { render } from 'react-dom'

function App () {
  const [state, setState] = useState('')

  return (
    <>
      <input value={state} onChange={(e) => setState(e.target.value)} />
      <ExpensiveCpt />
    </>
  )
}

function ExpensiveCpt () {
  const now = Perform.now()
  while(Perform.now() - now > 100) {
    // 模拟耗时的操作
  }
  
  return <div>耗时的组件</div>
}

render(<App />, document.querySelector('#root'))
```

![](http://cdn.liwuhou.cn/tmp/20220405181701.png)

这个 App 组件在每次输入框中输入的时候都会有 100ms 的延迟，即使 `ExpensiveCpt` 组件没有任何的 `props` 传值，内部也没有任何的 `state` 流转，可还是刷新了 `ExpensiveCpt` 组件。

这里可以做的优化有很多，比如给 `ExpensiveCpt` 包一层 `React.memo()` 方法，让组件不会每次都随着父组件去 render

```jsx
const ExpensiveCpt = React.memo(() => {
  const now = performance.now()
  while(performance.now() - now < 100) {}
  
  return <div> I'm Expensive! </div>

})

  

function App () {
  const [state, setState] = React.useState('')

  return (
    <div>
      <input value={state} onChange={(e) => setState(e.target.value)} />
      <ExpensiveCpt />
    </div>
  )
}
```

也可以通过提取变化的部分封装，这样 React 就单独地去 render 这一部分。

```jsx
const Input = () => {
  const [state, setState] = React.useState('')

  return (
    <input value={state} onChange={(e) => setState(e.target.value)} />
  )
}

function App () {
  return (
    <>
      <Input />
      <ExpensiveCpt />
    </>
  )
}
```

这样在输入框输入的时候，就只是会对 `<Input />`  进行重新 render，就不会重复地去渲染耗时的 `<ExpensiveCpt />` 组件了。

### 框架源码学习路径
可以先从小而美的各种库看起，比如 redux、axios 这些，从一些文章中就可以跟着直接看
接着看看模块化地比较好的 Vue，一个模块就是一个文件，分别看些单个模块的文章，配合源码

### React 源码学习路径
建议自顶向下学，先搞懂理念，再理清楚架构，最后着眼源码中的实现。
理念可以从官网文档、React conf、核心开发者博客阅读到
React 的架构分为 Scheduler、Reconciler、Renderer 三大模块
知道这三大块的实现原理，然后再从`React.render` 这个入口方法看起。