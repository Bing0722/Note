# error

Rust 将错误分为两类：可恢复错误（Recoverable Errors）和 不可恢复错误（Unrecoverable Errors）。

## 1. 用panic!宏处理不可恢复错误

不可恢复错误是一些不可修复的错误，如：

- 数组访问越界
- 空指针引用
- 除零错误

Rust 提供了 `panic!` 宏来处理不可恢复错误，当程序遇到不可恢复错误时，会调用 `panic!` 宏，打印错误信息并退出程序。

```rust
fn main() {
    panic!("crash and burn");
}
```

## 2. 使用Result<T, E>处理可恢复错误

可恢复错误是一些可以通过返回 `Result<T, E>` 类型来处理的错误，`Result` 枚举有两个变体：`Ok(T)` 和 `Err(E)`。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

** 匹配不同的错误类型 **

```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(ref error) if error.kind() == ErrorKind::NotFound => {
            match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => {
                    panic!("Problem creating the file: {:?}", e)
                },
            }
        },
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

当 `File::open` 返回一个 `Err` 值时，我们检查错误是否是 `ErrorKind::NotFound` 类型，如果是，我们尝试创建文件 `hello.txt`，否则我们会调用 `panic!` 宏。

## 3. 使用Result<T, E>简化错误处理

Rust 提供了一些方法来简化错误处理，如 `unwrap` 和 `expect` 方法。

- `unwrap` 方法：如果 `Result` 是 `Ok`，返回 `Ok` 中的值，否则调用 `panic!` 宏。
- `expect` 方法：与 `unwrap` 方法类似，但是可以指定 `panic!` 的错误信息。

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

## 4. 传播错误

在 Rust 中，可以使用 `match` 表达式来传播错误。

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
```

在 Rust 中，可以使用 `?` 运算符来简化传播错误，当调用一个返回 `Result` 类型的函数时，可以在调用后面加上 `?` 运算符，如果返回 `Ok`，则返回 `Ok` 中的值，否则将错误传播给调用者。

```rust
use std::fs::File;
use std::io;
use std::io::Read;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```

