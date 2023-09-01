Rust 中的 Struct 翻译为结构体，是一种灵活的复合型数据结构，在其它的编程语言中往往具有其它的名称，例如 `object`、`dict` 或 `record` 等。相比[[Tuple|元组]], Struct 会更富有语义化，因为可以给每个字段一个有含义的名称。

```Rust
struct Person {
  active: bool,
  username: String,
  email: String,
  sign_in_count: u64,
  hobby: &str[],
}
```

### 结体体的定义

一个结构体由几个部分组成：

- 通过关键字 `struct` 定义
- 一个清晰明确的结构体 `名称`
- 几个有名字的结构体 `字段`

创建结构体实例，要用相应的结构体类型去约束变量，然后结构体实例中的字段都要按结构体定义去定义和赋值

```Rust
let person = Person {
  active: true,
  username: String::from("william"),
  email: String::from("hugewilliam@foxmail.com"),
  sign_in_count: 1,
  hobby: ["Program"]
}
```

需要注意的点：

1. 初始化实例时，每个字段都需要进行初始化
2. 初始化时的字段顺序不需要按照定义结构体时的顺序

与 ecmascript 类似，在 Rust 中，如果有变量名跟结构体的字段名相同的话，可以简化。

```Rust
struct User {
  active: bool,
  username: String,
}

let active = true;
let username = String::from("william");

// 那么
let user = User {
  // active: active, // 可以简化为
  active,
  usernmae,
};
```

### 结构体的可变操作

在实例化一个结构体的时候，Rust 只允许我们将这个结构体声明可全部字段都可变的形式，不允许我们针对某个字段指定可变。

```Rust
struct Person {
  name: String,
  age: u8,
}

let mut p = Person { // 只能将整个结构体实例可变
  name: String::from("william"),
  age: 18,
};

p.age = 19;
p.name = "abby";

p = Person { // 也允许直接改实例指向
  name: String::from(".."),
  age: 1
}
```

### 结构体更新语法

在实际的使用场景中，有一类使用情况很常见：通过已有的结构体实例来创建新的结构体实例。而 Rust 支持使用 `..` 操作符来展开已有的结构体实例，有点类型 ecmascript 的解构语法。但又有些不同的点需要注意一下。

```Rust
let user2 = User {
  active: user.active,
  username: user.username,
  sign_in_count: user.sign_in_count,
  email: String::from("another@mail.com"), // 仅 email 这个字段不同
};
// 可以使用 .. 操作符简化代码
let user3 = User {
  email: String::from("another@mail.com"),
  ..user2 // 这里还不能使用上分号
};
```

使用结构体更新语法需要注意的几个点：

1. `..` 语法只是表明，凡是我们没有显式声明的字段（例子中的除也`email` 之外的所有字段）才自动从展开的结构体实例中获取
2. `..` 只能在结构体的尾部使用，且不能使用分号结尾
3. `..` 与赋值语法 `=` 相似，同样也会在赋值的时候发生 [[Ownership|所有权]] 的转移，而具有 [[Ownership#Copy 特性|Copy 特性]] 的值同样也不会存在所有权转移的问题。
4. 发生所有权转移的时候，被 `..` 解构的对象中一些具有 `Copy` 特性的字段和未被解构的其它字段都是可以正常使用的
5. 只要在 `..` 的时候发生所有权转移，那么整个地访问被解构的结构体实例会发生所有权错误

```Rust
let user1 = User {
  email: String::from("hugewilliam@foxmail.com"),
  username: String::from("hugewilliam"),
  active: true,
  sign_in_count: 1,
};

let user2 = User {
  email: String::from("another@mail.com"),
  sign_in_count: user1.sign_in_count,
  ..user1
};

println!("{}", user1.active); // 不会报错
println!("{}", user1.email); // 也不会报错
println!("{}", user1); // 整个报错
println!("{}", user1.username); // 报错

```

### 元组结构体（Tuple Struct)

结构体必须要有名称，但结构体的字段可以没有名称，这种形式的结构体看起来很像元组，因此也被称为元组结构体。

```Rust
struct Color(i32, i32, i32);
struct Point3d(i32, i32, i32);

let black = Color(0, 0, 0);
let origin = Point3d(0, 0, 0);

// 解构赋值要这样用
let Point3d(x, y, z) = origin;
```

这样的形式既利用了元组的优势，又极大地丰富了代码的可读形式与含义。实在是一举两得。

### 单元结构体（Unit-like Struct）

与[[Char & Bool & Unit type#单元类型（unit type)|单元类型]]很像，单元结构体没有任何字段和属性，一般用来帮助我们定义某个不需关注其内容的类型，让我们只关心它的行为。

```Rust
struct AlwayEqual;

let subject = AlwayEqual;

impl SomeTrait for AlwayEqual{
  todo!();
}
```

### 结构体数据的所有权

结构体数据也受 Rust 中的[[Ownership|所有权]]法则约束，一个结构体中，如果声明的时候，没有自身拥有所有权的数据，换句话说就是，结构体从其它地方借用了数据，那么就需要开发者显式地为这个借用的数据声明一个[[Lifetime#结构体上的生命周期|生命周期]]。

```Rust
struct User {
  username: &str, // 这里的 &str 不属于 Struct 结构拥有，所以算是借用，编译会报错：这里需要一个生命周期
  active: bool,
}
```

### 打印结构体信息

结构体的结构比较复杂，在 Rust 中默认没有实现它的 `Display`特征，所以直接用 `println!("{}", struct);` 会报错。

而使用 `"{:?}"` 的话，又要我们实现 `Debug` 特征，这是因为 Rust 默认不会为我们实现 `Debug` 特征，如果为了实现它，就有两种方式供我们选择。

一种是手动实现一个结构体的 `Debug` 特征，这种就比较复杂而且繁琐了。
第二种就是使用 `derive` 派生，这种虽然简单，但是也有一些限制，见 [附录 D](https://course.rs/appendix/derive.html)。

使用很简单，往要打印的结构体定义上加这么一些注释就可以了

```Rust
#[derive(Debug)] // 关键代码
struct Person {
  name: string,
  age: number,
}

let p = Person {
  name: "william",
  age: 18
};

println!("{:?}", p);
```

此时就不会报任何的错误，还会正确的输出结构体。

```bash
$ cargo run
{ name: "william", age: 18 }
```
