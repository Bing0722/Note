# Rust生态系统常用Crate详解

## 1. 异步运行时和并发

### 1.1 Tokio - 异步运行时之王

```toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
```

**核心功能**：

- 异步运行时
- 网络I/O（TCP/UDP）
- 文件系统操作
- 定时器和延迟
- 同步原语（Mutex、RwLock、Semaphore等）

```rust
use tokio::net::{TcpListener, TcpStream};
use tokio::io::{AsyncReadExt, AsyncWriteExt};

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    // TCP服务器示例
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server running on 127.0.0.1:8080");
    
    while let Ok((stream, addr)) = listener.accept().await {
        println!("New connection: {}", addr);
        tokio::spawn(handle_connection(stream));
    }
    
    Ok(())
}

async fn handle_connection(mut stream: TcpStream) -> Result<(), Box<dyn std::error::Error>> {
    let mut buffer = [0; 1024];
    let n = stream.read(&mut buffer).await?;
    let request = String::from_utf8_lossy(&buffer[..n]);
    
    let response = "HTTP/1.1 200 OK\r\n\r\nHello from Tokio!";
    stream.write_all(response.as_bytes()).await?;
    
    Ok(())
}
```

**常用features**：

```toml
tokio = { version = "1.0", features = [
    "macros",      # #[tokio::main], #[tokio::test]
    "rt-multi-thread", # 多线程运行时
    "net",         # 网络功能
    "fs",          # 文件系统
    "time",        # 定时器
    "sync",        # 同步原语
    "signal",      # 信号处理
    "process",     # 进程管理
] }
```

### 1.2 async-std - 标准库风格的异步运行时

```toml
[dependencies]
async-std = { version = "1.12", features = ["attributes", "tokio1"] }
use async_std::net::{TcpListener, TcpStream};
use async_std::prelude::*;
use async_std::task;

#[async_std::main]
async fn main() -> std::io::Result<()> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    
    let mut incoming = listener.incoming();
    while let Some(stream) = incoming.next().await {
        let stream = stream?;
        task::spawn(async {
            handle_connection(stream).await.unwrap();
        });
    }
    
    Ok(())
}

async fn handle_connection(mut stream: TcpStream) -> std::io::Result<()> {
    let mut buffer = [0; 1024];
    stream.read(&mut buffer).await?;
    stream.write_all(b"HTTP/1.1 200 OK\r\n\r\nHello!").await?;
    Ok(())
}
```

### 1.3 rayon - 数据并行处理

```toml
[dependencies]
rayon = "1.8"
use rayon::prelude::*;

fn parallel_processing_example() {
    let numbers: Vec<i32> = (0..1_000_000).collect();
    
    // 并行映射
    let squares: Vec<i32> = numbers
        .par_iter()
        .map(|&x| x * x)
        .collect();
    
    // 并行过滤和归约
    let sum_of_evens: i32 = numbers
        .par_iter()
        .filter(|&&x| x % 2 == 0)
        .sum();
    
    println!("Sum of evens: {}", sum_of_evens);
    
    // 并行排序
    let mut data = vec![3, 1, 4, 1, 5, 9, 2, 6, 5, 3];
    data.par_sort();
    println!("Sorted: {:?}", data);
}

// 自定义并行迭代
fn custom_parallel_work() {
    let data = vec!["apple", "banana", "cherry", "date"];
    
    let results: Vec<String> = data
        .par_iter()
        .map(|&fruit| {
            // 模拟计算密集型操作
            std::thread::sleep(std::time::Duration::from_millis(100));
            format!("processed: {}", fruit)
        })
        .collect();
    
    println!("{:?}", results);
}
```

## 2. Web框架和HTTP

### 2.1 axum - 现代化Web框架

```toml
[dependencies]
axum = "0.7"
tokio = { version = "1.0", features = ["full"] }
tower = "0.4"
serde = { version = "1.0", features = ["derive"] }
use axum::{
    extract::{Path, Query, Json},
    response::{Html, Json as ResponseJson},
    routing::{get, post},
    Router,
    http::StatusCode,
};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize)]
struct User {
    id: u64,
    name: String,
    email: String,
}

#[derive(Deserialize)]
struct CreateUser {
    name: String,
    email: String,
}

#[derive(Deserialize)]
struct Pagination {
    page: Option<usize>,
    limit: Option<usize>,
}

async fn main() {
    let app = Router::new()
        .route("/", get(root))
        .route("/users", get(get_users).post(create_user))
        .route("/users/:id", get(get_user))
        .route("/health", get(health_check));
    
    let listener = tokio::net::TcpListener::bind("0.0.0.0:3000")
        .await
        .unwrap();
        
    println!("Server running on http://0.0.0.0:3000");
    axum::serve(listener, app).await.unwrap();
}

async fn root() -> Html<&'static str> {
    Html("<h1>Hello, Axum!</h1>")
}

async fn health_check() -> StatusCode {
    StatusCode::OK
}

async fn get_users(Query(pagination): Query<Pagination>) -> ResponseJson<Vec<User>> {
    let page = pagination.page.unwrap_or(1);
    let limit = pagination.limit.unwrap_or(10);
    
    // 模拟数据库查询
    let users = vec![
        User { id: 1, name: "Alice".to_string(), email: "alice@example.com".to_string() },
        User { id: 2, name: "Bob".to_string(), email: "bob@example.com".to_string() },
    ];
    
    ResponseJson(users)
}

async fn get_user(Path(id): Path<u64>) -> Result<ResponseJson<User>, StatusCode> {
    // 模拟数据库查询
    let user = User {
        id,
        name: format!("User {}", id),
        email: format!("user{}@example.com", id),
    };
    
    Ok(ResponseJson(user))
}

async fn create_user(Json(payload): Json<CreateUser>) -> Result<ResponseJson<User>, StatusCode> {
    let user = User {
        id: 123, // 模拟生成的ID
        name: payload.name,
        email: payload.email,
    };
    
    Ok(ResponseJson(user))
}
```

### 2.2 reqwest - HTTP客户端

```toml
[dependencies]
reqwest = { version = "0.11", features = ["json", "multipart"] }
use reqwest::{Client, multipart};
use serde::{Deserialize, Serialize};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct ApiResponse {
    data: Vec<Post>,
}

#[derive(Serialize, Deserialize, Debug)]
struct Post {
    id: u32,
    title: String,
    body: String,
}

async fn http_client_examples() -> Result<(), reqwest::Error> {
    let client = Client::new();
    
    // GET请求
    let response = client
        .get("https://jsonplaceholder.typicode.com/posts")
        .header("User-Agent", "MyApp/1.0")
        .send()
        .await?;
    
    let posts: Vec<Post> = response.json().await?;
    println!("Retrieved {} posts", posts.len());
    
    // POST请求 - JSON
    let new_post = Post {
        id: 0,
        title: "My New Post".to_string(),
        body: "This is the content".to_string(),
    };
    
    let response = client
        .post("https://jsonplaceholder.typicode.com/posts")
        .json(&new_post)
        .send()
        .await?;
    
    println!("POST response: {}", response.status());
    
    // POST请求 - Form data
    let mut form_data = HashMap::new();
    form_data.insert("name", "John Doe");
    form_data.insert("email", "john@example.com");
    
    let response = client
        .post("https://httpbin.org/post")
        .form(&form_data)
        .send()
        .await?;
    
    let text = response.text().await?;
    println!("Form response: {}", text);
    
    // 文件上传
    let form = multipart::Form::new()
        .text("field", "value")
        .file("file", "path/to/file.txt")
        .await?;
    
    let response = client
        .post("https://httpbin.org/post")
        .multipart(form)
        .send()
        .await?;
    
    Ok(())
}

// 带重试的HTTP客户端
async fn resilient_http_client() -> Result<(), Box<dyn std::error::Error>> {
    let client = Client::builder()
        .timeout(std::time::Duration::from_secs(10))
        .build()?;
    
    let max_retries = 3;
    let mut retries = 0;
    
    loop {
        match client.get("https://httpbin.org/delay/1").send().await {
            Ok(response) => {
                println!("Success: {}", response.status());
                break;
            }
            Err(e) if retries < max_retries => {
                retries += 1;
                println!("Retry {}/{}: {}", retries, max_retries, e);
                tokio::time::sleep(std::time::Duration::from_secs(1)).await;
            }
            Err(e) => {
                println!("Failed after {} retries: {}", max_retries, e);
                return Err(e.into());
            }
        }
    }
    
    Ok(())
}
```

### 2.3 hyper - 低级HTTP库

