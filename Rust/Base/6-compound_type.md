# 复合类型

## 1. 元组

元组是一个将多个其他类型的值组合在一起的复合类型。元组的长度是固定的，一旦声明，它的长度不能增加或减少。

```rust
fn main() {
    let tup: (i32, f64, u8) = (500, 6.4, 1); // 元组的类型标注
    
    println!("The value of y is: {}", tup.1); // 访问元组的元素

    let (x, y, z) = tup; // 解构元组
    
    let unit_tup = (); // 空元组

    println!("The value of y is: {}", y);
}
```

## 2. 数组

数组是一个将多个相同类型的值组合在一起的复合类型。数组的长度是固定的，一旦声明，它的长度不能增加或减少。

```rust
fn main() {
    let a = [1, 2, 3, 4, 5];            // 数组的类型标注
    let b: [i32; 5] = [1, 2, 3, 4, 5];  // 数组的类型标注

    let first = a[0];                   // 访问数组的元素

    let c = [3; 5];                     // 重复初始化数组元素

    println!("The first element of a is: {}", first);
}
```
