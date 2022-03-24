在函数式编程中，有这么一个 currying函数，接收一个函数，返回一个柯里化后的函数

```ts
const func = (a: string, b: number, c: boolean) => {}
```

返回后的函数

```ts
(a: string) => (b: number) => (c: boolean) => void
```

观察可知，这个函数的返回值类型和参数类型是有关系的，所以这个柯里化函数的类型的定义要用到类型编程

每一个参数就返回一层函数，具体多少层数是不确定的，因此会有到递归

```ts
type CurriedFunc<Params, Return> =
  Params extends []
    ? () => Return
    : Params extends [infer Arg]
      ? (arg: Arg) => Return
      : Params extends [infer Arg, ...infer Rest]
        ? (arg: Arg) => CurriedFunc<Rest, Return>
        : never

declare function currying<Func>(fun: Func):
  Func extends (...args: infer Params) => infer Return ? curriedFunc<Params, Return> : never
```

返回值要对 Func 做一些类型运算，通过模式匹配提取参数和返回值的类型，传入 CurriedFunc 中来构造新的函数类型

构造的函数层数不确定，所以用到了递归，每次提取了 infer 声明的首个变量 Arg，其余的参数用 infer 提取到变量 Rest 来继续构造函数

直到 Params 是只有一个参数的时候，就构造一个 `(arg: Arg) => Return` 的函数类型返回，如果一开始 Params 就为空的话，就返回一个 `() => Return` 的函数类型

这样就提取出了 Params 中的所有元素，递归构造出了柯里化函数类型