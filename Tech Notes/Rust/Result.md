在[[Panic]]那节说过，一些不可恢复的，对程序而言毁灭性的错误就要 `panic!` 出去，但这并不意味着碰见啥错都要 `panic`。一些错误往往可以使用一些重试策略或者友好提示来告知使用的用户，抑或做一些兜底处理。事实上，Rust 中的标准库、第三方包都有很多对 `panic` 事件的处理，最常见的就是返回一个 `Result` 枚举。

### Result 探秘

`Result` 其实本质上就是一个[[Enum|枚举]]类型，定义在标准库里，并且由于非常常用，所以也被包含在了 `prelude` 包中，因此开发的时候，无须我们再 `use std::result::Result` 地引用。

```Rust
enum Result<T, E> {
  Ok(T),
  Err(E),
}
```

`Result<T, E>` 有两个泛型，`T` 用来代表某些操作成功时存入的正确类型，`E` 则是代表错误时存入的错误值。

有了 `Result`，我们可以把某类可能导致错误的操作给封装起来，对外只暴露一个 `Result<T, E>` 枚举类型，然后让外部使用 [[Match]] 等去处理操作成功或是错误的场景。

```Rust
use std::fs::File;
use std::io::{Error, ErrorKind};
fn main() {
  match open_file("./苍老师秘录.avi") {
    Ok(_f) => println!("批判性观看"),
    Err(error) => {
      if let ErrorKind::NotFound = error.kind() {
        println!("404： 在 40T 的学习资料中都找不到该文件！");
      } else {
        println!("发生了其它的错误，请检查是否有学习资料访问权限")
      }
    }
  }
}

fn open_file(file_path: &str) -> Result<File, Error> {
  // File::open 方法本身返回的就是一个 Result 类型
  File::open(file_path)
}
```

上述例子中，外部使用 `match` 和 `if let` 来匹配值与错误，对两种情况都做了对应的处理。在 Rust 中，处理异常就是这么的轻松写意，此时隔壁的同事们还在要么写 `if(err) return nil`，要么就各种 `try...catch`……

### unwrap 和 expect

接下来就要介绍的两位，是一对非常头铁的兄弟。众所周知，在 Rust 中的 `match` 具有穷尽匹配的特性，以至于你使用它去处理 `Result` 的时候，不能忽视 `Err` 分支。但有些时候，我们写原型或者简单的代码示例时，想忽略这些异常的 case，那就可以使用接下来要出场的两兄弟了。

它们的作用基本类似，就是只管 `Result` 的 `Ok` 情况，或者 `Option<T>` 的 `Some(T)` 情况，其它的情况就通通 [[Panic|panic]]。

```Rust

let f = File::open("xxx.txt").unwrap();

let f = File::open("yyy.txt").expect("It should be existed");

let f: Some<i32> = None;

println!("{}", f.unwrap());
println!("{}", f.expect("It can't be a None"));
```

就是这么地莽，如果以上的几个文件不存在就直接 `panic`，`expect` 遇上错误也是会 `panic`，只是会带上我们传入的错误提示信息，相当于把错误信息换成我们传入的信息。

### 传播错误

当我们封装一个操作的时候，如果操作出错了，可能不会直接就在封装的方法里进行处理，而是把错误传递出去，让外层去做处理，也就是对出现的错误进行传播。举个 🌰，我们封装一个读取某个文件内容的函数，这个函数中有两个可能出错的操作，一个就是文件不存在的文件打开错误，一个就是读写数据出错。我们返回一个两个操作都成功时的文件内容数据和任一可能出错时的错误的 `Result` 出去，让外层去处理。

```Rust
use std::fs::File;
use std::io::{self, Read};

fn read_text_from_file(filename: &str) -> Result<String, io::Error> {
  let f = File::open(filename);

  let mut f = match f {
    Ok(file) => file,
    Err(e) => return Err(e), // 将错误提前返回，向上传播
  };
  let mut s = String::new();
  match f.read_to_string(&mut s) {
    Ok(_) => Ok(s),
    Err(e) => Err(e), // 同样这里的 match 表达式会返回结果或者向上传播 Error
  }
}
```

通过上面的函数，可知结果会返回一个 `Result<String, io::Error>` 的类型，外层调用它的时候就会进行 `Result` 的解套，同时也要妥善处理好 `Result::Err` 的情况。

