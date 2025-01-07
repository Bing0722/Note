# Control Flow

## 1. 条件语句( `if` 、 `else` )

`if` 语句用于根据条件执行不同的代码块。Rust 中的 `if` 语句条件必须是一个 `bool` 类型。

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
    let number = if condition { 5 } else { 6 };

    println!("The value of number is: {}", number);
}
```

## 2. 循环语句( `loop` 、 `while` 、 `for` )

### 2.1. `loop` 循环

`loop` 关键字用于创建一个无限循环, 可以通过 `break` 语句来退出循环。

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
