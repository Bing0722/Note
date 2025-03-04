# concurrency

Rust 的无畏并发(fearless concurrency)

## 1. 使用 `spawn` 创建线程

通过使用 `std::thread::spawn` 函数来创建线程，传入一个闭包作为线程的执行体。

```rust
use std::thread;
use std::time::Duration;

fn main() {
    let handle = thread::spawn(|| {
        for i in 1..10 {
            println!("hi number {} from the spawned thread", i);
            thread::sleep(Duration::from_millis(1));
        }
    });

    for i in 1..5 {
        println!("hi number {} from the main thread", i);
        thread::sleep(Duration::from_millis(1));
    }

    handle.join().unwrap();
}
```

通过 `join` 方法来等待线程结束, `join` 方法返回一个 `Result` 类型，如果线程正常结束，返回 `Ok`，否则返回 `Err`， `unwrap` 方法用于解析 `Result` 类型，如果是 `Ok`，返回 `Ok` 中的值，否则直接 panic。

使用 `move` 关键字来强制闭包获取其使用的环境值的所有权，这样闭包就可以在创建线程后继续使用这些值。

```rust
use std::thread;

fn main() {
    let v = vec![1, 2, 3];

    let handle = thread::spawn(move || {
        println!("Here's a vector: {:?}", v);
    });

    handle.join().unwrap();
}
```

使用 `move` 关键字之后，`v` 的所有权被移动到闭包中，所以在闭包中可以继续使用 `v`，但是在闭包外部就不能再使用 `v` 了。

## 2. 使用消息传递在线程间传递数据

Rust 中的消息传递是通过通道(channel)来实现的，通道是一个多生产者单消费者的队列。

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let val = String::from("hi");
        tx.send(val).unwrap();
    });

    let received = rx.recv().unwrap();
    println!("Got: {}", received);
}
```

发送多个消息

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
    let (tx, rx) = mpsc::channel();

    thread::spawn(move || {
        let vals = vec![
            String::from("hi"),
            String::from("from"),
            String::from("the"),
            String::from("thread"),
        ];

        for val in vals {
            tx.send(val).unwrap();
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

这里 `rx` 不再显示调用 `recv` 方法，而是将 `rx` 作为一个迭代器，这样当通道被关闭时，迭代器就会结束。

多个生产者

```rust
use std::sync::mpsc;
use std::thread;

fn main() {
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
            thread::sleep(Duration::from_secs(1));
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
            thread::sleep(Duration::from_secs(1));
        }
    });

    for received in rx {
        println!("Got: {}", received);
    }
}
```

## 3. 共享状态并发

Rust 中的共享状态并发是通过 `Arc` 和 `Mutex` 来实现的。

```rust
use std::sync::{Arc, Mutex};
use std::thread;

fn main() {
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

使用 `Arc` 来创建一个引用计数的 `Mutex`，这样就可以在多个线程中共享 `counter`。

## 4. `RefCell<T>` / `Rc<T>` 和 `Mutex<T>` / `Arc<T>` 相似性

`RefCell<T>` 和 `Rc<T>` 与 `Mutex<T>` 和 `Arc<T>` 的相似性：

- `RefCell<T>` 和 `Mutex<T>` 都是内部可变的，`Rc<T>` 和 `Arc<T>` 都是引用计数的。
- `RefCell<T>` 和 `Rc<T>` 适用于单线程，`Mutex<T>` 和 `Arc<T>` 适用于多线程。
- `RefCell<T>` 和 `Mutex<T>` 都是非线程安全的，`Rc<T>` 和 `Arc<T>` 都是线程安全的。
