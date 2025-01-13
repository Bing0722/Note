# trait

trait 是 Rust 中的一种特性，它定义了一些行为，可以被类型实现。特征类似于其他语言中的接口，但是更加强大。

## 1. 定义tarit

```rust
trait Summary {
    fn summarize(&self) -> String;
}
```

这种情况下，Summary 是一个 trait，它定义了一个方法 summarize，该方法接受一个 self 参数，并返回一个 String 类型的值。

## 2. 实现trait

```rust
struct NewsArticle {
    headline: String,
    location: String,
    author: String,
    content: String,
}

impl Summary for NewsArticle {
    fn summarize(&self) -> String {
        format!("{}, by {} ({})", self.headline, self.author, self.location)
    }
}
```

在这个例子中，我们为 NewsArticle 结构体实现了 Summary trait。这意味着 NewsArticle 结构体必须实现 summarize 方法。

## 3. 默认实现

```rust
trait Summary {
    fn summarize(&self) -> String {
        String::from("(Read more...)")
    }
}
```

在这个例子中，我们为 Summary trait 提供了一个默认实现。这意味着实现 Summary trait 的类型可以选择是否实现 summarize 方法。

## 4. trait 作为参数

```rust
fn notify(item: &impl Summary) {
    println!("Breaking news! {}", item.summarize());
}
```

在这个例子中，notify 函数接受一个实现了 Summary trait 的参数。

## 5. trait bound

`impl Trait` 适用于简单的情况，但实际上是更长形式的语法糖。trait bound 是一种更通用的语法，它允许我们指定多个 trait。

```rust
fn notify<T: Summary>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

使用 trait bound 时，我们可以指定多个 trait，如下：

```rust
fn notify(item: &(impl Summary + Display)) {
    println!("Breaking news! {}", item.summarize());
}

fn notify<T: Summary + Display>(item: &T) {
    println!("Breaking news! {}", item.summarize());
}
```

这里使用 `+` 运算符指定了多个 trait。

## 6. where 从句

使用过多的 trait bound 会使函数签名变得复杂。在这种情况下，我们可以使用 where 从句来简化函数签名。

```rust
fn some_function<T: Display + Clone, U: Clone + Debug>(t: &T, u: &U) -> i32 {
    // code here
}

fn some_function<T, U>(t: &T, u: &U) -> i32
    where T: Display + Clone,
          U: Clone + Debug
{
    // code here
}
```

## 7. trait 作为返回值

```rust
fn returns_summarizable() -> impl Summary {
    NewsArticle {
        headline: String::from("Penguins win the Stanley Cup Championship!"),
        location: String::from("Pittsburgh, PA, USA"),
        author: String::from("Iceburgh"),
        content: String::from("The Pittsburgh Penguins once again are the best hockey team in the NHL."),
    }
}
```

在使用 impl Trait 时，返回值只能有一个类型。如果我们需要返回多个类型，可以使用 trait 对象。

## 8. 使用 trait bound 有条件地实现方法

```rust
struct Pair<T> {
    x: T,
    y: T,
}

impl<T> Pair<T> {
    fn new(x: T, y: T) -> Self {
        Self {
            x,
            y,
        }
    }
}

impl<T: Display + PartialOrd> Pair<T> {
    fn cmp_display(&self) {
        if self.x >= self.y {
            println!("The largest member is x = {}", self.x);
        } else {
            println!("The largest member is y = {}", self.y);
        }
    }
}
```

这里，我们为 Pair<T> 结构体实现了 cmp_display 方法，该方法只有当 T 类型实现了 Display 和 PartialOrd trait 时才有效。

blanket 实现

```rust
impl<T: Display> ToString for T {
    // code here
}
```

这里，我们为所有实现了 Display trait 的类型实现了 ToString trait。
