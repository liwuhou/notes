`Box` 是 Rust 中使用最为频繁的智能指针。`Box` 能让你**特意地**将数据放置在[[Stack & Heap#Heap 堆内存|堆（`heap`）内存]]中，而不是[[STach & Heap#Stack 栈内存|栈内存]]里。

```Rust
let b = Box<i32>(5); // 存储一个原本是会放在栈内存中的数据类型

println!("{}", b); // 智能指针智能的地方就在能自己解引用。

```

这样“包装”操作的好处除了可以强制地把数据分配在堆上，还能让编译器放弃对类型大小必须在编译时就确定下来的执念。具体是什么意思呢。在[[Trait#函数中返回特征类型]]中，一个函数的返回是一个特征对象，但是如果我们真的通过分支语句返回实现了相同特征但不同类型的数据时，编译器是会报错的，这就是 Rust 编译器的执念。

```Rust
// 结构体 Dog 和 Cat 都实现了特征 Pet
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

通过 `Box` 包装一层就可以蒙混过编译器的关。

```Rust
pnb fn get_me_a_pet(name: &str) -> Box<dyn Pet> {
  if name == "Anz" {
    Box::new(Dog {name: name.to_string()})
  } else {
    Box::new(Cat {name: name.to_string()})
  }
}

fn main() {
  let Anz = get_me_a_pet("Anz");

  println("{}", Anz.sleep()); // 这是 ok 的
  println("{}", Anz.name); // 这不 ok，因为编译器虽然知道这个类型实现了这个特征，但不确定具体是哪个类型，有什么属性
}
```

可见，返回的是一个 `Box<dyn Pet>` 类型，`dyn` 是动态的意思，用于表明返回的对象虽然是动态分发的，但是还是满足 `Pet` 特征的，所以特征中实现的方法都可以使用。同时，动态分发也会阻止编译器有选择的内联方法代码，这也会相应地禁用一些优化。