```toml
[dependencies]
hyper = { version = "0.14", features = ["full"] }
use hyper::service::{make_service_fn, service_fn};
use hyper::{Body, Request, Response, Server, Method, StatusCode};
use std::convert::Infallible;

async fn hyper_server_example() {
    let make_svc = make_service_fn(|_conn| async {
        Ok::<_, Infallible>(service_fn(handle_request))
    });
    
    let addr = ([127, 0, 0, 1], 3000).into();
    let server = Server::bind(&addr).serve(make_svc);
    
    println!("Server running on http://{}", addr);
    
    if let Err(e) = server.await {
        eprintln!("Server error: {}", e);
    }
}

async fn handle_request(req: Request<Body>) -> Result<Response<Body>, Infallible> {
    match (req.method(), req.uri().path()) {
        (&Method::GET, "/") => {
            Ok(Response::new(Body::from("Hello, Hyper!")))
        }
        (&Method::POST, "/echo") => {
            Ok(Response::new(req.into_body()))
        }
        _ => {
            let mut not_found = Response::new(Body::from("Not Found"));
            *not_found.status_mut() = StatusCode::NOT_FOUND;
            Ok(not_found)
        }
    }
}
```

## 3. 数据库和ORM

### 3.1 sqlx - 异步SQL库

```toml
[dependencies]
sqlx = { version = "0.7", features = ["runtime-tokio-rustls", "postgres", "chrono", "uuid"] }
use sqlx::{PgPool, Row};
use chrono::{DateTime, Utc};

#[derive(sqlx::FromRow, Debug)]
struct User {
    id: i32,
    name: String,
    email: String,
    created_at: DateTime<Utc>,
}

async fn sqlx_examples() -> Result<(), sqlx::Error> {
    // 连接数据库
    let database_url = "postgresql://username:password@localhost/database";
    let pool = PgPool::connect(database_url).await?;
    
    // 原始SQL查询
    let rows = sqlx::query("SELECT id, name, email FROM users WHERE active = $1")
        .bind(true)
        .fetch_all(&pool)
        .await?;
    
    for row in rows {
        let id: i32 = row.get("id");
        let name: String = row.get("name");
        println!("User: {} - {}", id, name);
    }
    
    // 使用结构体映射
    let users: Vec<User> = sqlx::query_as::<_, User>(
        "SELECT id, name, email, created_at FROM users WHERE created_at > $1"
    )
    .bind(Utc::now() - chrono::Duration::days(30))
    .fetch_all(&pool)
    .await?;
    
    println!("Found {} recent users", users.len());
    
    // 插入数据
    let new_user = sqlx::query!(
        "INSERT INTO users (name, email) VALUES ($1, $2) RETURNING id",
        "John Doe",
        "john@example.com"
    )
    .fetch_one(&pool)
    .await?;
    
    println!("Created user with ID: {}", new_user.id);
    
    // 事务
    let mut tx = pool.begin().await?;
    
    sqlx::query!("UPDATE users SET name = $1 WHERE id = $2", "Jane Doe", new_user.id)
        .execute(&mut *tx)
        .await?;
    
    sqlx::query!("INSERT INTO user_logs (user_id, action) VALUES ($1, $2)", new_user.id, "updated")
        .execute(&mut *tx)
        .await?;
    
    tx.commit().await?;
    
    Ok(())
}
```

### 3.2 diesel - 强类型ORM

```toml
[dependencies]
diesel = { version = "2.1", features = ["postgres", "chrono"] }
diesel_migrations = "2.1"
// schema.rs (通过diesel print-schema生成)
table! {
    users (id) {
        id -> Int4,
        name -> Varchar,
        email -> Varchar,
        created_at -> Timestamptz,
    }
}

table! {
    posts (id) {
        id -> Int4,
        user_id -> Int4,
        title -> Varchar,
        content -> Text,
        published -> Bool,
    }
}

// models.rs
use diesel::prelude::*;
use chrono::{DateTime, Utc};

#[derive(Queryable, Debug)]
pub struct User {
    pub id: i32,
    pub name: String,
    pub email: String,
    pub created_at: DateTime<Utc>,
}

#[derive(Insertable)]
#[diesel(table_name = users)]
pub struct NewUser<'a> {
    pub name: &'a str,
    pub email: &'a str,
}

#[derive(Queryable, Associations, Debug)]
#[diesel(belongs_to(User))]
pub struct Post {
    pub id: i32,
    pub user_id: i32,
    pub title: String,
    pub content: String,
    pub published: bool,
}

// main.rs
use diesel::pg::PgConnection;
use diesel::prelude::*;

fn diesel_examples() -> Result<(), Box<dyn std::error::Error>> {
    use crate::schema::users::dsl::*;
    
    let database_url = "postgresql://username:password@localhost/database";
    let mut connection = PgConnection::establish(&database_url)?;
    
    // 查询所有用户
    let all_users = users.load::<User>(&mut connection)?;
    println!("Found {} users", all_users.len());
    
    // 条件查询
    let john = users
        .filter(name.eq("John"))
        .first::<User>(&mut connection)
        .optional()?;
    
    if let Some(user) = john {
        println!("Found John: {:?}", user);
    }
    
    // 插入新用户
    let new_user = NewUser {
        name: "Alice",
        email: "alice@example.com",
    };
    
    let inserted_user: User = diesel::insert_into(users)
        .values(&new_user)
        .get_result(&mut connection)?;
    
    println!("Inserted user: {:?}", inserted_user);
    
    // 更新用户
    let updated_count = diesel::update(users.find(inserted_user.id))
        .set(name.eq("Alice Smith"))
        .execute(&mut connection)?;
    
    println!("Updated {} users", updated_count);
    
    Ok(())
}
```

### 3.3 redis - Redis客户端

```toml
[dependencies]
redis = { version = "0.24", features = ["tokio-comp"] }
use redis::AsyncCommands;

async fn redis_examples() -> redis::RedisResult<()> {
    // 连接Redis
    let client = redis::Client::open("redis://127.0.0.1/")?;
    let mut con = client.get_async_connection().await?;
    
    // 基本操作
    let _: () = con.set("my_key", 42).await?;
    let value: i32 = con.get("my_key").await?;
    println!("Value: {}", value);
    
    // 列表操作
    let _: () = con.lpush("my_list", vec!["item1", "item2", "item3"]).await?;
    let items: Vec<String> = con.lrange("my_list", 0, -1).await?;
    println!("List items: {:?}", items);
    
    // 哈希操作
    let _: () = con.hset_multiple("user:1", &[
        ("name", "John"),
        ("email", "john@example.com"),
        ("age", "30")
    ]).await?;
    
    let user_data: std::collections::HashMap<String, String> = con.hgetall("user:1").await?;
    println!("User data: {:?}", user_data);
    
    // 发布/订阅
    let mut pubsub = con.into_pubsub();
    pubsub.subscribe("channel1").await?;
    
    // 在另一个连接中发布消息
    let mut pub_con = client.get_async_connection().await?;
    let _: () = pub_con.publish("channel1", "Hello, Redis!").await?;
    
    // 接收消息
    let msg = pubsub.on_message().next().await;
    if let Some(msg) = msg {
        let payload: String = msg.get_payload()?;
        println!("Received: {}", payload);
    }
    
    Ok(())
}
```

## 4. 序列化和反序列化

### 4.1 serde - 序列化框架

