Match 语法绝对在 Rust 中是很常用的语法，一些流程控制及变量赋值都可以使用它来完成，且比一般的 `if...else` 优雅。

```Rust
match target {
  模式1 => 表达式,
  模式2 => {
    语句1;
    语句2;
    表达式 // 没有逗号
  },
  模式3 | 模式4 => 表达式, // 多个可能匹配的模式
  _ => 表达式3, // 相当于其它编程语言中 swtich 的 default
}
```

### 何为模式

模式是 Rust 中的特殊语法之一，除了与 `match` 表达式联用外，还有其它一些语法也运用了模式的匹配能力。
像 `let` 语句。

```Rust
let (x, y) = (1, 2);
```

这其中的 `(x, y)` 就是模式匹配，连自信又普通的 `let x = 1;` 中的 `x` 也是模式。
函数参数就自不必说了 `fn foo(x: i32) -> i32 {}`。

### match 匹配

话不多说，先看个例子：

```Rust
enum Coin {
  Penny,
  Nickel,
  Dime,
  Quarter,
}

fn value_in_centes(coin: Coin) -> u8 {
  match coin {
    Coin::Penny => {
      println!("Lucky penny!");
      1
    },
    Coin::Nickel => 5,
    Coin::Dime => 10,
    Coin::Quarter => 25,
  }
}

let m: u8 = value_in_centes(Coin::Penny); // 1
```

上述代码比较简单，通过 match 语法，可以很轻松地去 `coin` 变量进行模式匹配，而由于 match 又是一个表达式，所以可以直接在函数中返回，实在是优雅。再说回 match 语句，match 中可以有多个匹配值的分支，每个分支都由 **模式代码与处理匹配上该模式的处理代码**组成。拿上述代码中的第一个匹配分支举例来说，`Coin::Penny` 即是模式代码，而 `=>` 运算符后面的代码即是处理代码，最后还有个表达式 `1`，表示返回了 `1`。
这里要注意的是，因为 match 的每一个分支都必须要是一个表达式，且所有分支的表达式最终返回值的类型必须是要相同的。
当 match 表达式执行的时候，目标值 `coin` 会按从上到小的顺序依次地与各个 match 分支进行匹配，如若匹配上了，就会执行相应分支的处理代码然后结束。如果模式不匹配，就会继续与下一个模式进行匹配。Rust 的编译器很智能，会穷举目标值所有可能匹配的模式，如果开发者没有处理所有可能的情况，就会被编译器报错伺候。

```Rust
// 还是上面的 Coin
match coin {
  Coin::Penny => 1,
  Coin::Nickel => 5,
  Coin::Dime => 10,
  // 后面的 Quarter 不写了，就会被 rustc 报错
}
```

开发者也可以选择对所有未成功匹配的模式做处理，使用 `_` 来表示未被匹配的模式，类型于其它编程语言中 `switch` 的 `default`

```Rust
match coin {
  Coin::Penny => 1,
  Coin::Nickel => 5,
  Coin::Dime => 10,
  _ => 25,
}
```

同时，match 语法也可以处理类似逻辑运算符 `或` 的逻辑，只要用 `|` 来隔开模式就行

```Rust
match coin {
  Coin::Penny => 1,
  Coin::Nickel | Coin::Dime => 10,
  _ => 25,
}
```

### 模式绑定

这个模式绑定是模式匹配的另一个重要的功能，它能够赋予开发者从模式中取出绑定的值。

```Rust
enum Color {
  Red(u8),
  Green(u8),
  Blue(u8),
}

let color = Color::Red(255);
let this_color = match color {
  Color::Red(hexValue) => {
    println!("{}", hexValue);
    hexValue
  },
  Color::Green(hexValue) => hexValue,
  COlor::Blue(hexValue) => hexValue,
}

println!("{}", this_color); // 255
```

上述代码的含义在于，匹配 `Color::Red(hexValue)` 模式时，我们把它内部存储的值绑定到了 `hexValue` 变量身上，因此在模式的执行代码中，可以获取和操作这个值。

### if let 匹配

`if let` 语句跟 `match` 语句最不一样的就是 `if let` 语句可以只对值可能存在的某一个模式进行匹配，而不用去管其它的可能存在的值。`match` 则必须面面俱到才行，哪怕你用不上都要加上一个 `- => ()` 来避免编译器报错。这个也是因为 Rust 中 匹配模式分为可驳模式和不可驳模式的特性。像 `match` 就是不可驳模式，所以匹配必须面面俱到，而后面介绍的 `if let` 和 `while let` 就属于不可驳模式。

```Rust
if let 模式1 = target {
  语句 or 表达式
} else if let 模式2 = target {
  语句 or 表达式
} else {
  语句 or 表达式
}
```

当你想用 `match` 来处理某一个模式，而忽略其它模式的场景时，如果只用 `match` 写是这样的

