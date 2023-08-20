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

还有个有意思的地方，动态数组可以声明最小长度（容量），如果你预先就知道**数组要存储的元素的最小个数**，那么可以使用 `Vec::with_capacity` 声明一个在内存空间中预先分配了最小长度的动态数组。这样的好处是避免动态数组一下子插入大量的值，导致 rust 没有防备，频繁地在内存中分配和拷贝，提升一点性能。Rust 连这点都考虑到了，它真的，我哭死。

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
println!("{}", v.capacity()); // 4, 扩展为 4
```

调用 `vec.capacity()` 方法会返回 Rust 当前为我们 Vec 分配好的内在空间大小，而 `vec.len()` 则是返回当前 Vec 中已经存储的元素数量。前面说过 Vec 是动态长度的数组类型，也就是说，他的 `capacity` 是会动态变化的，当长度将要超过已有的 `capacity` 时，Rust 就会重新分配一段更大的连续内存空间，然后将原有的 Vec 拷贝过去。在 Rust 中，容量的调整策略是加倍的，例如 2 -> 4 -> 8 ...

### vec![]

可以使用宏 `vec!` 来创建数组，与 `Vec::new` 不同的是，`vec!` 能像字面量一样，创建的同时赋予初始值：

```Rust
let v = vec![1, 2, 3]; // v 推导为 Vec<i32> 类型
```

### 实现了 From/Into 特征的类型转换为 Vec

只要类型实现了 `From` 或 `Into` 特征，就可以转为 Vec

```Rust
let arr: [u8; 2] = [1, 2];
let v = Vec::from(arr);
let v: Vec<u8> = arr.into();
let v: Vec<u8> = Vec::from("hello");

let s = "hello".to_string();
let v = s.into_bytes(); // into_bytes 就不需要显性地给 v 加上 Vec<u8> 的类型了。
```

Vec 相比简单数组的好处在于，可以动态地调整数组的长度，而 Array 实现了 `From` 特征，可以让数组很方便地转为 Vec。

### 更新 Vec

可以使用 `push` 方法，追加元素

```Rust
let mut v = Vec::new();
v.push(1);
```

需要注意的是，这里声明的变量须为[[Variables#变量的可变性|可变类型（mutable）]]

同时，Vec 可以使用 `extend` 来扩展自身

```Rust
let mut v1 = vec![1, 2, 3];
let mut v2 = Vec::new();
let arr: [i32; 3] = [4, 5, 6];

v2.extend(&v1); // 接收一个实现了 Iterator 特征的值
v2.extend([1, 2, 3]);
v2.extend(arr);
```

### 从 Vec 中获取元素

在 Rust 中获取 Vec 元素有两种方式：

1. 通过下标获取元素
2. 通过 `.get(Idx)` 方法获取元素

```Rust
fn main() {
  let v = vec![1, 2, 3];

  println!("{}", &v[1]); // 2

  if let Some(t) = v.get(100) {
    println!("{t}");
  }
}
```

使用下标的方法就比较潇洒，很是随意，但是就会有下标越界的风险。一旦数组发生越界，程序可是会无情退出的啊。而 `.get` 方法， Rust 内部做了处理，它返回了一个 `Option<&T>` 类型，有值就是 `Some<T>` 类型，没有则是 `None`。因此 `.get` 方法安全度较高，虽然看着调用会变得比较啰嗦。

这两种方法，赋予了 Rust 既灵活又安全的特性，当你在某个场景，有把握数组访问不会越界的时候，可以直接用下标获取。否则的话，就使用 `.get` 方法。

### Vec 与其元素共存亡

与结构体一样，`Vector` 类型在超出作用域的时候，会被自动地删除。

```Rust
{
  let v = vec![1, 2, 3];
} // 此处超出作用域后，v 就被销毁
```

当 `Vector` 被删除之后，它内部存储的内容也会被随之删除。这种方案为了内存安全考虑，做法显得有些简单粗暴。但是，如果 `Vector` 中有元素被引用后，事情就变得没那么简单了。

### 同时借用多个数组元素

在[[References & Borrowing#可变引用|借用与引用的规则]]章节说过，可以有多个不可变引用，但同一时刻，只能有一个可变的引用。而在 `Vec` 中也是一样的。

```Rust
fn main() {
  let v = vec![1, 2, 3];
  let t = &v[1];
  v.push(4); // .push 方法是可变引用（&self）

  println!("{}", t); // 如果没有这行代码，程序还不至于报错
}
```

`let t = &v[1];` 进行了不可变借用，而 `v.push()` 进行了可变借用，如果后面没有再访问不可变借用，也就是不再使用变量 `t` 的话，还相安无事。但如果在可变借用后还使用了，就会构成同一时刻同一作用域内存在可变引用和不可变引用而报错的情况。

再深入探讨一下，明明一个是查询元素，一个是在数组尾部追加元素，看着是完全不相干的操作，是不是这个编译器太执着了。有这想法其实也正常，但要意识到的是，`Vector` 是一个长度可变的数组，它本质上是如果旧数组不够用了，就会在内存空间中寻找一块更大的内存空间，把旧数组的数据都迁移过去，所以原本不可变引用可能会引用到一块无效或者是完全不相干的内存。所以最稳的做法就是不允许进行此类的操作。

### 遍历 Vec

需要迭代遍历数组，也可以直接使用迭代的方式，这种方式比用下标去访问的方式更安全。

```Rust
let mut v = vec![1, 2, 3, 4, 5];

