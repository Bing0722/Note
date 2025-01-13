# cargo

## 1. 进一步认识 Cargo

Cargo 是 Rust 的构建系统和包管理工具。Cargo 能够帮助我们构建、测试和运行 Rust 项目。

### 1.1. 采用发布配置自定义构建

在 Rust 中，Cargo 有两种主要的构建配置：

- `cargo build`：采用 dev 配置，构建一个未优化的二进制文件。
- `cargo build --release`：采用 release 配置，构建一个优化的二进制文件。

```bash
$ cargo build
$ cargo build --release
```

## 2. 将crate发布到crates.io

### 2.1. 编写一个有用的文档注释

在 Rust 中，我们可以使用文档注释来为代码添加注释。文档注释使用 `///` 开头。

````rust
/// Add two numbers together.
///
/// # Examples
///
/// ```
/// let result = add(2, 3);
/// assert_eq!(result, 5);
/// ```
fn add(a: i32, b: i32) -> i32 {
    a + b
}
````

使用 `cargo doc` 命令可以生成文档。

```bash
$ cargo doc
```

### 2.2. 注释包含项的结构

在 Rust 中，我们可以使用 `//!` 注释来注释包含项的结构。

````rust
//! # My Crate
//!
//! `my_crate` is a collection of utilities to make performing certain calculations more convenient.

/// Add two numbers together.
///
/// # Examples
///
/// ```
/// let result = my_crate::add(2, 3);
/// assert_eq!(result, 5);
/// ```
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}
````

## 3. 创建 Cargo.io 账户

> 注：本节内容需要访问 [crates.io](https://crates.io/) 网站。