```Rust
let v = Some(3u8);
match v {
  Some(3) => println!("Three"),
  _ => (), // 其它的情况不处理
}
```

就会显得比较啰嗦，这个时候，使用 `if let` 就会比较方便。

```Rust
let v = Some(3u8);
if let Some(3u8) = v {
  println!("Three");
}
if let Some(1u8) = v {
  println!("不会执行");
}
```

这样就完事了，当你想只匹配一个条件且其它条件忽略的时候，就可以用 `if let`，否则都用 `match`。
同时，`if let` 也可以接 `else` 、 `else if let`。

```Rust
enum Foo {
  Bar,
  Baz,
  Baba,
}

let v = Foo::Bar;
if let Foo::Baz = v {
  println!("Baz");
} else if let Foo::Baba = v {
  println!("Baba");
} else {
  println!("Bar");
}

// 其实就等价于 match 中
match v {
  Foo::Baz => {
    println!("Baz");
  },
  Foo::Baba => {
    println!("Baba");
  },
  _ => println!("Bar");
}
```

### while let 条件循环

与 `if let` 类似，`while let` 只要模式匹配上，就可以继续进行 `while` 循环。

```Rust
let mut stack = Vec::new();

stack.push(1);
stack.push(2);
stack.push(3);

while let Some(top) = stack.pop() {
  println!("{}", top);
}

// 等价
loop {
  if let Some(top) = stack.pop() {
    println!("{}", top);
  } else {
    break;
  }
}
```

