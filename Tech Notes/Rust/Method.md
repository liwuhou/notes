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
