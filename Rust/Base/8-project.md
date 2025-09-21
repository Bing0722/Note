# mangage project

在 Rust 中，包(Package)、Crate 和 模块(Module) 是管理项目的基本单元。

## 1. 包(Package)

包是一个或多个 `crate` 的集合，它包含一个 `Cargo.toml` 文件，用于描述如何构建这些 `crate`。

- 一个包最多可以包含一个库 `crate` (Library Crate)。
- 一个包可以包含多个二进制 `crate` (Binary Crate)。
- 一个包必须至少包含一个 `crate` (库或二进制)。

## 2. Crate

Crate 是一个二进制或库，它可以被其他包引用。

- 库Crate：包含可重用的代码，没有 `main` 函数，入口文件是 `src/lib.rs`。
- 二进制Crate：包含可执行代码，入口文件是 `src/main.rs` 或 `src/bin/*.rs`。

## 3. 模块

模块是用来组织代码的逻辑单元，用于将代码划分成不同的作用域。

默认情况下模块是私有的，可以使用 `pub` 关键字将模块暴露给外部。

- 使用 `mod` 关键字定义模块。
- 使用 `pub` 关键字将模块暴露给外部。
- 使用 `use` 关键字导入模块。

```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

use front_of_house::hosting;

fn main() {
    hosting::add_to_waitlist();
}
```

通常将模块拆分到多个文件中，可以使用 `mod.rs` 文件来组织模块。

```rust
// src/front_of_house/mod.rs
pub mod hosting;

// src/front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```

## 4. 使用外部包

使用 `Cargo.toml` 文件中的 `dependencies` 字段来引入外部包。

```toml
[dependencies]
rand = "0.8.4"
```
