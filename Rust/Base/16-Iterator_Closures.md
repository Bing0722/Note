# Iterator and Closures

## 1. 闭包(Closures)

闭包是一种可以捕获环境中变量的匿名函数。闭包可以捕获三种环境变量：`FnOnce`、`FnMut` 和 `Fn`。

### 1.1. 闭包语法

```rust
fn main() {
    let add_one = |x| x + 1;

    let result = add_one(1);
    println!("The result is {}", result);
}
```

在这个例子中，我们定义了一个闭包 `add_one`，它接受一个参数 `x`，并返回 `x + 1`。

```rust
fn add_one_v1    (x: u32) -> u32 { x + 1 }
let add_one_v2 = |x: u32| -> u32 { x + 1 };
let add_one_v3 = |x|             { x + 1 };
let add_one_v4 = |x|               x + 1 ;
```

上述代码中的 `add_one_v1`、`add_one_v2`、`add_one_v3` 和 `add_one_v4` 是等价的。

第一个是函数定义，第二个是完整标注的闭包定义，第三个是省略了参数类型标注的闭包定义，第四个是省略了参数类型标注和返回值类型标注的闭包定义。

### 1.2. 闭包捕获环境变量

#### 1.2.1. 捕获不可变引用

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    let only_borrows = || println!("From closure: {list:?}");

    println!("Before calling closure: {list:?}");
    only_borrows();
    println!("After calling closure: {list:?}");
}
```

#### 1.2.2. 捕获可变引用

```rust
fn main() {
    let mut list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    let mut_borrows = || {
        list.push(4);
    };

    println!("Before calling closure: {list:?}");
    mut_borrows();
    println!("After calling closure: {list:?}");
}
```

#### 1.2.3. 移动所有权

```rust
fn main() {
    let list = vec![1, 2, 3];
    println!("Before defining closure: {list:?}");

    thread::spawn(move || println!("From thread: {list:?}"))
        .join()
        .unwrap();

    // error: value borrowed here after move
    // println!("After calling closure: {list:?}");
}
```

### 1.3. 闭包的Fn trait

- `FnOnce`：闭包可以被调用一次。
- `FnMut`：闭包可以被多次调用。
- `Fn`：闭包可以被多次调用，但不会改变捕获的环境变量。

```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
fn main() {
    let mut list = [
        Rectangle { width: 10, height: 1 },
        Rectangle { width: 3, height: 5 },
        Rectangle { width: 7, height: 12 },
    ];
    list.sort_by_key(|r| r.width);
    println!("{list:#?}");
}
```

sort_by_key 被定义为接收一个 FnMut 闭包的原因是它会多次调用这个闭包: 对 slice 中的每个元素调用一次

## 2. 迭代器(Iterators)

迭代器是一种用于在集合（如数组、向量等）上进行逐元素操作的抽象。Rust 的迭代器实现遵循惰性计算原则，即只有在需要时才会执行迭代操作。

迭代器的核心特性：

- `Iterator` trait：定义了迭代器的行为。
    - `next` 方法：返回一个 `Option` 类型的值，表示迭代器的下一个元素。
- `IntoIterator` trait：定义了如何将一个集合转换为迭代器。
    - `into_iter` 方法：返回一个迭代器。

### 2.1. 使用迭代器

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    println!("{v2:?}");
}
```

### 2.2. 迭代器适配器

迭代器适配器是一种用于对迭代器进行转换的方法，它们可以将一个迭代器转换为另一个迭代器。

- `map`：对迭代器中的每个元素应用一个函数。
- `filter`：对迭代器中的每个元素应用一个过滤器。
- `collect`：将迭代器转换为集合。
- `fold`：对迭代器中的每个元素应用一个累加器。
- `sum`：对迭代器中的每个元素求和。

```rust
fn main() {
    let v1 = vec![1, 2, 3];
    let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();
    println!("{v2:?}");
}
```

### 2.3. 迭代器消费适配器

迭代器消费适配器是一种用于对迭代器进行消费的方法，它们会消耗迭代器中的元素。

```rust
#[cfg(test)]
mod tests {
    #[test]
    fn iterator_sum() {
        let v1 = vec![1, 2, 3];
        let v1_iter = v1.iter();
        let total: i32 = v1_iter.sum();
        assert_eq!(total, 6);
    }
}
```