```toml
[dependencies]
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
serde_yaml = "0.9"
toml = "0.8"
use serde::{Serialize, Deserialize, Serializer, Deserializer};
use std::collections::HashMap;

#[derive(Serialize, Deserialize, Debug)]
struct User {
    id: u64,
    name: String,
    email: String,
    #[serde(skip_serializing_if = "Option::is_none")]
    phone: Option<String>,
    #[serde(default)]
    is_active: bool,
    #[serde(rename = "created_at")]
    creation_date: String,
}

#[derive(Serialize, Deserialize, Debug)]
struct Config {
    database: DatabaseConfig,
    server: ServerConfig,
    features: HashMap<String, bool>,
}

#[derive(Serialize, Deserialize, Debug)]
struct DatabaseConfig {
    host: String,
    port: u16,
    #[serde(skip)]
    password: String, // 不会被序列化
}

#[derive(Serialize, Deserialize, Debug)]
struct ServerConfig {
    #[serde(default = "default_port")]
    port: u16,
    #[serde(with = "string_or_number")]
    timeout: u64,
}

fn default_port() -> u16 {
    8080
}

// 自定义序列化/反序列化
mod string_or_number {
    use serde::{de, Deserialize, Deserializer, Serializer};
    
    pub fn serialize<S>(value: &u64, serializer: S) -> Result<S::Ok, S::Error>
    where
        S: Serializer,
    {
        serializer.serialize_u64(*value)
    }
    
    pub fn deserialize<'de, D>(deserializer: D) -> Result<u64, D::Error>
    where
        D: Deserializer<'de>,
    {
        #[derive(Deserialize)]
        #[serde(untagged)]
        enum StringOrNumber {
            String(String),
            Number(u64),
        }
        
        match StringOrNumber::deserialize(deserializer)? {
            StringOrNumber::String(s) => s.parse().map_err(de::Error::custom),
            StringOrNumber::Number(n) => Ok(n),
        }
    }
}

fn serde_examples() -> Result<(), Box<dyn std::error::Error>> {
    // JSON序列化
    let user = User {
        id: 1,
        name: "John Doe".to_string(),
        email: "john@example.com".to_string(),
        phone: Some("+1234567890".to_string()),
        is_active: true,
        creation_date: "2023-01-01".to_string(),
    };
    
    let json = serde_json::to_string_pretty(&user)?;
    println!("JSON:\n{}", json);
    
    // JSON反序列化
    let user_from_json: User = serde_json::from_str(&json)?;
    println!("Deserialized: {:?}", user_from_json);
    
    // YAML处理
    let yaml_str = r#"
database:
  host: localhost
  port: 5432
server:
  port: 3000
  timeout: "30"
features:
  auth: true
  cache: false
"#;
    
    let config: Config = serde_yaml::from_str(yaml_str)?;
    println!("Config: {:?}", config);
    
    // TOML处理
    let toml_str = r#"
[database]
host = "localhost"
port = 5432

[server]
port = 3000
timeout = 30

[features]
auth = true
cache = false
"#;
    
    let config_from_toml: Config = toml::from_str(toml_str)?;
    println!("Config from TOML: {:?}", config_from_toml);
    
    Ok(())
}
```

### 4.2 bincode - 二进制序列化

```toml
[dependencies]
bincode = "1.3"
use bincode;

fn bincode_example() -> Result<(), Box<dyn std::error::Error>> {
    let user = User {
        id: 1,
        name: "John Doe".to_string(),
        email: "john@example.com".to_string(),
        phone: Some("+1234567890".to_string()),
        is_active: true,
        creation_date: "2023-01-01".to_string(),
    };
    
    // 序列化为二进制
    let encoded: Vec<u8> = bincode::serialize(&user)?;
    println!("Encoded size: {} bytes", encoded.len());
    
    // 从二进制反序列化
    let decoded: User = bincode::deserialize(&encoded)?;
    println!("Decoded: {:?}", decoded);
    
    Ok(())
}
```

## 5. 命令行工具

### 5.1 clap - 命令行参数解析

```toml
[dependencies]
clap = { version = "4.0", features = ["derive"] }
use clap::{Parser, Subcommand, ValueEnum};

#[derive(Parser, Debug)]
#[command(name = "myapp")]
#[command(about = "A sample CLI application")]
#[command(version = "1.0")]
struct Args {
    /// 输入文件
    #[arg(short, long)]
    input: String,
    
    /// 输出文件
    #[arg(short, long)]
    output: Option<String>,
    
    /// 详细输出
    #[arg(short, long, action = clap::ArgAction::Count)]
    verbose: u8,
    
    /// 格式
    #[arg(short, long, default_value = "json")]
    format: Format,
    
    /// 子命令
    #[command(subcommand)]
    command: Commands,
}

#[derive(Subcommand, Debug)]
enum Commands {
    /// 处理文件
    Process {
        /// 并发数
        #[arg(short, long, default_value = "1")]
        threads: usize,
        /// 是否强制覆盖
        #[arg(short, long)]
        force: bool,
    },
    /// 验证文件
    Validate {
        /// 严格模式
        #[arg(short, long)]
        strict: bool,
    },
}

#[derive(ValueEnum, Clone, Debug)]
enum Format {
    Json,
    Yaml,
    Toml,
}

fn main() {
    let args = Args::parse();
    
    match args.verbose {
        0 => println!("Running in quiet mode"),
        1 => println!("Running in normal mode"),
        2 => println!("Running in verbose mode"),
        _ => println!("Running in debug mode"),
    }
    
    println!("Input file: {}", args.input);
    if let Some(output) = args.output {
        println!("Output file: {}", output);
    }
    
    println!("Format: {:?}", args.format);
    
    match args.command {
        Commands::Process { threads, force } => {
            println!("Processing with {} threads, force: {}", threads, force);
        }
        Commands::Validate { strict } => {
            println!("Validating, strict mode: {}", strict);
        }
    }
}
```

### 5.2 color-eyre - 错误报告

```toml
[dependencies]
color-eyre = "0.6"
eyre = "0.6"
use color_eyre::{eyre::{Result, eyre, Context}, Report};

fn main() -> Result<()> {
    color_eyre::install()?;
    
    let result = risky_operation()
        .wrap_err("Failed to perform risky operation")
        .wrap_err("Application startup failed");
    
    match result {
        Ok(value) => println!("Success: {}", value),
        Err(e) => {
            eprintln!("Error: {:?}", e);
            return Err(e);
        }
    }
    
    Ok(())
}

fn risky_operation() -> Result<String> {
    let data = read_config_file()
        .wrap_err("Could not read configuration")?;
    
    if data.is_empty() {
        return Err(eyre!("Configuration file is empty"));
    }
    
    Ok(format!("Processed: {}", data))
}

fn read_config_file() -> Result<String> {
    std::fs::read_to_string("config.txt")
        .wrap_err("Failed to read config.txt")
}
```

### 5.3 indicatif - 进度条

```toml
[dependencies]
indicatif = "0.17"
use indicatif::{ProgressBar, ProgressStyle, MultiProgress};
use std::time::Duration;

fn progress_examples() {
    // 简单进度条
    let pb = ProgressBar::new(1000);
    pb.set_style(
        ProgressStyle::default_bar()
            .template("{spinner:.green} [{elapsed_precise}] [{bar:40.cyan/blue}] {pos}/{len} ({eta})")
            .unwrap()
            .progress_chars("#>-"),
    );
    
    for i in 0..1000 {
        std::thread::sleep(Duration::from_millis(5));
        pb.inc(1);
    }
    pb.finish_with_message("完成！");
    
    // 多个进度条
    let multi = MultiProgress::new();
    let pb1 = multi.add(ProgressBar::new(100));
    let pb2 = multi.add(ProgressBar::new(200));
    
    pb1.set_style(
        ProgressStyle::default_bar()
            .template("Task 1: {bar:40.cyan/blue} {pos}/{len}")
            .unwrap()
    );
    
    pb2.set_style(
        ProgressStyle::default_bar()
            .template("Task 2: {bar:40.green/red} {pos}/{len}")
            .unwrap()
    );
    
    let handle1 = std::thread::spawn(move || {
        for _ in 0..100 {
            std::thread::sleep(Duration::from_millis(20));
            pb1.inc(1);
        }
        pb1.finish_with_message("Task 1 完成");
    });
    
    let handle2 = std::thread::spawn(move || {
        for _ in 0..200 {
            std::thread::sleep(Duration::from_millis(10));
            pb2.inc(1);
        }
        pb2.finish_with_message("Task 2 完成");
    });
    
    handle1.join().unwrap();
    handle2.join().unwrap();
}
```

## 6. 日志和监控

### 6.1 tracing - 结构化日志

```toml
[dependencies]
tracing = "0.1"
tracing-subscriber = { version = "0.3", features = ["env-filter"] }
tracing-appender = "0.2"
use tracing::{info, warn, error, debug, trace, instrument};
use tracing_subscriber::{layer::SubscriberExt, util::SubscriberInitExt};

fn setup_tracing() {
    tracing_subscriber::registry()
        .with(
            tracing_subscriber::EnvFilter::try_from_default_env()
                .unwrap_or_else(|_| "myapp=debug,tower_http=debug".into()),
        )
        .with(tracing_subscriber::fmt::layer())
        .init();
}

#[instrument]
async fn process_user(user_id: u64, action: &str) -> Result<String, &'static str> {
    info!(user_id, action, "开始处理用户操作");
    
    // 模拟一些工作
    tokio::time::sleep(Duration::from_millis(100)).await;
    
    if user_id == 0 {
        error!("无效的用户ID");
        return Err("无效用户ID");
    }
    
    debug!("处理中间步骤");
    
    // 带有结构化字段的日志
    info!(
        user_id = user_id,
        action = action,
        duration_ms = 100,
        "用户操作完成"
    );
    
    Ok(format!("用户{}的{}操作已完成", user_id, action))
}

// 带有span的异步函数
#[instrument(skip(data))]
async fn process_data(name: &str, data: &[u8]) -> usize {
    let span = tracing::info_span!("processing", data_len = data.len());
    let _enter = span.enter();
    
    trace!("开始处理数据");
    
    // 模拟处理
    tokio::time::sleep(Duration::from_millis(50)).await;
    
    let result = data.len() * 2;
    debug!(result, "数据处理完成");
    
    result
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    setup_tracing();
    
    info!("应用程序启动");
    
    let result = process_user(123, "login").await?;
    info!(result, "操作结果");
    
    let data = b"hello world";
    let processed = process_data("test", data).await;
    info!(processed, "处理结果");
    
    Ok(())
}
```

