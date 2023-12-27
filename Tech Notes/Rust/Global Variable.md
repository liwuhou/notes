往往在编程中，我们会被告诫谨慎使用全局变量。因为全局都可以访问的一个变量，任何涉及该变量的函数输出都会受到外部状态的影响，导致我们难以测试，也难以发现和修复其可能产生的 bug。但不可否认，某些场景下，全局变量确实可以简化我们的代码，特别是在一些涉及到状态共享的场景，比如某个全局数据的共享、全局 ID 的生成，而在 Rust 中，全局变量使用时会有一定的约束。

全局变量从初始化的时机与可变性来说，又会有不同的一些类型，

### 编译期初始化

也是我们大多数使用的情况，在编译期进行初始化，后续在运行期进行访问或者修改，常用的场景如：静态配置、计数器、状态值等。

#### 静态常量

全局常量可以在程序的任一部分使用，但如果是定义在某个模块中的全局变量，使用时需要引入对应的模块。
一个静态的常量声明也很简单易懂，适合用来作为静态配置：

```Rust
use std::fs::File;
use std::io::{BufRead, BufReader};

const CARGO_PATH: &str = "Cargo.toml";

fn main() -> Result<(), Box<dyn std::error::Error>> {
  BufReader::new(File::open(CARGO_PATH)?)
    .lines()
    .filter_map(Result::ok)
    .for_each(|ref line| println!("{}", line));

  Ok(())
}
```

常量与变量的区别在于：

- 声明关键字使用 `const` 而不是 `let`
- 定义常量时，必须要指明类型，不能省略
- 定义常量时变量的命名规则大写+下划线形式
- 常量可以在任意作用域中进行定义，其生命周期会贯穿整个程序的生命周期，在编译时编译器会尽可能地将其内联到代码中，**所以在不同地方对同一常量的引用并不能保证访问到相同的内存地址**
- 常量的赋值只能是常量表达式/数学表达式，也就是必须是能在编译期就确定的值，如果需要在运行时才能得出结果的值，则编译器会咔咔报错
- 变量出现重复定义会发生[[Variables#变量的遮蔽（Shadowing）|变量的遮蔽]]

#### 静态变量

静态变量与静态常量对应，是一个作用域在全局的**可变变量**。常用于全局数据统计，例如实现一个变量统计程序当前的请求总数：

```Rust
static mut REQUEST_RECV: usize = 0;

fn main() {
  unsafe {
    REQUEST_RECV += 1;
    assert_eq!(REQUEST_RECV, 1); // 这里即使是访问也要被 `unsafe` 包裹
  }
}
```

Rust 要求必须使用 `unsafe` 语句块才能访问和修改 `static` 变量，这是因为在某些场景（比如多线程同时修改访问）会不安全。但如果你能保证只在同一线程下且不太在乎数据准确性的话，可以尽情地去用全局静态变量。

与全局静态常量的相同点：
由于是编译时初始化，所以跟全局静态常量一样，全局静态变量也需要你在编译期时就确定他的值。

与全局静态常量的不同点：

- 静态常量不会被内联，在整个程序中，静态常量只有一个实例，所有的引用指向的都是同一个内存地址
- 存储在静态变量中的值必须要实现 `Sync` trait
- 还有就是访问和修改都要使用 `unsafe` 包裹

#### 原子类型

静态变量虽然可以实现全局的状态管理，但它并不是线程安全的，而原子类型则是很好的跨线程安全的解决方式。

```Rust
use std::sync::atomic::{AtomicUsize, Ordering};

static REQUEST_RECV: AtomicUsize = AtomicUsize::new(0); // 换成具有原子安全性的usize类型

fn main() {
  for _ in 0..100 {
    REQUEST_RECV.fetch_add(1, Ordering::Relaxed);
  }

  println!("{:?}", REQUEST_RECV); // 100
}
```

### 运行期初始化

上述的全局变量都是属于需要在编译期就初始化完成的，如果碰到一些需要在运行时才能确定的值就无力了。虽然 Rust 语言没有提供这种能力，但社区里还是有一些解决的手段的，其中就有 [lazy_static](https://crates.io/crates/lazy_static)，或是 [once_cell](https://crates.io/crates/once_cell)。如果你 Rust 版本是 1.70.0 及以上，也可以尝试使用标准库里的 `cell::OnceCell` 与 `sync::OnceLock`。前者用于单线程，后者用于多线程，都是用于存储堆上的信息，并且只能被赋值一次。

使用 `OnceLock` 特性。

```Rust
use std::sync::OnceLock;

static NAMES: OnceLock<String> = OnceLock::new();

fn main() {
  NAMES.get_or_init(|| "hello world".to_string()); // 在这里的时候才初始化值

  println!("{}", NAMES.get().unwrap());
}
```

`OnceLock` 其实就是提供一个只允许我们修改一次值的容器，但凡我们已经初始化过（init）值了，就不会再被更改了，达到了在运行时初始化变量的目的。
也可以声明一个 `OnceLock<Mutex<T>>` 来实现懒初始化的全局变量。

使用 `once_cell` 外部库也可以，[事实上标准库的 `OnceLock` 就是从 `once_cell` 中引进的](https://github.com/rust-lang/rust/pull/105587)。

使用 `lazy_static`

```Rust
use lazy_static::lazy_static;
use std::sync::Mutex;

lazy_static! {
  // lazy_static 是个宏方法
  static ref NAMES: Mutex<String> = Mutex::new(String::from("Hello ")); // 懒初始化
}

fn main() {
  let mut v = NAMES.lock().unwrap();
  v.push_str("world!");
  println!("{}", v); // Hello world!
}
```

`lazy_static` 包裹的代码，会在程序运行到 main 函数第一行代码的时候才进行初始化，真的把 lazy 做到了极致。

更多 Rust 懒初始化全局变量的方法见 [How do I create a global, mutable singleton?](https://stackoverflow.com/questions/27791532/how-do-i-create-a-global-mutable-singleton)

也可以借用 `Box::leak` 主动将局部变量“泄漏”给全局变量使用。

```Rust
struct Config {
  config: String
}

static mut CONFIG: Option<&mut Config> = None;

fn main() {
  // 在 main 中才初始化
  let c = Box::new(Config { config: "config".to_string() });

  unsafe {
    // 将`c`从内存中泄漏，变成`'static`生命周期
    CONFIG = Some(Box::leak(c));
  }
}

```

### 总结

- 编译期初始化全局变量：`const` 创建常量，`static` 创建变量， `Atomic` 创建原子类型
- 运行期初始化全局变量，`lazy_static` 用于懒初始化，`Box::leak` 用于主动泄漏一个变量为全局变量，也可以使用 `OnceCell` 与 `OnceLock` 初始化全局变量