```Rust
fn main() {
  // 拿得到就 println 出来，拿不到就算了，也算是妥善处理了
  if let Ok(s) = read_text_from_file("xxx.txt") => {
    println("s")
  }
}
```

`read_text_from_file` 这个方法是实现了错误的传播，但这个代码量实在是有点多了，假设要处理的 `Result` 类型越多，我们内部的代码里要写的 `match` 也就越多，但其实我们操作遇到错误都是直接传递出去的，不会在封闭的函数内部处理的，有什么办法可以简化这一步，让我们写的代码可以不用这么啰嗦地去匹配和传递这些错误呢。
答案是有的，让编译器帮我做这些就好了，Rust 中实现了个语法糖，可以颇具语义又便捷地实现这一套东西。

### ?

废话不多说，直接 show `?` 改造过的代码：

```Rust
use std::fs::File;
use std::io;

fn read_text_from_file(filename: &str) -> Result<String, io::Error> {
  let mut f = File::open(filename)?;
  let mut s = String::new();
  f.read_to_string(&mut s)?;
  Ok(s)
}
```

是的，就这样，没有更多了。代码量跟上面我们用 `match` 去匹配值与错误的量相比，直接少了一半不止。这里稍微解释一下，这个 `?` 其实是一个宏，它的作用与下面的 `match` 是等价的。

```Rust
fn read_file() -> Result<(), io::Error> {
  let mut f = match File::open("xxx.txt") {
    Ok(file) => file, // 成功就把值赋给可变变量 f
    Err(err) => return Err(err), // 出错就把值抛出外层
  }

  // 而同样的功能，使用 ? 之后
  let mut f = File::open("xxx.txt")?;
}
```

`?` 除了自动实现向外抛出错误外，还实现了自动对错误类型进行提升。让我们可以在接收的地方，定义一个大而全的错误类型，来覆盖封装的操作里返回的所有可能的错误类型。

```Rust
fn open_file() -> Result<File, Box<dyn std::error::Error>> {
  let mut f = File::open("xxx.txt")?;
  Ok(f)
}
```

