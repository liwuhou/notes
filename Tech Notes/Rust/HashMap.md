在 Rust 中，HashMap 是一个以键值形式存储数据的集合类型，并提供了平均复杂度为 `O(1)` 的查询方法。其类型为 `HashMap<K, V>`。

### 创建 HashMap

#### HashMap::new

HashMap 提供了一个关联方法 `new`，可以用来创建 HashMap，与 Vec 和 String 不同的是，HashMap 没有包含在 Rust 的 `prelude` 中，所以需要把它从标准库中引入到当前作用域来。

```Rust
use std::collections::HashMap;

fn main() {
  let mut map: HashMap<&str, i32> = HashMap::new();

  map.insert("red", 0);
  map.insert("green", 1);
}
```

#### HashMap::with_capacity

与 [[Vector#Vec::with_capacity|Vector]] 一样，如果预先知道要存储的键值对个数，可以使用 `HashMap::with_capacity` 创建预先分配了足够多内存空间的 HashMap，避免后续大量插入导致频繁的内在分配和拷贝，以提升性能。

#### 使用迭代器和 colloect 方法转换为 HashMap

迭代器中提供了一个 `collect` 方法，可以把某些类型转为集合类型，这其中也存在把 `Vec<(T, U)>` 转为 `HashMap<T, U>` 的操作。

```Rust
use std::collections::HashMap;

fn main() {
  let list = vec![
    ("foo".to_string(), 1),
    ("bar".to_string(), 2),
    ("baz".to_string(), 3),
  ]

  let map: HashMap<_, _> = list.into_iter().collect(); // 类型 HashMap<_, _> 是让 Rust 编译器去推导
}
```

通过 `into_iter()` 方法将列表转为迭代器，接着通过 `collect` 方法收集。由于在 `collect` 方法内部支持转换为多种类型的目标集合，所以这里我们需要手动地为 `map` 标注类型为 `HashMap<_, _>`，告诉编译器我们要收集的是 HashMap 集合类型，但具体的 `K`、`V` 类型，通过数据自行推导。

如果没有标类型，编译器由于不知道 `collect` 方法最终要转换成什么类型，就会无情地抛错了。

```plain
error[E0282]: type annotations needed // 需要类型标注
  --> src/main.rs:10:9
   |
10 |     let teams_map = teams_list.into_iter().collect();
   |         ^^^^^^^^^ consider giving `teams_map` a type
```

#### Hash::from

对于上面的例子，其实也可以直接使用 `Hash::from` 来把合适类型的 Vec 转成 HashMap。

```Rust
let map = HashMap::from(list);
```

### 插入 HashMap

可以使用 `insert(K, V)` 来插入键值对到一个 HashMap，但要注意的是，`K`、`V` 类型要与 `HashMap<K, V>` 里对应。

### 查询 HashMap

HashMap 也能通过 `get` 方法获取元素，但要注意的是，`get`方法返回的一样是 `Option<T>` 类型，要注意为 `None` 的情况。

```Rust
let mut map: HashMap<&str, i32> = HashMap::new();
map.insert('key', 1);
// ...
if let Some(val) = map.get('key') {
  // 有对应 key-value 的情况
  println!("{}", val);
} else {
  // 找不到，为 None 的情况
}

// get 方法返回的都是 Option<&T> 类型
let value: Option<&i32> = map.get('key')
```

`map.get('key')` 返回的是 `Option<&i32>` 类型，当然我们也可以直接拿到 `i32` 元素。

```Rust
let value: i32 = map.get('key').copied().unwrap_or(0);
```

这里的 `copied` 方法是 `Option` 枚举类型为具有[[Ownership#Copy 特性|Copy 特性]] 的值提供的，可以把 `Option<&mut T>` 或 `Option<&T>` 类型转成 `Option<T>` 类型。

````Rust
impl<T> Option<&T> {
  /// Maps an `Option<&T>` to an `Option<T>` by copying the contents of the option.
  ///
  /// # Examples
  ///
  /// ```
  /// let x = 12;
  /// let opt_x = Some(&x);
  /// assert_eq!(opt_x, Some(&12));
  /// let copied = opt_x.copied();
  /// assert_eq!(copied, Some(12));
  /// ```
  #[must_use = "`self` will be dropped if the result is not used"]
  #[stable(feature = "copied", since = "1.35.0")]
  #[rustc_const_unstable(feature = "const_option", issue = "67441")]
  pub fn copied(self) -> Option<T>
  where
    T: Copy
  {
    match self {
      Some(&mut t) => Some(t),
      None => None,
    }
  }
  /// Maps an `Option<&T>` to an `Option<T>` by cloning the contents of the option.
  ///
  /// # Examples
  ///
  /// ```
  /// let x = 12;
  /// let opt_x = Some(&x);
  /// assert_eq!(opt_x, Some(&12));
  /// let cloned = opt_x.cloned();
  /// assert_eq!(cloned, Some(12));
  /// ```
  #[must_use = "`self` will be dropped if the result is not used"]
  #[stable(feature = "rust1", since = "1.0.0")]
  pub fn cloned(self) -> Option<&T>
  where
    T: Clone,
  {
    match self {
      Some(t) => Some(t.clone()),
      None => None,
    }
  }
}
````

如果值具有 `Clone` 特性的话，也可以使用 `cloned` 方法

然后这个 [[Option#unwrap_or|unwrap_or(default)]] 也是 `Option` 里提供的，代表着，如果 `Option` 类型是 `some` 则取 `Some` 中的值，如果是 None，则取传入的 `default` 。

这也是个快速地解 `Option` 方式，还有 [[Option#unwrap|unwrap]]方法，当你有绝对把握对应的 `Option` 类型一定是 `some` 类型时，可以直接用 `unwrap` 解。但是如果是 `None`，那就程序直接报错不干了。

所以 `unwrap_or(default)` 方法会更稳一点。

与 [[Vector]] 类似，HashMap 也可以使用 `map[key]` 的形式取值，但一样的，如果没有值，编译器就报错给你看。这个其实也是上面 `Option::unwrap` 方法的语法糖。

```Rust
let mut map: HashMap<&str, i32> = HashMap::new();
map.insert("red", 0);
assert_eq!(map["red"], map.get("red").unwrap());
```

也可以通过循环的方法，依次遍历 HashMap 中的键值对

```Rust
for (key, value) in &map {
  println!("{}: {}", key, value);
}
```

### 更新 HashMap

上面[[#插入 HashMap]]中有提到可以使用 `insert` 方法对 HashMap 插入键值对。如果键原本不存在，那就插入，如果存在，则更新值。

```Rust
let mut map: HashMap<&str, i32> = HashMap::new();

map.insert("red", 1);
map.insert("red", 2);
println!("{}", map.get("red").unwrap()); // 2
```

但如果我们有需要是要，键名不存在的时候插入，存在的时候就不插入了要咋整呢。
你会想，用 `get` 方法判断一下。

```Rust
// 承接上面代码片段
if let None = map.get("green") {
  map.insert("green", 1);
}

// 或是使用 contains_key 方法
if map.contains_key("green") {
  map.insert("green", 1);
}
```

看着也可以，但是不够 Rusty，事实上 Rust 提供了相关的方法让我们这么操作，那就是 `entry` 和 `or_insert` 方法。

```Rust
// 承接上面代码片段
map.entry("yellow").or_insert(1);
println!("{}", map["yellow"]); // 1

map.entry("yellow").or_insert(100000);
println!("{}", map.["yellow"]); // 还是 1
```

同时，Rust 还贴心地在 `or_insert` 方法返回了这个值的可变引用 `&mut T`。方便让你在这个 HashMap 里有值的情况下，对这个值进行修改。Rust，它真的，我哭死。

```Rust
let black_count = map.entry("black").or_insert(0);
assert_eq!(black_count, 0);

*black_count += 1;
assert_eq!(*black_count, 1);

```
