# oop

面向对象编程（Object-Oriented Programming，OOP）是一种编程范式，它以对象为基本单元，将数据和操作封装在对象中，通过对象之间的交互来实现程序的功能。

## 1. 面向对象的语言特性

面向对象编程有以下几个特性：

- 封装（Encapsulation）：将数据和操作封装在对象中，隐藏对象的内部细节，只暴露必要的接口。
- 继承（Inheritance）：子类可以继承父类的属性和方法。
- 多态（Polymorphism）：同一个方法可以根据对象的不同表现出不同的行为。

## 2. Rust 中的面向对象编程

### 2.1. 封装

Rust 中的封装通过 `struct` 和 `impl` 实现，可以使用 `pub` 关键字指定公有或私有。

```rust
pub struct Person {
    name: String,
    age: u8,
}

impl Person {
    pub fn new(name: &str, age: u8) -> Self {
        Self {
            name: name.to_string(),
            age,
        }
    }

    pub fn say_hello(&self) {
        println!("Hello, my name is {}, I'm {} years old.", self.name, self.age);
    }

    fn private_method(&self) {
        println!("This is a private method.");
    }
}
```

### 2.2. 继承

Rust 中没有继承的概念，但可以通过组合和 trait 实现类似的功能。

```rust
pub struct Student {
    person: Person,
    grade: u8,
}

impl Student {
    pub fn new(name: &str, age: u8, grade: u8) -> Self {
        Self {
            person: Person::new(name, age),
            grade,
        }
    }

    pub fn study(&self) {
        println!("I'm studying in grade {}.", self.grade);
    }
}
```
