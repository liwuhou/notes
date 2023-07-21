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
