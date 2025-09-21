# Rust学习总结 - 从C++到Rust

## 1. Rust语言概述

### 1.1 设计理念

- **内存安全**：在编译时防止内存泄漏、悬垂指针、缓冲区溢出
- **零成本抽象**：高级特性不影响运行时性能
- **并发安全**：编译时防止数据竞争
- **系统级编程**：可直接操作硬件，无垃圾回收器

### 1.2 与C++的主要区别

- **所有权系统**：替代手动内存管理
- **模式匹配**：比C++的switch更强大
- **trait系统**：类似C++概念但更灵活
- **宏系统**：比C++宏更安全和强大
- **错误处理**：Result类型替代异常

## 2. 基础语法

### 2.1 变量与常量

```rust
// 不可变变量（默认）
let x = 5;
// 可变变量
let mut y = 10;
y = 15;

// 常量
const MAX_SIZE: u32 = 100_000;

// 变量遮蔽（shadowing）
let x = "hello"; // 可以改变类型
```

### 2.2 数据类型

#### 标量类型

```rust
// 整数类型
let a: i32 = -42;      // 有符号32位
let b: u64 = 100;      // 无符号64位
let c = 0xff;          // 十六进制
let d = 0b1010;        // 二进制

// 浮点类型
let x: f32 = 3.14;
let y: f64 = 2.718;    // 默认类型

// 布尔类型
let is_active: bool = true;

// 字符类型（Unicode）
let heart = '❤';
```

#### 复合类型

```rust
// 元组
let tuple: (i32, f64, char) = (42, 3.14, 'x');
let (x, y, z) = tuple; // 解构

// 数组
let arr: [i32; 5] = [1, 2, 3, 4, 5];
let zeros = [0; 10]; // 10个0
```

### 2.3 函数

```rust
fn main() {
    println!("Hello, world!");
    let result = add(5, 3);
}

fn add(x: i32, y: i32) -> i32 {
    x + y // 表达式，无分号
}

// 发散函数（never type）
fn panic_fn() -> ! {
    panic!("This function never returns");
}
```

## 3. 所有权系统（Ownership）

### 3.1 核心概念

1. **每个值都有一个所有者**
2. **同时只能有一个所有者**
3. **所有者离开作用域时，值被丢弃**

```rust
fn ownership_example() {
    let s1 = String::from("hello"); // s1拥有字符串
    let s2 = s1;                    // 所有权转移给s2
    // println!("{}", s1);          // 错误！s1已失效
    println!("{}", s2);             // 正确
}
```

### 3.2 借用（Borrowing）

```rust
fn borrowing_example() {
    let s1 = String::from("hello");
    let len = calculate_length(&s1); // 借用，不转移所有权
    println!("'{}' has length {}", s1, len); // s1仍然有效
}

fn calculate_length(s: &String) -> usize {
    s.len()
} // s离开作用域，但因为是借用，不会释放内存
```

### 3.3 可变借用

```rust
fn mutable_borrow() {
    let mut s = String::from("hello");
    change_string(&mut s);
    println!("{}", s); // "hello, world"
}

fn change_string(s: &mut String) {
    s.push_str(", world");
}
```

### 3.4 借用规则

- 可以有多个不可变借用
- 只能有一个可变借用
- 不能同时有可变和不可变借用
- 借用必须在所有者生命周期内有效

### 3.5 生命周期（Lifetimes）

```rust
// 生命周期注解
fn longest<'a>(x: &'a str, y: &'a str) -> &'a str {
    if x.len() > y.len() {
        x
    } else {
        y
    }
}

// 结构体中的生命周期
struct ImportantExcerpt<'a> {
    part: &'a str,
}
```

## 4. 结构体与枚举

### 4.1 结构体

```rust
// 普通结构体
struct User {
    username: String,
    email: String,
    age: u32,
    active: bool,
}

// 元组结构体
struct Color(i32, i32, i32);

// 单元结构体
struct AlwaysEqual;

impl User {
    // 关联函数（类似静态方法）
    fn new(username: String, email: String, age: u32) -> User {
        User {
            username,
            email,
            age,
            active: true,
        }
    }
    
    // 方法
    fn is_adult(&self) -> bool {
        self.age >= 18
    }
    
    // 可变方法
    fn set_age(&mut self, age: u32) {
        self.age = age;
    }
}
```