### 6.2 log - 传统日志

```toml
[dependencies]
log = "0.4"
env_logger = "0.10"
use log::{info, warn, error, debug};

fn main() {
    env_logger::init();
    
    info!("应用程序启动");
    debug!("这是调试信息");
    warn!("这是警告");
    error!("这是错误信息");
}
```

### 6.3 metrics - 指标收集

```toml
[dependencies]
metrics = "0.21"
metrics-exporter-prometheus = "0.12"
use metrics::{counter, histogram, gauge, describe_counter, describe_histogram};
use std::time::Instant;

fn setup_metrics() {
    let builder = metrics_exporter_prometheus::PrometheusBuilder::new();
    builder.install().expect("failed to install recorder");
    
    describe_counter!("requests_total", "请求总数");
    describe_histogram!("request_duration_seconds", "请求持续时间");
}

fn handle_request(path: &str) {
    let start = Instant::now();
    
    counter!("requests_total", 1, "path" => path.to_string());
    
    // 模拟请求处理
    std::thread::sleep(std::time::Duration::from_millis(100));
    
    let duration = start.elapsed();
    histogram!("request_duration_seconds", duration);
    
    gauge!("active_connections", 42.0);
}
```

## 7. 加密和安全

### 7.1 ring - 加密原语

```toml
[dependencies]
ring = "0.16"
base64 = "0.21"
use ring::{digest, hmac, pbkdf2, rand, signature};
use base64::{Engine as _, engine::general_purpose};

fn cryptography_examples() -> Result<(), Box<dyn std::error::Error>> {
    // 哈希
    let data = b"hello world";
    let hash = digest::digest(&digest::SHA256, data);
    println!("SHA256: {}", hex::encode(hash.as_ref()));
    
    // HMAC
    let key = hmac::Key::new(hmac::HMAC_SHA256, b"secret key");
    let signature = hmac::sign(&key, data);
    println!("HMAC: {}", hex::encode(signature.as_ref()));
    
    // PBKDF2 密码哈希
    let password = "my_password";
    let salt = b"salt123456789012"; // 16字节盐
    let mut pbkdf2_hash = [0u8; digest::SHA256_OUTPUT_LEN];
    
    pbkdf2::derive(
        pbkdf2::PBKDF2_HMAC_SHA256,
        std::num::NonZeroU32::new(100_000).unwrap(),
        salt,
        password.as_bytes(),
        &mut pbkdf2_hash,
    );
    
    println!("PBKDF2: {}", hex::encode(pbkdf2_hash));
    
    // 随机数生成
    let rng = rand::SystemRandom::new();
    let mut random_bytes = [0u8; 32];
    rand::SecureRandom::fill(&rng, &mut random_bytes)?;
    println!("随机数: {}", hex::encode(random_bytes));
    
    Ok(())
}

// Base64编码/解码
fn base64_example() -> Result<(), base64::DecodeError> {
    let data = b"Hello, World!";
    let encoded = general_purpose::STANDARD.encode(data);
    println!("编码: {}", encoded);
    
    let decoded = general_purpose::STANDARD.decode(&encoded)?;
    println!("解码: {}", String::from_utf8_lossy(&decoded));
    
    Ok(())
}
```

### 7.2 jsonwebtoken - JWT处理

```toml
[dependencies]
jsonwebtoken = "9"
serde = { version = "1.0", features = ["derive"] }
chrono = { version = "0.4", features = ["serde"] }
use jsonwebtoken::{encode, decode, Header, Algorithm, Validation, EncodingKey, DecodingKey};
use serde::{Deserialize, Serialize};
use chrono::{Utc, Duration};

#[derive(Debug, Serialize, Deserialize)]
struct Claims {
    sub: String,    // subject
    exp: usize,     // expiration time
    iat: usize,     // issued at
    #[serde(skip_serializing_if = "Option::is_none")]
    role: Option<String>,
}

fn jwt_example() -> Result<(), Box<dyn std::error::Error>> {
    let secret = "my_secret_key";
    let encoding_key = EncodingKey::from_secret(secret.as_ref());
    let decoding_key = DecodingKey::from_secret(secret.as_ref());
    
    // 创建token
    let now = Utc::now();
    let claims = Claims {
        sub: "user123".to_string(),
        exp: (now + Duration::hours(24)).timestamp() as usize,
        iat: now.timestamp() as usize,
        role: Some("admin".to_string()),
    };
    
    let token = encode(&Header::default(), &claims, &encoding_key)?;
    println!("Token: {}", token);
    
    // 验证token
    let validation = Validation::new(Algorithm::HS256);
    let token_data = decode::<Claims>(&token, &decoding_key, &validation)?;
    
    println!("解码后的claims: {:?}", token_data.claims);
    
    Ok(())
}
```

## 8. 配置管理

### 8.1 config - 配置管理

```toml
[dependencies]
config = "0.13"
serde = { version = "1.0", features = ["derive"] }
use config::{Config, ConfigError, File, Environment};
use serde::{Deserialize, Serialize};

#[derive(Debug, Serialize, Deserialize)]
struct AppConfig {
    database: DatabaseConfig,
    server: ServerConfig,
    logging: LoggingConfig,
}

#[derive(Debug, Serialize, Deserialize)]
struct DatabaseConfig {
    host: String,
    port: u16,
    name: String,
    username: String,
    password: String,
    max_connections: u32,
}

#[derive(Debug, Serialize, Deserialize)]
struct ServerConfig {
    host: String,
    port: u16,
    workers: usize,
}

#[derive(Debug, Serialize, Deserialize)]
struct LoggingConfig {
    level: String,
    file: Option<String>,
}

impl AppConfig {
    fn new() -> Result<Self, ConfigError> {
        let config = Config::builder()
            // 添加默认配置
            .set_default("server.host", "127.0.0.1")?
            .set_default("server.port", 8080)?
            .set_default("server.workers", 4)?
            .set_default("database.port", 5432)?
            .set_default("database.max_connections", 10)?
            .set_default("logging.level", "info")?
            // 从文件读取配置
            .add_source(File::with_name("config/default").required(false))
            .add_source(File::with_name("config/local").required(false))
            // 从环境变量读取配置（例如：APP_DATABASE__HOST）
            .add_source(Environment::with_prefix("APP").separator("__"))
            .build()?;
        
        config.try_deserialize()
    }
}

fn config_example() -> Result<(), ConfigError> {
    let app_config = AppConfig::new()?;
    println!("配置: {:#?}", app_config);
    
    // 使用配置
    println!("数据库连接: {}:{}", app_config.database.host, app_config.database.port);
    println!("服务器监听: {}:{}", app_config.server.host, app_config.server.port);
    
    Ok(())
}
```

### 8.2 dotenv - 环境变量管理

```toml
[dependencies]
dotenv = "0.15"
use dotenv::dotenv;
use std::env;

fn main() -> Result<(), Box<dyn std::error::Error>> {
    // 从.env文件加载环境变量
    dotenv().ok();
    
    let database_url = env::var("DATABASE_URL")
        .expect("DATABASE_URL must be set");
    
    let port = env::var("PORT")
        .unwrap_or_else(|_| "8080".to_string())
        .parse::<u16>()?;
    
    println!("Database URL: {}", database_url);
    println!("Port: {}", port);
    
    Ok(())
}
```

## 9. 测试工具

### 9.1 proptest - 属性测试

```toml
[dependencies]
proptest = "1.0"
use proptest::prelude::*;

fn add(a: i32, b: i32) -> i32 {
    a + b
}

fn multiply(a: i32, b: i32) -> i32 {
    a * b
}

proptest! {
    #[test]
    fn test_add_commutative(a: i32, b: i32) {
        prop_assert_eq!(add(a, b), add(b, a));
    }
    
    #[test]
    fn test_add_associative(a: i32, b: i32, c: i32) {
        prop_assert_eq!(add(add(a, b), c), add(a, add(b, c)));
    }
    
    #[test]
    fn test_multiply_by_zero(a: i32) {
        prop_assert_eq!(multiply(a, 0), 0);
    }
    
    #[test]
    fn test_string_reverse_twice(s in ".*") {
        let reversed_twice: String = s.chars().rev().collect::<String>()
            .chars().rev().collect();
        prop_assert_eq!(s, reversed_twice);
    }
}

// 自定义策略
fn my_strategy() -> impl Strategy<Value = (i32, String)> {
    (1..100, "[a-z]{3,10}")
}

proptest! {
    #[test]
    fn test_with_custom_strategy((num, s) in my_strategy()) {
        prop_assert!(num > 0);
        prop_assert!(s.len() >= 3);
        prop_assert!(s.len() <= 10);
    }
}
```

