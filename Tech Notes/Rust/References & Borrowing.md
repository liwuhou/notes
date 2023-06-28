前面说过，在调用函数的传参的时候，也会发生所有权的转移。如果 Rust 仅仅支持转移所有权的方式去获取一个值，那程序编写和运行起来就太蛋疼了。
有没有什么方法，能让 Rust 像其它（js， java）编程语言一样，使用某个变量的指针或者引用呢？答案当然是可以的。所以现在是时候聊聊 **引用(References)** 和 **借用(Borrowing)** 的概念了。

### 引用与解引用

```Rust
fn main() {
  let x: i32 = 5;
  let y: &i32 = &x; // 取 x 值的引用
  let ref z = x; // 取引用的另一种语法

  assert_eq!(5, x);
  assert_eq!(5, *y); // 解引用，取值
  assert_eq!(5, *z); // 解引用，取值
}
```

Rust 支持取变量的引用，类似于 c++ 中取指针的概念，然后也可以通过去指针变量加 `*` 来取指针指向的值。

### 不可变引用

同声明变量一样，引用也分可变和不可变。默认是不可变的。
`&` 符号是引用，它可以允许你使用值，但不获取所有权。如果尝试去获取值的所有权，就会被编译器啪啪报错。

```Rust
fn main() {
  let s1 = String::from("Hello");
  let r1 = &s1;

  r1.push_str("World"); // 报错，r1 不是可变引用
}
```

不可变引用的好处就是，传递的时候，不用操作会被影响到值的改变，特别是调用函数的时候。

```Rust
fn main() {
  let s = String::from("hello");

  let len = get_len(&s); // 放心传

}
fn get_len(some_str: &String) -> usize {
  some_str.len()
}
```

### 可变引用

可变引用，允许你声明一个可变的值引用

```Rust
fn main() {
  let mut s = String::from("Hello");

  change(&mut s); // 传递也要有 &mut 符，表示传入可变引用
}

fn change(some_str: &mut String) { // 声明类型是可变引用
  some_str.push_str("World"); // 这里改变了值
}
```

但有一个点要注意的是，**可变引用同一“时刻”内只能存在一个**。

```Rust
let mut s = String::from("hello");

let r1 = &mut s;
let r2 = &mut s;

println!("{}, {}", r1, r2); // 报错，这里会报 s 的引用存在多个
```

注意这个同一时刻，比较有内容。如果把 `println` 方法改成下面这样就不会报错。

```Rust
let mut s = String::from("hello");

let _r1 = &mut s;
let r2 = &mut s;

println!("{}", r2); // 不报错

```

并且**可变引用与不可变引用，不可同一时刻同时存在**。

```Rust
let mut s = String::from("hello");

let r1 = &s;
let r2 = &s;
// 如果 r1 和 r2 的使用在此结束就还好
let r3 = &mut s;

println!("{}, {}", r1, r2)
// 在 r3 之后还使用了 r1 和 r2，Rust 就会认为 s 的可变引用和不可变引用同时存在了
// 然后编译器就会啪啪报错
```

最早的 Rust 编译器（Rust 1.31 前），只要 `r1`、`r2` 和 `r3` 只要在同一花括号（作用域）内，就会触发**无法同时借用可变和不可变的规则**。
但是新的编译器中，优化为在引用作用域结束位置变成最后一次使用的位置，因此如果 `r1` 和 `r2` 在 `r3` 声明之后就再也不访问的话，就不算是同时借用可变和不可变引用。

对于这种编译器的优化，Rust 还专门起了个名字 —— Non-Lexical Lifetimes（NLL），用于找到某个引用在作用域结束前最后一次被使用的位置。

### 悬垂引用

悬垂引用也称作悬垂指针，指的是指针在指向某个值之后，这个值被释放了，而指针仍然存在，其指向的内存可能不存在任何值或是已经其它值所取代。而 Rust 编译器可以确保引用不会变成悬垂状态，因为你如果想要释放数据，那么就必须要停止其引用，否则就要啪啪报错。

```Rust
fn main() {
  let reference_to_notion = dangle();
}

fn dangle() -> &String {
  let s = String::from('hh');
  &s // 返回 s 的引用
} // s 离开作用域后就会被丢弃，其内存也会被释放，但却返回了 s 的引用，这里就会报错
```

### 总结

总的来说，借用规则如下：

- 同一时刻，你只能拥有要么一个可变引用, 要么任意多个不可变引用
- 引用必须总是有效的
