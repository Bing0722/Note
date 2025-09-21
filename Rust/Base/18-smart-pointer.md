# smart pointer

智能指针是一个数据结构，表现类似指针，但是也拥有额外的元数据和功能。智能指针通常用于管理分配给堆上内存的内存。
智能指针是一种常见的设计模式，它使用 Rust 的所有权系统来确保在内存不再需要时释放内存。

## 1. 使用 Box<T> 指向堆上数据

Box<T> 的使用场景：

- 当有一个在编译时未知大小的类型，而又需要在确切大小的上下文中使用这个类型值的时候。
- 当有大量数据并希望在确保数据不被拷贝的情况下转移所有权的时候。
- 当希望拥有一个值并只关心它的类型是否实现了特定 trait 而不是其具体类型时。

```rust
fn main() {
    let b = Box::new(5);
    println!("b = {}", b);
}
```

## 2. 使用 Box<T> 在堆上存储递归类型

递归类型是指在定义自身的结构体或枚举体中使用自身的情况。这种类型的大小在编译时是未知的，所以不能直接使用。

```rust
enum List {
    Cons(i32, Box<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let list = Cons(1, Box::new(Cons(2, Box::new(Cons(3, Box::new(Nil)))));
}
```

## 3. 通过 Deref trait 将智能指针当作常规引用处理

智能指针实现了 Deref trait，它允许智能指针结构体实例表现得像引用一样，这样就可以编写使用引用的代码而实际上使用的是智能指针。

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &self::Target {
        &self.0
    }
}

fn main() {
    let x = 5;
    let y = MyBox::new(x);

    assert_eq!(5, x);
    assert_eq!(5, *y);
}
```

这个例子展示了 Rust 是如何通过 Deref trait 将智能指针当作常规引用处理的。当我们在代码中使用 `*y` 时，Rust 实际上会调用 `*(y.deref())`。

## 4. 函数和方法的隐式 Deref 强制转换

Rust 也提供了一个功能叫做 Deref 强制转换，它可以把实现了 Deref trait 的类型的引用转换为其他类型的引用。

```rust
use std::ops::Deref;

struct MyBox<T>(T);

impl<T> MyBox<T> {
    fn new(x: T) -> MyBox<T> {
        MyBox(x)
    }
}

impl<T> Deref for MyBox<T> {
    type Target = T;

    fn deref(&self) -> &T {
        &self.0
    }
}

fn hello(name: &str) {
    println!("Hello, {}!", name);
}

fn main() {
    let m = MyBox::new(String::from("Rust"));
    hello(&m);
    // hello(&(*m)[..]);
}
```

这个例子展示了 Rust 是如何通过 Deref 强制转换将 `&MyBox<String>` 转换为 `&String` 的。当我们在代码中使用 `&m` 时，Rust 实际上会调用 `&(*m)`。

### 4.1. Deref 强制转换如何与可变性交互

Rust 在发现类型和 trait 实现满足三种情况时会进行 Deref 强制转换：

- 当 T: Deref<Target=U> 时从 &T 到 &U。
- 当 T: DerefMut<Target=U> 时从 &mut T 到 &mut U。
- 当 T: Deref<Target=U> 时从 &mut T 到 &U。

> Rust 不存在从 &T 到 &mut U 的 Deref 强制转换。

## 5. 使用 Drop trait 运行清理代码

Drop trait 类似于 C++ 中的析构函数，它允许我们在值离开作用域时执行一些代码。

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("my stuff") };
    let d = CustomSmartPointer { data: String::from("other stuff") };
    println!("CustomSmartPointers created.");
}
```

这个例子展示了 Rust 是如何通过 Drop trait 运行清理代码的。当 `c` 和 `d` 离开作用域时，Rust 会自动调用 `drop` 方法。

## 6. 通过 std::mem::drop 提早丢弃值

Rust 不允许我们显式调用 Drop trait 的 `drop` 方法，因为 Rust 会在值离开作用域时自动调用 `drop` 方法。但是 Rust 提供了一个名为 `std::mem::drop` 的函数，它接受一个值并立即丢弃它。

```rust
struct CustomSmartPointer {
    data: String,
}

impl Drop for CustomSmartPointer {
    fn drop(&mut self) {
        println!("Dropping CustomSmartPointer with data `{}`!", self.data);
    }
}

fn main() {
    let c = CustomSmartPointer { data: String::from("some data") };
    println!("CustomSmartPointer created.");
    c.drop();    // error: explicit use of destructor method
    drop(c);     // OK
    println!("CustomSmartPointer dropped before the end of main.");
}
```

这个例子展示了 Rust 是如何通过 `std::mem::drop` 提早丢弃值的。当 `c` 调用 `drop` 方法时，Rust 会报错；当 `c` 传递给 `drop` 函数时，Rust 会立即丢弃 `c`。

## 7. Rc<T> 引用计数智能指针

Rc<T> 是一个引用计数智能指针，它允许我们在程序的多个部分之间共享数据，但是它只能用于单线程场景。

