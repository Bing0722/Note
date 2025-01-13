# collections

Rust 中的集合(Collection)是一种存储多个值的数据结构，它允许我们将多个值组合成一个整体进行管理。

Rust 标准库提供了多种集合类型， 接下来我们将介绍 Rust 中的常用集合类型。

- vector： 允许我们一个接一个地存储一系列相同类型的值。
- 字符串(string)： 一个字符序列。
- hash map： 允许我们将键与值相关联，一个键对应一个值。

## 1. Vector

Vector 是 Rust 中最常用的集合类型，它允许我们一个接一个地存储一系列相同类型的值。

**新建 Vector**

```rust
fn main() {
    let v: Vec<i32> = Vec::new(); //  Vec::new() creates a new empty vector.
    let v = vec![1, 2, 3]; //  vec! macro creates a new vector that holds the values we give it.
}
```

**更新 Vector**

```rust
fn main() {
    let mut v = Vec::new(); // must be mutable to update
    v.push(5); // add a value to the end of the vector
    v.push(6);
}
```

**读取 Vector**

```rust
fn main() {
    let v = vec![1, 2, 3, 4, 5];
    let third: &i32 = &v[2]; // get the third element
    match v.get(2) { // get the third element
        Some(third) => println!("The third element is {}", third),
        None => println!("There is no third element."),
    }
}
```

在 Rust 中，当我们尝试访问一个不存在的元素时，Rust 会引发 panic 并终止程序。为了避免这种情况，我们可以使用 `get` 方法，它返回一个 `Option<&T>`。

**遍历 Vector**

```rust
fn main() {
    let v = vec![100, 32, 57];
    for i in &v {
        println!("{}", i);
    }
}
```

**修改 Vector 中的值**

```rust
fn main() {
    let mut v = vec![100, 32, 57];
    for i in &mut v {
        *i += 50;
    }
}
```

## 2. 字符串(String)

Rust 标准库提供了一个 `String` 类型，它是一个可增长的、可变的、可拥有的 UTF-8 编码的字符串类型。

Rust 核心语言中只有一种字符串类型：字符串 slice `str`，它通常以不可变引用 `&str` 的形式出现，而 `String` 类型由标准库提供，不少Rust核心语言，它是一个可变的、可拥有的、UTF-8 编码的字符串类型。

**新建 String**

使用 `String::new()` 创建一个空字符串。

使用 `to_string()` 方法将字符串字面值转换为 `String` 类型。

使用 `String::from()` 函数也可以从字符串字面值创建一个 `String`。

```rust
fn main() {
    let mut s1 = String::new();
}

fn main() {
    let data = "initial contents";
    let s = data.to_string();
    let s = "initial contents".to_string();

    // use the function String::from can also create a String from a string literal
    let s = String::from("initial contents");
}
```

**更新 String**

更新字符串有 `push_str` 和 `push` 方法。

`push_str` 方法将字符串切片附加到字符串。
`push` 方法将一个字符附加到字符串。

```rust
fn main() {
    let mut s = String::from("foo");
    s.push_str("bar"); // push_str() appends a string slice to a String
    s.push('l'); // push() appends a single character to a String
}
```

**连接字符串**

使用 `+` 运算符或 `format!` 宏连接字符串。

```rust
fn main() {
    let s1 = String::from("Hello, ");
    let s2 = String::from("world!");
    let s3 = s1 + &s2; // s1 has been moved here and can no longer be used
    // let s4 = format!("{}-{}", s1, s2);
}
```

**索引字符串**

Rust 不支持使用索引访问字符串，因为字符串是一个 UTF-8 编码的字节序列，而不是一个字节序列。

Rust 允许我们使用切片语法来获取字符串的一部分。

```rust
fn main() {
    let s = String::from("hello");
    // let h = s[0]; // error: cannot index into a value of type `String`

    let s = String::from("你好");
    let h = &s[0..3]; // slicing by bytes
    let h = &s[0..1]; // error: this range will panic at runtime
}
```

**字符串遍历**

Rust 中的字符串是一个字符序列，我们可以使用 `chars` 方法来遍历字符串中的字符。

也可以使用 `bytes` 方法来遍历字符串中的字节。

```rust
fn main() {
    for c in "你好".chars() {
        println!("{}", c);
    }

    for b in "你好".bytes() {
        println!("{}", b);
    }
}
```

## 3. Hash Map

Hash Map 是一种键值对集合，它允许我们将一个键与一个值相关联。

**新建 Hash Map**

Rust 标准库提供了一个 `HashMap` 类型，它是一个哈希映射，允许我们将键与值相关联。

Rust 中使用 `HashMap::new()` 创建一个空的哈希映射。

使用 `insert` 方法向哈希映射中插入键值对。

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);
}
```

**读取 Hash Map**

使用 `get` 方法获取哈希映射中的值。

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    let team_name = String::from("Blue");
    let score = scores.get(&team_name).copied().unwrap_or(0);
    match score {
        Some(score) => println!("The score of {} is {}", team_name, score),
        None => println!("No score for {}", team_name),
    }
}
```

使用 `get` 方法返回一个 `Option<&V>`，如果键存在则返回 `Some(&value)`，否则返回 `None`。

然后使用 `copied` 方法将 `Option<&V>` 转换为 `Option<V>`，最后使用 `unwrap_or` 方法获取值或默认值。

**遍历 Hash Map**

使用 `iter` 方法遍历哈希映射中的键值对。

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    for (key, value) in &scores {
        println!("{}: {}", key, value);
    }
}
```

**Hash Map 的所有权**

对于 `i32` 这样的实现了Copy trait类型，它们的值会被复制到哈希映射中。

像 `String` 这样的拥有所有权的类型，它们的值会被移动到哈希映射中，哈希映射会获取这些值的所有权。

```rust
use std::collections::HashMap;

fn main() {
    let field_name = String::from("Favorite color");
    let field_value = String::from("Blue");

    let mut map = HashMap::new();
    map.insert(field_name, field_value);

    // field_name and field_value are invalid at this point
}
```

在上面的例子中，`field_name` 和 `field_value` 的所有权被移动到哈希映射中，所以它们在这之后就无效了。

**更新 Hash Map**

在 Rust 中，更新哈希映射的值有两种方式：覆盖和插入。

使用 `insert` 方法可以覆盖哈希映射中的值。

使用 `entry` 方法可以插入一个不存在哈希映射中的值, 如果存在则无影响。

```rust
use std::collections::HashMap;

fn main() {
    let mut scores = HashMap::new();
    scores.insert(String::from("Blue"), 10);
    scores.insert(String::from("Yellow"), 50);

    scores.insert(String::from("Blue"), 25); // overwrite the value

    scores.entry(String::from("Blue")).or_insert(50); // insert the value if the key does not exist
}
```