### 4.2 枚举

```rust
// 基础枚举
enum IpAddrKind {
    V4,
    V6,
}

// 带数据的枚举
enum IpAddr {
    V4(u8, u8, u8, u8),
    V6(String),
}

// Option枚举（内置）
enum Option<T> {
    Some(T),
    None,
}

// Result枚举（内置）
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 4.3 模式匹配

```rust
fn match_example(ip: IpAddr) {
    match ip {
        IpAddr::V4(a, b, c, d) => {
            println!("IPv4: {}.{}.{}.{}", a, b, c, d);
        }
        IpAddr::V6(addr) => {
            println!("IPv6: {}", addr);
        }
    }
}

// if let 简化匹配
fn if_let_example(config_max: Option<u8>) {
    if let Some(max) = config_max {
        println!("Max is {}", max);
    }
}
```

## 5. 集合类型

### 5.1 Vector

```rust
// 创建和使用Vec
let mut v: Vec<i32> = Vec::new();
v.push(1);
v.push(2);
v.push(3);

// 宏创建
let v = vec![1, 2, 3];

// 访问元素
let third = &v[2];                    // 可能panic
let third = v.get(2);                 // 返回Option<&T>

// 遍历
for i in &v {
    println!("{}", i);
}

// 可变遍历
for i in &mut v {
    *i += 50;
}
```

### 5.2 HashMap

```rust
use std::collections::HashMap;

let mut scores = HashMap::new();
scores.insert(String::from("Blue"), 10);
scores.insert(String::from("Red"), 50);

// 从Vec创建
let teams = vec![String::from("Blue"), String::from("Yellow")];
let initial_scores = vec![10, 50];
let scores: HashMap<_, _> = teams.iter().zip(initial_scores.iter()).collect();

// 获取值
let score = scores.get("Blue"); // Option<&V>

// 遍历
for (key, value) in &scores {
    println!("{}: {}", key, value);
}
```

### 5.3 字符串

```rust
// 创建字符串
let mut s = String::new();
let s = "initial contents".to_string();
let s = String::from("initial contents");

// 更新字符串
s.push_str("bar");
s.push('!');

// 连接字符串
let s1 = String::from("Hello, ");
let s2 = String::from("world!");
let s3 = s1 + &s2; // s1不再有效

// format!宏
let s = format!("{}-{}", s2, s3);

// 字符串切片
let hello = &s[0..5];
```

## 6. 错误处理

### 6.1 不可恢复错误 - panic!

```rust
fn panic_example() {
    panic!("crash and burn");
}

// 数组越界也会panic
let v = vec![1, 2, 3];
v[99]; // panic!
```

### 6.2 可恢复错误 - Result

```rust
use std::fs::File;
use std::io::ErrorKind;

fn result_example() {
    let f = File::open("hello.txt");
    
    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}

// 简化版本
fn simplified_result() -> Result<File, Box<dyn std::error::Error>> {
    let f = File::open("hello.txt")?; // ? 操作符
    Ok(f)
}
```

## 7. 泛型、Trait和生命周期

### 7.1 泛型

```rust
// 泛型函数
fn largest<T: PartialOrd + Copy>(list: &[T]) -> T {
    let mut largest = list[0];
    for &item in list.iter() {
        if item > largest {
            largest = item;
        }
    }
    largest
}

// 泛型结构体
struct Point<T> {
    x: T,
    y: T,
}

impl<T> Point<T> {
    fn x(&self) -> &T {
        &self.x
    }
}

// 特定类型实现
impl Point<f32> {
    fn distance_from_origin(&self) -> f32 {
        (self.x.powi(2) + self.y.powi(2)).sqrt()
    }
}
```

### 7.2 Trait（特性）

```rust
// 定义trait
pub trait Summary {
    fn summarize(&self) -> String;
    
    // 默认实现
    fn summarize_author(&self) -> String {
        String::from("(Read more...)")
    }
}

// 实现trait
pub struct NewsArticle {
    pub headline: String,
    pub content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {}", self.headline, self.summarize_author())
    }
}

// trait作为参数
pub fn notify(item: impl Summary) {
    println!("Breaking news! {}", item.summarize());
}

// trait bound语法
pub fn notify<T: Summary>(item: T) {
    println!("Breaking news! {}", item.summarize());
}

