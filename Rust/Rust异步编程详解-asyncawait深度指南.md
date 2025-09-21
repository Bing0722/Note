# Rust异步编程详解 - async/await深度指南

## 1. 异步编程概述

### 1.1 为什么需要异步编程？

**传统同步编程的问题**：

- I/O操作会阻塞线程，造成资源浪费
- 大量线程会导致上下文切换开销
- 内存占用高（每个线程需要栈空间）

**异步编程的优势**：

- 非阻塞I/O，提高并发性能
- 更少的系统资源占用
- 更适合I/O密集型应用

### 1.2 Rust异步模型特点

```rust
// Rust的异步是零成本抽象
// 编译时会生成状态机，运行时无额外开销

// 异步函数示例
async fn hello_world() {
    println!("Hello, async world!");
}

// 返回Future trait对象
fn main() {
    let future = hello_world(); // 这里不会执行函数体
    // 需要运行时来驱动Future
}
```

## 2. Future Trait 深入理解

### 2.1 Future Trait 定义

```rust
use std::pin::Pin;
use std::task::{Context, Poll};

trait Future {
    type Output;
    
    fn poll(self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output>;
}

// Poll枚举
enum Poll<T> {
    Ready(T),    // 任务完成
    Pending,     // 任务尚未完成，需要再次轮询
}
```

### 2.2 手动实现简单的Future

```rust
use std::future::Future;
use std::pin::Pin;
use std::task::{Context, Poll, Waker};
use std::time::{Duration, Instant};

// 一个延迟Future
struct TimerFuture {
    when: Instant,
    waker: Option<Waker>,
}

impl TimerFuture {
    fn new(duration: Duration) -> Self {
        TimerFuture {
            when: Instant::now() + duration,
            waker: None,
        }
    }
}

impl Future for TimerFuture {
    type Output = ();
    
    fn poll(mut self: Pin<&mut Self>, cx: &mut Context<'_>) -> Poll<Self::Output> {
        if Instant::now() >= self.when {
            Poll::Ready(())
        } else {
            // 保存waker用于将来唤醒
            self.waker = Some(cx.waker().clone());
            
            // 在另一个线程中设置定时器
            let waker = self.waker.as_ref().unwrap().clone();
            let when = self.when;
            std::thread::spawn(move || {
                std::thread::sleep(when - Instant::now());
                waker.wake();
            });
            
            Poll::Pending
        }
    }
}
```

## 3. async/await 语法

### 3.1 基本语法

```rust
// async fn 自动返回 impl Future<Output = T>
async fn fetch_data() -> String {
    // 模拟异步操作
    tokio::time::sleep(Duration::from_millis(100)).await;
    "Data fetched".to_string()
}

// 等价的手动实现
fn fetch_data_manual() -> impl Future<Output = String> {
    async {
        tokio::time::sleep(Duration::from_millis(100)).await;
        "Data fetched".to_string()
    }
}

#[tokio::main]
async fn main() {
    let result = fetch_data().await;
    println!("{}", result);
}
```

### 3.2 async块

```rust
async fn async_block_example() {
    let async_block = async {
        println!("Inside async block");
        42
    };
    
    let result = async_block.await;
    println!("Result: {}", result);
}

// 带move的async块
async fn async_move_example() {
    let data = vec![1, 2, 3, 4, 5];
    
    let async_block = async move {
        println!("Data: {:?}", data);
        data.len()
    };
    
    let len = async_block.await;
    // data在这里不再可用，因为已经被move
}
```

### 3.3 async闭包

```rust
// 注意：async闭包目前还不稳定，需要nightly版本
// 这里展示概念

// 当前可用的替代方案
fn async_closure_alternative() -> impl Future<Output = ()> {
    let closure = || async {
        println!("Async closure alternative");
    };
    
    closure()
}
```

## 4. 常用异步运行时

### 4.1 Tokio - 最流行的异步运行时

```rust
// Cargo.toml
/*
[dependencies]
tokio = { version = "1.0", features = ["full"] }
*/

use tokio::time::{sleep, Duration};

#[tokio::main]
async fn main() {
    println!("Starting async program");
    
    // 并发执行多个任务
    tokio::join!(
        task_one(),
        task_two(),
        task_three()
    );
    
    println!("All tasks completed");
}

async fn task_one() {
    sleep(Duration::from_secs(1)).await;
    println!("Task one completed");
}

async fn task_two() {
    sleep(Duration::from_secs(2)).await;
    println!("Task two completed");
}

async fn task_three() {
    sleep(Duration::from_secs(1)).await;
    println!("Task three completed");
}
```

### 4.2 async-std - 标准库风格的异步运行时

```rust
// Cargo.toml
/*
[dependencies]
async-std = { version = "1.12", features = ["attributes"] }
*/

use async_std::task;
use async_std::prelude::*;

#[async_std::main]
async fn main() {
    let tasks = vec![
        task::spawn(async_task("Task 1", 1000)),
        task::spawn(async_task("Task 2", 500)),
        task::spawn(async_task("Task 3", 1500)),
    ];
    
    // 等待所有任务完成
    for task in tasks {
        task.await;
    }
}

async fn async_task(name: &str, delay_ms: u64) {
    task::sleep(std::time::Duration::from_millis(delay_ms)).await;
    println!("{} completed", name);
}
```

## 5. 异步并发模式

### 5.1 并发执行 - join! 和 try_join!

