# functions

## 1. 函数的基本定义

Rust 使用 `fn` 关键字定义函数，函数的基本语法如下：

```rust
fn function_name(parameter1: Type1, parameter2: Type2) -> ReturnType {
    // 函数体
    return value; // 返回值（可选）
}
```

- `function_name`：函数名, 使用蛇形命名法（snake_case）。
- `parameter1: Type1, parameter2: Type2`：参数列表，每个参数都需要指定类型。
- `ReturnType`：返回值类型，`->` 后面的类型表示函数的返回值类型。
- `return value`：返回值，`return` 关键字用于返回值，可以省略。

## 2. 表达式和语句

Rust 中，**表达式（Expression）** 会返回一个值，而 **语句（Statement）** 不返回值。

函数体的最后一行通常是一个表达式，不需要加分号。如果加了分号，他就变成了语句，函数将返回 `()`(空元组)。

```rust
fn main() {
    let result = calculate();
    println!("The result is {}", result);
}

fn calculate() -> i32 {
    let x = 5;  // 语句, 不返回值
    x + 2       // 表达式, 返回值
}
```

## 3. 函数的返回值

函数的返回值可以是任意类型，甚至可以是另一个函数。

```rust
fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn sub(a: i32, b: i32) -> i32 {
    a - b
}

fn math(op: &str) -> fn(i32, i32) -> i32 {
    match op {
        "add" => add,
        "sub" => sub,
        _ => panic!("Unknown operation"),
    }
}

fn main() {
    let a = 10;
    let b = 5;
    let op = "add";

    let result = math(op)(a, b);
    println!("The result is {}", result);
}
```
