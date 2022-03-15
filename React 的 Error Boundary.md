在 React16 之前，React 没办法捕获到组件内部的 javascript 错误，这会导致 React 内部的状态被破坏。并在下次渲染的时候，产生可能无法追踪的错误。
为了解决这些问题，React 引入了一个新的概念——错误边界

赋予 React 组件能够捕获发生在其了组件任何位置的 Javascript 错误，并打印或是上报这些错误信息，同时展示降级 UI，而不会渲染这些崩溃的组件

> 错误边界无法捕获以下场景中产生的错误
> 
> - 事件处理
> - 异步代码（例如 setTimeout 和 requestAnimationFrame 回调函数）
> - 服务商渲染
> - 它自身抛出来的错误（并非它的子组件）
> 

### 定义一个错误边界
如果一个 class 组件中定义了 static `getDerivedStateFromError` 或 `componentDidCatch` (或两个) 生命周期，那么这个组件就会变成一个错误边界（Error Boundary）
当抛出错误的时候，可以使用 `getDerivedStateFromError` 渲染备用 UI，使用 `componentDidCatch` 去打印或上报一些错误信息的操作。

```TSX
import React from 'react'

interface Props {}
interface State {
  hasError: boolean
}

export default class ErrorBound extends React.Component<Props, State> {
  constructore(props: Props) {
    super(props)
    this.state = {
      hasError: false
    }
  }

  static getDerivedStateFromError() {
    return { hasError: true }
  }

  componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
    // 上报错误
    arms.push('/error_boundary', errorInfo.componentStack || error.message)
  }

  return () {
    return this.state.hasError ? (
      <div>出错啦</div>
    ) : (
      this.props.children
    )
  }
}
```

然后像正常一样去使用这个错误边界组件即可

```jsx
<ErrorBoundary>
  <RestComponent />
</ErrorBoundary>
```

### getDerivedStateFromError

```js
static getDerivedStateFromError(error)
```

此生命周期会在后代组件抛出错误后被调用。它将抛出的错误作为参数，并返回一个值以更新 state

```jsx
class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props)
    this.state = { hasError: false }
  }

  static getDerivedStateFromError(error) {
    // 返回一个新的 state 以更新视图显示降级 UI
    return { hasError: true }
  }

  render() {
    if (this.state.hasError) {
      return (
        <h1>Something went wrong.</h1>
      )
    } else {
      return this.props.children
    }
  }

}
```

> 值得注意的是，`getDerivedStateFromError()` 会在渲染阶段调用，因此不允许出现副作用。如需产生副作用的情况，就要使用 `componentDidCatch` 

### componentDidCatch

```jsx
componentDidCatch(error, info)
```

此生命周期在后代组件抛出错误后被调用，它接收两个参数：
1. `error` —— 抛出的错误
2. `info` —— 带有 `componentStack` key 的对象，其中包括[有关组件引发错误的栈信息](https://zh-hans.reactjs.org/docs/error-boundaries.html#component-stack-traces)。

`componentDidCatch()` 会在 commit 阶段被调用，因此允许执行副作用，它可以被用来记录错误之类的情况

```TSX
class ErrorBoundary extends React.Component {
  componentDidCatch(error, info) {
    logComponentStackToMySevice(info.componentStack)
  }
}
```

> React 的开发和生产构建的版本在 `componentDidCatch()` 的方式上有轻微差别
> 在开发模式，错误会冒泡至 `window`，这意味着任何 `window.onerror` 或 `window.addEventListener('error', callback)` 会中断这些已经被 `componentDidCatch` 捕获的错误。
> 相反，在生产模式下，错误不会冒泡，这意味着任何根错误处理器只会接受那些没有显式地被 `componentDidCatch` 捕获的错误。

如果发生了错误，虽然可以使用 `setState` 在 `componentDidCatch` 中渲染降级 UI，但在未来的版本中将不推荐这么做，可以使用 `getDerivedStateFromError` 来处理降级 UI 的渲染