```rust
use tokio::time::{sleep, Duration};
use std::io;

async fn concurrent_execution() {
    // 所有任务并发执行，等待全部完成
    let (a, b, c) = tokio::join!(
        fetch_data_from_db(),
        fetch_data_from_api(),
        process_local_data()
    );
    
    println!("Results: {}, {}, {}", a, b, c);
}

async fn concurrent_with_error_handling() -> Result<(), io::Error> {
    // 如果任何一个失败，立即返回错误
    let (a, b) = tokio::try_join!(
        async_operation_that_might_fail(),
        another_async_operation()
    )?;
    
    println!("Both operations succeeded: {}, {}", a, b);
    Ok(())
}

async fn fetch_data_from_db() -> String {
    sleep(Duration::from_millis(100)).await;
    "DB data".to_string()
}

async fn fetch_data_from_api() -> String {
    sleep(Duration::from_millis(200)).await;
    "API data".to_string()
}

async fn process_local_data() -> String {
    sleep(Duration::from_millis(50)).await;
    "Local data".to_string()
}

async fn async_operation_that_might_fail() -> Result<String, io::Error> {
    sleep(Duration::from_millis(100)).await;
    Ok("Success".to_string())
}

async fn another_async_operation() -> Result<String, io::Error> {
    sleep(Duration::from_millis(150)).await;
    Ok("Another success".to_string())
}
```

### 5.2 选择执行 - select! 宏

```rust
use tokio::time::{sleep, Duration, timeout};

async fn select_pattern() {
    tokio::select! {
        result = long_running_task() => {
            println!("Long task completed: {}", result);
        }
        _ = sleep(Duration::from_secs(1)) => {
            println!("Timeout reached!");
        }
        msg = receive_message() => {
            println!("Received message: {}", msg);
        }
    }
}

async fn long_running_task() -> String {
    sleep(Duration::from_secs(5)).await;
    "Task result".to_string()
}

async fn receive_message() -> String {
    sleep(Duration::from_millis(800)).await;
    "Important message".to_string()
}

// 带偏向的select
async fn biased_select() {
    let mut counter = 0;
    
    loop {
        tokio::select! {
            biased; // 按照分支顺序检查，不随机
            
            _ = sleep(Duration::from_millis(10)) => {
                counter += 1;
                if counter > 5 {
                    break;
                }
            }
            msg = receive_urgent_message(), if counter < 3 => {
                println!("Urgent: {}", msg);
            }
        }
    }
}

async fn receive_urgent_message() -> String {
    sleep(Duration::from_millis(5)).await;
    "URGENT!".to_string()
}
```

## 6. 异步任务管理

### 6.1 spawning 任务

```rust
use tokio::task;
use std::time::Duration;

async fn task_spawning() {
    // 创建新任务，在后台运行
    let handle1 = task::spawn(async {
        tokio::time::sleep(Duration::from_secs(1)).await;
        "Task 1 result"
    });
    
    let handle2 = task::spawn(async {
        tokio::time::sleep(Duration::from_secs(2)).await;
        "Task 2 result"  
    });
    
    // 等待任务完成
    let result1 = handle1.await.unwrap();
    let result2 = handle2.await.unwrap();
    
    println!("{}, {}", result1, result2);
}

// 带错误处理的任务
async fn task_with_error_handling() {
    let handle = task::spawn(async {
        // 可能失败的操作
        if rand::random::<bool>() {
            Ok("Success")
        } else {
            Err("Failed")
        }
    });
    
    match handle.await {
        Ok(Ok(result)) => println!("Task succeeded: {}", result),
        Ok(Err(err)) => println!("Task failed: {}", err),
        Err(join_err) => println!("Task panicked: {:?}", join_err),
    }
}
```

### 6.2 JoinSet - 管理多个任务

```rust
use tokio::task::JoinSet;

async fn join_set_example() {
    let mut set = JoinSet::new();
    
    // 添加多个任务
    for i in 0..5 {
        set.spawn(async move {
            tokio::time::sleep(Duration::from_millis(100 * i)).await;
            format!("Task {} completed", i)
        });
    }
    
    // 逐个等待任务完成
    while let Some(res) = set.join_next().await {
        match res {
            Ok(output) => println!("{}", output),
            Err(err) => println!("Task failed: {:?}", err),
        }
    }
}

// 有限制的并发
async fn limited_concurrency() {
    use tokio::sync::Semaphore;
    use std::sync::Arc;
    
    let semaphore = Arc::new(Semaphore::new(3)); // 最多3个并发任务
    let mut set = JoinSet::new();
    
    for i in 0..10 {
        let permit = semaphore.clone();
        set.spawn(async move {
            let _permit = permit.acquire().await.unwrap();
            println!("Task {} started", i);
            tokio::time::sleep(Duration::from_secs(1)).await;
            println!("Task {} finished", i);
            i
        });
    }
    
    while let Some(res) = set.join_next().await {
        println!("Task result: {:?}", res);
    }
}
```

## 7. 异步I/O操作

### 7.1 文件I/O

```rust
use tokio::fs;
use tokio::io::{AsyncReadExt, AsyncWriteExt};

async fn file_operations() -> Result<(), Box<dyn std::error::Error>> {
    // 写文件
    let mut file = fs::File::create("test.txt").await?;
    file.write_all(b"Hello, async file I/O!").await?;
    file.flush().await?;
    
    // 读文件
    let mut file = fs::File::open("test.txt").await?;
    let mut contents = String::new();
    file.read_to_string(&mut contents).await?;
    println!("File contents: {}", contents);
    
    // 一次性读取整个文件
    let contents = fs::read_to_string("test.txt").await?;
    println!("File contents: {}", contents);
    
    // 删除文件
    fs::remove_file("test.txt").await?;
    
    Ok(())
}
```

