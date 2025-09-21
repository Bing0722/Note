# async await

并行与并发是两个不同的概念，但是在编程中经常会混淆。并行是指两个或多个任务同时执行，而并发是指两个或多个任务在一段时间内交替执行。并行是并发的子集，也就是说并行是并发的一种特例。

在 Rust 中，我们可以使用 `async` 和 `await` 关键字来实现异步编程。`async` 关键字用于声明异步函数，`await` 关键字用于等待异步函数的结果。

## 1. Features 和 async

### 1.1. 第一个异步程序

```rust
use trpl::Html;

async fn page_title(url: &str) -> Option<String> {
    let response = trpl::get(url).await;
    let response_text = response.text().await;
    // let response_text = trpl::get(url).await.text().await;
    Html::parse(&response_text)
        .select("title")
        .map(|title_element| title_element.inner_html())
}
```

这个例子中的两个函数都是异步函数，`page_title` 函数是一个异步函数，`get` 和 `text` 方法也是异步函数。`await` 关键字用于等待异步函数的结果。

在 Rust 中，异步函数的返回值是一个 `Future` 对象，`Future` 对象是一个异步任务的抽象。`Future` 对象可以通过 `await` 关键字来等待异步任务的结果。

而且 在 Rust 中 `future` 是惰性的，只有在调用 `await` 关键字时才会执行异步任务。

在 Rust 中遇到一个 `async` 函数时，编译器会生成一个 `Future` 对象，这个 `Future` 对象会在调用 `await` 关键字时执行异步任务。

所以上面的函数等价于：

```rust
fn page_title(url: &str) -> impl Future<Output = Option<String>> + '_ {
    async move {
        let response = trpl::get(url).await;
        let response_text = response.text().await;
        Html::parse(&response_text)
            .select("title")
            .map(|title_element| title_element.inner_html())
    }
}
```

### 1.2. 运行异步函数

使用 `trpl::run` 函数来运行异步函数： 它获取一个 future 作为参数，并在当前线程上运行它。

```rust
use trpl::Html;

fn main() {
    let args: Vec<String> = std::env::args().collect();

    trpl::run(async{
        let url = &args[1];
        match page_title(url).await {
            Some(title) => println!("Title: {}", title),
            None => println!("No title found"),
        }
    })
}
```

## 2. 并发与async

### 2.1. 计数

```rust
use std::time::Duration;

fn main() {
    trpl::run(async{
        let handler = trpl::spawn_task(async{
            for i in 0..10 {
                println!("Task 1: {}", i);
                trpl::sleep(Duration::from_secs(1)).await;
            }
        });

        for i in 0..5 {
            println!("Main: {}", i);
            trpl::sleep(Duration::from_secs(1)).await;
        }

        // wait for the task to finish
        handler.await.unwrap();
    })
}
```

在这个例子中，我们创建了一个异步任务 `handler`，然后在主线程中打印数字。在主线程中，我们使用 `trpl::sleep` 函数来模拟一个耗时的操作。

那么使用 `trpl::join` 函数来等待多个异步任务的结果。

```rust
use std::time::Duration;

fn main() {
    trpl::run(async{
        let handler1 = trpl::spawn_task(async{
            for i in 0..10 {
                println!("Task 1: {}", i);
                trpl::sleep(Duration::from_secs(1)).await;
            }
        });

        let handler2 = trpl::spawn_task(async{
            for i in 0..5 {
                println!("Task 2: {}", i);
                trpl::sleep(Duration::from_secs(1)).await;
            }
        });

        trpl::join(handler1, handler2).await;
    })
}
```

## 3. 消息传递

在线程间通信中，我们可以使用 `channel` 来传递消息。在异步编程中，我们可以使用 `trpl::channel` 函数来创建一个异步通道。

```rust
fn main(){
    trpl::run(async {
        let (tx, mut rx) = trpl::channel();

        let val = String::from("hi");
        tx.send(val).unwrap();

        let received = rx.recv().await.unwrap();
        println!("Got: {received}");
    })
}
```

### 3.1. 多个消息

多个消息的发送和接收是并发的，所以我们可以使用 `trpl::join` 函数来等待多个异步任务的结果。

```rust
fn main(){
    trpl::run(async {
        let (tx, mut rx) = trpl::channel();

        let tx_fut = async {
            let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("future"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async {
            while let Some(value) = rx.recv().await {
                eprintln!("received '{value}'");
            }
        };

        trpl::join(tx_fut, rx_fut).await;
    });
}
```

### 3.2. 使用任意多个 future

我们可以使用 `trpl::join` 函数来等待多个异步任务的结果, 或者使用它的变体 `trpl::join3` ... 等等。

我们也可以使用 `trpl::join!` 宏 和 `trpl::join_all` 函数来等待多个异步任务的结果。

```rust
fn main(){
    trpl::run(async {
        let (tx, mut rx) = trpl::channel();

        let tx1 = tx.clone();
        let tx1_fut = async move {
            let vals = vec![
                String::from("hi"),
                String::from("from"),
                String::from("the"),
                String::from("future"),
            ];

            for val in vals {
                tx1.send(val).unwrap();
                trpl::sleep(Duration::from_millis(500)).await;
            }
        };

        let rx_fut = async move {
            while let Some(value) = rx.recv().await {
                println!("received '{value}'");
            }
        };

        let tx_fut = async move {
            let vals = vec![
                String::from("more"),
                String::from("messages"),
                String::from("for"),
                String::from("you"),
            ];

            for val in vals {
                tx.send(val).unwrap();
                trpl::sleep(Duration::from_secs(1)).await;
            }
        };

        trpl::join!(tx1_fut, tx_fut, rx_fut);
}
```

这里使用 `trpl::join!` 宏来等待多个异步任务的结果。

他和 `trpl::join_all` 函数接受一个 `Vec`，并返回一个 `Future` 对象，而 `trpl::join!` 宏接受多个 `Future` 对象，并返回一个 `Future` 对象。
