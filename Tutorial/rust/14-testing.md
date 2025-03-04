# testing

在 Rust 中测试函数是用来验证非测试代码是否是按照期望的方式运行的。

## 1. 编写测试

在 Rust 中，测试函数使用 `#[test]` 属性标记。测试函数通常包含在一个名为 `tests` 的模块中。

在 Rust 中，我们可以使用 `assert!` 宏来验证一个表达式是否为真。如果表达式为假，`assert!` 宏会触发一个 panic。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert_eq!(add(2 + 2), 4);
    }
}
```

在这个例子中，我们定义了一个函数 `add`，它接受两个整数参数，并返回它们的和。然后我们定义了一个测试模块 `tests`，并在其中定义了一个测试函数 `it_works`。测试函数使用 `assert_eq!` 宏来验证 `add` 函数是否按照预期工作。

## 2. 使用 `cargo test` 运行测试

在 Rust 中，我们可以使用 `cargo test` 命令来运行测试。`cargo test` 命令会自动查找项目中的测试函数，并运行它们。

```bash
$ cargo test
```

`cargo test` 命令会输出测试结果，包括测试函数的名称、运行时间、通过或失败等信息。

## 3. 自定义失败信息

在 Rust 中，我们可以使用 `assert!` 宏来自定义失败信息。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    #[test]
    fn it_works() {
        assert!(add(2 + 2) == 4, "add(2, 2) should be 4");
    }
}
```

## 4. 使用 `#[should_panic]` 属性

在 Rust 中，我们可以使用 `#[should_panic]` 属性来测试一个函数是否会触发 panic。

```rust
fn div(a: i32, b: i32) -> i32 {
    if b == 0 {
        panic!("division by zero");
    }
    a / b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    #[should_panic]
    fn test_div() {
        div(1, 0);
    }
}
```

## 5. 使用 Result<T, E> 进行测试

在 Rust 中，我们可以使用 `Result<T, E>` 类型来处理错误。我们可以使用 `assert!` 宏来验证一个 `Result` 是否是 `Ok` 或 `Err`。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() -> Result<(), String> {
        if add(2, 2) == 4 {
            Ok(())
        } else {
            Err(String::from("add(2, 2) should be 4"))
        }
    }
}
```

不能对使用 `Result<T, E>` 类型的测试函数使用 `#[should_panic]` 属性。

不要使用 `Result<T, E>` 值使用问号表达式 `?` 。而是使用 `assert!` 宏来验证 `Result` 是否是 `Ok` 或 `Err`。

## 6. 控制测试如何运行

`cargo test` 命令有一些选项可以控制测试如何运行。

- `--test-threads`：设置测试线程数。
- `--show-output`：显示测试输出。
- `test-name`：只运行指定名称的测试函数, 例如 `cargo test test_name`
- `--ignored`：运行被忽略的测试函数。
    - `--include-ignored`：运行全部的测试函数。

```bash
cargo test --test-threads=1

cargo test --show-output

cargo test test_name

cargo test --ignored

cargo test --include-ignored
```

## 7. 测试的组织结构

Rust 中通常有两种测试组织结构：单元测试和集成测试。

- 单元测试：测试一个模块、函数或方法的行为。
- 集成测试：测试整个程序的行为。

### 7.1. 单元测试

在 Rust 中，单元测试通常包含在与被测试代码相同的文件中。单元测试使用 `#[cfg(test)]` 属性标记。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn it_works() {
        assert_eq!(add(2 + 2), 4);
    }
}
```

通过 `use super::*` 将test模块的所有内容引入到当前模块中。

在编译时，Rust 会忽略 `#[cfg(test)]` 属性标记的代码。

而且，Rust 还允许测试私有函数。

### 7.2. 集成测试

在 Rust 中，集成测试通常包含在 `tests` 目录中。集成测试使用 `#[cfg(test)]` 属性标记。

```rust
// src/lib.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// tests/integration_test.rs
#test
mod tests {
    use mylib::add;

    #[test]
    fn it_works() {
        assert_eq!(add(2 + 2), 4);
    }
}
```

我们并不需要在 `tests` 目录中的测试文件中使用 `#[cfg(test)]` 属性标记，因为 `tests` 目录中的代码只会在运行 `cargo test` 时被编译。

### 7.3. 集成测试中的子模块

在 Rust 中，我们可以在集成测试中使用子模块。

```rust
// src/lib.rs
pub fn add(a: i32, b: i32) -> i32 {
    a + b
}

// tests/common/utils.rs
fn setup() {
    // setup code
}

// tests/integration_test.rs
use mylib::add;
mod utils;

#test
mod tests {

    utils::setup();

    #[test]
    fn it_works() {
        assert_eq!(add(2 + 2), 4);
    }
}
```