### 7.2 网络I/O

```rust
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

// 异步TCP服务器
async fn tcp_server() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server listening on 127.0.0.1:8080");
    
    loop {
        let (socket, addr) = listener.accept().await?;
        println!("New connection: {}", addr);
        
        // 为每个连接spawning新任务
        tokio::spawn(handle_client(socket));
    }
}

async fn handle_client(mut socket: TcpStream) -> Result<(), Box<dyn std::error::Error>> {
    let mut buf = [0; 1024];
    
    loop {
        match socket.read(&mut buf).await {
            Ok(0) => break, // 连接关闭
            Ok(n) => {
                let msg = String::from_utf8_lossy(&buf[0..n]);
                println!("Received: {}", msg);
                
                // 回显消息
                socket.write_all(&buf[0..n]).await?;
            }
            Err(err) => {
                println!("Error: {}", err);
                break;
            }
        }
    }
    
    Ok(())
}

// 异步TCP客户端
async fn tcp_client() -> Result<(), Box<dyn std::error::Error>> {
    let mut stream = TcpStream::connect("127.0.0.1:8080").await?;
    
    stream.write_all(b"Hello from client!").await?;
    
    let mut buf = [0; 1024];
    let n = stream.read(&mut buf).await?;
    let response = String::from_utf8_lossy(&buf[0..n]);
    println!("Server response: {}", response);
    
    Ok(())
}
```

### 7.3 HTTP 客户端

```rust
// Cargo.toml
/*
[dependencies]
reqwest = { version = "0.11", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
*/

use reqwest;
use serde::{Deserialize, Serialize};

#[derive(Serialize, Deserialize, Debug)]
struct Post {
    id: u32,
    title: String,
    body: String,
    #[serde(rename = "userId")]
    user_id: u32,
}

async fn http_requests() -> Result<(), reqwest::Error> {
    let client = reqwest::Client::new();
    
    // GET 请求
    let posts: Vec<Post> = client
        .get("https://jsonplaceholder.typicode.com/posts")
        .send()
        .await?
        .json()
        .await?;
    
    println!("Retrieved {} posts", posts.len());
    
    // POST 请求
    let new_post = Post {
        id: 0,
        title: "My new post".to_string(),
        body: "This is the content".to_string(),
        user_id: 1,
    };
    
    let response = client
        .post("https://jsonplaceholder.typicode.com/posts")
        .json(&new_post)
        .send()
        .await?;
    
    println!("POST response status: {}", response.status());
    
    Ok(())
}
```

## 8. 异步同步原语

### 8.1 Mutex (异步互斥锁)

```rust
use tokio::sync::Mutex;
use std::sync::Arc;

async fn async_mutex_example() {
    let data = Arc::new(Mutex::new(0));
    let mut handles = vec![];
    
    for i in 0..10 {
        let data = Arc::clone(&data);
        let handle = tokio::spawn(async move {
            let mut num = data.lock().await;
            *num += 1;
            println!("Task {} incremented value to {}", i, *num);
        });
        handles.push(handle);
    }
    
    // 等待所有任务完成
    for handle in handles {
        handle.await.unwrap();
    }
    
    println!("Final value: {}", *data.lock().await);
}
```

### 8.2 RwLock (异步读写锁)

```rust
use tokio::sync::RwLock;
use std::sync::Arc;

async fn async_rwlock_example() {
    let data = Arc::new(RwLock::new(vec![1, 2, 3, 4, 5]));
    
    // 多个读操作可以并发
    let read_tasks: Vec<_> = (0..5)
        .map(|i| {
            let data = Arc::clone(&data);
            tokio::spawn(async move {
                let vec = data.read().await;
                println!("Reader {} sees: {:?}", i, *vec);
            })
        })
        .collect();
    
    // 等待所有读操作完成
    for task in read_tasks {
        task.await.unwrap();
    }
    
    // 写操作是独占的
    {
        let mut vec = data.write().await;
        vec.push(6);
        println!("Added 6 to vector");
    }
    
    // 再次读取
    let vec = data.read().await;
    println!("Final vector: {:?}", *vec);
}
```

### 8.3 Semaphore (信号量)

```rust
use tokio::sync::Semaphore;
use std::sync::Arc;

async fn semaphore_example() {
    let semaphore = Arc::new(Semaphore::new(3)); // 最多3个许可
    let mut handles = vec![];
    
    for i in 0..10 {
        let semaphore = Arc::clone(&semaphore);
        let handle = tokio::spawn(async move {
            let _permit = semaphore.acquire().await.unwrap();
            println!("Task {} acquired permit", i);
            
            // 模拟工作
            tokio::time::sleep(Duration::from_secs(2)).await;
            
            println!("Task {} releasing permit", i);
            // permit在这里自动释放
        });
        handles.push(handle);
    }
    
    for handle in handles {
        handle.await.unwrap();
    }
}
```

### 8.4 Channel (异步通道)

