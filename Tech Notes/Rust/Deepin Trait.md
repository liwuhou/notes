### 运算符是个语法糖

你知道吗？在 Rust 中，许多的去处符都可以被重载，在这里运算符其实只是特征方法调用的语法糖而已。例如 `a + b` 里的 `+` 其实是 `std::ops::Add` 特征里的 `add` 方法的调用。因此我们可以很轻易的为自定义类型实现该特征来支持些类型的各种运算符操作。

```Rust
// 实现一个乘法运算的方法
use std::ops::Mul;
fn multiply<T: Mul<Output = T>>(a: T, b: T) -> T {
  a * b
}

#[derive(Debug)]
struct Foo<T> {
  value: T
}

// 为自定义类型实现乘法运算，先要实现乘法 Mul 特征
impl<T> Mul<Self> for Foo<T>
where
  T: Mul<Output = T>
{
  type Output = Foo<T>;
  fn mul(self, rhs: Self) -> Self::Output {
    Foo {
      value: self.value * rhs.value
    }
  }
}

fn main() {
  println!("{}", multiply(1i32, 2i32));
  println!("{}", multiply(2.0f64, 3.1415926f64));

  let foo1 = Foo { value: 123 };
  let foo2 = Foo { value: 124 };
  println!("{:?}", foo1 * foo2);
  // println!("{:?}", multiply(foo1, foo2)); // 这样也可以
}
```

