Proxy 和 Reflect 都是 Javascript 的内置对象，前者是创建一个对象的代理，后者则是提供了拦截 Javascript 操作的方法。

### Proxy

可以用于创建一个原始对象的代理，并可对这个代理对象定义其 `getting` 、 `setting` 和属性。

```javascript
const proxed = new Proxy(target, handler)
```

- `Proxy` 的构造器接收两个参数
- `target`: 需要被代码的原理对象
- `handler`: 