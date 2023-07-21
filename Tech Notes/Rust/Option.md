在枚举那篇文章中提到过[[Enum#Option 枚举处理空值|Option]],主要是用它来解决一些变量是否存在有效值的问题。再回顾一下 `Option` 的定义：

```Rust
enum Option<T> {
  Some<T>,
  None,
}
```

其实吧，这个枚举的作用就在于，可以定义一个变量他有可能有值（`Some<T>`），也有可能无值（`None`)。
