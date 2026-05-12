# 引言

本书是 Rust 编程语言的主要参考手册。

> [!NOTE]
> 关于本书中已知的错误和遗漏，请参阅我们的 [GitHub issues]。如果你发现编译器行为与此处文本不一致的情况，请提交 issue，以便我们判断哪一方是正确的。

## Rust 版本发布

Rust 每六周发布一个新语言版本。
该语言的第一个稳定版本是 Rust 1.0.0，随后是 Rust 1.1.0，依此类推。
工具（`rustc`、`cargo` 等）和文档（[标准库][standard library]、本书等）随语言版本一起发布。

本书的最新版本（与最新的 Rust 版本对应）始终可以在 <https://doc.rust-lang.org/reference/> 找到。
之前的版本可以通过在 "reference" 目录前添加 Rust 版本来找到。
例如，Rust 1.49.0 的参考手册位于 <https://doc.rust-lang.org/1.49.0/reference/>。

## 《Rust 参考手册》不是什么

本书不作为语言入门教程。
假定读者已经具备该语言的背景知识。
有一本[配套书籍][book]可用来帮助获得这种背景知识。

本书也不作为语言发行版中包含的[标准库][standard library]的参考。
这些库的文档是通过从其源代码中提取文档属性来单独生成的。
许多人们可能认为是语言特性的东西实际上是 Rust 中的库特性，所以你要找的内容可能在那里，而不是这里。

同样，本书通常不记录 `rustc` 作为工具或 Cargo 的具体细节。
`rustc` 有自己的[手册][rustc book]。
Cargo 有一本[手册][cargo book]，其中包含一个[参考][cargo reference]。
但仍有少数页面（例如[链接][linkage]）描述了 `rustc` 的工作方式。

本书也仅作为稳定版 Rust 中可用内容的参考。
关于正在开发中的不稳定特性，请参阅[不稳定特性手册][Unstable Book]。

Rust 编译器（包括 `rustc`）会执行优化。
本参考手册不指定哪些优化是允许或不允许的。
相反，可以把编译后的程序看作一个黑盒。
你只能通过运行它、给它输入并观察其输出来探测它。
通过这种方式发生的一切都必须符合本参考手册的规定。

## 如何使用本书

本书并不假设你是按顺序阅读的。
每章通常可以独立阅读，但会交叉链接到其他章节，以引用它们涉及但未讨论的语言层面。

阅读本文档主要有两种方式。

第一种是回答特定问题。
如果你知道哪个章节回答了该问题，可以直接跳到目录中的该章。
否则，你可以按 `s` 键或点击顶部栏的放大镜图标，搜索与问题相关的关键词。
例如，你想知道 let 语句中创建的临时值何时被丢弃。
如果你还不知道[临时值的生命周期][lifetime of temporaries]是在[表达式章节][expressions chapter]中定义的，你可以搜索 "temporary let"，第一个搜索结果就会带你到那一节。

第二种是全面提升你对语言某个层面的了解。
在这种情况下，只需浏览目录，直到看到你想进一步了解的内容，然后开始阅读。
如果某个链接看起来有趣，点击它并阅读该节。

话虽如此，阅读本书没有错误的方式。以你觉得最有帮助的方式阅读即可。

### 约定

与所有技术书籍一样，本书在信息展示方面有一些约定。
这些约定记录在此处。

* 定义术语的语句将该术语以*斜体*显示。
  每当该术语在该章之外使用时，通常是一个指向包含此定义的小节的链接。

  *示例术语*就是一个正在被定义的术语的示例。

* 正文描述最新的稳定版本。与先前版本的差异在版本块中单独说明：

  > [!EDITION-2018]
  > 在 2018 版本之前，行为是这样的。从 2018 版本开始，行为是那样的。

* 包含关于本书状态的有用信息或指出有用但大多超出范围的信息的注释，放在注释块中。

  > [!NOTE]
  > 这是一个示例注释。

* 示例块展示演示某个规则或指出某些有趣方面的示例。有些示例可能有隐藏行，可以通过将鼠标悬停或点击示例时出现的眼睛图标来查看。

  > [!EXAMPLE]
  > 这是一个代码示例。
  > ```rust
  > println!("hello world");
  > ```

* 展示语言中的不安全行为或可能令人困惑的语言特性交互的警告，放在特殊的警告框中。

  > [!WARNING]
  > 这是一个示例警告。

* 文本中的内联代码片段放在 `<code>` 标签内。

  较长的代码示例放在语法高亮的框中，右上角有复制、执行和显示隐藏行的控件。

  ```rust
  # // 这是隐藏行。
  fn main() {
      println!("This is a code example");
  }
  ```

  除非另有说明，所有示例都针对最新版本编写。

* 语法和词法产生式在[记法][Notation]章节中描述。

r[example.rule.label]
* 规则标识符出现在每个语言规则之前，用方括号括起来。这些标识符提供了一种引用和链接到语言中特定规则的方式（[例如][example rule]）。规则标识符使用句点将各部分从最通用到最具体地分隔开（例如 [destructors.scope.nesting.function-body]）。在窄屏幕上，规则名称会折叠显示为 `[*]`。

  点击规则名称可以链接到该规则。

  > [!WARNING]
  > 规则的组织方式目前尚在调整中。目前，这些标识符名称在各版本之间并不稳定，如果发生变化，指向这些规则的链接可能会失效。我们计划在组织方式稳定后，使得指向规则名称的链接在版本之间不会断开。

* 有关联测试的规则会在其下方包含一个 `Tests` 链接（在窄屏幕上，链接为 `[T]`）。点击该链接会弹出一个测试列表，可以点击查看测试。例如，参见 [input.encoding.utf8]。

  将规则链接到测试是一项持续进行的工作。概览请参见[测试摘要](test-summary.md)章节。

## 贡献

我们欢迎各种形式的贡献。

你可以通过向 [the Rust Reference repository] 提交 issue 或 pull request 来为本书做出贡献。
如果本书没有回答你的问题，并且你认为答案属于本书的范围，请随时[提交 issue][file an issue]或在 [Zulip] 上的 `t-lang/doc` 频道中询问。
了解人们最常将本书用于什么目的，有助于我们将注意力集中在使这些章节尽可能完善上。
当然，如果你看到任何错误或非规范性但未明确标明的内容，也请[提交 issue][file an issue]。

[book]: ../book/index.html
[github issues]: https://github.com/rust-lang/reference/issues
[standard library]: std
[the Rust Reference repository]: https://github.com/rust-lang/reference/
[Unstable Book]: https://doc.rust-lang.org/nightly/unstable-book/
[cargo book]: ../cargo/index.html
[cargo reference]: ../cargo/reference/index.html
[example rule]: example.rule.label
[expressions chapter]: expressions.html
[file an issue]: https://github.com/rust-lang/reference/issues
[lifetime of temporaries]: expressions.html#temporaries
[linkage]: linkage.html
[rustc book]: ../rustc/index.html
[Notation]: notation.md
[Zulip]: https://rust-lang.zulipchat.com/#narrow/stream/237824-t-lang.2Fdoc