```rust
use tokio::sync::{mpsc, oneshot, broadcast};

// 多生产者单消费者通道
async fn mpsc_example() {
    let (tx, mut rx) = mpsc::channel::<String>(32);
    
    // 生产者任务
    for i in 0..5 {
        let tx = tx.clone();
        tokio::spawn(async move {
            let msg = format!("Message from task {}", i);
            tx.send(msg).await.unwrap();
        });
    }
    
    // 关闭发送端
    drop(tx);
    
    // 消费者
    while let Some(msg) = rx.recv().await {
        println!("Received: {}", msg);
    }
}

// 一次性通道
async fn oneshot_example() {
    let (tx, rx) = oneshot::channel::<String>();
    
    tokio::spawn(async move {
        tokio::time::sleep(Duration::from_secs(1)).await;
        tx.send("Hello from oneshot!".to_string()).unwrap();
    });
    
    let result = rx.await.unwrap();
    println!("Oneshot result: {}", result);
}

// 广播通道
async fn broadcast_example() {
    let (tx, mut rx1) = broadcast::channel::<String>(16);
    let mut rx2 = tx.subscribe();
    let mut rx3 = tx.subscribe();
    
    tokio::spawn(async move {
        tx.send("Broadcast message 1".to_string()).unwrap();
        tx.send("Broadcast message 2".to_string()).unwrap();
    });
    
    // 多个接收者
    let h1 = tokio::spawn(async move {
        while let Ok(msg) = rx1.recv().await {
            println!("Receiver 1: {}", msg);
        }
    });
    
    let h2 = tokio::spawn(async move {
        while let Ok(msg) = rx2.recv().await {
            println!("Receiver 2: {}", msg);
        }
    });
    
    let h3 = tokio::spawn(async move {
        while let Ok(msg) = rx3.recv().await {
            println!("Receiver 3: {}", msg);
        }
    });
    
    tokio::join!(h1, h2, h3);
}
```

## 9. 异步迭代器和流

### 9.1 Stream trait

```rust
// Cargo.toml
/*
[dependencies]
futures = "0.3"
tokio-stream = "0.1"
*/

use futures::stream::{self, StreamExt};
use tokio_stream::wrappers::IntervalStream;

async fn stream_example() {
    // 从Vec创建流
    let mut stream = stream::iter(vec![1, 2, 3, 4, 5]);
    
    while let Some(item) = stream.next().await {
        println!("Item: {}", item);
    }
    
    // 使用流适配器
    let doubled: Vec<i32> = stream::iter(1..=10)
        .map(|x| x * 2)
        .filter(|&x| x > 10)
        .collect()
        .await;
    
    println!("Doubled and filtered: {:?}", doubled);
}

// 定时器流
async fn timer_stream() {
    let interval = tokio::time::interval(Duration::from_secs(1));
    let mut stream = IntervalStream::new(interval)
        .take(5); // 只取前5个
    
    while let Some(_) = stream.next().await {
        println!("Tick at {:?}", std::time::Instant::now());
    }
}

// 自定义流
use futures::stream::Stream;
use std::pin::Pin;
use std::task::{Context, Poll};

struct CounterStream {
    current: usize,
    max: usize,
}

impl CounterStream {
    fn new(max: usize) -> Self {
        CounterStream { current: 0, max }
    }
}

impl Stream for CounterStream {
    type Item = usize;
    
    fn poll_next(mut self: Pin<&mut Self>, _cx: &mut Context<'_>) -> Poll<Option<Self::Item>> {
        if self.current < self.max {
            let current = self.current;
            self.current += 1;
            Poll::Ready(Some(current))
        } else {
            Poll::Ready(None)
        }
    }
}

async fn custom_stream_example() {
    let mut stream = CounterStream::new(5);
    
    while let Some(num) = stream.next().await {
        println!("Counter: {}", num);
    }
}
```

## 10. 错误处理模式

### 10.1 异步错误处理最佳实践

```rust
use tokio::fs;
use std::io;

// 定义自定义错误类型
#[derive(Debug)]
enum AppError {
    Io(io::Error),
    Parse(std::num::ParseIntError),
    Network(reqwest::Error),
}

impl From<io::Error> for AppError {
    fn from(err: io::Error) -> AppError {
        AppError::Io(err)
    }
}

impl From<std::num::ParseIntError> for AppError {
    fn from(err: std::num::ParseIntError) -> AppError {
        AppError::Parse(err)
    }
}

impl From<reqwest::Error> for AppError {
    fn from(err: reqwest::Error) -> AppError {
        AppError::Network(err)
    }
}

// 使用?操作符简化错误处理
async fn read_and_parse_number() -> Result<i32, AppError> {
    let content = fs::read_to_string("number.txt").await?;
    let number: i32 = content.trim().parse()?;
    Ok(number)
}

// 异步错误传播
async fn complex_async_operation() -> Result<String, AppError> {
    let number = read_and_parse_number().await?;
    
    let client = reqwest::Client::new();
    let response = client
        .get(&format!("https://api.example.com/data/{}", number))
        .send()
        .await?;
    
    let text = response.text().await?;
    Ok(text)
}
```

### 10.2 超时和取消

```rust
use tokio::time::{timeout, Duration};
use tokio_util::sync::CancellationToken;

async fn timeout_example() {
    let result = timeout(Duration::from_secs(2), slow_operation()).await;
    
    match result {
        Ok(value) => println!("Operation completed: {}", value),
        Err(_) => println!("Operation timed out"),
    }
}

async fn slow_operation() -> String {
    tokio::time::sleep(Duration::from_secs(3)).await;
    "Slow result".to_string()
}

// 使用取消令牌
async fn cancellation_example() {
    let token = CancellationToken::new();
    let child_token = token.child_token();
    
    let task = tokio::spawn(cancellable_task(child_token));
    
    // 2秒后取消任务
    tokio::time::sleep(Duration::from_secs(2)).await;
    token.cancel();
    
    match task.await {
        Ok(result) => println!("Task completed: {:?}", result),
        Err(_) => println!("Task was cancelled or panicked"),
    }
}

async fn cancellable_task(token: CancellationToken) -> Result<String, &'static str> {
    for i in 0..10 {
        tokio::select! {
            _ = token.cancelled() => {
                return Err("Task was cancelled");
            }
            _ = tokio::time::sleep(Duration::from_millis(500)) => {
                println!("Step {}", i);
            }
        }
    }
    Ok("Task completed successfully".to_string())
}
```

