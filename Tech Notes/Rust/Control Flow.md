众所周知，Rust 跟大部分编程语言一样，都是从上到下执行的，也跟其它编程语言一样，拥有流程控制语句。下面就介绍一下 `if` 条件语句与 `loop` 循环语句。

### 条件语句

条件语句可以创造一些代码执行的条件，通过给定的条件语句判断要执行的代码。

```Rust
fn main() {
  let condition = true;

  if condition {
    // condition is true
  } else {
    // else ...
  }
}
```

### 与 let 语句结合

同时，Rust 还可以使用 `if` 语句来为 `let` 赋值，用法看起来有点像三元表达式。

```Rust
let number = if condition {
  5
} else {
  6
};

let number = if condition {5} else {6}; // some
```

这里需要注意的点是，`if` 和 `else` 分支返回的类型需要一致，除此之外，块中的最末尾的代码必须是表达式，才能为变量赋值。

### 使用 else if 构建多重分支

同其它语言一样，Rust 也可以构建多重条件分支。执行程序的时候，会从上到下顺序执行每一个分支判断，一旦有任一分支的条件为真，则跳出 `if` 语句块。

```Rust
fn main() {
  let n = 6;

  if n % 4 == 0 {
    println!("number is divisible by 4");
  } else if n % 3 == 0 {
    println!("number is divisible by 3");
  } else if n % 2 == 0 {
    println!("number is divisible by 2");
  } else {
    println!("number is not divisible by 4, 3, or 2");
  }
}
```

当然，如果有多个分支话，大量的 `if` `else` 会让代码变得丑陋，可读性也会大大降低，这个时候可以考虑用 [[Match]] 重构。

### 循环语句

在 Rust 语言中有三种循环方式：`for`、`while` 和 `loop`。

#### for 循环

for 循环的语法很简单，声明一个当前循环内的变量，用一个[[Collections|集合类型]]的值用来遍历。

```Rust
for 当前循环变量 in 集合 {
  // 循环代码
}
```

例如从一数到十

```Rust
for i in 1..=10 {
  println!("{}", i);
}
```

这个 `1..=10` 就是一个序列，意为从 `1` 到 `10`，包含 `10` 的序列。还有一个点需要注意的是，使用 `for` 语句时，往往给的是集合类型的引用形式。因为如果不使用引用的话，集合类型的所有权就会被转移到 `for` 语句块中，后续的代码就无法再使用这个集合了。
