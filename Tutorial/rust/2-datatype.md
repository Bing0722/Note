# datatype

## 1. 标量类型 (Scalar Types)

标量类型代表一个单独的值。Rust 有四种基本的标量类型：

- 整型 (Integer)
- 浮点型 (Floating-Point)
- 布尔型 (Boolean)
- 字符型 (Character)

### 1.1 整型 (Integer)

| 长度    | 有符号(i) | 无符号(u) |
| ------- | --------- | --------- |
| 8-bit   | i8        | u8        |
| 16-bit  | i16       | u16       |
| 32-bit  | i32       | u32       |
| 64-bit  | i64       | u64       |
| 128-bit | i128      | u128      |
| arch    | isize     | usize     |

- `isize` 和 `usize` 类型依赖运行程序的计算机架构：64 位架构上它们是 64 位的，32 位架构上它们是 32 位的。
- 默认类型是 `i32`。

```rust
let a: i32 = 42;    // 有符号32位整型
let b: u64 = 100;   // 无符号64位整型
```

### 1.2 浮点型 (Floating-Point)

Rust 有两种浮点数类型：

- `f32`：32位单精度浮点数
- `f64`：64位双精度浮点数

默认类型是 `f64`。

```rust
let x: f32 = 3.14;  // 单精度浮点数
let y: f64 = 2.718; // 双精度浮点数
```

### 1.3 布尔型 (Boolean)

布尔类型有两个值：`true` 和 `false`, 类型为 `bool`。

```rust
let is_true: bool = true;
let is_false = false; // 类型推导
```

### 1.4 字符型 (Character)

字符类型代表单个 Unicode 字符，类型为 `char`, 占用 4 个字节。

```rust
let c: char = 'A';
let emoji = '🦀'; // Unicode 字符
```

## 2. 复合类型 (Compound Types)

复合类型可以将多个值组合成一个类型。Rust 有两种基本的复合类型：

- 元组 (Tuple)
- 数组 (Array)

### 2.1 元组 (Tuple)

元组是一个将多个不同类型的值组合在一起的类型。元组的长度是固定的，一旦声明后就不能改变。

```rust
let tup: (i32, f64, char) = (42, 3.14, 'A');    // 申明一个元组
let (x, y, z) = tup;                            // 解构元组
let first = tup.0;                              // 访问元组元素
```

### 2.2 数组 (Array)

数组是一个将多个相同类型的值组合在一起的类型。数组的长度是固定的，且存储在栈上。

```rust
let arr: [i32; 3] = [1, 2, 3];                  // 申明一个数组
let first = arr[0];                             // 访问数组元素
let arr = [0; 5];                               // 申明一个包含5个元素的数组，每个元素的值为0
```
