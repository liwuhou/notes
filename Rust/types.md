Rust 的类型总的来说分为基本类型和复合类型。基本类型即是 rust 中的最小的原子类型。

### 基本类型

- 数值类型：有符号整数（`i8`, `i16`, `i32`, `i64`， `isize`），无符号整数（`u8`, `u16`, `u32`, `u64`, `usize`）和浮点数（`f32`, `f64`)，以及有理数和复数
- 字符串：字符串字面量和字符串切片 `&str`
- 布尔类型：`true` 和 `false`
- 字符类型：表示单个 Unicode 字符存储为 4 个字节
- 单元类型：即 `()`，其唯一的值也是 `()`

### 类型推导与手动标注

**Rust 会在编译期时，尝试为所有变量的类型加上类型推导**，Rust 编译器很聪明，会自动根据变量的上下文去推导出变量的类型。但也有一些情况下编译器推导不出来，这个时候就会抛出错误，因此需要手动给一个标注。

```rust
let guess = '42'.parse().expect("Not a number!");
```

这段代码将 `'42'` 进行解析，而编译器在这里无法正确推导出正确的类型：整数？浮点数？字符串？因此编译器报qaj

```bash
$ cargo build 
  Compiling no_type_annotations v0.1.0 (file:///projects/no_type_annotations) 
error[E0282]: type annotations needed
 --> src/main.rs:2:9
  | 
2 |     let guess = "42".parse().expect("Not a number!");
  |         ^^^^^ consider giving `guess` a type
```

这里手动为变量增加一个显式的类型标注

```rust
let guess: i32 = '42'.parse();
// or
let guess = '42'.parse::<i32>();
```