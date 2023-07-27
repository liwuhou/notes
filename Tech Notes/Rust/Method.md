这里的方法（Method），并不是指函数。而是类、对象中的方法。

```Rust
object.method(); // . 操作符调用对象中的方法
```

在 Rust 中，方法往往跟 [[Struct|结构体]]、[[Enum|枚举]]、[[Trait|特征]]一起使用。

Rust 中，使用 `impl` (_implementation_,实现) 来定义一个对象的方法，这点与传统语言很不一样。

![](http://cdn.liwuhou.cn/tmp/20230725235151.png)

```Rust
// Rust 中
// 定义 Circle 的属性
struct Circle {
  x: f64,
  y: f64,
  radius: f64,
}

// 定义 Circle 的方法/行为
impl Circle {
  // new 是 Circle 的关联函数，因为它的第一个参数不是 self，且 new 并不是关键字
  // 基本用来初始化当前结构体的实例
  fn new(x: f64, y: f64, radius: f64) -> Circle {
    Circle {
      x,
      y,
      radius,
    }
  }

  // Circle 的方法，&self 表示借用当前的 Circle 结构体
  fn area(&self) -> f64 {
    std::f64::consts::PI * (self.radius * self.radius)
  }
}

let c = Circle::new(0.0, 0.0, 0.5);
println("this circle's area is {}", c.area());

```

ts/es6

```ts
class Circle {
  // 定义属性
  x: number
  y: number
  radius: number

  constructor(x: number, y: number, radius: number) {
    // 初始化
    this.x = x
    this.y = y
    this.radius = radius
  }

  // 方法
  area(): number {
    return Math.PI * this.radius ** 2
  }
}
```

es5……

```js
function Circle(x, y, radius) {
  // 属性
  this.x = x
  this.y = y
  this.radius = radius
}

// js：我定义方法也是分开的呀
Circle.prototype.area = function () {
  return Math.PI * this.radius * this.radius
}
```

### self, &self, &mut self

在对象的方法中操作对象自身，由于方法的本质还是函数，故方法参数仍受所有权法则约束。下面是几种 `self` 的引用：

- `self`：其实是 `self: Self` 的简写，表示将对象的所有权转移到该方法中，这种形式只能说有，但是用得很少
- `&self`： 其实是 `self: &Self` 的简写，表示对对象的[[References & Borrowing#不可变引用|不可变引用]]
- `&mut self`：没错，其实也是 `self: &mut Self` 的简写，表示[[References & Borrowing#可变借用|可变借用]]

回到上面的 Rust 代码的例子中，`area()` 方法只是访问结构体中的数据，并不想去改变它，所以使用了 `&self` 不可变借用。如果方法需要去改变当前的结构体，则需把第一个参数改为 `&mut self`即可。而直接使用 `self` 的情况其实很少见，一般这么用也往往只是想把当前的对象转成另外的一个对象，并且在转换完成之后，也不需要关注之前的对象，防止之前的对象被错误的调用。

### 关联函数

rust 中的关联函数，这个其实跟 ts 中的类的静态方法有异曲同工之妙，但又有稍许不同。在 Rust 中，只要结构体中的方法的参数不包含 `self`，那这个“东西”就不是方法了，而是一个函数。也就是说不能通过 `对象.方法()` 的形式调用。而是通过 `类::关联函数()` 来使用。例如上述示例中的 `Circle::new()` 关联函数。

```Rust
impl Circle {
  fn new(w: u32, h: u32) -> Circle | Self { // 这里 Self 和 Circle 都是等价的
    Circle {
      width: w,
      height: h,
    }
  }

  fn area(&self) {
    return std::f64::consts::PI * self.radius * self.radius
  }
}
```

因为是函数，所以不能用 `.` 的方式来调用，只能通过 `::` 来调用。这个“方法”位于结构体的命名空间中，`::` 语法用于关联函数和模块创建的命名空间。而 `area()` 方法，声明了 `self` 参数，所以他就是在对象实例上的，与对象绑定的，可以直接通过 `对象.方法()` 用。

> 在 Rust 中有一个约定俗成的规则，使用 `new` 来作为构造器的名称，出于设计上的考虑，Rust 特地没有用 `new` 来作为关键字。

### 多个 `impl` 定义

Rust 支持为结构体等实体定义多个 `impl` 块，提供了多种组织代码的灵活性，当然如果没有规范，也会显得杂乱，代码松散。

### 为枚举实现方法

Rust 不仅能为 [[Struct|struct]] 实现方法，也能为 [[Enum|enum]] 和 [[Trait|trait]] 实现方法。

```rust
enum Message {
  Quit,
  Move { x: i32, y: i32 },
  Write(i32),
  ChangeColor(i32, i32, i32),
}
impl Message {
  fn call(&self) {
    match *self {
      Message::Quit => println!("Quit"),
      Message::Move { x, y } => println!("Move {{{}, {}}}", x, y),
      Message::Write(str) => println!("Write {}", str),
      Message::ChangeColor(r, g, b) => println!("ChangeColor rgb({}, {}, {})", r, g, b),
    }
  }
}

fn main() {
  let m = Message::Move { x: 1, y: 0 };
  m.call(); // Move {1, 0}
}

```
