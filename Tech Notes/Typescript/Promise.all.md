类型与类型之间有关系的时候，就自然使用上类型编程，在 Promise 中，all 方法和 race 方法是这样的

```ts
interface PromiseConstructor {
  all<T extends readonly unknown[] | []>
    (values: T): promise<{
      -readonly [P in keyof T]: Awaited<T[P]>
    }>

  race<T extends readonly unknown[] | []>
    (values: T): Promise<Awaited<T[number]>>

}
```

因为 `promise.all` 是等所有的 promise 执行完一起返回

![](http://cdn.liwuhou.cn/tmp/20220315184932.png)

而 `promise.race` 是有一个只执行完就返回，返回的类型都需要用到参数 `Promise` 的 `value` 类型

![](http://cdn.liwuhou.cn/tmp/20220315185005.png)

在 `Promise.all` 的类型实现中，返回一个新的数组类型也可以使用映射类型语法来构造一个全新的索引类型（class、对象、数组等聚合多个元素的类型都是索引类型）

新的索引类型的索引来自之前的数组 `T`，也就是 `P in keyof T`，值的类型是之前的值的类型，但要做下 `Promise` 的 `value` 类型提取，用内置的高级类型 `Awaited` 获取

同时需要把 `readonly` 去掉，也就是 `-readonly`