```rust
enum List {
    Cons(i32, Rc<List>),
    Nil,
}

use List::{Cons, Nil};
use std::rc::Rc;

fn main() {
    let a = Rc::new(Cons(5, Rc::new(Cons(10, Rc::new(Nil)))));
    println!("count after creating a = {}", Rc::strong_count(&a));  // a  reference count = 1
    let b = Cons(3, Rc::clone(&a));
    println!("count after creating b = {}", Rc::strong_count(&a)); // a reference count = 2
    {
        let c = Cons(4, Rc::clone(&a));
        println!("count after creating c = {}", Rc::strong_count(&a)); // a reference count = 3
    }
    println!("count after c goes out of scope = {}", Rc::strong_count(&a)); // a reference count = 2
}
```

## 8. RefCell<T> 和内部可变性模式

RefCell<T> 是一个在运行时而不是在编译时执行借用规则的类型。RefCell<T> 代表其数据的唯一所有权，所以我们可以在运行时借用数据的多个部分。

```rust
use std::cell::RefCell;

fn main() {
    let x = RefCell::new(42);
    let y = x.borrow_mut();
    // let z = x.borrow_mut(); // error: already borrowed: BorrowMutError
    println!("{}", y);
}
```

如果存在多个可变引用，RefCell<T> 的 borrow_mut 方法会在运行时检查是否违反了借用规则，如果违反了则会导致程序崩溃。

## 9. 结合 Rc<T> 和 RefCell<T> 来拥有多个可变引用

Rc<T> 允许多个所有者共享数据，RefCell<T> 允许在运行时而不是在编译时执行借用规则。

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List {
    Cons(Rc<RefCell<i32>>, Rc<List>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let value = Rc::new(RefCell::new(5));
    let a = Rc::new(Cons(Rc::clone(&value), Rc::new(Nil)));
    let b = Cons(Rc::new(RefCell::new(3)), Rc::clone(&a));
    let c = Cons(Rc::new(RefCell::new(4)), Rc::clone(&a));
    *value.borrow_mut() += 10;
    println!("{:?}", a);
    println!("{:?}", b);
    println!("{:?}", c);
}
```

## 10. 引用循环与内存泄漏

Rust 的内存安全性保证不允许内存泄漏，但是当使用 Rc<T> 和 RefCell<T> 时，我们需要注意可能出现的引用循环。

```rust
use std::rc::Rc;
use std::cell::RefCell;

#[derive(Debug)]
enum List {
    Cons(i32, RefCell<Rc<List>>),
    Nil,
}

use List::{Cons, Nil};

fn main() {
    let a = Rc::new(Cons(5, RefCell::new(Rc::new(Nil))));
    println!("a initial rc count = {}", Rc::strong_count(&a));
    println!("a next item = {:?}", a.tail());
    let b = Rc::new(Cons(10, RefCell::new(Rc::clone(&a))));
    println!("a rc count after b creation = {}", Rc::strong_count(&a));
    println!("b initial rc count = {}", Rc::strong_count(&b));
    println!("b next item = {:?}", b.tail());
    if let Some(link) = a.tail() {
        *link.borrow_mut() = Rc::clone(&b);
    }
    println!("b rc count after changing a = {}", Rc::strong_count(&b));
    println!("a rc count after changing a = {}", Rc::strong_count(&a));
    // println!("a next item = {:?}", a.tail()); // error: stack overflow
}

impl List {
    fn tail(&self) -> Option<&RefCell<Rc<List>>> {
        match self {
            Cons(_, item) => Some(item),
            Nil => None,
        }
    }
}
```

这个例子展示了 Rust 是如何通过引用循环和内存泄漏来保证内存安全性的。当 `a` 和 `b` 互相引用时，它们的引用计数永远不会为 0，所以它们指向的内存永远不会被释放。

## 11. 弱引用和 Rc<T>

Rc<T> 的弱引用是一种不增加引用计数的引用，它允许我们引用 Rc<T> 实例但不会造成引用计数增加。

```rust
use std::rc::{Rc, Weak};
use std::cell::RefCell;

struct Node {
    value: i32,
    parent: RefCell<Weak<Node>>, // 弱引用
    children: RefCell<Vec<Rc<Node>>>, // 强引用
}

fn main() {
    let parent = Rc::new(Node {
        value: 1,
        parent: RefCell::new(Weak::new()), // 初始化为空的弱引用
        children: RefCell::new(vec![]),
    });

    let child = Rc::new(Node {
        value: 2,
        parent: RefCell::new(Rc::downgrade(&parent)), // 使用弱引用指向父节点
        children: RefCell::new(vec![]),
    });

    // 将子节点加入到父节点的 children 中
    parent.children.borrow_mut().push(Rc::clone(&child));

    println!("Parent strong count = {}", Rc::strong_count(&parent)); // 输出：1
    println!("Parent weak count = {}", Rc::weak_count(&parent));   // 输出：1
    println!("Child strong count = {}", Rc::strong_count(&child)); // 输出：1
}
```