## 11. 性能优化技巧

### 11.1 避免不必要的async

```rust
// 不好的做法 - 不需要async的函数
async fn bad_sync_function(x: i32) -> i32 {
    x * 2  // 没有异步操作
}

// 好的做法 - 只在需要时使用async
fn good_sync_function(x: i32) -> i32 {
    x * 2
}

// 混合使用
async fn mixed_operations() {
    let sync_result = good_sync_function(10);
    let async_result = actual_async_operation().await;
    println!("Results: {}, {}", sync_result, async_result);
}

async fn actual_async_operation() -> String {
    tokio::time::sleep(Duration::from_millis(10)).await;
    "async result".to_string()
}
```

### 11.2 批量操作

```rust
// 批量处理避免频繁切换
async fn batch_processing(items: Vec<String>) -> Vec<String> {
    let futures: Vec<_> = items
        .into_iter()
        .map(|item| process_item(item))
        .collect();
    
    // 并发处理所有项目
    futures::future::join_all(futures).await
}

async fn process_item(item: String) -> String {
    // 模拟异步处理
    tokio::time::sleep(Duration::from_millis(10)).await;
    format!("processed: {}", item)
}
```

### 11.3 缓冲和背压控制

```rust
use tokio::sync::mpsc;
use futures::stream::{self, StreamExt};

// 控制并发数量防止资源耗尽
async fn controlled_concurrency() {
    let items = (0..1000).collect::<Vec<_>>();
    
    // 使用buffer_unordered限制并发数
    let results: Vec<_> = stream::iter(items)
        .map(|item| expensive_async_operation(item))
        .buffer_unordered(10) // 最多10个并发操作
        .collect()
        .await;
    
    println!("Processed {} items", results.len());
}

async fn expensive_async_operation(item: i32) -> i32 {
    tokio::time::sleep(Duration::from_millis(100)).await;
    item * 2
}

// 使用有界通道实现背压
async fn backpressure_example() {
    let (tx, mut rx) = mpsc::channel::<i32>(10); // 缓冲区大小为10
    
    // 生产者
    let producer = tokio::spawn(async move {
        for i in 0..100 {
            // send会在缓冲区满时阻塞
            if tx.send(i).await.is_err() {
                break;
            }
            println!("Sent: {}", i);
        }
    });
    
    // 慢消费者
    let consumer = tokio::spawn(async move {
        while let Some(item) = rx.recv().await {
            tokio::time::sleep(Duration::from_millis(100)).await;
            println!("Consumed: {}", item);
        }
    });
    
    tokio::join!(producer, consumer);
}
```

## 12. 常见陷阱和最佳实践

### 12.1 避免阻塞异步运行时

```rust
// 错误做法 - 在异步函数中使用阻塞操作
async fn bad_blocking_example() {
    // 这会阻塞整个异步运行时！
    std::thread::sleep(Duration::from_secs(1));
    
    // 这也会阻塞运行时
    let _result = std::fs::read_to_string("file.txt");
}

// 正确做法
async fn good_async_example() {
    // 使用异步sleep
    tokio::time::sleep(Duration::from_secs(1)).await;
    
    // 使用异步文件操作
    let _result = tokio::fs::read_to_string("file.txt").await;
}

// 如果必须使用阻塞操作，使用spawn_blocking
async fn necessary_blocking_example() {
    let result = tokio::task::spawn_blocking(|| {
        // CPU密集型或阻塞操作
        std::thread::sleep(Duration::from_secs(1));
        expensive_cpu_computation()
    }).await.unwrap();
    
    println!("Blocking operation result: {}", result);
}

fn expensive_cpu_computation() -> i32 {
    // 模拟CPU密集型计算
    (0..1_000_000).sum()
}
```

### 12.2 正确处理生命周期

```rust
// 错误：返回借用的异步函数
// async fn bad_lifetime_example(data: &str) -> &str {
//     tokio::time::sleep(Duration::from_millis(10)).await;
//     data // 编译错误：生命周期问题
// }

// 正确做法1：返回拥有的数据
async fn good_lifetime_example1(data: &str) -> String {
    tokio::time::sleep(Duration::from_millis(10)).await;
    data.to_string()
}

// 正确做法2：使用生命周期参数
async fn good_lifetime_example2<'a>(data: &'a str) -> &'a str {
    tokio::time::sleep(Duration::from_millis(10)).await;
    data
}

// 结构体中的异步方法
struct DataProcessor {
    name: String,
}

impl DataProcessor {
    async fn process_data(&self, input: &str) -> String {
        tokio::time::sleep(Duration::from_millis(10)).await;
        format!("{}: processed {}", self.name, input)
    }
    
    // 消费self的异步方法
    async fn finalize(self) -> String {
        tokio::time::sleep(Duration::from_millis(10)).await;
        format!("{} finalized", self.name)
    }
}
```

### 12.3 错误的异步闭包使用

