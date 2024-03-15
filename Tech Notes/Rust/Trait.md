Rust 中的特征，在其它编程语言中有着另外的一个名字 —— 接口（interface）。在 Rust 中，特征定义了一组可以被共享的行为，只要实现了特征，你就能使用这个行为。像在 [[Generics]] 中提到的，定义了一个泛型参数 `T`，只有 `T` 实现了 `std::ops::Add` 才可以进行合法的加法操作。

```Rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
  a + b
}
```

> `Output` 用来定义两个变量相加之后返回的数据类型

这其实也点明了特征的作用 —— 特征定义了一组可以被共享的行为，只要实体实现了特征，就能使用特征定义的行为。

### 定义特征

如果有不同的类型具有相同的行为，例如上面的 `i32` 类型和 `f64` 类型都具有加法的行为。那么就可以定义一个特征，为这些类型实现该特征。
举个更具体的例子，我养了一只猫猫和一条修勾，他们都具有相同的行为：吃饭、睡觉、拆家。对于铲屎官来说，这俩货的行为基本都是相似的，可以共享的。因此是可以抽象为这么一个特征：

```Rust
pub trait Pet {
  fn eat(&self) -> String;
  fn sleep(&self) -> String {
    // 这个是特征默认实现方法
    format!("{} is sleep", &self.name)
  }
  fn break_down_your_furniture(&self, furniture: &str) -> String;
}
```

这里声明特征需要用到关键字 `trait`，上面代码中 `Pet` 即是特征名。而在大括号中定义的都是该特征具有的方法（行为）。
特征不用管方法的具体实现，只需要给出方法的签名，正如猫猫和狗狗都会拆家，但他们的具体行为与破坏力又都有一些不同。

### 为类型实现特征

假设我们有狗狗的结构体 `Dog`，现在为他们实现 `Pet` 的特征。

```Rust
struct Dog {
  name: String
}

impl Pet for Dog {
  fn eat(&self) -> String {
    format!("{} is {}", &self.name, "eating.")
  }
  fn break_down_your_furniture(&self, funiture: &str) -> String {
    format!("{} breaks down your {}", &self.name, funiture)
  }
}

fn main() {
  let anz = Dog { name: String::from("Anz") }
  println!("{}", anz.eat()); // Anz is eating.
  println!("{}", anz.sleep()); // Anz is sleeping.
  println!("{}", anz.break_down_your_furniture("sofa")); // Anz breaks down your safa
}
```

可以看到，使用的是 `impl Pet for Dog`，读作“为 Dog 实现特征 Pet”。可以看到，我们并没有为特征实现 `sleep` 方法，所以结构体实例调用自身 `sleep` 方法的时候，调用的其实是特征 `Pet` 默认实现的 `sleep` 方法。当然，如果 `Dog` 结构体实现了对应的 `sleep` 方法，那么调用的就会是 `Dog` 自身的 `sleep`。说白了特征里的默认实现只是一个“缺省方法`。

> 顺便一提，这货就是安仔(Anz)。
> ![](http://cdn.liwuhou.cn/tmp/20230802074149.png)

### 使用特征的关联函数声明类型实体

我们不仅可以使用特征去约束类型，也可以直接使用特征来生成相应的类型实例，比如 `Pet` 特征有这么一个 `new` 方法。

```Rust
trait Pet {
  fn new(name: &str) -> Self; // 这里的 Self 指代实现者的类型
}

// 然后为 `Cat` 和 `Dog` 类型实现特征 Pet
struct Cat {
  name: String
}
struct Dog {
  name: String
}

impl Pet for Cat {
  fn new(name: &str) -> Cat { // 这里可以换成具体生成的类型
    Cat {
      name: name.to_string(),
    }
  }
}

impl Pet for Dog {
  fn new(name: &str) -> Self { // 也可以接着使用 Self 指代
    Dog {
      name: name.to_string(),
    }
  }
}