### 9.2 criterion - 性能基准测试

```toml
[dev-dependencies]
criterion = { version = "0.5", features = ["html_reports"] }

[[bench]]
name = "my_benchmark"
harness = false
// benches/my_benchmark.rs
use criterion::{criterion_group, criterion_main, Criterion, black_box};

fn fibonacci_recursive(n: u64) -> u64 {
    match n {
        0 => 1,
        1 => 1,
        n => fibonacci_recursive(n-1) + fibonacci_recursive(n-2),
    }
}

fn fibonacci_iterative(n: u64) -> u64 {
    let mut a = 1;
    let mut b = 1;
    
    for _ in 0..n {
        let temp = a + b;
        a = b;
        b = temp;
    }
    
    a
}

fn criterion_benchmark(c: &mut Criterion) {
    c.bench_function("fib recursive 20", |b| {
        b.iter(|| fibonacci_recursive(black_box(20)))
    });
    
    c.bench_function("fib iterative 20", |b| {
        b.iter(|| fibonacci_iterative(black_box(20)))
    });
    
    // 比较多个实现
    let mut group = c.benchmark_group("fibonacci");
    group.bench_function("recursive", |b| {
        b.iter(|| fibonacci_recursive(black_box(10)))
    });
    group.bench_function("iterative", |b| {
        b.iter(|| fibonacci_iterative(black_box(10)))
    });
    group.finish();
}

criterion_group!(benches, criterion_benchmark);
criterion_main!(benches);
```

### 9.3 mockall - Mock测试

```toml
[dependencies]
mockall = "0.11"
async-trait = "0.1"
use mockall::{automock, predicate::*};
use async_trait::async_trait;

#[async_trait]
#[automock]
trait DataService {
    async fn fetch_user(&self, id: u64) -> Result<String, String>;
    fn get_config(&self, key: &str) -> Option<String>;
}

struct UserProcessor {
    service: Box<dyn DataService>,
}

impl UserProcessor {
    fn new(service: Box<dyn DataService>) -> Self {
        UserProcessor { service }
    }
    
    async fn process_user(&self, id: u64) -> Result<String, String> {
        let user_data = self.service.fetch_user(id).await?;
        let prefix = self.service.get_config("user_prefix")
            .unwrap_or_else(|| "User".to_string());
        
        Ok(format!("{}: {}", prefix, user_data))
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    use tokio;
    
    #[tokio::test]
    async fn test_process_user_success() {
        let mut mock_service = MockDataService::new();
        
        mock_service
            .expect_fetch_user()
            .with(eq(123))
            .times(1)
            .returning(|_| Ok("John Doe".to_string()));
        
        mock_service
            .expect_get_config()
            .with(eq("user_prefix"))
            .times(1)
            .returning(|_| Some("Customer".to_string()));
        
        let processor = UserProcessor::new(Box::new(mock_service));
        let result = processor.process_user(123).await;
        
        assert_eq!(result.unwrap(), "Customer: John Doe");
    }
    
    #[tokio::test]
    async fn test_process_user_service_error() {
        let mut mock_service = MockDataService::new();
        
        mock_service
            .expect_fetch_user()
            .with(eq(404))
            .times(1)
            .returning(|_| Err("User not found".to_string()));
        
        let processor = UserProcessor::new(Box::new(mock_service));
        let result = processor.process_user(404).await;
        
        assert_eq!(result.unwrap_err(), "User not found");
    }
}
```

## 10. 实用工具库

### 10.1 uuid - UUID生成

```toml
[dependencies]
uuid = { version = "1.0", features = ["v4", "serde"] }
use uuid::Uuid;

fn uuid_examples() {
    // 生成随机UUID (v4)
    let id = Uuid::new_v4();
    println!("UUID v4: {}", id);
    
    // 从字符串解析
    let uuid_str = "550e8400-e29b-41d4-a716-446655440000";
    let parsed = Uuid::parse_str(uuid_str).unwrap();
    println!("Parsed UUID: {}", parsed);
    
    // 零UUID
    let nil = Uuid::nil();
    println!("Nil UUID: {}", nil);
    
    // 转换为字节数组
    let bytes = id.as_bytes();
    println!("UUID bytes: {:?}", bytes);
    
    // 从字节数组创建
    let from_bytes = Uuid::from_bytes(*bytes);
    println!("From bytes: {}", from_bytes);
}
```

### 10.2 chrono - 日期时间处理

```toml
[dependencies]
chrono = { version = "0.4", features = ["serde"] }
use chrono::{DateTime, Utc, Local, Duration, NaiveDate, TimeZone};
use chrono::format::strftime::StrftimeItems;

fn datetime_examples() {
    // 当前时间
    let now_utc = Utc::now();
    let now_local = Local::now();
    
    println!("UTC now: {}", now_utc);
    println!("Local now: {}", now_local);
    
    // 解析时间字符串
    let dt = DateTime::parse_from_rfc3339("2023-01-01T12:00:00Z").unwrap();
    println!("Parsed: {}", dt);
    
    // 格式化
    let formatted = now_utc.format("%Y-%m-%d %H:%M:%S").to_string();
    println!("Formatted: {}", formatted);
    
    // 日期运算
    let tomorrow = now_utc + Duration::days(1);
    let last_week = now_utc - Duration::weeks(1);
    
    println!("Tomorrow: {}", tomorrow);
    println!("Last week: {}", last_week);
    
    // 创建特定日期
    let date = NaiveDate::from_ymd_opt(2023, 1, 1).unwrap();
    let datetime = date.and_hms_opt(12, 0, 0).unwrap();
    let utc_dt = Utc.from_utc_datetime(&datetime);
    
    println!("Created datetime: {}", utc_dt);
    
    // 时区转换
    let tokyo = chrono_tz::Asia::Tokyo;
    let tokyo_time = now_utc.with_timezone(&tokyo);
    println!("Tokyo time: {}", tokyo_time);
}
```

### 10.3 regex - 正则表达式

```toml
[dependencies]
regex = "1.0"
lazy_static = "1.4"
use regex::Regex;
use lazy_static::lazy_static;

lazy_static! {
    static ref EMAIL_REGEX: Regex = Regex::new(
        r"^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$"
    ).unwrap();
    
    static ref PHONE_REGEX: Regex = Regex::new(
        r"^\+?1?[-.\s]?\(?([0-9]{3})\)?[-.\s]?([0-9]{3})[-.\s]?([0-9]{4})$"
    ).unwrap();
}

fn regex_examples() {
    // 基本匹配
    let text = "Hello, my email is john@example.com and phone is +1-555-123-4567";
    
    // 验证邮箱
    let emails: Vec<&str> = EMAIL_REGEX.find_iter(text)
        .map(|m| m.as_str())
        .collect();
    println!("Emails: {:?}", emails);
    
    // 验证电话
    if PHONE_REGEX.is_match("+1-555-123-4567") {
        println!("Valid phone number");
    }
    
    // 捕获组
    let re = Regex::new(r"(\w+)\s+(\w+)").unwrap();
    let caps = re.captures("John Doe").unwrap();
    println!("First name: {}", &caps[1]);
    println!("Last name: {}", &caps[2]);
    
    // 替换
    let re = Regex::new(r"\b\w{4}\b").unwrap();
    let result = re.replace_all("This is a test", "****");
    println!("Replaced: {}", result);
    
    // 分割
    let re = Regex::new(r"[,\s]+").unwrap();
    let parts: Vec<&str> = re.split("apple, banana orange,cherry").collect();
    println!("Parts: {:?}", parts);
}
```

### 10.4 once_cell - 延迟初始化

```toml
[dependencies]
once_cell = "1.19"
use once_cell::sync::{Lazy, OnceCell};
use std::collections::HashMap;

// 全局延迟初始化
static GLOBAL_CONFIG: Lazy<HashMap<String, String>> = Lazy::new(|| {
    let mut map = HashMap::new();
    map.insert("database_url".to_string(), "postgresql://localhost".to_string());
    map.insert("redis_url".to_string(), "redis://localhost".to_string());
    map
});

// 单次初始化
static INSTANCE: OnceCell<ExpensiveToCompute> = OnceCell::new();

struct ExpensiveToCompute {
    value: u64,
}

impl ExpensiveToCompute {
    fn new() -> Self {
        println!("Computing expensive value...");
        std::thread::sleep(std::time::Duration::from_millis(1000));
        ExpensiveToCompute { value: 42 }
    }
    
    fn get_instance() -> &'static ExpensiveToCompute {
        INSTANCE.get_or_init(|| ExpensiveToCompute::new())
    }
}

fn once_cell_example() {
    // 使用Lazy
    println!("Config: {:?}", &*GLOBAL_CONFIG);
    
    // 使用OnceCell
    let instance1 = ExpensiveToCompute::get_instance();
    let instance2 = ExpensiveToCompute::get_instance(); // 不会重新计算
    
    println!("Instance 1: {}", instance1.value);
    println!("Instance 2: {}", instance2.value);
    println!("Same instance: {}", std::ptr::eq(instance1, instance2));
}
```

