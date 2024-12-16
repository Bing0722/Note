# Markdown Advance

## 创建表

## 创建折叠部分

在 `<details>` 块中，使用 `<summary>` 标记让读者知道里面的内容。 标签显示在  的右侧。

<details>

<summary>Tips for collapsed sections</summary>

### You can add a header

You can add text within a collapsed section.

You can add an image or a code block, too.

```ruby
   puts "Hello World"
```

</details>

（可选）若要使该部分在默认情况下显示为打开状态，请将 open 属性添加到 `<details>` 标签：`<details open>`

## 代码块

## 创建关系图

Here is a simple flow chart:

```mermaid
graph TD;
    A-->B;
    A-->C;
    B-->D;
    C-->D;
```

## 数学表达式

This sentence uses `$` delimiters to show math inline: $\sqrt{3x-1}+(1+x)^2$

## 自动链接引用和URL

...
