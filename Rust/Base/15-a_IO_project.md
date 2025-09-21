# A IO Project

我们将要做一个简单的命令行工具，这个工具可以搜索一个文件中的字符串并打印出包含这个字符串的行。这个工具的名字叫 `minigrep`。

## 1. 从命令行读取参数

我们将要从命令行读取两个参数：

- 第一个参数是要搜索的字符串。
- 第二个参数是要搜索的文件名。

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    dbg!(args);
}
```

## 2. 将参数值保存到变量中

```rust
use std::env;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    println!("Searching for {}", query);
    println!("In file {}", filename);
}
```

## 3. 读取文件

使用 `fs` 模块的 `read_to_string` 函数读取文件内容。

```rust
use std::env;
use std::fs;

fn main() {
    let args: Vec<String> = env::args().collect();

    let query = &args[1];
    let filename = &args[2];

    let contents = fs::read_to_string(filename).expect("Something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

## 4. 重构改进模块性能和错误处理

### 4.1. 提取参数解析器

```rust
fn parse_config(args: &[String]) -> (&str, &str) {
    let query = &args[1];
    let filename = &args[2];

    (query, filename)
}
```

### 4.2. 组合配置值

```rust
struct Config {
    query: String,
    file_path: String,
}

fn parse_config(args: &[String]) -> Config {
    let query = args[1].clone();
    let file_path = args[2].clone();

    Config { query, file_path }
}
```

### 4.3. 创建一个 `config` 的构造函数

```rust
impl Config {
    fn new(args: &[String]) -> Config {
        let query = args[1].clone();
        let file_path = args[2].clone();

        Config { query, file_path }
    }
}
```

### 4.4. 增加错误处理

```rust
impl Config {
    fn new(args: &[String]) -> Config {
        if args.len() < 3 {
            panic!("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

### 4.5. 从 `new` 函数返回 `Result` 而不是调用 `panic!`

```rust
use std::env;
use std::fs;
use std::process;

fn main () {
    let args: Vec<String> = env::args().collect();
    let config = Config::build(&args).unwrap_or_else(|err| {
        panic!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    let contents = fs::read_to_string(config.file_path).expect("Something went wrong reading the file");

    println!("With text:\n{}", contents);
}

struct Config {
    query: String,
    file_path: String,
}

impl Config {
    fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}
```

### 4.6. 从 `main` 函数中提取逻辑

```rust
fn main () {
    let args: Vec<String> = env::args().collect();
    let config = Config::build(&args).unwrap_or_else(|err| {
        panic!("Problem parsing arguments: {}", err);
        process::exit(1);
    });

    run(config);
}

fn run (config: Config){
    let contents = fs::read_to_string(config.file_path).expect("Something went wrong reading the file");

    println!("With text:\n{}", contents);
}
```

### 4.7. `run` 函数中返回错误

```rust
fn run (config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{}", contents);

    Ok(())
}
```

### 4.8. 将代码拆分到库creat

```rust
// src/lib.rs
use std::error::Error;
use std::fs;

pub struct Config {
    pub query: String,
    pub file_path: String,
}

impl Config {
    pub fn build(args: &[String]) -> Result<Config, &'static str> {
        if args.len() < 3 {
            return Err("not enough arguments");
        }

        let query = args[1].clone();
        let file_path = args[2].clone();

        Ok(Config { query, file_path })
    }
}

pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    println!("With text:\n{contents}");

    Ok(())
}

// main.rs
use std::env;
use std::process;

use ch13_minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        println!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    println!("Searching for {}", config.query);
    println!("In file {}", config.file_path);

    if let Err(e) = ch13_minigrep::run(config) {
        println!("Application error: {e}");
        process::exit(1);
    }
}
```

## 5. 采用测试驱动开发完善库的功能

因为编译器不知道我们在创建 `query` 还是 `contents` 的 `slice`，所以我们需要添加生命周期注解。

```rust
pub fn search<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let mut result = Vec::new();

    for line in contents.lines() {
        if line.contains(query) {
            result.push(line);
        }
    }

    result
}

#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn one_result() {
        let query = "duct";
        let contents = "\
            Rust:
            safe, fast, productive.
            Pick three.";

        assert_eq!(vec!["safe, fast, productive."], search(query, contents));
    }
}
```

## 6. 处理环境变量

写一个大小写不敏感的搜索函数。

```rust
pub fn run(config: Config) -> Result<(), Box<dyn Error>> {
    let contents = fs::read_to_string(config.file_path)?;

    let results = if config.ignore_case {
        search_case_insensitive(&config.query, &contents)
    } else {
        search(&config.query, &contents)
    };

    for line in results {
        println!("{line}");
    }

    Ok(())
}

pub fn search_case_insensitive<'a>(query: &str, contents: &'a str) -> Vec<&'a str> {
    let query = query.to_lowercase();
    let mut results = Vec::new();

    for line in contents.lines() {
        if line.to_lowercase().contains(&query) {
            results.push(line);
        }
    }

    results
}

#[test]
fn case_insensitive() {
    let query = "rUsT";
    let contents = "\
        Rust:
        safe, fast, productive.
        Pick three.
        Trust me.";

    assert_eq!(
        vec!["Rust:", "Trust me."],
        search_case_insensitive(query, contents)
    );
}
```

设置环境变量 `IGNORE_CASE` 来忽略大小写。

```bash
IGNORE_CASE=1 cargo run to poem.txt
```

## 7. 将错误信息输出到标准错误而不是标准输出

使用 `eprintln!` 宏，将错误信息输出到标准错误。

```rust
use std::env;
use std::process;

use minigrep::Config;

fn main() {
    let args: Vec<String> = env::args().collect();

    let config = Config::build(&args).unwrap_or_else(|err| {
        eprintln!("Problem parsing arguments: {err}");
        process::exit(1);
    });

    if let Err(e) = ch13_minigrep::run(config) {
        eprintln!("Application error: {e}");
        process::exit(1);
    }
}
```