fn main() {
  // 使用
  // 可以直接用类型生成实例
  let cat = Cat::new("BigHead");
  // 也可以用先生关联函数来生成
  let dog: Dog = Pet::new("Anz");
  // 要注意的是，这里的 `Dog` 类型不能省略，否则编译器不知道具体类型，就会报错
  let c: Cat = Pet::new("foo");
}
```

#### 特征定义与实现的位置（孤儿规则）

在 Rust 中，如果你想要为某个类型实现某个特征，那么**这个类型或者特征必须至少有一个是在当前作用域内定义的**，否则你无法改动到这个类型的特征。例如上面的 `Pet`，我们在 `Pet` 定义的位置，可以为标准库中的 `String` 类型实现 `Pet`，因为在 `Pet` 定义的位置。但我们没办法为 `String` 实现 `Display` 特征。这个规则也被戏称为”孤儿规则（Orphan Rules）“， 它能确保其它人不会改动到你的代码，你也不会破坏到不相关的其它代码。

### 使用特征来约束函数参数

我们还可以使用特征来约束函数的参数类型，意为具有 xx 特征的类型才可传入。可以假设有这么一个方法，家里的宠物一拆家就能通知到我回去揍它。那么期望传入的就是具有 `Pet` 特征的某个宠物了。

```Rust
pub fn notify_owner(pet: &impl Pet) {
  println!("Breaking news! {}", pet.break_down_your_furniture("furniture"));
  // Breaking news! Anz breaks down your furniture.
}
```

上面的 `fn notify_owner(pet: &impl Pet)` 只是语法糖，完整的写法是这样的。

```Rust
pub fn notify_owner<T: Pet>(pet: &T) {}
```

语法糖能快速地帮我们声明一个类型泛型，然后与参数绑定。它看着更简洁，一些简单的场景下非常的适用。例如一个参数的情况下，如果有两个参数，并且这两个参数不是同一个类型，也能用 `&impl` 语法糖声明。例如：

```Rust
let cat = Cat { name: "BigHead" };
let dog = Dog { name: "Anz" };

// cat 和 dog 是两个不同的类型（Cat 结构体和 Dog 结构体），但都实现了相同的特征 Pet

fn notify(pet1: &impl Pet, pet2: &impl Pet) {
  println!("{}, {}", pet1.break_down_your_furniture("home"), pet2.break_down_your_furniture("home"));
}
// &impl Trait 语法糖的 notify 的函数签名实际等价于
// fn notify<T: Pet, U: Pet>(pet1: &T, pet2: &T)

notify(&cat, &dog);
```

可以看到 `&impl` 的语法糖真正的样子，也由此，他只能用在不同类型但都实现了相同特征的参数的情况。如果两个参数都是同一个类型，就不能用这个语法糖了，而是要拆出来，还原本来的样子，用相同的泛型参数去限制。

```Rust
let big_cat = Cat { name: "BigHead" };
let small_cat = Cat { name: "SmallCat" };

fn notify<T: Pet>(pet1: &T, pet2: &T) {
  todo!();
}

notify(&big_cat, &small_cat);
```

`fn notify<T: Pet>(pet1: &T, pet2: &T)` 说明 `pet1` 和 `pet2` 都是相同的类型（`Cat` 结构体），同时 `<T: Pet>` 又约束了类型必须实现了特征 `Pet`。

### 多重特征约束

我们类型可以实现不止一个的特征，比如狗不仅有宠物 `Pet` 的特征，还有一些同时能胜任搜救犬和导盲犬等工作犬 `Assisant`。同样的函数也可以限制同时实现两个以上特征的类型参数。

```Rust
struct Dog {} // 狗狗结构体

impl Pet for Dog {} // 为狗狗实现特征宠物

impl Assisant for Dog {} // 工作犬

// 语法糖
fn some_fn1(dog: &(impl Pet + Assisant)) {todo!()}

