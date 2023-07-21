对于一些有限结果的集合，可以使用枚举类型来表示，例如表示红绿蓝三原色：

```Rust
enum Color {
  Red,
  Green,
  Blue,
}
```

枚举分为枚举类型和枚举值组成，枚举类型是一个类型，它包含了所有枚举成员。而枚举成员就是枚举值，是该枚举类型中的某个成员的实例。

### 枚举值

```Rust
let red: Color = Color::Red; // 红色
let mut leaves: Color = Color::Green; // 绿色

leaves = red;
```

既然枚举值是枚举类型的成员实例，那么就可以通过 `::` 操作符来访问枚举类型下的各枚举值。然后不同的成员枚举值都可以给彼此的枚举类型赋值。
除此之外，Rust 中的 Enum 还可以实现带值的枚举类型。比如用来表示色值的 RGB 编码。

```Rust
enum Color {
  Red(u8),
  Green(u8),
  Blue(u8),
}

fn main() {
  let color1 = Color::Red(255);
  let color2 = Color::Blue(125);
}
```

不仅如此，Rust 还可以将任何类型的数据都放入枚举成员当中。例如：

```Rust
enum Message {
  Quit,
  Move { x: i32, y: i32 },
  Write(String),
  ChangeColor(i32, i32, i32),
}

fn main() {
  let m1 = Message::Quit;
  let m2 = Message::Move{ x: 1, y: 2 };
  let m3 = Message::ChangeColor(255, 255, 255);
}
```

当然我们也可以使用[[Struct|结构体]]来表示上述的消息枚举，但就会显得啰嗦，而且每一处结构体其实都有了自己的类型，无法在需要同一类型的地方进行使用。

```Rust
struct QuitMessage; // 单元结构体
struct MoveMessage {
  x: i32,
  y: i32,
}
struct WriteMessage(String); // 元组结构体
struct ChangeColorMessage(i32, i32, i32); // 元组结构体

fn send_msg(msg: /**? */) {

}
```

所以使用的是枚举的话就方便很多，函数只需定义其枚举类型就都可以传入函数中处理了。

```Rust
fn send_msg(msg: Message) {
  todo!();
}
```

那么问题来了，既然一个枚举的成员可能由多个不同类型的枚举值组成，那么在处理这种多种不同类型的枚举成员的时候，又该如何处置它们呢。
这就要引出 Rust 中的，[[Match|match]] 语法了。

### Option 枚举处理空值

在 Rust 的标准库 `prelude` 中，包含了一个常用的枚举 —— [[Option]]。`Option` 包含了两个成员，一个是代表含有值的 `Some(T)`， 一个则是表示没有值的 `None`。其定义如下：

```Rust
enum Option<T> {
  Some(T),
  None,
}
```

其中 `T` 是泛型参数，`Some(T)` 用于表示枚举成员的数据类型是 `T`。由于 `Option<T>` 枚举被包含在 `prelude` 库中，即 Rust 最常用的库，所以我们都无需引用，甚至无需使用 `Option::` 前缀就能使用。

```Rust
let some_num = Some(5);
let some_str: Option<String> = None<String>;
```

在使用的时候，如果有值，可以直接使用 `Some` 传递，如果还没有值，就要专门约束一下变量的类型，告诉 Rust `Option<T>` 是什么类型，因为单单通过 `None`，编译器并不能推断出变量的类型。

`Option` 的存在，使得我们和 Rust 可以知道有一个变量并不一定拥有一个有效的值，某种意义上它与空值是具有相同的意义的。

这个时候就要使用 [[Match|match]] 模式匹配语法来帮助我们判断值的情况了。

```Rust
let abstract_num: Option<i32> = None;
let some_num = Some(5);

let plus_five(x: Option<i32>) -> i32 {
  match x {
    None => 0,
    Some(i) => i + 5,
  }
}

plus_five(abstract_num); // 0
plus_five(some_num); // 10

```

[更多 `Option` 的使用见此处](https://doc.rust-lang.org/std/option/enum.Option.html)