## 11. 数据处理

### 11.1 csv - CSV处理

```toml
[dependencies]
csv = "1.3"
serde = { version = "1.0", features = ["derive"] }
use csv::{Reader, Writer, ReaderBuilder, WriterBuilder};
use serde::{Deserialize, Serialize};
use std::io;

#[derive(Debug, Deserialize, Serialize)]
struct Person {
    name: String,
    age: u8,
    city: String,
    salary: Option<f64>,
}

fn csv_examples() -> Result<(), Box<dyn std::error::Error>> {
    // 写入CSV
    let people = vec![
        Person { name: "Alice".to_string(), age: 30, city: "New York".to_string(), salary: Some(75000.0) },
        Person { name: "Bob".to_string(), age: 25, city: "Los Angeles".to_string(), salary: Some(65000.0) },
        Person { name: "Charlie".to_string(), age: 35, city: "Chicago".to_string(), salary: None },
    ];
    
    let mut wtr = Writer::from_path("people.csv")?;
    for person in &people {
        wtr.serialize(person)?;
    }
    wtr.flush()?;
    
    // 读取CSV
    let mut rdr = Reader::from_path("people.csv")?;
    for result in rdr.deserialize() {
        let person: Person = result?;
        println!("{:?}", person);
    }
    
    // 自定义分隔符
    let data = "name;age;city\nAlice;30;New York\nBob;25;Los Angeles";
    let mut rdr = ReaderBuilder::new()
        .delimiter(b';')
        .from_reader(data.as_bytes());
    
    for result in rdr.records() {
        let record = result?;
        println!("Record: {:?}", record);
    }
    
    // 处理无头部的CSV
    let data = "Alice,30,New York\nBob,25,Los Angeles";
    let mut rdr = ReaderBuilder::new()
        .has_headers(false)
        .from_reader(data.as_bytes());
    
    for result in rdr.records() {
        let record = result?;
        if record.len() >= 3 {
            println!("{}, age {}, from {}", &record[0], &record[1], &record[2]);
        }
    }
    
    Ok(())
}
```

### 11.2 polars - 高性能数据框架

```toml
[dependencies]
polars = { version = "0.35", features = ["lazy", "csv", "json", "strings"] }
use polars::prelude::*;

fn polars_examples() -> PolarsResult<()> {
    // 创建DataFrame
    let df = df! {
        "name" => ["Alice", "Bob", "Charlie", "Diana"],
        "age" => [25, 30, 35, 28],
        "city" => ["NY", "LA", "Chicago", "Boston"],
        "salary" => [50000, 60000, 70000, 55000],
    }?;
    
    println!("Original DataFrame:");
    println!("{}", df);
    
    // 过滤和选择
    let filtered = df
        .clone()
        .lazy()
        .filter(col("age").gt(lit(27)))
        .select([col("name"), col("age"), col("salary")])
        .collect()?;
    
    println!("\nFiltered (age > 27):");
    println!("{}", filtered);
    
    // 聚合操作
    let summary = df
        .clone()
        .lazy()
        .group_by([col("city")])
        .agg([
            col("age").mean().alias("avg_age"),
            col("salary").sum().alias("total_salary"),
            col("name").count().alias("count"),
        ])
        .collect()?;
    
    println!("\nSummary by city:");
    println!("{}", summary);
    
    // 排序
    let sorted = df
        .clone()
        .lazy()
        .sort("salary", SortOptions::default())
        .collect()?;
    
    println!("\nSorted by salary:");
    println!("{}", sorted);
    
    // 添加新列
    let with_bonus = df
        .clone()
        .lazy()
        .with_columns([
            (col("salary") * lit(0.1)).alias("bonus"),
            when(col("age").gt(lit(30)))
                .then(lit("Senior"))
                .otherwise(lit("Junior"))
                .alias("level"),
        ])
        .collect()?;
    
    println!("\nWith bonus and level:");
    println!("{}", with_bonus);
    
    // 从CSV读取
    let df_from_csv = LazyFrame::scan_csv("people.csv", ScanArgsCSV::default())?
        .collect()?;
    
    println!("\nFrom CSV:");
    println!("{}", df_from_csv);
    
    Ok(())
}
```

## 12. 图像和多媒体处理

### 12.1 image - 图像处理

```toml
[dependencies]
image = { version = "0.24", features = ["png", "jpeg", "gif", "webp"] }
use image::{ImageBuffer, RgbImage, DynamicImage, ImageFormat};
use image::imageops::{FilterType, resize, rotate90, flip_horizontal};
use std::path::Path;

fn image_processing_examples() -> Result<(), Box<dyn std::error::Error>> {
    // 创建新图像
    let mut img: RgbImage = ImageBuffer::new(800, 600);
    
    // 绘制渐变
    for (x, y, pixel) in img.enumerate_pixels_mut() {
        let r = (x as f32 / 800.0 * 255.0) as u8;
        let g = (y as f32 / 600.0 * 255.0) as u8;
        let b = 128;
        *pixel = image::Rgb([r, g, b]);
    }
    
    // 保存图像
    img.save("gradient.png")?;
    
    // 读取图像
    let mut dynamic_img = image::open("input.jpg")?;
    
    // 获取图像信息
    println!("图像尺寸: {}x{}", dynamic_img.width(), dynamic_img.height());
    println!("颜色类型: {:?}", dynamic_img.color());
    
    // 调整大小
    let resized = resize(&dynamic_img, 400, 300, FilterType::Lanczos3);
    resized.save("resized.jpg")?;
    
    // 旋转
    let rotated = rotate90(&dynamic_img);
    rotated.save("rotated.jpg")?;
    
    // 水平翻转
    let flipped = flip_horizontal(&dynamic_img);
    flipped.save("flipped.jpg")?;
    
    // 裁剪
    let cropped = dynamic_img.crop(100, 100, 300, 200);
    cropped.save("cropped.jpg")?;
    
    // 转换格式
    dynamic_img.save_with_format("output.webp", ImageFormat::WebP)?;
    
    // 图像滤镜
    let blurred = dynamic_img.blur(2.0);
    blurred.save("blurred.jpg")?;
    
    let brightness_adjusted = dynamic_img.brighten(30);
    brightness_adjusted.save("bright.jpg")?;
    
    let contrast_adjusted = dynamic_img.adjust_contrast(1.5);
    contrast_adjusted.save("contrast.jpg")?;
    
    Ok(())
}

// 生成缩略图
fn generate_thumbnails(input_path: &str, sizes: &[u32]) -> Result<(), Box<dyn std::error::Error>> {
    let img = image::open(input_path)?;
    
    for &size in sizes {
        let thumbnail = img.thumbnail(size, size);
        let output_path = format!("thumbnail_{}x{}.jpg", size, size);
        thumbnail.save(&output_path)?;
        println!("生成缩略图: {}", output_path);
    }
    
    Ok(())
}
```

### 12.2 rodio - 音频播放

```toml
[dependencies]
rodio = "0.17"
use rodio::{Decoder, OutputStream, Sink, Source};
use std::fs::File;
use std::io::BufReader;
use std::time::Duration;

fn audio_examples() -> Result<(), Box<dyn std::error::Error>> {
    // 获取输出流和句柄
    let (_stream, stream_handle) = OutputStream::try_default()?;
    
    // 创建音频接收器
    let sink = Sink::try_new(&stream_handle)?;
    
    // 播放音频文件
    let file = BufReader::new(File::open("music.mp3")?);
    let source = Decoder::new(file)?;
    sink.append(source);
    
    // 控制音量
    sink.set_volume(0.5);
    
    // 暂停和恢复
    std::thread::sleep(Duration::from_secs(2));
    sink.pause();
    println!("暂停播放...");
    
    std::thread::sleep(Duration::from_secs(1));
    sink.play();
    println!("恢复播放...");
    
    // 等待播放完成
    sink.sleep_until_end();
    
    // 生成音调
    let sine_wave = rodio::source::SineWave::new(440.0) // A4音调
        .take_duration(Duration::from_secs(2))
        .amplify(0.20);
    
    let sink2 = Sink::try_new(&stream_handle)?;
    sink2.append(sine_wave);
    sink2.sleep_until_end();
    
    Ok(())
}

// 音频效果处理
fn audio_effects_example() -> Result<(), Box<dyn std::error::Error>> {
    let (_stream, stream_handle) = OutputStream::try_default()?;
    let sink = Sink::try_new(&stream_handle)?;
    
    let file = BufReader::new(File::open("input.wav")?);
    let source = Decoder::new(file)?
        .convert_samples::<f32>()
        .amplify(0.8) // 调整音量
        .fade_in(Duration::from_secs(2)) // 淡入效果
        .take_duration(Duration::from_secs(10)); // 限制播放时长
    
    sink.append(source);
    sink.sleep_until_end();
    
    Ok(())
}
```

