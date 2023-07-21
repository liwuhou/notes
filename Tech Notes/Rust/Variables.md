跟传统的只支持声明可变变量或者只支持声明不可变变量的编程语言有所不同，在 Rust 中，你既可以声明可变变量，也可以声明不可变变量。选择声明不可变变量会为编程提供安全性，而选择声明可变变量则是为编程提供了灵活性。

### 变量命名

在命名方面，和其它语言没有区别，不过当给变量命名时，需要遵循 [Rust 命名规范](https://course.rs/practice/naming.html)。

> Rust 语言有一些**关键字**（_keywords_），和其他语言一样，这些关键字都是被保留给 Rust 语言使用的，因此，它们不能被用作变量或函数的名称。在 [附录 A](https://course.rs/appendix/keywords.html) 中可找到关键字列表。

### 变量「绑定」

对于变量的赋值，传统语言会先为变量开辟一个内存空间，然后将值存放在这个空间中，例如 js 的 `var a = 10;`。而在 Rust 中，与其说是赋值，绑定的说法要更加的恰当。

为何不用赋值，而是绑定的呢？这里就涉及到了 Rust 最核心的原则 —— [[Ownership|所有权（Ownership）]]，任何的内存对象都是有主人的，而且一般的情况下，它完全地属于它的主人，绑定这个操作就是将这个对象绑定给一个变量，让这个变量成为这个内存对象的主人（之前的主人就失去了这个内存对象的所有权）。

### 变量的可变性

Rust 的变量默认是不可变的，如果尝试着改变一个不可变的变量的值，那么使用 `cargo run` 等带检查的指令运行项目将会无情地报错。

```rust
fn main() {
  let x = 5;
  println!("The value of x is {}", x);
  x = 6; // change the value
  println!("The value of x is {}", x);
}
```

运行一遍

```bash
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
error[E0384]: cannot assign twice to immutable variable `x`
 --> src/main.rs:4:5
  |
2 |     let x = 5;
  |         -
  |         |
  |         first assignment to `x`
  |         help: consider making this binding mutable: `mut x`
3 |     println!("The value of x is: {}", x);
4 |     x = 6;
  |     ^^^^^ cannot assign twice to immutable variable

error: aborting due to previous error
```

具体错误的原因为 `cannot assign twice to immutable variable x`，即无法对不可变的变量进行重复的赋值。

如果想要对一个变量进行重复赋值，那就要在一开始的时候就声明为可变变量。声明一个可变变量的操作也非常简单，在`let`命令的后面跟一个`mut`的关键字，`mut` 只能修改变量的值，而不能修改类型。

```rust
fn main() {
  let mut x = 5;
  x = 6; // Not any errors
}
```

选择变量不可变或是可变的特性，更多的还是取决于使用场景。如果在同一个内存位置更新实例可能比复制一遍并返回新分配的实例要快的话，就可以使用可变变量。通常在使用较小数据结构时，通过创建新的实例这种函数式的风格会让代码更容易理解，所以也值得使用较小的性能开销来换取代码的清晰度。

### 变量的遮蔽（shadowing）

Rust 允许在某个变量已经声明的情况下，再次对这个变量名进行声明，官方管这种情况叫 **shadowing**，rust 中文社区翻译为 **变量的遮蔽**。

```rust
fn main() {
  let x = 5;
  // 在main函数的作用域内对之前的x进行遮蔽
  let x = x + 1;
  {
    // 在当前的花括号作用域内，对之前的x进行遮蔽
    let x = x * 2;
    println!("The value of x in the inner scope is: {}", x);
  }
  println!("The value of x is: {}", x); }
```

运行的结果为

```bash
$ cargo run
   Compiling variables v0.1.0 (file:///projects/variables)
   ...
The value of x in the inner scope is: 12
The value of x is: 6
```

这和 `mut` 变量的使用是不同的，第二个 `let` 生成了完全不同的新变量，他只是拥有前一个变量相同的名字而已，这种其实内存对象进行了一次再分配，而 `mut` 变量，修改的都是同一个内存地址中的值，不会发生内存对象的重新分配。

shadowing 的好处在于，如果你在某个作用域内无需再使用之前的变量（在被遮蔽后，无法再访问到之前的同名变量），就可以重复的使用变量名字，而不用绞尽脑汁去想更多的名字，毕竟变量的命名实在是太费脑细胞了 😂。

### 变量与常量

在 Rust 中，除了不可变的变量，还有**常量（constant）**。与不可变变量一样，常量也是绑定到一个常量名且不允许更改的值，但是常量与变量之间存在一些差异：

- 常量使用 `const` 关键字而不是 `let` 关键字来声明，并且值的类型必须标注
- 常量不允许使用 `mut`。常量不仅仅默认不可变，而且自始至终不可变，因为常量在编译完成后，已经确定它的值。

```rust
const MAX_POINTS: u32 = 1000_1000;
```

常量可以在任意作用域内声明，包括全局作用域，在声明的作用域内，常量在程序运行的整个过程中都有效。对于需要在多处代码共享一个不可变的值时非常有用，例如游戏中允许玩家赚取的最大点数或光速。

> 在实际使用中，最好将程序中用到的硬编码值都声明为常量，对于代码后续的维护有莫大的帮助。如果将来需要更改硬编码的值，你也只需要在代码中更改一处即可。

### 变量的解构

`let` 表达式不仅仅可以用于变量的绑定，还能进行复杂变量的解构：从一个相对复杂的变量中，匹配出该变量的一部分内容：

```rust
fn main() {
  let (a, mut b): (bool, bool) = (true, false);
  println!("a = {:?}, b = {:?}", a, b);

  b = true
  assert_eq!(a, b);
}
```

### 解构式赋值

在 [Rust 1.59](https://course.rs/appendix/rust-versions/1.59.html) 版本后，我们可以在赋值语句的左式中使用元组、切片和结构体模式了。

```rust
struct Struct {
    e: i32
}

fn main() {
    let (a, b, c, d, e);

    (a, b) = (1, 2);
    [c, .., d, _] = [1, 2, 3, 4, 5];
    Struct { e, .. } = Struct { e: 5 };

    assert_eq!([1, 2, 1, 4, 5], [a, b, c, d, e]);
}
```

这种使用方式跟之前的 `let` 保持了一致性，但是 `let` 会重新绑定，而这里仅仅是对之前绑定的变量进行再赋值。

需要注意的是，使用 `+=` 的赋值语句还不支持解构式赋值。

> 这里用到了模式匹配的一些语法，如果大家看不懂没关系，可以在学完模式匹配章节后，再回头来看。
