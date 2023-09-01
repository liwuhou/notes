在 Rust 中，每个值都有作用域，作用域一过，这个值就会被软件回收。而我们引用（借用）某些值的时候，为了避免出现[[References & Borrowing#悬垂引用]]问题，编译器会进行一系列的 “借用检查” 和引用的生命周期检查。

### 借用检查

在 Rust 内部，有一个借用检查器(Borrow checker)，用来检查代码中程序借用其它值的引用指针时，每个借用行为的正确性。

```Rust
{                       // 下面就是借用检查器示意
  let r;                // ---------+-- 'a
                        //          |
  {                     //          |
    let x = 5;          // -+-- 'b  |
    r = &x;             //  |       |
  }                     // -+       |
                        //          |
  println!("r: {}", r); //          |
}                       // ----------+
```

上面的代码中，变量 `r` 明显是有比变量 `x` 更长的生命周期的，也就是 `'a` 要大于 `'b`。 `'b` 出了代码第 7 行的 `}`，也就是出了块级作用域之后，就被销毁了，而拥有更长生命周期的 `r`，却引用了它，这就构成了悬垂引用。编译器第一时间肯定会报错的。

### 函数中的生命周期

在函数中，受其传参与函数体，会影响最终函数的返回值。如果一个函数经过一定条件抉择，返回了其参数之中的引用，由于单看函数体，编译器和我们程序员都无法判断返回的引用的生命周期。这个时候，就需要我们显性的利用“生命周期标注”来让引用的生命周期有迹可循。

先来看一个例子，一个函数接收两个字符串切片（也就是引用），通过一定条件（判断长度），返回符合条件的切片。

```Rust
fn main() {
  let string1 = String::from("1111");
  let string2 = "222";

  let result = longest(string1.as_str(), string2);
  println!("The longest string is {}", result)
}

fn longest(x: &str, y: &str) -> &str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```

这个代码看上去好像没有什么问题，函数中，`x` 和 `y` 都在同一个作用域中，也就是说是具有相同的生命周期的，那不管函数返回的是哪一个，都是符合生命周期法则的才对。
然而，这只是拥有上帝视角的我们能看到的迹象。像编译器，它其实就拎着函数 `longest` 的实现来看，并不知道参数 `x` 和 `y` 的生命周期跟返回值的生命周期之间的关系，也不能在函数传参运行之前，就知道会返回哪一个值。所以就会啪啪地给程序员报错，跟你说，你这代码写得有问题，它需要更多的信息来推导整个函数中引用的生命周期关系，需要你辅助地增加一些标注。我们运行一下，看看报错的内容。

```rust
error[E0106]: missing lifetime specifier
 --> src/main.rs:9:33
  |
9 | fn longest(x: &str, y: &str) -> &str {
  |               ----     ----     ^ expected named lifetime parameter
  |
  = help: this function's return type contains a borrowed value, but the signature does not say whether it is borrowed from `x` or `y`
help: consider introducing a named lifetime parameter
  |
9 | fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  |           ++++     ++          ++          ++

For more information about this error, try `rustc --explain E0106`.
```

这个报错也是非常简洁但详细地指出问题，和修复的手段。前半部分，编译器语重心长地跟你说，我们需要给函数返回加一个生命周期标注啊，因为这个函数返回值是一个借用类型（引用），我并不能确定他是引用的参数 `x` 还是 `y`。
后半部分就直接上方法，让你参考着改一下，然后呢，就引出生命周期标注语法 —— `'a`。

### 生命周期标注语法

**这个生命周期的标注语法，并不会改变任何引用的实际作用域。**它只是用来帮助我们“取悦”编译器的，让编译器能意识到各个引用的生命周期，别一惊一乍地动不动就报错。

生命周期标注以 `'` 开关，英文读作 `tick`，名称往往是一个单独的小写字母，大多数时候直接使用 `'a` 来作为生命周期的名称。如果是引用类型的参数，就要把生命周期位于引用符号 `&` 之后，并用一个空格来与参数隔开：

```Rust
&i32 // 普通引用
&'a i32 // 具有显式生命周期的引用
&'a mut i32 // 具有显式生命周期的可变引用
```

正如上面据说，生命周期标注并不会对引用的作用域有丝毫地影响，本质上也只是让编译器知道多个引用之间的关系。回到前面的例子，我们添加上标注。

```Rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  if x.len() > y.len() {
    x
  } else {
    y
  }
}
```

在这之中，需要注意的点有：

- 和泛型一样，使用生命周期参数时，需要先声明 `<'a>`
- 返回值的生命周期会取与其相关的参数中的生命周期最短的那一个

第一点好理解，因为同一个函数中生命周期可以有多个，这里声明就是让函数签名更直观一些。但第二点又要如何理解呢。
其实，还是我们[[#借用检查]]里的内容，具体的引用传入函数中时，其生命周期 `'a` 的具体大小，就是所有 `'a` 参数中最“短命”的那个，也就是取所有标注了 `'a` 的生命周期的交集。这样的处理是方便返回的引用能 100% 地笃定绝对是可以无脑地使用的。让我们来举个 🌰，看一下下面的代码：

```Rust
fn main() {
  let mut result: &str;
  let string1 = String::from("longlonglonglong");

  {
    let string2 = String::from("short");
    result = longest(string1.as_str(), string2.as_str());
    println!("{}", result); // 1
  }
  println!("{}", result); // 2
}

fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
  // 实现我故意省略的
}
```

上面的代码中，有两处作用域，一个是 `main` 函数的作用域，一个就是使用 `{}` 括起来的局部作用域。从[[#借用检查]]的内容可知，`string2` 的作用域是最小的，只有 `{}` 包裹起来的部分。这里我们看 `longest` 的函数签名是可以知道 `string1`、`string2` 和 `result` 们的关系的，此时的 `result` 的生命周期就是 `string1` 和 `string2` 二者中的交集，也就是 `string2` 的部分。所以我们在 代码 1 处访问 `result` 的时候，是可以正常访问的。但是脱离了这个块级作用域之后，在代码 2 处访问 `result` 时就报错了。我们来运行一下看看：

```Rust
error[E0597]: `string2` does not live long enough
  --> src/main.rs:7:44
   |
6  |         let string2 = String::from("short");
   |             ------- binding `string2` declared here
7  |         result = longest(string1.as_str(), string2.as_str());
   |                                            ^^^^^^^^^^^^^^^^ borrowed value does not live long enough
8  |         println!("{}", result);
9  |     }
   |     - `string2` dropped here while still borrowed
10 |     println!("{}", result);
   |                    ------ borrow later used here

For more information about this error, try `rustc --explain E0597`.
```

报错的意思是，`string2` 活得不够久，哈哈，这就奇怪了。我们知道 `longest` 函数的实现，他会返回最长长度的字符串，所以一定是 `string1` 会返回，那 `result` 为啥还要看 `string2` 的眼色。这背后的理由让人暖心。在日常的开发中，更改某类函数或者使用哪类方法库是很家常便饭的事，我们并不知道每一个函数具体的实现，可能就 peek 一眼函数的签名，这里跟编译器也是同样的处境。编译器在推断这些变量的生命周期的时候，不可能会去执行一遍函数来确定返回值是怎样，所以编译器是非常保守的。当一段代码可能出错的时候，它就会认定必然是会出错的，坚决地抛出错误。这也是为什么返回值的生命周期会取所有相关参数中最”短命“的那个的生命周期。
这种检查方式也倒逼着我们开发去写出更稳健更安全的代码，即使哪天需求变化，我要返回长度最短的那个字符串了，也不用去担心因为生命周期问题导致的报错。
真的太暖心了。rust 它真的，我哭死。

### 结构体上的生命周期

在[[Struct#结构体数据的所有权]]中说过，如果如果声明的结构体借用了外部的引用，那么是要为这个结构体和数据都标注一个生命周期的。与[[Generics|泛型]]相似，使用时也需要声明。

```Rust
struct Cat<'a> {
  name: &'a str,
}
```

这个也说明，`name` 字段所引用的字符串与结构体 `Cat` 的实例同在，或者说至少结构体不在的时候，字符串也还要在，再换句话说就是，`name` 字段所引用的字符串活得要比结构体实例更久。

举个 🌰：

```Rust
#[derive(Debug)]
struct Cat<'a> {
  name: &'a str,
}

fn main() {
  let i;
  {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    i = Cat {
      name: first_sentence,
    };
  } // first_sentence 在这里就已经被释放了
  println!("{:?}",i);
}
```

上面的代码中，可以看到，结构体 `i` 中的 `name` 字段所借用的字符串 `first_sentence` 在 `}` 块级作用域结束的时候就会被释放掉。因此在随后的 `println!` 中使用就会导致无效引用而报错。也就是说，结构体所借用的字符串活得比结构体还短。