这里可以看到的是，原本在 `File::open` 操作中，出现错误时返回的错误类型应该是 `std::io::Error`的，但我们函数定义的错误类型却是 `std::error::Error`。这是因为前者实现了后者，因此 `std::io::Error` 可以转换为 `std::error::Error`。
这里的原理就是 `std::io::Error` 中实现了标准库里定义的 `From` 特征，该特征有一个 `from` 方法，可以把一个类型转换成另一个类型，而 `?` 会去自动调用这个方法。再结合 [[Deepin Trait#特征对象的定义|特征对象]] 的概念，类型就可以汇聚到 `Box<dyn std::error::Error>` 中进行约束了。
我们实现自定义的错误的时候，也可以去实现 `From` 特征的 `from` 方法，后续 `?` 就会自动帮我们把错误转换为对应的错误类型。

为啥要有这些子类型转成大类型的操作，还不是为了方便我们开发者，可以不用去管这些具象的子错误，能用一个大而全的错误来概括出现的错误，与此同时带来的好处就是写代码可以更无脑，因此 `?` 也支持链式调用，只要返回的错误类型都受最终的错误类型约束就可以了。

```Rust
fn read_text_from_file() -> Result<String, std::io::Error> {
  let mut s = String::new();

  File::open("xxx.txt")?.read_to_string(&mut s)?;
  Ok(s)
}
```

连这都想到了，Rust 真的，我哭死。

### ? 用于 Option 的返回

`?` 不仅可以用来处理 `Result` 类型，也可以用来处理 `Option` 类型。与处理 `Result` 时相似，在 `?` 中，如果遇到了 `None`，那么就会直接返回 `None`。

举个 🌰，读取某段字符串中第一行的最后一个字符，这时就是一个 `Option` 类型的值，我们不使用 `?` 的话，就要这么啰嗦：

```Rust
fn last_char_of_first_line(text: &str) -> Option<char> {
  let mut line = text.lines();

  match line.next() { // 因为这里的 next 可能返回的是 Option::None
    Some(s) => s.chars().last(), // 返回这一行最后一个字符
    None => None, // 没有就返回 null
  }
}
```

有点啰嗦是吧，关键是咱们调用这个 `last_char_of_first_line` 的地方，还要包一层 `match` 或者 `if let` 来处理函数返回的 `Option` 值。

```Rust
fn main() {
  let text = "Hello\nworld";
  match last_char_of_first_line(text) {
    Some(c) => println!("{}", c),
    None => println!("Nothing"),
  }
}
```

这里观察 `last_char_of_first_line` 方法，能够发现我们在 `match` 中，碰到 `Option::None` 的时候，就跟上面在 `match` 中碰到 `Result::Err` 一样，是直接返回它自身的，也没有做任何的处理，然后在外层中处理它。所以这里完全可以使用 `?` 来搞定这个场景。

改写一下:

```Rust
let last_char_of_first_line(text: &str) -> Option<char> {
  text.lines().next()?.chars().last()
}

fn main() {
  let text = "Hello\nworld";
  if let Some(c) = last_char_of_first_line(text) {
    // 使用 if let，只处理 Option::Some 的情况，Option::None 直视
    println!("{}", c);
  }
}
```

由上面的例子中也可以很容易的得出规律：

1. `?` 适用的场景是某个函数中，函数返回类型是 `Result<_,_>` 或是 `Option<_>` 的场景
2. `?` 只是一个使用 `match` 处理 `Result` 和 `Option` 类型，并且使用 `return` 来提前返回的语法糖
3. `?` 会自动调用 `Error` 类型的 `from` 方法，能自动转换为更大的错误类型
4. 在 `Result` 或 `Option` 的值后跟上 `?`，表示如果值是正面的，也就是 `Result::Ok` 或 `Option::Some` 就忽略 `?`，接着执行
5. `?` 处理 `Result::Ok(T)` 或 `Option::Some(T)`，如果是值是正面的，那该语句返回的是解套了的 `T` 类型
6. 如果值返回的是负面的，也就是 `Result::Err` 或 `Option::None` 的时候，就直接中断，提前返回相应的 `Err` 或 `None`

规律 5 挺重要的，这也是很多使用 `?` 的新手司机容易混淆的点，首先因为有提前 `return` 所以要在函数里使用。并且还要保证受函数的返回类型约束。先看一下下面的例子：

```Rust
fn get_option() -> Option<i32> {
  let x = Some(1);
  let y = x?;
  Some(y) // ? 的值一定要有变量对象来接收，并且如果其是 Some<T> 类型，会直接把 T 赋给 y
  // 下面这样为啥报错了？想想规律 5
  // x?
}
```

基于以上规律，如果要在 `main` 方法中使用 `?`，那就要适当地改造 `main` 方法的返回类型，一般都是使用 `Result<(), Box<dyn std::error::Error>` 来接收，并且在函数的最后，返回一个 `Ok(())`。例如：

```Rust
fn main() -> Result<(), Box<dyn std::error::Error>> {
  let f = std::fs::File::open("xxx.txt")?;
  // 操作 f

  Ok(()) //  最后返回一个 `Ok(())`
}
```

顺便提一嘴，`Box<dyn Trait>` 这种[[Deepin Trait#特征对象的定义|特征对象]]的用法，是一种很常用的，用来抽象一类实现了共同特征的类型的方法。

### try!

在 `?` 语法面世之前，Rust 有个宏方法也是用来实现提前返回错误的，它就是末路英雄 `try!`。直接看他的源码定义可以发现，就是个方法用来处理 `Result` 类型，`Ok` 直接返回值，`Err` 就提前 `return` 出错误：

```Rust
marco_rules! try {
  ($e:expr) => (match $e {
    Ok(Val) => Val,
    Err(err) => return Err(::std::convert::From::from(err)),
  })
}
```

跟 `?` 对比一下：

```Rust
let x = File::open("xxx.txt")?;

let x = try!(File::open("xxx.txt"));
```

劣势也是很明显的，不能使用链式调用，也不能用来处理 `Option` 类型的值，并且，在现在版本的 Rust 中使用 `try!`，IDE 上直接就标红，编译能通过，但也会报一个 `warning` 提示你使用 `?` 来代替 `try!`，只能说在后面的开发中，要尽量避免使用 `try!`，因为它尽早要淘汰了。