上述代码，会在 `stack.pop` 方法返回的元素非空的时候，会执行 `while` 语句中的方法。这就是 `while let` 的特性，我们也可以通过 [[Control Flow#loop 循环|loop]] + [[#if let 匹配|if let]]或者[[#match匹配|match]] 来实现这个功能，就是会更加啰嗦就是了。

### matches! 宏

`matched!` 是 Rust 提供的一个宏方法，用来比较一个表达式和模式是否匹配。

举个例子

```Rust
enum MyEnum {
  Foo,
  Bar,
}

let arr = [MyEnum::Bar, MyEnum::Foo, MyEnum::Foo];

// 通过arr的filter方法过滤出只有 MyEnum::Foo 的情况
for item in arr.iter().filter(|x| x == MyEnum::Foo) {
  // 能行吗？
}
```

`filter` 是迭代器的过滤方法，通过表达式 `x == MyEnum::Foo` 返回 `true` 或是 `false` 来决定要不要过滤出元素。
但上述代码中，如果直接这么写 `arr.iter().filter(|x| x == MyEnum::Foo)` 是会报错的，因为直接地把一个变量跟枚举成员做比较是不行的。
然后我们可以想到，能使用 `match` 语法，就是好像有点啰嗦了点……

```Rust
for item in arr.iter().filter(|x| match x {
  MyEnum::Foo => true,
  MyEnum::Bar => false,
}) {
  // 可以是可以，就是看着……
}
```

这个时候就要引出我大 Rust 提供的 `matches!` 方法了，这个方法接收两个参数。一个值与一个模式，如果匹配会返回 `true`，不匹配返回 `false`。所以真的是完美匹配 `filter` 方法。

```Rust
for item in arr.iter().filter(|x| matches!(x, MyEnum::Foo)) {
  // 优雅
}
```

### 变量的遮蔽

在使用 `if let` 语法或者是 `match` 的时候，都会产生一个代码块，而在此代码块中绑定的变量都相当于新变量，如果使用了代码块之外的同名变量，那么该变量在代码块中会被[[Variables#变量的遮蔽（shadowing）|遮蔽]]

```Rust
let age = Some(30);
println!("{:?}", age); // Some(30)
if let Some(age) = age {
  println!("{}", age); // 30
}
println!("{:?}", age); // Some(30)
match age {
  Some(age) => println!(age), // 30
  _ => (),
}
println!("{:?}", age); // Some(30)
```

在上述代码中，`if let` 语句中，`=` 右边的 `Some(i32)` 类型的 age 被左边的 `i32` 类型的新 `age` 遮蔽了，该遮蔽会一直持续到整个 `if let` 语句块的结束。因此第三个 `println！` 输出的 `age` 依然是 `Some(i32)` 类型。

在这两种语法中， 变量的遮蔽很难看出来，所以小心起见，这里可以使用不同的变量名，避免发生变量遮蔽，以至于难以理解。

```Rust
let age = Some(30);
println!("{:?}", age); // Some(30)
if let Some(x) = age {
  println!("{}", x); // 30
}
println!("{:?}", age); // Some(30)
match age {
  Some(x) => println!("{}", x), // 30
  _ => (),
}
println("{:?}", age); // Some(30)
```

### 单分支多模式

在 `match` 中，使用 `|` 语句可以匹配多个模式，它代表着逻辑 `或`。

```Rust
let x = 1;
match x {
  1 | 2 => println!("1 | 2"),
  3 => println!("3"),
  _ => println!("other"),
}
```

### 通过序列匹配值的范围

如果有两个或者几个的值还好，如果是有很多值，或者是一个序列范围的值，用 `..=` 匹配值的范围就很方便

```Rust
let x: u32 = 102;
match x {
  1..=99 => println("一百以内"),
  100..=200 => println("两百以内"),
  _ => println!("很大就是了"),
}
```

需要注意的是，`match` 的序列匹配只允许数字或是字符，因为要确保它们是连续的。而且只能用 `..=` 划分的闭区间，不能使用 `..` 这种半开区间。

```Rust
let x = 'c';

match x {
  'a'..='j' => println!("early ASCII letter"),
  'k'..='z' => println!("late ACSII letter"),
  _ => println!("something else"),
}
```

同时编译器在编译期可以检查该序列是否为空，数字值和字符是 Rust 中仅有的可以用于判断是否为空的类型。

### 匹配守卫

上面的例子中，代码的表现力已经很强了，能让 `x` 在某个连续的区间中，但是如果是负数或者浮点数的比较就无能为力了。或者有更为复杂的判断条件又该怎么办呢。
这里就要介绍匹配守卫了。首先看看，它长这个样子， `模式 if condition => expression`。当模式匹配上的时候，再经过一个额外的 `if` 语句判断是否满足条件，如果满足就执行语句。说白了就是为模式提供了更进一步的判断。而且在 `if` 语句中，还可以使用模式中创建的变量，举个粟子。

```Rust
let num = Some(4);

match num {
  Some(x) if x < 5 => println!("less than 5: {}", x),
  Some(x) => pringln!("{}", x),
  None => (),
}
```

匹配守卫也可以使用外部的变量，或者通过一个返回 `bool` 类型的函数来判断。

```Rust
fn main() {
  let num = Some(4);
  let bo = true
  match num {
    Some(x) if less_than_five(x) => println!("less than 5"),
    Some(_x) if bo => println!("greater than 5"),
    _ => (),
  }
}

fn less_than_five(x: i32) -> bool {
  x < 5
}
```

### 解构嵌套的结构体或枚举

`match` 语法也可以嵌套，而且，`match` 的模式也支持匹配嵌套的项。

```Rust
fn main() {
  enum Color {
    Rgb(i32, i32, i32),
    Hsv(i32, i32, i32),
  }
  enum Message {
    ChangeColor(Color),
  }

  let message = Message::ChangeColor(Color::Rgb(0, 0, 0));

  match message {
    Message::ChangeColor(color) => match color {
      Color::Rgb(r, g, b) => println!("Rgb({}, {}, {})", r, g, b),
      Color::Hsv(h, s, v) => println!("Hsv({}, {}, {})", h, s, v),
    },
    _ => (),
  }

  // 也可以直接匹配嵌套的项
  match message {
    Message::ChangeColor(Color::Rgb(r, g, b)) => println!("Rgb({}, {}, {})", r, g, b),
    Message::ChangeColor(Color::Hsv(h, s, v)) => println!("Hsv({}, {}, {})", h, s, v),
  }
}

```

### `_` 忽略模式中的值

有个有意思的地方值得注意，在模式匹配中，可以使用 `_变量名` 的模式忽略某些未被使用的变量。此时 `_变量名` 仍会将值绑定到变量，但如果直接使用 `_` 当变量名的话，则值完全不会绑定。我们看下下面的例子就知道了。

```Rust
let s = Some(String::from("Hello")); // Some 中的值是一个 String

if let Some(_s) = s {
  todo!();
}

println!("{:?}", s); // 这里会报错，显示 s 已被移动了，再使用就报错
```

而如果是 `_` 的话，就完全不会有问题

```Rust
let s = Some(String::from("Hello"));

if let Some(_) = s {
  todo!();
}

println!("{:?", s); // 完全正常
```

### 用 `..` 忽略剩余值

```Rust
struct Point {
  x: i32,
  y: i32,
  z: i32,
}

let origin = Point { x: 0, y: 0, z: 0 };

// 使用 `..` 忽略剩余值
match origin {
  Point {x, .. } => pringln!("{}", x),
}

// 也可以用来忽略中间的某些值

if let Point {x, .., z} = origin {
  println!("{}, {}", x, z);
}

// 与 `_` 相同，但 `..` 能忽略一序列的值
if let Point {x, _, z} = origin {
  println!("{}, {}", x, z);
}
```

但 `..` 必须是无歧义的，如果期望匹配和忽略的值不明确，Rust 就会报错。

```Rust
fn main() {
  let numbers = (2, 4, 8, 16, 32);

  match numbers {
    (.., second, ..) => { // 报错
      println!("Some numbers: {}", second)
    },
  }
}
```
