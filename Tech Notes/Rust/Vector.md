`Vector` 是 Rust 中的一种动态数组，与 [[Array]] 不同的是，Vec 可以存放不限个数的值，而 `Array` 的数组长度是固定的。动态数组虽然只能存放相同类型的元素，但这个元素在内存中是紧密排列的，一个紧挨着另一个，因此访问其中的某个元素的成本特别低。当然你也可以用一些方式来存放不同的类型的元素，就是之前讲过的[[Enum]]或[[Deepin Trait#特征对象的定义|特征对象]]。

### 创建动态数组

在 Rust 中，创建动态数组主要是使用 `Vec::new()` 关联函数或 `vec![]` 宏函数。

#### Vec::new

使用 `Vec::new` 创建动态数组是最 rusty 的方式，它调用了 Vec 中的 `new` 关联函数：

```Rust
let v: Vec<i32> = Vec::new();
```

Rust 编译器具有类型推导的能力，如果你在后续代码中有往数组变量里 `push` 元素，那么会自动地推导出数组的类型。

```Rust
let mut v = Vec::new();
v.push(1); // v 会被推导为 Vec<i32> 类型
```

但你如果只是声明了一个动态数组，没有更新过数组的值，也就是不给编译器推导的机会，那就别怪编译器报你的错了。

#### Vec::with_capacity(length)

还有个有意思的地方，动态数组可以声明最小长度，如果你预先就知道**数组要存储的元素的最小个数**，那么可以使用 `Vec::with_capcity` 声明一个在内存空间中预先分配了最小长度的动态数组。这样的好处是避免动态数组一下子插入大量的值，导致 rust 没有防备，频繁地在内存中分配和拷贝，提升一点性能。Rust 连这点都考虑到了，它真的，我哭死。

```Rust
let v: Vec<String> = Vec::with_capacity(2);

println!("{}", v.len()); // 0
println!("{}", v.capacity()); // 2
v.push(String::new());
v.push(String::new());
println!("{}", v.len()); // 2
println!("{}", v.capacity()); // 2
v.push(String::new());
println!("{}", v.len()); // 3
println!("{}", v.capacity()); // 4
```

### vec![]

可以使用宏 `vec!` 来创建数组，与 `Vec::new` 不同的是，`vec!` 能像字面量一样，创建的同时赋予初始值：

```Rust
let v = vec![1, 2, 3]; // v 推导为 Vec<i32> 类型
```

### 更新 Vec

可以使用 `push` 方法，追加元素

```Rust
let mut v = Vec::new();
v.push(1);
```

需要注意的是，这里声明的变量须为[[Variables#变量的可变性|可变类型（mutable）]]

### 从 Vec 中获取元素
