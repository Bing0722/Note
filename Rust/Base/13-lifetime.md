# lifetime

## 1. 生命周期(Lifetime)

生命周期是 Rust 中的一种特性，它用于避免悬垂引用(dangling references)。生命周期是一种编译时的概念，它描述了引用的有效范围。

生命周期：确保引用在合适的时间内有效。 引用有效的范围。

每个引用都有生命周期，大多数时候生命周期是隐含的，Rust 会自动推断生命周期。但是在某些情况下，我们需要显式地指定生命周期。

### 1.1. 借用检查器(Borrow Checker)

Rust 的借用检查器会检查引用的生命周期，确保引用在合适的时间内有效。

借用检查器： 确保数据存活时间长于引用的存活时间，比较作用域，以确定所有的借用是否有效。

```rust
fn main() {
    let r;                      // ---------+-- 'a
    {                           //          |
        let x = 5;              // -+-- 'b  |
        r = &x;                 //  |       |
    }                           // -+       |
    println!("r: {}", r);       //          |
}
```

在这个例子中，我们定义了一个引用 `r`，它指向了一个整数 `x`。但是 `x` 的生命周期比 `r` 的生命周期要短，所以这段代码是错误的。

### 1.2. 函数中的泛型生命周期

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result = longest(string1.as_str(), string2);
    println!("The longest string is {}", result);
}

fn longest(x: &str, y: &str) -> &str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

在这个例子中，我们定义了一个函数 `longest`，它接受两个字符串引用，并返回一个字符串引用。但是这段代码是错误的，因为返回的引用的生命周期是不确定的。

### 1.3. 生命周期注解

生命周期注解不会改变引用存活的时间，而是描述多个引用之间的生命周期关系。

生命周期注解的语法：

- 必须以单引号 `'` 开头。
- 通常是小写字母 `'a` `'b` `'c` 等。
- 并且非常简短。
- 位置：紧跟在引用的 `&` 的后边，用空格与引用类型分隔。

```rust
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}
```

在这个例子中，我们为函数 `longest` 添加了生命周期注解 `'a`，它表示 `x` 和 `y` 的生命周期是相同的。

返回的引用的生命周期是两个输入引用的生命周期中较短的那个。

```rust
fn main() {
    let string1 = String::from("abcd");
    let string2 = "xyz";

    let result;
    {
        let string3 = String::from("longest");
        result = longest(string1.as_str(), string3.as_str());
    }
    println!("The longest string is {}", result);
}
```

这是一个错误的例子，因为 `string3` 的生命周期比 `result` 的生命周期要短。

这个例子中，我们定义了一个字符串 `string3`，它的生命周期比 `string1` 和 `string2` 都要短。但是 `result` 的生命周期是等于 `string1` 和 `string3` 的生命周期中较短的那个。

从函数返回一个引用时，返回类型的生命周期参数需要与其中一个参数的生命周期参数相匹配。

### 1.4. struct 中的生命周期

```rust
struct ImportantExcerpt<'a> {
    part: &'a str,
}

fn main() {
    let novel = String::from("Call me Ishmael. Some years ago...");
    let first_sentence = novel.split('.').next().expect("Could not find a '.'");
    let i = ImportantExcerpt { part: first_sentence };
}
```

在这个例子中，我们定义了一个结构体 `ImportantExcerpt`，它包含一个字符串引用 `part`。`ImportantExcerpt` 结构体的生命周期参数 `'a` 表示 `part` 的生命周期。

### 1.5. 生命周期省略规则

Rust 有三条生命周期省略规则：

- 编译器为每一个引用参数都分配一个生命周期参数
- 如果只有一个输入生命周期参数，那么它被赋予所有输出生命周期参数
- 如果有多个输入生命周期参数，其中一个是 `&self` 或 `&mut self`，那么 `self` 的生命周期被赋予所有输出生命周期参数

```rust
fn first_word(s: &str) -> &str {
    let bytes = s.as_bytes();
    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[..i];
        }
    }
    &s[..]
}
```

### 1.6 静态生命周期

`'static` 生命周期表示受影响的引用可以在整个程序的持续时间内存活，所有的字符串字面值都拥有 `'static` 生命周期。

```rust
fn main() {
    let s: &'static str = "I have a static lifetime.";

    println!("{}", s);
}
```

在这个例子中，我们定义了一个字符串引用 `s`，它的生命周期是 `'static`。
