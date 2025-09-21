# Struct

## 1. 定义结构体

结构体是 Rust 中的一种自定义数据类型，它允许你将多个相关的数据组合在一起。

使用 `struct` 关键字定义一个结构体， 结构体的字段需要指定类型和名称。

```rust
struct User {
    username: String,
    email: String,
    sign_in_count: u64,
    active: bool,
}
```

## 2. 创建结构体实例

创建结构体实例时，需要指定每个字段的值。

```rust
fn main() {
    let user1 = User {
        username: String::from("bing"),
        email: String::from("123@example.com"),
        sign_in_count: 1,
        active: true,
        }
}
```

如果结构体实例是可变的(mut)，那么可以修改字段的值。

## 3. 使用结构体更新语法

当你需要创建一个新的结构体实例，但是只想修改其中的一部分字段时，可以使用结构体更新语法。

```rust
fn main() {
    let user1 = User {
        username: String::from("bing"),
        email: String::from("123@example.com"),
        sign_in_count: 1,
        active: true,
    }

    let user2 = User {
        username: String::from("bing"),
        email: String::from("123@example.com"),
        sign_in_count: user1.sign_in_count,
        active: user1.active,
    }

    let user3 = User {
        username: String::from("bing"),
        ..user1,
    }
}
```

上面是结构体更新语法的两种方式，第一种是逐个字段赋值，第二种是使用 `..` 字段初始化简写。

## 4. 元组结构体

元组结构体是一种特殊的结构体，它的字段没有名称，只有类型。

```rust
struct Color(i32, i32, i32);
```

元组结构体 (Tuple Struct) 的每个字段可以通过索引访问。

## 5. 单元结构体

单元结构体 (Unit-like Struct) 是一种特殊的结构体，它没有字段。

```rust
struct Empty;
```

单元结构体通常用于实现特定的 trait。

## 6. 结构体方法

结构体方法是一个关联到结构体的函数，它允许你在结构体上定义自定义的行为。

```rust
impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```

结构体方法的第一个参数通常是 `self`，它代表调用该方法的结构体实例。

## 7. 关联函数

关联函数是一个关联到结构体的函数，它不接受 `self` 参数。

```rust
impl Rectangle {
    fn square(size: u32) -> Rectangle {
        Rectangle { width: size, height: size }
    }
}
```

关联函数通常用于创建结构体实例。

通常, 结构体可以有多个 `impl` 块。
