Rust 中的特征，在其它编程语言中有着另外的一个名字 —— 接口（interface）。在 Rust 中，特征定义了一组可以被共享的行为，只要实现了特征，你就能使用这个行为。像在 [[Generics]] 中提到的，定义了一个泛型参数 `T`，只有 `T` 实现了 `std::ops::Add` 才可以进行合法的加法操作。

```Rust
fn add<T: std::ops::Add<Output = T>>(a: T, b: T) -> T {
  a + b
}
```

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

#### 特征定义与实现的位置（孤儿规则）

在 Rust 中，如果你想要为某个类型实现某个特征，那么**这个类型或者特征必须至少有一个是在当前作用域内定义的**，否则你无法改动到这个类型的特征。例如上面的 `Pet`，我们在 `Pet` 定义的位置，可以为标准库中的 `String` 类型实现 `Pet`，因为在 `Pet` 定义的位置。但我们没办法为 `String` 实现 `Display` 特征。这个规则也被戏称为”孤独规则“， 它能确保其它人不会改动到你的代码，你也不会破坏到不相关的其它代码。

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