## 13. 网络和协议

### 13.1 tonic - gRPC框架

```toml
[dependencies]
tonic = "0.10"
prost = "0.12"
tokio = { version = "1.0", features = ["macros", "rt-multi-thread"] }

[build-dependencies]
tonic-build = "0.10"
// build.rs
fn main() -> Result<(), Box<dyn std::error::Error>> {
    tonic_build::compile_protos("proto/hello.proto")?;
    Ok(())
}

// proto/hello.proto
/*
syntax = "proto3";
package hello;

service Greeter {
    rpc SayHello (HelloRequest) returns (HelloResponse);
    rpc SayHelloStream (HelloRequest) returns (stream HelloResponse);
}

message HelloRequest {
    string name = 1;
}

message HelloResponse {
    string message = 1;
}
*/

// src/server.rs
use tonic::{transport::Server, Request, Response, Status, Streaming};
use hello::greeter_server::{Greeter, GreeterServer};
use hello::{HelloRequest, HelloResponse};

pub mod hello {
    tonic::include_proto!("hello");
}

#[derive(Debug, Default)]
pub struct MyGreeter {}

#[tonic::async_trait]
impl Greeter for MyGreeter {
    async fn say_hello(
        &self,
        request: Request<HelloRequest>,
    ) -> Result<Response<HelloResponse>, Status> {
        println!("收到请求: {:?}", request);
        
        let reply = HelloResponse {
            message: format!("Hello {}!", request.into_inner().name),
        };
        
        Ok(Response::new(reply))
    }
    
    type SayHelloStreamStream = tokio_stream::wrappers::ReceiverStream<Result<HelloResponse, Status>>;
    
    async fn say_hello_stream(
        &self,
        request: Request<HelloRequest>,
    ) -> Result<Response<Self::SayHelloStreamStream>, Status> {
        let (tx, rx) = tokio::sync::mpsc::channel(4);
        let name = request.into_inner().name;
        
        tokio::spawn(async move {
            for i in 0..5 {
                let response = HelloResponse {
                    message: format!("Hello {} - 消息 {}", name, i),
                };
                
                if tx.send(Ok(response)).await.is_err() {
                    break;
                }
                
                tokio::time::sleep(Duration::from_secs(1)).await;
            }
        });
        
        Ok(Response::new(tokio_stream::wrappers::ReceiverStream::new(rx)))
    }
}

#[tokio::main]
async fn grpc_server_main() -> Result<(), Box<dyn std::error::Error>> {
    let addr = "0.0.0.0:50051".parse()?;
    let greeter = MyGreeter::default();
    
    Server::builder()
        .add_service(GreeterServer::new(greeter))
        .serve(addr)
        .await?;
    
    Ok(())
}

// src/client.rs
use hello::greeter_client::GreeterClient;
use hello::HelloRequest;

async fn grpc_client_example() -> Result<(), Box<dyn std::error::Error>> {
    let mut client = GreeterClient::connect("http://[::1]:50051").await?;
    
    let request = tonic::Request::new(HelloRequest {
        name: "World".into(),
    });
    
    let response = client.say_hello(request).await?;
    println!("响应: {}", response.into_inner().message);
    
    // 流式请求
    let request = tonic::Request::new(HelloRequest {
        name: "Stream".into(),
    });
    
    let mut stream = client.say_hello_stream(request).await?.into_inner();
    
    while let Some(response) = stream.message().await? {
        println!("流式响应: {}", response.message);
    }
    
    Ok(())
}
```

### 13.2 websocket处理

```toml
[dependencies]
tokio-tungstenite = "0.20"
futures-util = "0.3"
use tokio_tungstenite::{accept_async, connect_async, WebSocketStream};
use tokio_tungstenite::tungstenite::{Message, Result as WsResult};
use futures_util::{StreamExt, SinkExt, stream::SplitSink, stream::SplitStream};
use tokio::net::{TcpListener, TcpStream};
use std::collections::HashMap;
use std::sync::Arc;
use tokio::sync::{Mutex, mpsc};

// WebSocket服务器
async fn websocket_server() -> Result<(), Box<dyn std::error::Error>> {
    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("WebSocket server listening on ws://127.0.0.1:8080");
    
    let connections: Arc<Mutex<HashMap<String, mpsc::UnboundedSender<Message>>>> = 
        Arc::new(Mutex::new(HashMap::new()));
    
    while let Ok((stream, addr)) = listener.accept().await {
        let connections = Arc::clone(&connections);
        let client_id = format!("client_{}", addr);
        
        tokio::spawn(async move {
            if let Err(e) = handle_websocket_connection(stream, client_id, connections).await {
                println!("WebSocket连接错误: {}", e);
            }
        });
    }
    
    Ok(())
}

async fn handle_websocket_connection(
    stream: TcpStream,
    client_id: String,
    connections: Arc<Mutex<HashMap<String, mpsc::UnboundedSender<Message>>>>,
) -> WsResult<()> {
    let ws_stream = accept_async(stream).await?;
    let (mut ws_sender, mut ws_receiver) = ws_stream.split();
    
    let (tx, mut rx) = mpsc::unbounded_channel::<Message>();
    connections.lock().await.insert(client_id.clone(), tx);
    
    // 处理发送消息的任务
    let send_task = tokio::spawn(async move {
        while let Some(message) = rx.recv().await {
            if ws_sender.send(message).await.is_err() {
                break;
            }
        }
    });
    
    // 处理接收消息
    while let Some(message) = ws_receiver.next().await {
        match message? {
            Message::Text(text) => {
                println!("收到来自{}的消息: {}", client_id, text);
                
                // 广播给所有客户端
                let connections_guard = connections.lock().await;
                for (id, sender) in connections_guard.iter() {
                    if *id != client_id {
                        let broadcast_msg = Message::Text(format!("{}: {}", client_id, text));
                        let _ = sender.send(broadcast_msg);
                    }
                }
            }
            Message::Close(_) => {
                println!("客户端{}断开连接", client_id);
                break;
            }
            _ => {}
        }
    }
    
    // 清理连接
    connections.lock().await.remove(&client_id);
    send_task.abort();
    
    Ok(())
}

// WebSocket客户端
async fn websocket_client_example() -> Result<(), Box<dyn std::error::Error>> {
    let (ws_stream, _) = connect_async("ws://127.0.0.1:8080").await?;
    let (mut write, mut read) = ws_stream.split();
    
    // 发送消息的任务
    let send_task = tokio::spawn(async move {
        let mut interval = tokio::time::interval(Duration::from_secs(5));
        let mut counter = 0;
        
        loop {
            interval.tick().await;
            counter += 1;
            let message = Message::Text(format!("客户端消息 #{}", counter));
            
            if write.send(message).await.is_err() {
                break;
            }
        }
    });
    
    // 接收消息的任务
    let receive_task = tokio::spawn(async move {
        while let Some(message) = read.next().await {
            match message {
                Ok(Message::Text(text)) => {
                    println!("收到服务器消息: {}", text);
                }
                Ok(Message::Close(_)) => {
                    println!("服务器关闭连接");
                    break;
                }
                Err(e) => {
                    println!("接收消息错误: {}", e);
                    break;
                }
                _ => {}
            }
        }
    });
    
    // 等待任务完成
    tokio::select! {
        _ = send_task => {},
        _ = receive_task => {},
    }
    
    Ok(())
}
```

## 14. 嵌入式和系统编程

### 14.1 libc - 系统调用接口

