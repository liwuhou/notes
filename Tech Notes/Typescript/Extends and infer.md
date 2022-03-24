### extends 的类型条件判断

相当于 js 的三元， `T extends [] ? [] : T` 类型 T 如果属于 `extends` 右侧的类型的子集，则返回 `?` 后面的类型，否则使用 `:`后面的类型

获取 tuple 的 length 属性

Tuple 的类型有 `length`的 indexed，可以直接通过 `Tuple["length"]` 访问到

### extends union 判断的规则

`union` 类型是一票类型的可选值

```TypeScript
type T = 1 | 2 | 3 // union
type F = [1, 2, 3]
type TF = F[number] // union
type Equal = T extends TF ? 'equal' : 'inequal'
```

### infer 的使用

^d00720

`infer` 关键字表示在 `extends` 语句中待推断的类型变量

简单示例如下：

```TypeScript
type ParamType<T> = T extends (...args: infer P) => any ? P : T;
```

在这个条件语句 `T extends (...args: infer P) => any ? P : T` 中，`infer P` 表示待推断的函数参数。

整句表示为：如果 `T` 能赋值给 `(...args: infer P) => any`，则结果是 `(...args: infer P) => any` 类型中的参数 `P`，否则返回为 `T`。