> 这里的 `type Output = Foo<T>` 与 函数返回的 `Self::Output` 的语法是[[#关联类型]]

### 特征对象的定义

**特征对象就是指向某个特征的某类类型的实例**。比如在 [[Trait]] 中的 `Dog` 和 `Cat` 这这两个类型的实例，它们的类型都实现了 `Pet` 特征，他们都是满足 `Pet` 特征的特征对象。所以我们可以使用特征对象来约束某些类型。比如在 [[Trait#函数中返回特征类型]] 的例子里，就可以使用返回特征对象来限制函数的返回类型。

```Rust
// Dog 和 Cat 都实现了特征 Pet
let dog = Dog {}
let cat = Cat {}
let c: Vec<&impl Pet> = vec![Dog]; // 这里会报错， `impl Trait` 只可用于函数或方法的返回类型或入参类型

let c: Vec<Box<dyn Pet>> = vec![Box::new(dog), Box::new(cat)]; // 这种是可以的
let c: Vec<&dyn Pet> = vec![&dog, &cat]; // 这种也可以
```

这里变量 `c` 是存放了实现 `Pet` 特征的特征对象，可以看到第一种方式 `Vec<&impl Pet>` 的类型是不能通过编译的，Rust 会抛出错误，表明 `immpl Trait` 的形式只能用于函数或方法，因为它其实是个特征泛型的[[Trait#使用特征来约束函数参数|语法糖]]。

而使用了 `Box<dyn Pet>`，实际上是通过 [[Box]] 包装了一层，期望传入的是一个经过 `Box` 封装了的实现了 `Pet` 特征的特征对象。`&dyn Pet` 则是期望传入的是特征对象的引用，这里的关键字 `&dyn` 是 `Dynamic Dispatch` 动态分发的意思，意在告诉 Rust 编译器，让它放弃对类型全知全悉的执念，这类类型在运行时才能确定其类型，不要在编译期就给我报错不干了。

### `Box<T>` 与 `Box<dyn Trait>` 的区别

![](http://cdn.liwuhou.cn/tmp/20230807081458.png)

`Box<T>` 实际上是一类泛型的约束的，Rust 编译器会在编译期为每一个泛型参数生成对应的具体类型，见 [[Generics#泛型的性能]] 那章。这种方式称为**静态分布(Static Dispatch)**。

与静态分布对应的则是**动态分布(Dynamic Dispatch)**。只有在代码运行时，才能确定需要调用的是什么类型，上面的 `dyn` 关键字正是在强调“动态”这一特点。

而特征对象这类类型，就必须要使用动态分发了，否则编译器会在编译的时候就报出错误，因为它也不知道这块的类型具体是什么。

结合上面的内容及描述，不难可以发现：

- 特征对象的大小是不固定的，因为所有的对象的类型都只是实现了某个特征，但它们的定义是可能完全不同的
- 几乎问题使用特征对象的引用方式，如 `&dyn Trait` 或是 `Box<dyn Trait>`，这是因为引用的地址指针的大小是固定的，这是编译器喜欢的。
- 与常规引用不同的是，特征对象的引用具有两个指针，一个指针 `ptr` 指向的是具体实现了特征的某个类型的**实例**，一个指向的是一个虚表，这个表记录了所有实现特征的类型，及他们可以调用的属于特征的方法。

需要注意的，当你传入的某个实现了特征的类型实例的时候，这个实例就不会当成该类型的实例了，**而是这个特征对象的实例**，也因此实例只能调用类型实现在特征中的那些方法。

```Rust
struct Dog {
  name: String,
}

struct Cat {}

trait Pet {
  eat(&self) {
    println!("eating");
  }
}

impl Pet for Dog {}
impl Pet for Cat {}

fn get_me_a_pet(name: &str) -> Box<dyn Pet> {
  if (name == "Anz") {
    Dog {
      name: name.to_string(),
    }
  } else {
    Cat {}
  }
}

fn main() {
  const dog = get_me_a_pet("Anz")
  println!(dog.eat()); // eating
  println!(dog.name); // 报错: no field `name` on type `Box<dyn Pet>`
}
```

### 特征对象的安全

并不是所有特征都有特征对象，只有对象安全的特征才行。当一个特征的所有方法都有遵守如下的约束时，它的对象才是安全的：

- 方法的返回类型不能是 `Self`
- 方法没有任何泛型参数

这两个限制是非常必要的，因为只有遵守了上述两条规则，才能做到特征对象的可描述性，不用去在乎实现该特征的具体类型是什么了。
因为在使用特征对象的时候，我们是有意要将其具体的类型给抹去了的，

### 关联类型

这个关联类型与[[Method#关联函数|关联函数]]没有太多的相似之处。
这个关联类型其实就是在**特征定义的语句块中**，申明一个自定义类型，以便特征的方法签名中可以使用该类型：

```Rust
pub trait Iterator {
  type Item;

  fn next(&mut self) -> Option<Self::Item>;
}
```

上述是标准库中实现的，迭代哭的特性 `Iterator`，其有一个 `Item` 的关联类型，用于指代遍历的值的类型。

`next` 方法返回了一个被 `Option` 枚举类型包裹了的 `Item` 类型，如果迭代器中的值是 `i32` 类型，那么调用 `next` 方法就将获取一个 `Option<i32>` 类型的值。

这里的 `Self` 自然就是用来指代当前调用者的具体类型，所以 `Self::Item` 就是用来指代该类型实现中定义的 `Item` 类型。是不是跟关联函数还有些相似。

```Rust
struct Counter<T: Copy> {
  x: T,
}

impl<T: Copy> Iterator for Counter<T> {
  type Item = T; // 这里是必须的，不然后续的 Self::Item 就找不到类型了

  fn next(&mut self) -> Option<Self::Item> {
    Option(self.x)
  }
}
```

这样的好处自然是能增加代码的可读性了，也简化了很多写代码的篇幅。诚然也是可以使用[[Generics|泛型]]代替，但是泛型是有传染性的，你函数体中使用，那么函数签名就必须定义泛型参数，每次调用也都要传入方法中，好生麻烦。

### 默认泛型参数

前面说了可以使用泛型来代替关联类型，就是每次调用方法都要传入一个泛型，比较麻烦。我们其实也可以为某个泛型类型参数指定一个默认的具体类型的。

```Rust
trait Add<RHS=Self> {
  type Output;
  fn add(self, rhs: RHS) -> Self::Output;
}
```

可以看到，`Add` 特征默认是实现两个同类型的值的加法运算，所以其 `RHS` 泛型参数默认值是 `Self`。而如果想实现两个不同类型之间的加法运算，那这个值就不能忽略了。

```Rust
use std::ops::Add;

struct Millimeters(u32);
struct Meters(u32);

impl Add<Meters> for Millimeters {
  type Output = Millimeters;

  fn add(self, rhs: Meters) -> Millimeters {
    Millimeters(self.0 + rhs.0 * 1000)
  }
}
```

使用默认参数可以让我们减少实现时编写的代码，也能无需大幅修改现有代码即可扩展类型。

### 多个特征中同名方法的调用

我们可以在多个特征中定义同名方法，甚至在类型上也可以出现同名的方法：

```Rust
trait Pilot {
  fn fly(&self);
}

trait Wizard {
  fn fly(&self);
}

struct Human;

impl Pilot for Human {
  fn fly(&self) {
    println!("This is your captain speaking.");
  }
}

impl Wizard for Human {
  fn fly(&self) {
    println!("Up!");
  }
}

impl Human {
  fn fly(&self) {
    println!("*waving arms furiously*");
  }
}
```

在这其中，可以通过 `.` 操作符直接访问类型中定义的方法 `fly`。而像 `Pilot` 和 `Wizard` 虽然被类型重写了，但也可以通过某些方式调用到。

```Rust
fn main() {
  let h = Human;
  h.fly(); // *waving arms furiously*
  Wizard::fly(&h); // Up!
  Poilt::fly(&h); // This is your captain speaking.
}
```

能这么实现是因为特征中有 `self` 参数，但如果没有呢？也就是都是关联函数的时候。就需要用到完全限定语法了。

### 完全限定语法

完全限定语法可以让开发者明确地告知 Rust 编译器某个值的类型。其语法是这样的：

```Rust
<Type as Trait>::function(receive_if_method, next_args...);
```

这上面的定义中，第一个参数是方法接收器 `receive`，即方法才拥有的，接收的调用者的 self 参数，而关联函数就没有这个参数了。

```Rust
trait Pilot {
  fn fly() { // 没有 self 参数了，即为特征 Pilog 的关联方法
    println!("This is your captain speaking.")
  }
}
struct Human;

impl Human {
  fn fly() {
    println!("*waving arms furiously*");
  }
}

fn main() {
  Human::fly(); // *waving arms furiously*
  <Human as Pilot>::fly(); // This is your captain speaking.
}
```

众所周知，Rust 编译器是具有类型推导的，这也是我们为什么会很少用到完全限定语法的原因。因为大部分场景下，Rust 编译器都会自动进行类型推导，然后找到方法/函数的调用路径。而只有当存在多个同名方法或函数的时候，并且 Rust 无法区分你想调用的目标函数时，完全限定语法才能派上用场。

### 特征定义中的特征约束

有时候，我们**限定类型实现某个特征之前，会需要类型先实现另外某个特征**，比如小狗 `Dog` 要成为宠物 `Pet` 之前，它必须要先是一个动物 `Animal`（这有点废话）。

```Rust
struct Dog;

trait Animal {
  fn live(&self);
}

trait Pet: Animal { // 意为实现特征 Pet 之前得先实现 Animal 特征
  fn eat(&self) {
    self.live();
    println!("eating");
  }
}
```

这里举的例子有点烂，我其实就是想说明这个主次关系。类型 `Dog` 想实现 `Pet` 之前，得先实现 `Animal` 特征，从实现中也可以看到 `Pet` 特征中调用了 `Animal` 特征的方法，如果只实现 `Pet`，而不实现 `Animal` 那就乱了套了。好在 Rust 有特征约，能约束类型的特征。这也有点像 `Pet` 扩展了 `Animal` 特征一样。这样特征 `Animal` 就被称为 特征 `Pet` 的 `supertrait`，类似于面向对象里的超类，但 Rust 不谈继承，更多的像是一种约束。

### 在外部类型上实现外部特征（newtype）

前面提到了类型实现的限制——[[trait#特征定义与实现的位置（孤儿规则）|孤儿规则]]，简单点说就是，特征或者类型二者之间必须要有一个是在本地，才能在些类型上定义特征。

然而还有一个办法可以用来绕过规则，那就是使用 `newtype` 模式。在编程界里流传着这么一句话：没有什么是封装一层解决不了的，如果有，那就加多一层。假设我们现在想为 `Vec<t>` 实现特征 `Display`，这两位大哥都是在标准库中实现的，也就是开发者们基本是没办法满足孤儿原则。那咋办呢，我们先来看看 Rust 给我们的报错。

```Rust
use std::fmt;

impl<T> fmt::Display for Vec<T> {
  // ...
}
```

运行后直接报错：

```plain
error[E0117]: only traits defined in the current crate can be implemented for arbitrary types
--> src/main.rs:5:1
|
5 | impl<T> std::fmt::Display for Vec<T> {
| ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^------
| |                             |
| |                             Vec is not defined in the current crate
| impl doesn't use only types from inside the current crate
|
= note: define and implement a trait or new type instead
```

`note: define and implement a trait or new type instead`，给出了提示信息，要么我们去到定义里实现这个特征，要么我们使用 `new type`。那回到我们封装的那句至理名言，我们完全可以把 `Vec<T>` 包一层，变为一个新的类型，然后这个类型去实现外部文件定义的特征：

```Rust
use std::fmt;

struct Wrapper(Vec<String>); // 这就是我们包装了一层的 new type

impl fmt::Display for Wrapper {
  fn fmt(&self, f: &mut fmt::Formatter) -> fmt::Result {
    write!(f, "[{}]", self.0.join(", "))
    // 这里 self.0 是因为我们 Wrapper 类型是个元组类型，要去拿里面真正包装了的 Vec<T>
  }
}

fn main() {
  let w = Wrapper(vec![String::from("hello"), String::from("world")]);
  println!("{}", w);
  println!("{:?}", w.0);
  println!("{}", w.0[0]);
}
```

是的，奏系介摸简单。通过封装了一层，产生了一个新的类型，就可以往这个类型里实现特征了，绕开了孤儿原则。但可能有人就要说了，上面的代码太丑了，使用都要通过 `self.0` 这样去用，获取 `Vec` 的子项还要 `w.0[0]`，我的天，丑哭了。
但别急着被这代码劝退，Rust 还提供了一个特征叫 [[Deref]]，只要类型实现了该特征之后，就可以自动做一层类型转换的操作，能够实现为 `Wrapper` 拆箱和装箱的操作，就不用每个操作都要 `self.0` 了。

```Rust
// 从上面的代码里再延伸一下
use std::ops::Deref;

// 使用 Deref 特征解引用
impl Deref for Wrapper {
  type Target = Vec<String>

  fn deref(&self) -> &Self::Target {
    &self.0
  }
}
```

`new type` 的好处还不止于此，如果你不想 `Wrapper` 暴露底层数组的所有方法，还能为 `Wrapper` 去重载这些方法，实现隐藏的目的。
