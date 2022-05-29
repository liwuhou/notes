Rust 中的数值类型有两大类：整数和浮点数。

### 整数类型

整数是没有小数部分的数字，在 Rust 中，整数还分为有符号类整数和无符号类整数。

| 长度       | 有符号类型 | 无符号类型 |
| ---------- | ---------- | ---------- |
| 8 位       | `i8`       | `u8`       |
| 16 位      | `i16`      | `u16`      |
| 32 位      | `i32`      | `u32`      |
| 64 位      | `i64`      | `u64`      |
| 128 位     | `i128`     | `u128`     |
| 视架构而定 | `isize`    | `usize`    | 

观察上述可知，类型定义的形式统一为：`有无符号 + 类型大小（位数）`。