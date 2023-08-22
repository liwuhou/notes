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

### 查询 HashMap
