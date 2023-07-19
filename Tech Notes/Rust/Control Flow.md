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

```Rust
for item in &container {
  todo!();
}

// or
for item in container.iter() {
  todo!();
}
```

所以一般是使用集合变量的引用 `&container` 。

如果想在循环中修改该元素，那么也可以使用上 `mut` 关键字。

```Rust
for item in &mut collection {
  todo!();
}
```

有以下几种使用方式

| 使用方法                      | 等价使用方式                                      | 所有权     |
| ----------------------------- | ------------------------------------------------- | ---------- |
| `for item in collection`      | `for item in IntoIterator::into_iter(collection)` | 转移所有权 |
| `for item in &collection`     | `for item in collection.iter()`                   | 不可变借用 |
| `for item in &mut collection` | `for item in collection.iter_mut()`               | 可变借用   |

#### 获取循环中的元素的索引

使用迭代器中的 `enumerate()` 方法，会返回一个 `(索引, 元素)` 的元组。

```Rust
let a = [1, 2, 3, 4, 5];

for (idx, item) in a.iter().enumerate() {
  // idx 是索引， item 是元素
}
```

#### 两种循环方式的优劣对比

迭代一个数组有大概两类方法，一种是通过数组索引，去访问数组该索引中的元素。

```Rust
// 第一种

for i in 0..collection.len() {
  let item = collection[i];
}
```

第二种则是直接遍历数组的元素

```Rust
for item in collection {}
```

二者优劣如下：

- 性能：第一种使用方式是通过 `collection[i]` 的索引去访问元素的，Rust 会在每次循环中都去检查边界情况会有一些额外的性能开销。而第二种直接迭代元素的方式就不会触发这种检查，因为编译器会在编译时就完成分析并证明这种上访问是合法的。
- 安全：第一种方式中，对 `collection` 的索引访问是非连续的，存在一定可能性在两次访问时，`collection` 发生了变化（因为存在并发等场景嘛），导致脏数据的产生。而第二种的迭代方式是连续访问的，因此不存在这种风险（由于所有权的限制），在访问过程中，数据并不会发生变化。

由此，`for` 循环无需任何条件限制，也不需要通过索引去访问，最安全也是最常用的做法就是直接迭代集合类型中的元素。

### continue 和 break

同类 c 语言一样，在 [[#for 循环|for]]、[[#while 循环|while]] 和 [[#loop 循环|loop]] 等循环中，可以通过 `contine` 关键字来路过此次循环，或者通过 `break` 来提前结束当前整个循环。

当有多层的循环嵌套的时候，可以通过为每一个循环添加一个标签 `'label`，然后使用 `continue` 或 `break` 去操作指定标签的循环。说起来有些抽象，看一下下面的例子：

```Rust

// 填空
fn main() {
  let mut count = 0;
  'outer: loop {
    'inner1: loop {
      if count >= 20 {
        // 这只会跳出 inner1 循环
        break 'inner1; // 这里使用 `break` 也是一样的
      }
      count += 2;
    }
    count += 5;
    loop {
      if count >= 30 {
        break 'outer;
      }
      continue 'outer;
    }
  }
  assert!(count == 30);
}

```

另外，与那些语言不太相同的就是，跟 `loop` 循环配合的时候 `break` 关键字还能带一个返回值，有些类似 `return`。

```Rust
let mut count = 0;

let result = loop {
  count += 1;

  if count == 10 {
    break count * 2;
  }
};

println!("{}", result); // 20
```

### while 循环

同类 c 语言一样，while 循环可以使用一个条件来循环，当条件为 `true` 的时候就继续循环，为 `false` 就结束循环。

```Rust
let n = 0;
while n <= 5 {
  println!("{}", n);
  n += 1;
}

### loop 循环
```

#### while 循环与 for 循环的对比

同样的，`for` 循环能干的事，`while` 循环也能干，不过，还是 `for` 循环直接迭代元素的性能和安全性最好。

### loop 循环

`loop` 循环可能是最简单无脑和普适性最高的循环，可以适用于多种循环场景（能用，但不一定是最优选择）。`loop` 循环本身是一个简单的无限循环，可以在内部通过 `break` 关键字来控制循环何时结束。

`loop` 循环就像 `while(true)` 循环，一定要注意代码结束的条件，否则会无限循环下去，跑满你整个 cpu 核心。

另外有意思的是，`loop` 循环是一个表达式，可以返回一个值，因此能用于赋值。