```rust
// 错误做法 - 试图在map中使用async闭包
// async fn bad_async_closure() {
//     let items = vec![1, 2, 3, 4, 5];
//     let results: Vec<_> = items
//         .iter()
//         .map(async |&x| {  // 编译错误！
//             tokio::time::sleep(Duration::from_millis(10)).await;
//             x * 2
//         })
//         .collect();
// }

// 正确做法1：使用futures
async fn good_async_closure1() {
    let items = vec![1, 2, 3, 4, 5];
    let futures: Vec<_> = items
        .iter()
        .map(|&x| async move {
            tokio::time::sleep(Duration::from_millis(10)).await;
            x * 2
        })
        .collect();
    
    let results = futures::future::join_all(futures).await;
    println!("Results: {:?}", results);
}

// 正确做法2：使用Stream
async fn good_async_closure2() {
    use futures::stream::{self, StreamExt};
    
    let items = vec![1, 2, 3, 4, 5];
    let results: Vec<_> = stream::iter(items)
        .then(|x| async move {
            tokio::time::sleep(Duration::from_millis(10)).await;
            x * 2
        })
        .collect()
        .await;
    
    println!("Results: {:?}", results);
}
```

## 13. 高级异步模式

### 13.1 异步递归

```rust
use std::boxed::Box;

// 异步递归需要Box来避免无限大小类型
async fn async_factorial(n: u64) -> u64 {
    if n <= 1 {
        1
    } else {
        n * Box::pin(async_factorial(n - 1)).await
    }
}

// 或者使用async-recursion crate
// use async_recursion::async_recursion;
// 
// #[async_recursion]
// async fn clean_async_factorial(n: u64) -> u64 {
//     if n <= 1 {
//         1
//     } else {
//         n * clean_async_factorial(n - 1).await
//     }
// }

async fn recursion_example() {
    let result = async_factorial(5).await;
    println!("5! = {}", result);
}
```

### 13.2 异步构建器模式

```rust
use std::collections::HashMap;

struct AsyncRequestBuilder {
    url: String,
    headers: HashMap<String, String>,
    timeout: Option<Duration>,
}

impl AsyncRequestBuilder {
    fn new(url: String) -> Self {
        AsyncRequestBuilder {
            url,
            headers: HashMap::new(),
            timeout: None,
        }
    }
    
    fn header(mut self, key: String, value: String) -> Self {
        self.headers.insert(key, value);
        self
    }
    
    fn timeout(mut self, duration: Duration) -> Self {
        self.timeout = Some(duration);
        self
    }
    
    async fn send(self) -> Result<String, reqwest::Error> {
        let client = reqwest::Client::new();
        let mut request = client.get(&self.url);
        
        for (key, value) in self.headers {
            request = request.header(&key, &value);
        }
        
        if let Some(timeout) = self.timeout {
            request = request.timeout(timeout);
        }
        
        let response = request.send().await?;
        let text = response.text().await?;
        Ok(text)
    }
}

async fn builder_pattern_example() {
    let result = AsyncRequestBuilder::new("https://httpbin.org/get".to_string())
        .header("User-Agent".to_string(), "MyApp/1.0".to_string())
        .timeout(Duration::from_secs(10))
        .send()
        .await;
    
    match result {
        Ok(text) => println!("Response: {}", text),
        Err(e) => println!("Error: {}", e),
    }
}
```

### 13.3 异步状态机

```rust
// 手动实现状态机来理解async/await的工作原理
enum ConnectionState {
    Connecting,
    Connected,
    Authenticated,
    Closed,
}

struct Connection {
    state: ConnectionState,
    address: String,
}

impl Connection {
    fn new(address: String) -> Self {
        Connection {
            state: ConnectionState::Connecting,
            address,
        }
    }
    
    async fn connect(&mut self) -> Result<(), &'static str> {
        match self.state {
            ConnectionState::Connecting => {
                // 模拟连接过程
                tokio::time::sleep(Duration::from_millis(100)).await;
                self.state = ConnectionState::Connected;
                Ok(())
            }
            _ => Err("Invalid state for connection"),
        }
    }
    
    async fn authenticate(&mut self, credentials: &str) -> Result<(), &'static str> {
        match self.state {
            ConnectionState::Connected => {
                // 模拟认证过程
                tokio::time::sleep(Duration::from_millis(50)).await;
                if credentials == "secret" {
                    self.state = ConnectionState::Authenticated;
                    Ok(())
                } else {
                    Err("Authentication failed")
                }
            }
            _ => Err("Must be connected to authenticate"),
        }
    }
    
    async fn send_data(&self, data: &str) -> Result<(), &'static str> {
        match self.state {
            ConnectionState::Authenticated => {
                tokio::time::sleep(Duration::from_millis(20)).await;
                println!("Sent: {}", data);
                Ok(())
            }
            _ => Err("Must be authenticated to send data"),
        }
    }
}

async fn state_machine_example() {
    let mut conn = Connection::new("example.com:8080".to_string());
    
    if let Err(e) = conn.connect().await {
        println!("Connection failed: {}", e);
        return;
    }
    
    if let Err(e) = conn.authenticate("secret").await {
        println!("Authentication failed: {}", e);
        return;
    }
    
    if let Err(e) = conn.send_data("Hello, server!").await {
        println!("Send failed: {}", e);
        return;
    }
    
    println!("Operation completed successfully");
}
```

## 14. 异步测试

### 14.1 基本异步测试