// 原本的写法
fn some_fn2<T: Pet + Assisant>(dog: &T) {todo!()}
```

没错，是 `&(impl Trait1 + Trait2)`， 而不是 ~~`&impl(Trait1 + Trait2)`~~ 喔。

在使用 `impl` 关键字实现特征或实体方法的时候，如果实体带泛型，也能 `impl` 声明多重泛型。

```Rust
use std::fmt::Display;
struct Pair<T> {
  x: T,
  y: T,
}

impl<T> Pair<T> {
  fn new(x: T, y: T) -> Self {
    Self { x, y }
  }
}

impl<T: Display + PartialOrd> Pair<T> {
  fn cmp_display(&self) {
    if self.x >= self.y {
      println!("The largest member is x = {}", self.x);
    } else {
      println!("The largest member is y = {}", self.y);
    }
  }
}
```

这个 `cmp_display` 方法，并不是所有的 `Pair<T>` 结构体对象都可以拥有的，而是要同时实现了 `Display` + `PartialOrd` 的 `Pair<T>` 才可能拥有此方法。

### Where 约束

也是一个肥肠之好用的声明特征的语法，像上面多重特征约束的例子，虽然参数还不算多，特征约束的个数也不算多，所以函数签名还在一个能够接受的范围。但如果函数签名很复杂，那么 Rust 提供了一种在形式上看起来很简洁的写法来约束函数的签名。

比如我们有这么一个函数：

```Rust
pub fn some_fn<T: Pet + Assisant + Display, U: Display + Debug>(pet: &T, u: &U) -> i32 {}

// 用上语法糖后
pub fn some_fn(pet: &(impl Pet + Assisant + Display), u: &(impl Display + Debug)) -> i32 {}
```

那么可以使用 `where` 关键字改进一下写法，注意，上下几种写法在函数签名上的作用其实是完全等价的：

```Rust
pub fn some_fn<T, U>(pet: &T, u: &U) -> i32
where
  T: Pet + Assisant + Display,
  U: Display + Debug
{
  todo!();
}
```

### 函数中返回特征类型

代码大体上长这样的：

```Rust
pub fn get_me_a_pet(name: &str) -> impl Pet {
  Cat {
    name: name.to_string()
  }
}
```

这样写的好处在于，调用 `get_me_a_pet` 函数的人，不用关心你返回的真实类型，他只知道你这个类型实现了 `Pet` 特征，能使用特征中约束的方法。当你返回的类型比较复杂的时候，每次调用的人都要写出那玩意的类型（比如是什么 `T<U<X<Y>>>`，自己把 `T` 和 `U` 等参数脑补成各种名字长得跟火车一样的泛型），那无疑是一种灾难。调用的人真正关心的是该类型能干什么，所以此时返回一个特征类型就足以，真实的类型就让编译器去保留吧。

但这种方式也有一个限制，就是你函数体中只能返回一个具体的类型，你不能使用诸如条件分支这样的方法返回两种可能的类型，即使这两种类型都拥有相同的特征：

```Rust
pub fn get_me_a_pet(name: &str) -> impl Pet {
  if name == "Anz" {
    Dog {
      name: name.to_string(),
    }
  } else {
    Cat {
      name: name.to_string(),
    }
  }
}
```

这样的代码是无法通过编译的，会提示我们 `if` 和 `else` 返回了不同的类型，如果想实现这种返回不同类型的操作，可以了解一下 [[Deepin Trait#特征对象|特征对象]]

### 为自定义类型实现 + 操作

在 Rust 中除了数值可以进行加法操作，`String` 也可以，自然是 Rust 为该类型实现了 `std::ops::Add` 特征。同样的，如果我们想为自定义类型加上加法操作，那么只要实现该特征就可以了。

### Clone Trait 与 Copy Trait

Clone Trait 上有两个方法，`clone` 和 `clone_from` 方法，一般只实现 `clone` 方法即可，因为 `clone_from` 有默认实现。

```Rust
pub trait Clone {
  fn clone(&self) -> Self

  fn clone_from(&mut self, source: &self) {
    *self = source.clone();
  }
}
```

`clone` 与 `clone_from` 的差异在于，
