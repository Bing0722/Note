# Control Flow

## 1. 条件语句( `if` 、 `else` )

`if` 语句用于根据条件执行不同的代码块。Rust 中的 `if` 语句条件**必须**是一个 `bool` 类型。

### 1.1. `if` 语句

```rust
fn main() {
    let number = 7;

    if number < 5 {
        println!("Condition is true");
    } else {
        println!("Condition is false");
    }
}
```

### 1.2. `if` 语句中的多个条件

```rust
fn main() {
    let number = 7;

    if number % 4 == 0 {
        println!("Number is divisible by 4");
    } else if number % 3 == 0 {
        println!("Number is divisible by 3");
    } else if number % 2 == 0 {
        println!("Number is divisible by 2");
    } else {
        println!("Number is not divisible by 4, 3, or 2");
    }
}
```

### 1.3. `if` 表达式

```rust
fn main() {
    let condition = true;
    let number = if condition { 5 } else { 6 }; // 两个分支的返回值类型必须相同

    println!("The value of number is: {}", number);
}
```

## 2. 循环语句( `loop` 、 `while` 、 `for` )

### 2.1. `loop` 循环

`loop` 关键字用于创建一个**无限循环**, 可以通过 `break` 语句来退出循环。

```rust
fn main() {
    let mut count = 0;

    loop {
        println!("Count: {}", count);
        count += 1;

        if count == 5 {
            break; // 退出循环
        }
    }
}
```

### 2.2. `while` 循环

`while` 循环会在条件为 `true` 时重复执行代码块。它与 `if` 语句类似，条件必须是布尔值。

```rust
fn main() {
    let mut count = 0;

    while count < 5 {
        println!("Count: {}", count);
        count += 1;
    }
}
```

### 2.3. `for` 循环

`for` 循环用于遍历集合、数组、区间等。Rust 中的 `for` 循环非常强大，它可以直接遍历集合类型（如数组、范围等）。

```rust
// 遍历范围
fn main() {
    for number in 0..5 { // 0..5 是一个范围（Range）
        println!("Number: {}", number);
    }
}

// 遍历数组
fn main() {
    let arr = [10, 20, 30, 40, 50];

    for element in arr.iter() { // 使用 .iter() 遍历数组
        println!("Element: {}", element);
    }
}
```

## 3. match 表达式

`match` 是一个强大的控制流运算符，它允许将一个值与一系列模式进行比较，并根据匹配的模式执行相应的代码。

```rust
fn main() {
    let number = 5;

    match number {
        1 => println!("One"),
        2 => println!("Two"),
        3 => println!("Three"),
        _ => println!("Other"), // _ 通配符，匹配所有情况
    }
}
```

`match` 表达式可以返回一个值，这个值可以被赋给一个变量。

```rust
fn main() {
    let number = 5;

    let text = match number {
        1 => "One",
        2 => "Two",
        3 => "Three",
        _ => "Other",
    };

    println!("Text: {}", text);
}
```

## 4. if let 表达式

`if let` 表达式是一个更简洁的 `match` 表达式，用于匹配单个模式。

```rust
fn main() {
    let some_value = Some(5);

    if let Some(value) = some_value {
        println!("Value: {}", value);
    }
}
```

## 5. while let 表达式

`while let` 表达式是一个更简洁的 `loop` 循环，用于匹配单个模式。

```rust
fn main() {
    let mut values = vec![1, 2, 3, 4, 5];

    while let Some(value) = values.pop() {
        println!("Value: {}", value);
    }
}
```

## 6. 控制流程关键字

Rust 中的控制流程关键字：

- `if`：条件语句
- `else`：条件语句的分支
- `loop`：无限循环
- `while`：条件循环
- `for`：遍历循环
- `match`：模式匹配
- `break`：退出循环
- `continue`：跳过本次循环
- `return`：返回值
- `if let`：简化的 `match` 表达式
- `while let`：简化的 `loop` 循环
- `_`：通配符，匹配所有情况
- `..`：范围运算符
- `=>`：分支操作符
- `@`：绑定符号
- `|`：或操作符
