# enums

## 1. 定义和实例枚举

Rust 中使用 `enum` 关键字来定义枚举，每个变体可以有不同的类型和数量。

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}

fn main() {
    let up = Direction::Up;
    let quit = Message::Quit;
    let write = Message::Write(String::from("Hello"));
}
```

枚举的值可以通过 `::` 语法来访问。

同样，枚举也可以定义方法。

## 2. 模式匹配(Match)

Rust 中的 `match` 是一个强大的控制流运算符，它允许你将一个值与一系列模式进行比较，并根据匹配的模式执行相应的代码。

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn move_player(direction: Direction) {
    match direction {
        Direction::Up => println!("Move up"),
        Direction::Down => println!("Move down"),
        Direction::Left => println!("Move left"),
        Direction::Right => println!("Move right"),
    }
}

fn move_player2(direction: Direction) -> &str {
    match direction {
        Direction::Up => "Move up",
        _ => "Move in other direction",
    }
}
```

使用 `_` 通配符可以匹配所有其他情况。

## 3. Option 枚举

Rust 标准库中有一个非常常用的枚举类型 `Option`，它表示一个值可能存在，也可能不存在。

```rust
enum Option<T> {
    Some(T),
    None,
}

fn divide(x: f64, y: f64) -> Option<f64> {
    if y == 0.0 {
        None
    } else {
        Some(x / y)
    }
}

fn main() {
    let result = divide(10.0, 2.0);
    match result {
        Some(x) => println!("Result: {}", x),
        None => println!("Cannot divide by zero"),
    }
}
```

## 4. Result 枚举

`Result` 枚举用于处理可能出现错误的情况，它有两个变体：`Ok` 和 `Err`。

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

fn divide(x: f64, y: f64) -> Result<f64, String> {
    if y == 0.0 {
        Err(String::from("Cannot divide by zero"))
    } else {
        Ok(x / y)
    }
}

fn main() {
    let result = divide(10.0, 0.0);
    match result {
        Ok(x) => println!("Result: {}", x),
        Err(e) => println!("Error: {}", e),
    }
}
```

## 5. if let

`if let` 语法是 `match` 的一个简化版本，它允许你只处理一个模式。

```rust
enum Direction {
    Up,
    Down,
    Left,
    Right,
}

fn move_player(direction: Direction) {
    if let Direction::Up = direction {
        println!("Move up");
    } else {
        println!("Move in other direction");
    }
}
```
