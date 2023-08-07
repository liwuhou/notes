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
