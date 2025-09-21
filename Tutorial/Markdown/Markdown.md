# Markdown Tutorial

<!--toc:start-->
- [Markdown Tutorial](#markdown-tutorial)
  - [标题](#标题)
  - [文本样式](#文本样式)
  - [引用文本](#引用文本)
  - [引用代码](#引用代码)
  - [支持颜色模型](#支持颜色模型)
  - [链接](#链接)
  - [章节链接](#章节链接)
  - [相对链接](#相对链接)
  - [映像](#映像)
  - [列表](#列表)
  - [任务列表](#任务列表)
  - [使用表情符号](#使用表情符号)
  - [段落](#段落)
  - [脚注](#脚注)
  - [警报](#警报)
  <!--toc:end-->

## 标题

要创建标题，请在标题文本前添加一至六个`#`符号。你使用的`#`数量将决定层次结构级别和标题的大小。

```md
# A first-level heaeding
## A second-level heaeding
### A third-level heaeding
```

## 文本样式

可以在评论字段和`.md`文件中以粗体、斜体、删除线、下标或上标文本表示强调。

| Stytle | 语法             | 快捷键     | 实例                        |
|--------|------------------|------------|-----------------------------|
| 加粗   | `** **`or`__ __` | `Ctrl`+`b` | **bold**                    |
| 斜体   | `* *`or`_ _`     | `Ctrl`+`i` | *italic*                    |
| 删除线 | `~~ ~~`          | 无         | ~~mistaken~~                |
| 下标   | `<sub> </sub>`   | 无         | text<sub>subscript</sub>    |
| 上标   | `<sup> </sup>`   | 无         | text<sup>superscripts</sup> |
| 下划线 | `<ins> </ins>`   | 无         | <ins>under lined</ins>      |

## 引用文本

可以使用`>`来引用文本。
Text that is not a quote.
>Text that is a quote.

## 引用代码

使用单反引号可标注句子中的代码或命令。 反引号中的文本不会被格式化。

Use `git status` to list all new or modified files that haven't yet been commited.

要将代码或文本格式化为各自的不同块，请使用三反引号。
Some basic Git comands are:

```bash
git status
git add
git commit
```

## 支持颜色模型

The background color is `#ffffff` for light mode and `#000000` for dark mode.

| Color | 语法         | 示例              |
|-------|--------------|-------------------|
| HEX   | `#RRGGBB`    | `#0969DA`         |
| RGB   | `rgd(R,G,B)` | `rgd(9,105,218)`  |
| HSL   | `hsl(H,S,L)` | `hsl(212,92%,45)` |

## 链接

通过将链接文本用方括号 `[ ]` 括起来，然后将 URL 用括号 `( )` 括起来，可创建内联链接。

This site was built using [GitHub Pages](https://pages.github.com/).

## 章节链接

章节链接和一般链接相同，只不过括号里的内容改成`(#章节标题)`

## 相对链接

相对链接和一般链接相同，只不过括号里的内容改成`(doc/xxx.md)`类型，表示当前文件下的链接。

[Contribution guidelines for this project](docs/CONTRIBUTING.md)

## 映像

通过添加 `!` 并 将 `alt` 文本用 `[ ]` 括起来，可显示图像。
替换文字是等效于图像中信息的短文本。 然后将图像的链接用括号 `()` 括起来。
![Screenshot of a comment on a GitHub issue showing an image, added in the Markdown, of an Octocat smiling and raising a tentacle.](https://myoctocat.com/assets/images/base-octocat.svg)

## 列表

可以在一行或多行文本前面添加`-`、`*`或`+`来创建一个无序列表。

- George Washington
- John Adams
- Thomas Jefferson

要对列表排序，请在每行前面添加一个编号。

1. James Madison
2. James Monroe
3. John Quincy Adams

## 任务列表

若要创建任务列表，请在列表项前加连字符和空格，后接 `[ ]`。 要将任务标记为完成，请使用 `[x]`。

- [x] #739
- [ ] <https://github.com/octo-org/octo-repo/issues/740>
- [ ] Add delight to the experience when all tasks are complete 🎉

## 使用表情符号

@octocat 👍 This PR looks great - it's ready to merge! 🤭

## 段落

通过在文本行之间留一个空白行，可以创建一个新的段落。

## 脚注

您可以使用此括号语法为您的内容添加脚注：

Here is a simple footnote[^1].

A footnote can also have multiple lines[^2].

[^1]: My reference.
[^2]: To add line breaks within a footnote, prefix new lines with 2 spaces.
  This is a second line.

```md
Here is a simple footnote[^1].

A footnote can also have multiple lines[^2].

[^1]: My reference.
[^2]: To add line breaks within a footnote, prefix new lines with 2 spaces.
  This is a second line.
```

## 警报

警报是基于块引用语法的 Markdown 扩展，可用于强调关键信息。

> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.

```md
> [!NOTE]
> Useful information that users should know, even when skimming content.

> [!TIP]
> Helpful advice for doing things better or more easily.

> [!IMPORTANT]
> Key information users need to know to achieve their goal.

> [!WARNING]
> Urgent info that needs immediate user attention to avoid problems.

> [!CAUTION]
> Advises about risks or negative outcomes of certain actions.
```