```toml
[dependencies]
libc = "0.2"
nix = "0.27"
use libc::{getpid, getppid, getuid, getgid};
use nix::unistd::{fork, ForkResult, execv};
use nix::sys::wait::waitpid;
use std::ffi::CString;

fn system_info_example() {
    unsafe {
        println!("进程ID: {}", getpid());
        println!("父进程ID: {}", getppid());
        println!("用户ID: {}", getuid());
        println!("组ID: {}", getgid());
    }
}

// 进程管理示例
fn process_management_example() -> Result<(), Box<dyn std::error::Error>> {
    match unsafe { fork() }? {
        ForkResult::Parent { child } => {
            println!("父进程: 子进程PID = {}", child);
            waitpid(child, None)?;
            println!("父进程: 子进程已结束");
        }
        ForkResult::Child => {
            println!("子进程: 开始执行");
            
            // 执行新程序
            let program = CString::new("/bin/echo")?;
            let args = vec![
                CString::new("echo")?,
                CString::new("Hello from child process!")?,
            ];
            
            execv(&program, &args)?;
        }
    }
    
    Ok(())
}

// 文件系统操作
use nix::sys::stat::{stat, Mode, SFlag};
use nix::unistd::{chdir, getcwd};

fn filesystem_example() -> Result<(), Box<dyn std::error::Error>> {
    // 获取当前工作目录
    let cwd = getcwd()?;
    println!("当前目录: {:?}", cwd);
    
    // 获取文件信息
    let file_stat = stat("Cargo.toml")?;
    println!("文件大小: {} bytes", file_stat.st_size);
    println!("文件权限: {:o}", file_stat.st_mode & 0o777);
    
    // 检查文件类型
    let file_type = SFlag::from_bits_truncate(file_stat.st_mode);
    if file_type.contains(SFlag::S_IFREG) {
        println!("这是一个普通文件");
    } else if file_type.contains(SFlag::S_IFDIR) {
        println!("这是一个目录");
    }
    
    Ok(())
}
```

### 14.2 sysinfo - 系统信息

```toml
[dependencies]
sysinfo = "0.29"
use sysinfo::{System, SystemExt, CpuExt, ProcessExt, DiskExt, NetworkExt};

fn system_monitoring_example() {
    let mut sys = System::new_all();
    sys.refresh_all();
    
    // 系统信息
    println!("系统名称: {:?}", sys.name());
    println!("系统版本: {:?}", sys.os_version());
    println!("内核版本: {:?}", sys.kernel_version());
    println!("主机名: {:?}", sys.host_name());
    
    // CPU信息
    println!("\nCPU信息:");
    for (i, cpu) in sys.cpus().iter().enumerate() {
        println!("CPU {}: {}%, {}MHz", 
            i, cpu.cpu_usage(), cpu.frequency());
    }
    
    // 内存信息
    println!("\n内存信息:");
    println!("总内存: {} MB", sys.total_memory() / 1024 / 1024);
    println!("已用内存: {} MB", sys.used_memory() / 1024 / 1024);
    println!("可用内存: {} MB", sys.available_memory() / 1024 / 1024);
    println!("总交换空间: {} MB", sys.total_swap() / 1024 / 1024);
    println!("已用交换空间: {} MB", sys.used_swap() / 1024 / 1024);
    
    // 磁盘信息
    println!("\n磁盘信息:");
    for disk in sys.disks() {
        println!("磁盘: {:?}", disk.name());
        println!("  文件系统: {:?}", disk.file_system());
        println!("  总空间: {} GB", disk.total_space() / 1024 / 1024 / 1024);
        println!("  可用空间: {} GB", disk.available_space() / 1024 / 1024 / 1024);
    }
    
    // 网络信息
    println!("\n网络接口:");
    for (interface_name, data) in sys.networks() {
        println!("接口: {}", interface_name);
        println!("  接收: {} bytes", data.received());
        println!("  发送: {} bytes", data.transmitted());
    }
    
    // 进程信息
    println!("\n前10个消耗内存最多的进程:");
    let mut processes: Vec<_> = sys.processes().values().collect();
    processes.sort_by_key(|p| p.memory());
    processes.reverse();
    
    for process in processes.iter().take(10) {
        println!("PID: {}, 名称: {}, 内存: {} MB", 
            process.pid(), 
            process.name(), 
            process.memory() / 1024 / 1024);
    }
}
```

## 15. 性能和优化

### 15.1 flamegraph - 性能分析

```toml
[dependencies]
pprof = { version = "0.13", features = ["flamegraph", "protobuf-codec"] }
use pprof::flamegraph::{from_reader, Options};
use std::fs::File;

fn performance_analysis_example() {
    // 启动性能分析
    let guard = pprof::ProfilerGuard::new(100).unwrap();
    
    // 执行需要分析的代码
    expensive_computation();
    
    // 生成报告
    match guard.report().build() {
        Ok(report) => {
            let file = File::create("flamegraph.svg").unwrap();
            let mut options = Options::default();
            report.flamegraph(file, &mut options).unwrap();
            println!("火焰图已保存到 flamegraph.svg");
        }
        Err(e) => {
            println!("生成报告失败: {}", e);
        }
    }
}

fn expensive_computation() {
    for i in 0..1000000 {
        let _ = fibonacci(20);
        if i % 100000 == 0 {
            calculate_prime(1000);
        }
    }
}

fn fibonacci(n: u32) -> u64 {
    match n {
        0 => 0,
        1 => 1,
        _ => fibonacci(n - 1) + fibonacci(n - 2),
    }
}

fn calculate_prime(limit: usize) -> Vec<usize> {
    let mut primes = Vec::new();
    let mut is_prime = vec![true; limit + 1];
    
    for i in 2..=limit {
        if is_prime[i] {
            primes.push(i);
            for j in (i * i..=limit).step_by(i) {
                is_prime[j] = false;
            }
        }
    }
    
    primes
}
```

### 15.2 jemallocator - 内存分配器

```toml
[dependencies]
jemallocator = "0.5"

[target.'cfg(not(target_env = "msvc"))'.dependencies]
jemallocator = "0.5"
#[cfg(not(target_env = "msvc"))]
use jemallocator::Jemalloc;

#[cfg(not(target_env = "msvc"))]
#[global_allocator]
static GLOBAL: Jemalloc = Jemalloc;

fn memory_allocation_example() {
    // 现在所有的内存分配都使用jemalloc
    let large_vec: Vec<u64> = (0..1_000_000).collect();
    println!("分配了包含{}个元素的向量", large_vec.len());
    
    // jemalloc通常在多线程环境下提供更好的性能
    let handles: Vec<_> = (0..4)
        .map(|i| {
            std::thread::spawn(move || {
                let local_vec: Vec<u32> = (0..100_000).map(|x| x * i).collect();
                println!("线程{}完成分配", i);
                local_vec.len()
            })
        })
        .collect();
    
    for handle in handles {
        handle.join().unwrap();
    }
}
```

## 16. 开发工具和生态系统管理

### 16.1 Cargo扩展

```toml
# 在终端中安装这些工具
# cargo install cargo-watch
# cargo install cargo-expand  
# cargo install cargo-audit
# cargo install cargo-outdated
# cargo install cargo-tree
# 自动重新编译和运行
cargo watch -x run

# 查看宏展开后的代码
cargo expand

# 安全审计
cargo audit

# 检查过期依赖
cargo outdated

# 查看依赖树
cargo tree

# 清理未使用的依赖
cargo machete

# 格式化代码
cargo fmt

# 代码检查
cargo clippy
```

### 16.2 工作空间配置示例

```toml
# Cargo.toml (根目录)
[workspace]
members = [
    "web-server",
    "database",
    "shared",
    "cli-tool"
]

[workspace.dependencies]
tokio = { version = "1.0", features = ["full"] }
serde = { version = "1.0", features = ["derive"] }
tracing = "0.1"

# web-server/Cargo.toml
[package]
name = "web-server"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { workspace = true }
axum = "0.7"
shared = { path = "../shared" }

# shared/Cargo.toml
[package]
name = "shared"
version = "0.1.0"
edition = "2021"

[dependencies]
serde = { workspace = true }
```

## 总结

Rust的生态系统非常丰富，这份指南涵盖了最常用和最重要的crate：

### 核心类别：

- **异步运行时**: tokio, async-std
- **Web框架**: axum, warp, actix-web
- **HTTP客户端**: reqwest, hyper
- **数据库**: sqlx, diesel, redis
- **序列化**: serde, bincode
- **命令行**: clap, color-eyre
- **日志**: tracing, log
- **测试**: proptest, criterion, mockall
- **工具库**: uuid, chrono, regex

### 选择建议：

1. **新项目推荐**：优先选择维护活跃、文档完善的crate
2. **性能关键**：选择零成本抽象的库（如serde, tokio）
3. **生态兼容**：选择与tokio生态兼容的异步库
4. **稳定性**：production环境选择1.0+版本的crate

### 学习路径：

1. 从基础工具库开始（serde, chrono, clap）
2. 学习异步编程（tokio, futures）
3. 掌握Web开发（axum, reqwest）
4. 深入数据库操作（sqlx, redis）
5. 实践测试和性能优化

这个生态系统仍在快速发展，建议关注官方awesome-rust列表和社区推荐，及时了解新兴的优秀crate。