// 多个trait bound
pub fn notify<T: Summary + Display>(item: T) {
    // ...
}
```

### 7.3 重要的内置Trait

```rust
// Clone trait
#[derive(Clone)]
struct MyStruct {
    data: String,
}

// Copy trait（仅用于栈上数据）
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

// Debug trait
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}
```

## 8. 模块系统

### 8.1 包和Crate

```rust
// Cargo.toml
[package]
name = "my_project"
version = "0.1.0"
edition = "2021"

[dependencies]
rand = "0.8"
```

### 8.2 模块定义

```rust
// lib.rs or main.rs
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
        fn seat_at_table() {}
    }
    
    mod serving {
        fn take_order() {}
        fn serve_order() {}
        fn take_payment() {}
    }
}

// 使用模块
pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();
    
    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}

// use关键字
use crate::front_of_house::hosting;
// 或
use crate::front_of_house::hosting::add_to_waitlist;
```

### 8.3 将模块分离到文件

```rust
// lib.rs
mod front_of_house;

// front_of_house.rs
pub mod hosting;

// front_of_house/hosting.rs
pub fn add_to_waitlist() {}
```

## 9. 集合进阶

### 9.1 迭代器

```rust
let v1 = vec![1, 2, 3];

// 迭代器适配器
let v2: Vec<_> = v1.iter().map(|x| x + 1).collect();

// 过滤
let filtered: Vec<_> = v1.iter().filter(|&&x| x > 1).collect();

// 查找
let found = v1.iter().find(|&&x| x > 2);

// 折叠
let sum = v1.iter().fold(0, |acc, &x| acc + x);
let sum2 = v1.iter().sum::<i32>();

// 自定义迭代器
struct Counter {
    current: usize,
    max: usize,
}

impl Counter {
    fn new(max: usize) -> Counter {
        Counter { current: 0, max }
    }
}

impl Iterator for Counter {
    type Item = usize;
    
    fn next(&mut self) -> Option<Self::Item> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Some(current)
        } else {
            None
        }
    }
}
```

### 9.2 闭包

```rust
// 基本闭包
let add_one = |x| x + 1;
let result = add_one(5);

// 捕获环境变量
let x = 4;
let equal_to_x = |z| z == x;

// 闭包类型
// Fn - 不可变借用
// FnMut - 可变借用  
// FnOnce - 获取所有权

fn call_with_different_closures() {
    let mut s = String::from("hello");
    
    // FnOnce
    let consume = move || {
        println!("{}", s);
        s
    };
    
    // FnMut
    let mut change = |new_str| {
        s = new_str;
    };
    
    // Fn
    let borrow = || {
        println!("{}", s);
    };
}
```

## 10. 智能指针

### 10.1 Box<T>

```rust
// 在堆上存储数据
let b = Box::new(5);
println!("b = {}", b);

// 递归类型
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};
let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil))))));
```

### 10.2 Rc<T> - 引用计数智能指针

```rust
use std::rc::Rc;

let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
let b = Cons(3, Rc::clone(&a));
let c = Cons(4, Rc::clone(&a));

println!("count after creating c = {}", Rc::strong_count(&a));
```

### 10.3 RefCell<T> - 内部可变性

```rust
use std::cell::RefCell;
use std::rc::Rc;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

let value = Rc::new(RefCell::new(5));
let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));

*value.borrow_mut() += 10;
```

## 11. 并发

### 11.1 线程

```rust
use std::thread;
use std::time::Duration;

fn basic_threading() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread!", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread!", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}

// 移动闭包
fn move_closure() {
    let v = vec![1, 2, 3];
    
    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });
    
    handle.join().unwrap();
}
```

### 11.2 消息传递

```rust
use std::sync::mpsc;
use std::thread;

fn message_passing() {
    let (tx, rx) = mpsc::channel();
    
    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });
    
    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}

// 多个生产者
fn multiple_producers() {
    let (tx, rx) = mpsc::channel();
    let tx1 = mpsc::Sender::clone(&tx);
    
    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];
        
        for val in vals {
            tx1.send(val).unwrap();
        }
    });
    
    thread::spawn(move || {
        let vals = vec![
            String::from("more"),
            String::from("messages"),
            String::from("for"),
            String::from("you"),
        ];
        
        for val in vals {
            tx.send(val).unwrap();
        }
    });
    
    for received in rx {
        println!("Got: {}", received);
    }
}
```

### 11.3 共享状态并发

```rust
use std::sync::{Mutex, Arc};
use std::thread;

