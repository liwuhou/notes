### useRef

这个 `ref` 是 `reference` 的意思，表示一个引用的数据。在 React 的 class 组件中，经常使用jsx 元素属性中的 `ref` 来获取到其挂载之后的真实 dom

```jsx
class App extends React.Component {
  constructor(this) {
    super(this)
    this.refDiv1 = null
    this.refDiv2 = { current: null }
  }

  render() {
    return (
      <>
        <div ref={(dom) => this.refDiv1 = dom}>Hello world</div>
        <div ref={refDiv2}>这样也能获取</div>
      </>
    )
  }
}
```

在 FP 中，`ref` 的含义就多了起来，同样也可以用来获取 jsx 中的真实 dom

```jsx
function App() {
  const refDiv = React.useRef(null)
  const [$div, setDiv] = React.useState(null)

  return (
    <>
      <div ref={refDiv}>Hello world</div>
      <div ref={setDiv}>这样也能获取</div>
    </>
  )
}
```

是不是更简洁了，除此之外，也可以用 useState 获取 dom，属实是方便了很多。同 class 组件中的用法一致，如果是个 包含 `{current: null}` 的对象，那么可以直接传入 jsx 的 `ref`属性中，React 会自动赋值。

除此之外，FP 中的 `useRef` 这个 Hook，不仅用来获取 jsx 的真实 dom，还被赋予了更多的意义。

`useRef` 返回了一个拥有可变的属性 `.current` 的 js 对象，并且这个对象在组件的整个生命周期中都不会改变，也就是说，`useRef` 会在每次 render 组件的时候，都返回给你同一个对象。

值得注意的是，React 也不会监听 `useRef` 中内容的改变，改变这个 `.current` 属性并不会导致组件重新渲染。

这里贴一下 `useRef` 的实现源码

```js
function mountRef(initValue) {
  var hook = mountWorkInProgressHook()
  var ref = {
    current: initValue
  }

  {
    Object.seal(ref)
  }

  hook.memoizedState = ref
  return ref
}

function UpdateRef() {
  var hook = updateWorkInProgressHook()
  return hook.memoizedState
}
```

首先 React 的组件在挂载和更新时会调用对应 hooks 中的 `mountXxx` 方法和 `updateXxx` 方法，`mountWorkInProgressHook` 和 `updateWorkInProgreeHook` 也分别是组件挂载和更新获取当前处理的 hook的方法。当组件挂载的时候，也就是 `mountRef`，React 会创建一个拥有 `.current` 属性的冻结了的对象，赋值给组件对应的 fiber 节点的 `memoizedState`。有关 `Object.seal` 的方法可以看这里——[Object.seal() - JavaScript | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/seal)。而在更新的时候，只是将 fiber 中保存的这个对象返回出去而已。可以说是非常的简单了。

`useRef` 返回的对象本身不会改变这点挺关键，我们可以在 `useEffect` 和 `useCallback` 的第二个参数收集依赖的时候，只跟踪 `userRef` 返回的这个对象。在回调中使用 `.current` 属性来拿到我们想要的数据。

除此之外，获取 jsx 的真实 dom 还有其它的一些方法，比如可以获取，比如 jsx 中的 ref 可以传入一个回调函数，那么可以传入一个方法，来获取到 dom 的一些属性。

```jsx
function MeasureExample() {
  const [height, setHeight] = useState(0)

  const measuredRef = useCallback(node => {    
    if (node !== null) {      
      setHeight(node.getBoundingClientRect().height)
    }  
  }, [])
  
  return (
    <>
      <h1 ref={measuredRef}>Hello, world</h1>      
      <h2>The above header is {Math.round(height)}px tall</h2>
    </>
  )
}

```

也可以进一步封装，抽成一个 hook

```jsx
function MeasureExample() {
  const [rect, ref] = useClientRect()
  return (
    <>
      <h1 ref={ref}>Hello, world</h1>
      {rect !== null &&
        <h2>The above header is {Math.round(rect.height)}px tall</h2>
      }
    </>
  )
}

function useClientRect() {
  const [rect, setRect] = useState(null)
  const ref = useCallback(node => {
    if (node !== null) {
      setRect(node.getBoundingClientRect());
    }
  }, [])
  return [rect, ref]
}
```

ok，目前为止，我们不仅知道了怎么获取 jsx 对应的真实 dom，还知道了 `useRef` 衍生的一些用法。那么如果是要获取一个自定义组件的 ref 要怎么办呢。

### useImperativeHandle

```js
useImperativeHandle(ref, createHandle, [deps])
```

`useImperativeHandle` 必须用于被 `React.forwardRef` 包裹的组件当中，用于定义该组件暴露在外部 ref 中的属性或方法。

```jsx
const FancyInput = React.forwardRef((props, ref) => {
  const inputRef = React.useRef()

  useImperativeHandle(ref, () => ({
    diyFocus: () => inputRef.current.focus()
  }))

  return <input ref={inputRef} />
})

const App = () => {
  const fancyInputRef = React.useRef()

  return (
    <>
      <FancyInput ref={fancyInputRef} />
      <button onClick={() => fancyInputRef.current.diyFocus()}>Focus</button>
    </>
  )
}
```