```rust
#[cfg(test)]
mod tests {
    use super::*;
    use tokio::test;
    
    #[tokio::test]
    async fn test_async_function() {
        let result = async_factorial(4).await;
        assert_eq!(result, 24);
    }
    
    #[tokio::test]
    async fn test_timeout() {
        let result = tokio::time::timeout(
            Duration::from_millis(100),
            slow_operation()
        ).await;
        
        assert!(result.is_err()); // 应该超时
    }
    
    #[tokio::test]
    async fn test_concurrent_operations() {
        let start = std::time::Instant::now();
        
        let (a, b, c) = tokio::join!(
            tokio::time::sleep(Duration::from_millis(100)),
            tokio::time::sleep(Duration::from_millis(100)),
            tokio::time::sleep(Duration::from_millis(100))
        );
        
        let duration = start.elapsed();
        // 并发执行应该接近100ms，而不是300ms
        assert!(duration < Duration::from_millis(150));
    }
}
```

### 14.2 模拟和Mock

```rust
// 使用trait来支持测试中的模拟
#[async_trait::async_trait]
trait DataService {
    async fn fetch_data(&self, id: u32) -> Result<String, &'static str>;
}

struct RealDataService;

#[async_trait::async_trait]
impl DataService for RealDataService {
    async fn fetch_data(&self, id: u32) -> Result<String, &'static str> {
        // 真实的网络请求
        tokio::time::sleep(Duration::from_millis(100)).await;
        Ok(format!("Real data for ID: {}", id))
    }
}

struct MockDataService;

#[async_trait::async_trait]
impl DataService for MockDataService {
    async fn fetch_data(&self, id: u32) -> Result<String, &'static str> {
        // 模拟响应，无网络延迟
        if id == 999 {
            Err("Not found")
        } else {
            Ok(format!("Mock data for ID: {}", id))
        }
    }
}

async fn process_user_data(service: &dyn DataService, user_id: u32) -> String {
    match service.fetch_data(user_id).await {
        Ok(data) => format!("Processed: {}", data),
        Err(e) => format!("Error: {}", e),
    }
}

#[cfg(test)]
mod service_tests {
    use super::*;
    
    #[tokio::test]
    async fn test_successful_data_processing() {
        let service = MockDataService;
        let result = process_user_data(&service, 123).await;
        assert_eq!(result, "Processed: Mock data for ID: 123");
    }
    
    #[tokio::test]
    async fn test_error_handling() {
        let service = MockDataService;
        let result = process_user_data(&service, 999).await;
        assert_eq!(result, "Error: Not found");
    }
}
```

## 15. 实际应用示例

### 15.1 简单的异步Web服务器

```rust
// Cargo.toml
/*
[dependencies]
tokio = { version = "1.0", features = ["full"] }
hyper = { version = "0.14", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
*/

use hyper::service::{make_service_fn, service_fn};
use hyper::{Body, Request, Response, Server, StatusCode};
use serde::{Deserialize, Serialize};
use std::convert::Infallible;
use std::net::SocketAddr;

#[derive(Serialize, Deserialize)]
struct User {
    id: u32,
    name: String,
    email: String,
}

async fn handle_request(req: Request<Body>) -> Result<Response<Body>, Infallible> {
    match (req.method(), req.uri().path()) {
        (&hyper::Method::GET, "/users") => {
            let users = vec![
                User { id: 1, name: "Alice".to_string(), email: "alice@example.com".to_string() },
                User { id: 2, name: "Bob".to_string(), email: "bob@example.com".to_string() },
            ];
            
            let json = serde_json::to_string(&users).unwrap();
            Ok(Response::new(Body::from(json)))
        }
        (&hyper::Method::GET, path) if path.starts_with("/users/") => {
            let id_str = &path[7..]; // 去掉"/users/"
            match id_str.parse::<u32>() {
                Ok(id) => {
                    let user = User {
                        id,
                        name: format!("User {}", id),
                        email: format!("user{}@example.com", id),
                    };
                    let json = serde_json::to_string(&user).unwrap();
                    Ok(Response::new(Body::from(json)))
                }
                Err(_) => {
                    Ok(Response::builder()
                        .status(StatusCode::BAD_REQUEST)
                        .body(Body::from("Invalid user ID"))
                        .unwrap())
                }
            }
        }
        _ => {
            Ok(Response::builder()
                .status(StatusCode::NOT_FOUND)
                .body(Body::from("Not Found"))
                .unwrap())
        }
    }
}

async fn web_server_example() {
    let addr = SocketAddr::from(([127, 0, 0, 1], 3000));
    
    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle_request))
    });
    
    let server = Server::bind(&addr).serve(make_svc);
    
    println!("Server running on http://{}", addr);
    
    if let Err(e) = server.await {
        eprintln!("Server error: {}", e);
    }
}
```

### 15.2 异步任务队列

