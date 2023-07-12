Rust 中的 Struct 翻译为结构体，是一种灵活的复合型数据结构，在其它的编程语言中往往具有其它的名称，例如 `object`、`dict` 或 `record` 等。

```Rust
struct Person {
  active: bool,
  username: String,
  email: String,
  sign_in_count: u64,
  hobby: String[],
}
```