for i in 0..4 { // 别这么干
  println!(&v[i]);
}

// 推荐
for i in &v {
  //
}
for i in v.iter() {
  println!("{i}");
}

// 也可以在迭代过程中修改元素

for i in &mut v {
  *i += 1; // * 解引用
}
```

### 存储“不同”类型的元素

这个不同类型是打引号的，`Vec` 确实只能存储一种类型，但我们可以使用[[Enum|枚举类型]]和[[Deepin Trait#特征对象的定义|特征对象]]来包装不同的类型，对 `Vec` 而言，他的类型就是一种枚举或者特征对象类型。

#### 使用枚举实现 Vec 存储不同类型的元素

```Rust
#[derive(Debug)]
enum IpAddr {
  V4(String),
  V6(String),
}

fn main() {
  let v: Vec<IpAddr> = vec![IpAddr::V4("1.1.1.1".toString), IpAddr::V6("::1".to_string())];

  for i in &v {
    println("{:?}", i);
  }
}
```

数组的类型其实是 `Vec<IpAddr>`，所以很自然地可以存放枚举成员。

#### 使用特征对象实现 Vec 存储不同类型的元素

```Rust
// 定义一个共有的特征
trait IpAddr {
  fn display(&self);
}

struct V4(String);
struct V6(String);

impl IpAddr for V4 {
  fn display(&self) {
    println!("{:?}", self.0);
  }
}

impl IpAddr for V6 {
  fn display(&self) {
    println!("{:?}", self.0);
  }
}

fn main() {
  let v: Vec<Box<dyn IpAddr>> = vec![
    Box::new(V4("127.0.0.1".to_string())),
    Box::new(V6("::1".to_string())),
  ]

  for i in &v {
    i.display();
  }
}
```

这里的实现方式相比枚举类型的实现方式会更复杂一点。首先我们为 `V4` 和 `V6` 都实现了 特征 `IpAddr`，然后将它俩的实例使用 `Box::new()` 包裹起来，存在数组 `v` 中。有一点需要注意的是，我们在数组声明处必须手动地指定类型为 `Vec<Box<dyn Trait>>` 类型，表示数组存储的是特征 `IpAddr` 对象。虽然特征对象实现的篇幅较多，但它却是 Rust 中更为常见的实现方式，原因就在于特征对象它足够的灵活，相反编译器对枚举的限制就有很多，也无法动态地增加其它类型。特征对象的话，只要类型实体实现了相应的特征就可以加入某个特征对象的大军，而枚举你必须都找到枚举类型的定义处，增加一个类型。

### Vec 的排序

在 Rust 里，实现了两种排序算法，分别为稳定的排序 `sort` 和 `sort_by`，以及非稳定的排序 `sort_unstable` 和 `sort_unstable_by`。首先多了个 `by` 的关键字，是因为可以传递一个回调函数，来让我们自定义一些排序规则，譬如对结构体、枚举类型进入比较。然后这个所谓的非稳定并不是指这个排序算法本身不稳定，而是指在排序过程中对相等元素的处理方式不是稳定的。在稳定排序算法里，对相等元素不会进行重新排序，而在不稳定的算法里，则是不保证这一点 。

总体而言，非稳定排序算法在速度上会优于稳定排序算法，同时稳定排序还会额外地多分配数组一半的内存空间。

#### 整数数组的排序

整数数组的排序比较直接。

```Rust
let mut vec = vec![5, 1, 2, 4, 3];
vec.sort_unstable();
assert_eq!(vec, [1, 2, 3, 4, 5]);
```

#### 浮点数组的排序

浮点数的组的排序比较复杂，因为在浮点数中存在一个 `NAN` 值，这个值无法与其它的浮点数进行比较，因此浮点数并没有像整数那样实现全数值可比较的 `Ord` 特征，而是实现了部分可比较的 `PartialOrd` 特性。
为此，我们要对浮点数进行排序时，如果不确定是否包含了 `NAN` 值，那么就可以使用 `partial_cmp` 来作为大小判断的依据。

```Rust
let mut vec = vec![5.0, 2.0, 3.2, 1.0, 15f32]; // 15f32 就是 NAN 值
vec.sort_unstable_by(|x, y| x.partial_cmp(y));
assert_eq!(vec, vec![1.0, 2.0, 3.2, 5.0, 15f32]);
```