```rust
use tokio::sync::{mpsc, Mutex};
use std::sync::Arc;
use std::collections::VecDeque;

#[derive(Debug, Clone)]
struct Task {
    id: u32,
    data: String,
}

struct TaskQueue {
    queue: Arc<Mutex<VecDeque<Task>>>,
    sender: mpsc::Sender<Task>,
    receiver: Arc<Mutex<mpsc::Receiver<Task>>>,
}

impl TaskQueue {
    fn new() -> Self {
        let (sender, receiver) = mpsc::channel::<Task>(100);
        TaskQueue {
            queue: Arc::new(Mutex::new(VecDeque::new())),
            sender,
            receiver: Arc::new(Mutex::new(receiver)),
        }
    }
    
    async fn enqueue(&self, task: Task) -> Result<(), &'static str> {
        self.sender.send(task).await
            .map_err(|_| "Failed to enqueue task")
    }
    
    async fn start_workers(&self, num_workers: usize) {
        for worker_id in 0..num_workers {
            let receiver = Arc::clone(&self.receiver);
            tokio::spawn(async move {
                println!("Worker {} started", worker_id);
                
                loop {
                    let task = {
                        let mut rx = receiver.lock().await;
                        rx.recv().await
                    };
                    
                    match task {
                        Some(task) => {
                            println!("Worker {} processing task {}: {}", worker_id, task.id, task.data);
                            
                            // 模拟任务处理
                            tokio::time::sleep(Duration::from_millis(500)).await;
                            
                            println!("Worker {} completed task {}", worker_id, task.id);
                        }
                        None => {
                            println!("Worker {} shutting down", worker_id);
                            break;
                        }
                    }
                }
            });
        }
    }
}

async fn task_queue_example() {
    let queue = TaskQueue::new();
    
    // 启动3个工作线程
    queue.start_workers(3).await;
    
    // 添加一些任务
    for i in 1..=10 {
        let task = Task {
            id: i,
            data: format!("Task data {}", i),
        };
        queue.enqueue(task).await.unwrap();
    }
    
    // 等待所有任务完成
    tokio::time::sleep(Duration::from_secs(5)).await;
    println!("All tasks processed");
}
```

## 16. 调试和性能分析

### 16.1 异步代码调试技巧

```rust
// 使用tracing进行结构化日志记录
// Cargo.toml
/*
[dependencies]
tracing = "0.1"
tracing-subscriber = "0.3"
*/

use tracing::{info, warn, error, debug, instrument};

#[instrument]
async fn traced_async_function(user_id: u32) -> Result<String, &'static str> {
    info!("Starting operation for user {}", user_id);
    
    // 模拟异步工作
    tokio::time::sleep(Duration::from_millis(100)).await;
    debug!("Completed async work");
    
    if user_id == 0 {
        error!("Invalid user ID: 0");
        return Err("Invalid user ID");
    }
    
    info!("Operation completed successfully");
    Ok(format!("Result for user {}", user_id))
}

async fn debugging_example() {
    // 初始化tracing
    tracing_subscriber::fmt::init();
    
    info!("Application starting");
    
    let result = traced_async_function(123).await;
    match result {
        Ok(data) => info!("Success: {}", data),
        Err(e) => error!("Error: {}", e),
    }
}
```

### 16.2 性能监控

```rust
use std::time::Instant;

async fn performance_monitoring_example() {
    // 简单的时间测量
    let start = Instant::now();
    let result = expensive_async_operation(42).await;
    let duration = start.elapsed();
    
    println!("Operation took {:?}, result: {}", duration, result);
    
    // 更详细的性能分析
    let metrics = measure_async_performance().await;
    println!("Performance metrics: {:?}", metrics);
}

#[derive(Debug)]
struct PerformanceMetrics {
    total_time: Duration,
    setup_time: Duration,
    processing_time: Duration,
    cleanup_time: Duration,
}

async fn measure_async_performance() -> PerformanceMetrics {
    let total_start = Instant::now();
    
    // 设置阶段
    let setup_start = Instant::now();
    setup_operation().await;
    let setup_time = setup_start.elapsed();
    
    // 处理阶段
    let processing_start = Instant::now();
    processing_operation().await;
    let processing_time = processing_start.elapsed();
    
    // 清理阶段
    let cleanup_start = Instant::now();
    cleanup_operation().await;
    let cleanup_time = cleanup_start.elapsed();
    
    let total_time = total_start.elapsed();
    
    PerformanceMetrics {
        total_time,
        setup_time,
        processing_time,
        cleanup_time,
    }
}

async fn setup_operation() {
    tokio::time::sleep(Duration::from_millis(50)).await;
}

async fn processing_operation() {
    tokio::time::sleep(Duration::from_millis(200)).await;
}

async fn cleanup_operation() {
    tokio::time::sleep(Duration::from_millis(30)).await;
}
```

## 17. 总结和学习路径

### 17.1 异步编程核心概念回顾

1. **Future Trait**: 异步计算的抽象
2. **async/await**: 语法糖，让异步代码看起来像同步代码
3. **运行时**: 执行器，负责驱动Future完成
4. **并发vs并行**: 异步主要解决并发问题
5. **非阻塞I/O**: 异步编程的核心优势

### 17.2 最佳实践总结

- **避免在async函数中使用阻塞操作**
- **正确处理错误和超时**
- **合理使用并发控制（信号量、缓冲等）**
- **选择合适的运行时（Tokio vs async-std）**
- **使用结构化日志记录（tracing）**
- **编写测试验证异步行为**

### 17.3 进阶学习建议

1. **深入理解运行时内部机制**
2. **学习更多异步crate生态**（async-trait, futures-util等）
3. **研究异步Web框架**（axum, warp, actix-web）
4. **探索异步数据库驱动**（sqlx, tokio-postgres）
5. **学习分布式异步系统设计**

### 17.4 常用异步crate推荐

```toml
[dependencies]
# 运行时
tokio = { version = "1.0", features = ["full"] }
async-std = "1.12"

# 异步工具
futures = "0.3"
async-trait = "0.1"
pin-project = "1.0"

# 网络和HTTP
reqwest = { version = "0.11", features = ["json"] }
hyper = { version = "0.14", features = ["full"] }

# Web框架
axum = "0.7"
warp = "0.3"

# 数据库
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres"] }

# 日志和监控
tracing = "0.1"
tracing-subscriber = "0.3"
```

异步编程是现代Rust开发的重要组成部分，特别是在构建高性能的网络应用和服务时。通过理解Future、async/await语法、运行时机制以及各种并发模式，你可以构建出高效、可靠的异步应用程序。