fn shared_state() {
    let counter = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for _ in 0..10 {
        let counter = Arc::clone(&counter);
        let handle = thread::spawn(move || {
            let mut num = counter.lock().unwrap();
            *num += 1;
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.join().unwrap();
    }
    
    println!("Result: {}", *counter.lock().unwrap());
}
```

## 12. 高级特性

### 12.1 Unsafe Rust

```rust
unsafe fn dangerous() {}

fn unsafe_examples() {
    let mut num = 5;
    
    let r1 = &num as *const i32;
    let r2 = &mut num as *mut i32;
    
    unsafe {
        println!("r1 is: {}", *r1);
        println!("r2 is: {}", *r2);
        dangerous();
    }
}
```

### 12.2 高级Trait

```rust
// 关联类型
pub trait Iterator {
    type Item;
    fn next(&mut self) -> Option<Self::Item>;
}

// 默认泛型参数
use std::ops::Add;

#[derive(Debug, PartialEq)]
struct Point {
    x: i32,
    y: i32,
}

impl Add for Point {
    type Output = Point;
    
    fn add(self, other: Point) -> Point {
        Point {
            x: self.x + other.x,
            y: self.y + other.y,
        }
    }
}

// 完全限定语法
trait Pilot {
    fn fly(&self);
}

trait Wizard {
    fn fly(&self);
}

struct Human;

impl Pilot for Human {
    fn fly(&self) {
        println!("Flying a plane");
    }
}

impl Wizard for Human {
    fn fly(&self) {
        println!("Flying with magic");
    }
}

impl Human {
    fn fly(&self) {
        println!("Waving arms furiously");
    }
}

fn disambiguation() {
    let person = Human;
    Pilot::fly(&person);
    Wizard::fly(&person);
    person.fly();
}
```

### 12.3 宏

```rust
// 声明宏
macro_rules! vec {
    ( $( $x:expr ),* ) => {
        {
            let mut temp_vec = Vec::new();
            $(
                temp_vec.push($x);
            )*
            temp_vec
        }
    };
}

// 过程宏示例（需要在单独的crate中）
// 在Cargo.toml中添加：
// [lib]
// proc-macro = true
```

## 13. 测试

### 13.1 单元测试

```rust
#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn it_works() {
        assert_eq!(2 + 2, 4);
    }
    
    #[test]
    #[should_panic(expected = "Guess value must be less than or equal to 100")]
    fn greater_than_100() {
        Guess::new(200);
    }
    
    #[test]
    fn it_returns_result() -> Result<(), String> {
        if 2 + 2 == 4 {
            Ok(())
        } else {
            Err(String::from("two plus two does not equal four"))
        }
    }
}
```

### 13.2 集成测试

```rust
// tests/integration_test.rs
use my_crate;

#[test]
fn it_adds_two() {
    assert_eq!(4, my_crate::add_two(2));
}
```

## 14. 与C++对比总结

| 特性      | C++             | Rust               |
| --------- | --------------- | ------------------ |
| 内存管理  | 手动管理        | 所有权系统         |
| 空指针    | 可能存在        | Option<T>          |
| 错误处理  | 异常            | Result<T, E>       |
| 多态      | 虚函数表        | Trait对象          |
| 模板/泛型 | 模板            | 泛型 + Trait       |
| 头文件    | #include        | mod系统            |
| 宏        | 文本替换        | 语法感知           |
| 包管理    | 无标准          | Cargo              |
| 并发      | 线程 + 手动同步 | 所有权 + std::sync |

## 15. 学习建议

1. **从所有权系统开始**：这是Rust最核心的概念
2. **多写小项目**：CLI工具、简单的Web服务等
3. **阅读标准库源码**：了解惯用法
4. **参与开源项目**：提升实战经验
5. **学习async/await**：现代Rust开发必备
6. **掌握错误处理模式**：Result和Option的各种用法

## 16. 推荐资源

- **官方文档**：https://doc.rust-lang.org/book/
- **Rustlings练习**：https://github.com/rust-lang/rustlings
- **Rust By Example**：https://doc.rust-lang.org/rust-by-example/
- **The Cargo Book**：https://doc.rust-lang.org/cargo/
- **社区**：Reddit r/rust, Rust官方论坛