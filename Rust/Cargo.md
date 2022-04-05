Cargo 是 Rust 中的包管理器，相当于 Nodejs 中的 npm。一个好的管理工具非常的重要，Cargo 不仅提供了一系列的依赖，还提供了从项目的建立、构建和测试、运行直到部署的全部能力。

### 创建一个项目

```bash
cargo new [project_name]
```

直接用 `cargo new` 就可以创建一个项目，项目名是`project_name`，同时在命令行所在目录下也会有一个同名的项目文件夹。

`cargo new` 命令还支持两个参数，分别是 `--lib` 和 `--bin`，前者是创建一个依赖库项目，后者是一个可运行的项目。默认是 `--bin` 参数。

### 运行项目

```bash
cargo run
# or
cargo r
```

运行 `cargo run` 命令，rust 首先会检测下代码，接着运行编译，然后再执行一遍编译而成的二进制文件。也就是说，`cargo run` 等于：

```bash
cargo run 
# 等价于
cargo build
# 执行二进制文件
./target/debug/[project_name]
```

在运行和编译之前，都会对项目中的代码进行一遍检测和验证，如果项目经过迭代之后变得庞大，每次编译都要等待很长一段时间，那么有没有办法能快速检测代码是否通过，而不编译项目呢？答案是有的，就是 `cargo check`

### 检测代码

```bash
cargo check
# or
cargo c
```

`cargo check` 只是快速检查一下代码能否通过编译，而不会花时间去真正地编译，因此这个命令的速度非常的快，能节省大量的时间。

### 编译代码
在开发的时候，为了能让开发者快速地验证应用，cargo 采用了一种编译速度很快但应用运行速度较慢的措施。在项目打包部署的时候，如果想要高性能的 `release` 程序，cargo 也提供的方法：

```bash 
cargo build --release
# or
cargo build -r
```

这样虽然编译的速度慢了一点，但是运行的速度很快。

### Cargo.toml 和 Cargo.lock

`Cargo.toml` 和 `Cargo.lock` 是 cargo 的核心文件，cargo 的所有活动都基于这两个文件。

- `Cargo.toml` 是 cargo 特有的**项目数据描述文件**。存储了项目中所有的元配置信息，规定了项目如何构建、测试和运行。
- `Cargo.lock` 文件是 cargo 工具根据同一项目的 `toml` 文件生成的**项目依赖详细清单**，一般不用手动去修改它，相当于 node 中的 `package-lock.json` 文件。

#### Cargo.toml 中的 `package` 段落

`package` 中记录了当前项目的描述信息。

```toml
[package]
name = "project_name"
version = "0.1.0"
edition = "2021"
```

`name` 字段定义了项目的名称，`version`字段定义了项目的版本号，新项目默认是 `0.1.0`，`edition` 字段则是定义了 Rust 使用的大版本，如果该字段不存在，则默认为最新的大版本以提供向后兼容性。

#### Cargo.toml 中的`dependencies` 段落

`dependencies` 用来记录当前 rust 项目中使用的依赖，能够对项目中的各种依赖项进行方便、统一且灵活的管理。

```toml
[dependencies]
rand = "0.3" 
hammer = { version = "0.5.0"} 
color = { git = "https://github.com/bjz/color-rs" } 
geometry = { path = "crates/geometry" }
```

引入依赖的方式有几种：

- 引入 [crates.io](https://crates.io/) 中的依赖，只需一个包名和版本号即可，例如 `rand = "0.3"`，顺便一提，`0.3`字符串是一个 [`semver`](https://semver.org/) 格式的版本号，这点也跟 npm 相同。
- 从其它注册服务引入依赖包，`time = { registry = "https://mirrors.ustc.deu.cn/creates.io-index/" }`，还可以使用 `registries` 来指定镜像。
- 直接引入 git 仓库作为依赖包 `regex = { git = "https://github.com/rust-lang/regex" }
- 通过路径引入依赖 `hello_utils = { path = "./hello_utils" }


还有多种方式混合引入、根据不同的平台引入等，[详细看文档]([指定依赖项 - Rust语言圣经(Rust Course)](https://course.rs/cargo/reference/specify-deps.html))

