在枚举那篇文章中提到过[[Enum#Option 枚举处理空值|Option]],主要是用它来解决一些变量是否存在有效值的问题。再回顾一下 `Option` 的定义：

```Rust
enum Option<T> {
  Some<T>,
  None,
}
```

其实吧，这个枚举的作用就在于，可以定义一个变量他有可能有值（`Some<T>`），也有可能无值（`None`)。

### is_some

在 `Option` 类型中，不管是 `Some` 还是 `None`，都有些共同的方法，其中一个比较有用的就是 `is_some`，可以用来判断 Option 类型是 `Option::Some` 成员还是 `Option::None`。

```Rust
let x: Option<u32> = Some(2);
assert_eq!(x.is_some(), true);
let x: Option<u32> = None;
assert_eq!(x.is_some(), false);
```

### is_none

与 `Option::is_some` 对应，还有个 `is_none` 方法，用来判断值是否是 `Option::None`，可以看作是 `is_some` 的取反。

### unwrap 系列方法

`Option` 中，另一类比较强大且好用的方法就是 `unwrap` 类的方法了。“这一类” 方法的意思，有很多 `unwrap` 衍生的方法。

#### unwrap

头铁方法，直接返回 `Option::Some` 类型。你也许会问了，那如果值是 `Option::None` 咋整，编译器的回答永远只有一个 —— Panic。

```Rust
let x = Some("air");
assert_eq!(x.unwrap(), "air");
let x: Option<&str> = None;
assert_eq!(x.unwrap(), "air"); // panic
```

`unwrap` 方法太过头铁了，如果你对某个 `Option` 类型的取值很有信心，知道它肯定是 `Some`，那可以大胆使用 `unwrap`。但软件工程就没有永恒的事，谁知道哪一天，其它人维护（我感觉大概率会是几天后的你自己）就踩了你这个 `unwrap` 的坑。所以，Rust 官方也推荐使用 `unwrap_or`，`unwrap_or_else` 或 `unwrap_or_default` 方法，来对 `None` 情况做一层兜底。

#### unwrap_or

返回一个 `Some` 类型值或提供的默认值。

```Rust
assert_eq!(Some("car").unwrap_or("bike"), "car");
assert_eq!(None.unwrap_or("bike"), "bike");
```

#### unwrap_or_else

与 `unwrap_or` 方法类似，返回一个 `Some` 类型值，或是返回传入的函数调用的返回值。

```Rust
let x: Option<i32> = None;
let k = 2;
assert_eq!(x.unwrap_or_else(|| 2 * k), 4);
```

看着好像与 `unwrap_or` 的不同之处是传入的是个函数，可以动态的返回值。但 `unwrap_or_else` 传入的函数是懒式调用的，如果有 `Some` 值，就不会调用该函数，如果没有则会调用。

```Rust
fn getDefaultVal() -> i32 {
  println!("Executed!!");
  1
}

let x: Option<i32> = None;
assert_eq!(x.unwrap_or(getDefaultVal()), 1); // 执行了
assert_eq!(x.unwrap_or_else(getDefaultVal), 1); // 执行了

let x: Option<i32> = Some(1);
assert_eq!(x.unwrap_or(getDefaultVal()), 1); // 执行了
assert_eq!(x.unwrap_or_else(getDefaultVal), 1); // 没执行
```

#### unwrap_or_default

这个方法也是我最心水的，返回一个 Some 类型值，或者该类型的默认值。比如 `int` 类型就是 0，`char` 类型是 空串 `''`等。

```Rust
let x: Option<u32> = None;
let y: Option<u32> = Some(12);

assert_eq!(x.unwrap_or_default(), 0);
assert_eq!(y.unwrap_or_default(), 12);
```
