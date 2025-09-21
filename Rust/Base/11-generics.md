# generics、trait、lifetime

## 1. 泛型(Generic)

泛型是 Rust 中的一种特性，它允许我们编写出适用于多种类型的代码，而不是针对某一种特定类型。

### 1.1. 定义泛型

在 Rust 中，我们可以使用泛型来定义函数、结构体、枚举、方法等。

#### 1.1.1. 函数

```rust
fn largest<T: PartialOrd>(list: &[T]) -> T {
    let mut largest = list[0];

    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }

    largest
}
```

此处的 `<T: PartialOrd>` 表示泛型 `T` 必须实现 `PartialOrd` trait。

#### 1.1.2. 结构体

```rust
pub struct Point<T> {
    x: T,
    y: T,
}
```

#### 1.1.3. 枚举

```rust
enum Option<T> {
    Some(T),
    None,
}
```

#### 1.1.4. 方法

```rust
impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}
```

此处的 `impl` 语句表示为 `Point<T>` 结构体实现方法。必须在 `impl` 语句中指定泛型 `T`。

还可以为 `Point<T>` 结构体实现方法，如下：

```rust
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

但是两个同名的方法不能同时存在，因为 Rust 不支持泛型类型的方法重载。

结构体中的泛型类型参数不一定要与 `impl` 语句中的泛型类型参数一致。

```rust
impl<X1, Y1> Point<X1, Y1> {
    fn mixup<X2, Y2>(self, other: Point<X2, Y2>) -> Point<X1, Y2> {
        Point {
            x: self.x,
            y: other.y,
        }
    }
}
```

## 2. 泛型代码的性能

Rust 中的泛型代码是通过**单态化**（monomorphization）来实现的，即编译器会根据泛型代码的具体类型生成实际的代码。

这样做的好处是，泛型代码的性能和具体类型的代码一样高效。

所以，Rust 中的泛型代码不会因为使用泛型而导致性能下降。
