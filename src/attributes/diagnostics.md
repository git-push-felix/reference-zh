r[attributes.diagnostics]
# 诊断属性

以下[属性][attributes]用于控制或生成编译期间的诊断消息。

r[attributes.diagnostics.lint]
## Lint 检查属性

Lint 检查命名了一种潜在的不受欢迎的编码模式，例如不可达代码或遗漏的文档。

r[attributes.diagnostics.lint.level]
lint 属性 `allow`、`expect`、`warn`、`deny` 和 `forbid` 使用 [MetaListPaths] 语法来指定一个 lint 名称列表，以更改应用该属性的实体的 lint 级别。

对于任何 lint 检查 `C`：

r[attributes.diagnostics.lint.allow]
* `#[allow(C)]` 覆盖对 `C` 的检查，使得违规不会被报告。

r[attributes.diagnostics.lint.expect]
* `#[expect(C)]` 指示预期会发出 lint `C`。该属性将抑制 `C` 的发出，或者如果预期未被满足则发出警告。

r[attributes.diagnostics.lint.warn]
* `#[warn(C)]` 对 `C` 的违规发出警告但继续编译。

r[attributes.diagnostics.lint.deny]
* `#[deny(C)]` 在遇到 `C` 的违规后发出错误信号，

r[attributes.diagnostics.lint.forbid]
* `#[forbid(C)]` 与 `deny(C)` 相同，但也禁止之后更改 lint 级别，

> [!NOTE]
> `rustc` 支持的 lint 检查可通过 `rustc -W help` 找到，以及它们的默认设置，并在 [rustc 书][rustc book]中有文档记录。

```rust
pub mod m1 {
    // 此处忽略缺少文档
    #[allow(missing_docs)]
    pub fn undocumented_one() -> i32 { 1 }

    // 此处缺少文档发出警告
    #[warn(missing_docs)]
    pub fn undocumented_too() -> i32 { 2 }

    // 此处缺少文档发出错误
    #[deny(missing_docs)]
    pub fn undocumented_end() -> i32 { 3 }
}
```

r[attributes.diagnostics.lint.override]
Lint 属性可以覆盖前一个属性指定的级别，只要该级别不试图更改已禁止的 lint（除了 `deny`，在 `forbid` 上下文中允许但被忽略）。前一个属性是语法树中更高级别的属性，或按从左到右源顺序在同一实体上的前一个属性。

此示例展示了如何使用 `allow` 和 `warn` 来切换特定检查的开关：

```rust
#[warn(missing_docs)]
pub mod m2 {
    #[allow(missing_docs)]
    pub mod nested {
        // 此处忽略缺少文档
        pub fn undocumented_one() -> i32 { 1 }

        // 此处缺少文档发出警告，
        // 尽管上面有 allow。
        #[warn(missing_docs)]
        pub fn undocumented_two() -> i32 { 2 }
    }

    // 此处缺少文档发出警告
    pub fn undocumented_too() -> i32 { 3 }
}
```

此示例展示了如何使用 `forbid` 来禁止对该 lint 检查使用 `allow` 或 `expect`：

```rust,compile_fail
#[forbid(missing_docs)]
pub mod m3 {
    // 尝试切换警告在此处发出错误
    #[allow(missing_docs)]
    /// 返回 2。
    pub fn undocumented_too() -> i32 { 2 }
}
```

> [!NOTE]
> `rustc` 允许在[命令行][rustc-lint-cli]上设置 lint 级别，还支持设置报告的 lint 的[上限][rustc-lint-caps]。

r[attributes.diagnostics.lint.reason]
### Lint 原因

所有 lint 属性都支持一个额外的 `reason` 参数，用于说明添加某个属性的原因。如果 lint 在定义的级别上发出，此原因将作为 lint 消息的一部分显示。

```rust,edition2015,compile_fail
// `keyword_idents` 默认是 allow 的。此处我们将其 deny 以避免
// 在更新版次时迁移标识符。
#![deny(
    keyword_idents,
    reason = "we want to avoid these idents to be future compatible"
)]

// 此名称在 Rust 2015 版次中是允许的。我们仍然希望避免
// 此名称以保持未来兼容性并且不混淆最终用户。
fn dyn() {}
```

另一个示例，其中 lint 被 allow 并带有原因：

```rust
use std::path::PathBuf;

pub fn get_path() -> PathBuf {
    // `allow` 属性上的 `reason` 参数充当读者的文档。
    #[allow(unused_mut, reason = "this is only modified on some platforms")]
    let mut file_name = PathBuf::from("git");

    #[cfg(target_os = "windows")]
    file_name.set_extension("exe");

    file_name
}
```

r[attributes.diagnostics.expect]
### `#[expect]` 属性

r[attributes.diagnostics.expect.intro]
`#[expect(C)]` 属性为 lint `C` 创建一个 lint 预期。如果同一位置的 `#[warn(C)]` 属性会导致 lint 发出，则该预期将被满足。如果预期未满足，因为 lint `C` 不会被发出，则 `unfulfilled_lint_expectations` lint 将在该属性处发出。

```rust
fn main() {
    // 此 `#[expect]` 属性创建一个 lint 预期，即以下语句会发出
    // `unused_variables` lint。此预期未满足，因为 `question` 变量被
    // `println!` 宏使用。因此，`unfulfilled_lint_expectations` lint
    // 将在该属性处发出。
    #[expect(unused_variables)]
    let question = "who lives in a pineapple under the sea?";
    println!("{question}");

    // 此 `#[expect]` 属性创建一个将被满足的 lint 预期，因为
    // `answer` 变量从未被使用。通常会发出的 `unused_variables` lint
    // 被抑制。不会为该语句或属性发出警告。
    #[expect(unused_variables)]
    let answer = "SpongeBob SquarePants!";
}
```

r[attributes.diagnostics.expect.fulfillment]
Lint 预期仅由已被 `expect` 属性抑制的 lint 发出来满足。如果在该作用域中使用其他级别属性（如 `allow` 或 `warn`）修改了 lint 级别，则 lint 发出将相应地处理，而预期将保持未满足。

```rust
#[expect(unused_variables)]
fn select_song() {
    // 这将在 warn 级别发出 `unused_variables` lint，
    // 如 `warn` 属性所定义。这不会满足函数上方的预期。
    #[warn(unused_variables)]
    let song_name = "Crab Rave";

    // `allow` 属性抑制 lint 发出。这不会满足预期，因为它
    // 已被 `allow` 属性抑制，而不是函数上方的 `expect` 属性。
    #[allow(unused_variables)]
    let song_creator = "Noisestorm";

    // 此 `expect` 属性将抑制此变量处的 `unused_variables` lint 发出。
    // 函数上方的 `expect` 属性仍然不会被满足，因为此 lint 发出
    // 已被局部的 expect 属性抑制。
    #[expect(unused_variables)]
    let song_version = "Monstercat Release";
}
```

r[attributes.diagnostics.expect.independent]
如果 `expect` 属性包含多个 lint，每个 lint 被单独预期。对于 lint 组，只要组内有一个 lint 被发出就足够了：

```rust
// 此预期将被函数内部的未使用值满足，因为发出的
// `unused_variables` lint 在 `unused` lint 组内。
#[expect(unused)]
pub fn thoughts() {
    let unused = "I'm running out of examples";
}

pub fn another_example() {
    // 此属性创建两个 lint 预期。`unused_mut` lint 将被抑制，
    // 从而满足第一个预期。`unused_variables` 不会被发出，
    // 因为变量被使用了。因此该预期将未被满足，并将发出警告。
    #[expect(unused_mut, unused_variables)]
    let mut link = "https://www.rust-lang.org/";

    println!("Welcome to our community: {link}");
}
```

> [!NOTE]
> `#[expect(unfulfilled_lint_expectations)]` 的行为目前定义为始终生成 `unfulfilled_lint_expectations` lint。

r[attributes.diagnostics.lint.group]
### Lint 组

Lint 可以组织成命名组，以便相关 lint 的级别可以一起调整。使用命名组等效于列出该组内的 lint。

```rust,compile_fail
// 这将 allow "unused" 组中的所有 lint。
#[allow(unused)]
// 这将覆盖 "unused" 组中的 "unused_must_use" lint 为 deny。
#[deny(unused_must_use)]
fn example() {
    // 这不生成警告，因为 "unused_variables" lint
    // 在 "unused" 组中。
    let x = 1;
    // 这生成一个错误，因为结果未被使用且
    // "unused_must_use" 被标记为 "deny"。
    std::fs::remove_file("some_file"); // 错误：未使用的必须被使用的 `Result`
}
```

r[attributes.diagnostics.lint.group.warnings]
有一个特殊的组名为 "warnings"，包含所有处于 "warn" 级别的 lint。"warnings" 组忽略属性顺序，应用于实体内所有原本会发出警告的 lint。

```rust,compile_fail
# unsafe fn an_unsafe_fn() {}
// 这两个属性的顺序不重要。
#[deny(warnings)]
// unsafe_code lint 默认通常是 "allow"。
#[warn(unsafe_code)]
fn example_err() {
    // 这是一个错误，因为 `unsafe_code` 警告已被提升为 "deny"。
    unsafe { an_unsafe_fn() } // 错误：使用了 `unsafe` 块
}
```

r[attributes.diagnostics.lint.tool]
### 工具 lint 属性

r[attributes.diagnostics.lint.tool.intro]
工具 lint 允许使用作用域限定 lint，以 `allow`、`warn`、`deny` 或 `forbid` 某些工具的 lint。

r[attributes.diagnostics.lint.tool.activation]
工具 lint 仅在关联的工具处于活动状态时才被检查。如果 lint 属性（如 `allow`）引用了一个不存在的工具 lint，编译器不会警告该不存在的 lint，直到你使用该工具。

否则，它们的工作方式与常规 lint 属性完全相同：

```rust
// 将整个 `pedantic` clippy lint 组设置为 warn
#![warn(clippy::pedantic)]
// 静默 `filter_map` clippy lint 的警告
#![allow(clippy::filter_map)]

fn main() {
    // ...
}

// 仅为这个函数静默 `cmp_nan` clippy lint
#[allow(clippy::cmp_nan)]
fn foo() {
    // ...
}
```

> [!NOTE]
> `rustc` 目前识别 "[clippy]" 和 "[rustdoc]" 的工具 lint。

r[attributes.diagnostics.deprecated]
## `deprecated` 属性

r[attributes.diagnostics.deprecated.intro]
*`deprecated` 属性*将项标记为已弃用。`rustc` 将在使用 `#[deprecated]` 项时发出警告。`rustdoc` 将显示项弃用信息，包括 `since` 版本和 `note`（如果可用）。

r[attributes.diagnostics.deprecated.syntax]
`deprecated` 属性有多种形式：

- `deprecated` --- 发出一条通用消息。
- `deprecated = "message"` --- 在弃用消息中包含给定的字符串。
- [MetaListNameValueStr] 语法，带有两个可选字段：
  - `since` --- 指定项被弃用的版本号。`rustc` 目前不解释该字符串，但外部工具如 [Clippy] 可能会检查值的有效性。
  - `note` --- 指定应包含在弃用消息中的字符串。这通常用于提供关于弃用的解释和首选的替代方案。

r[attributes.diagnostic.deprecated.allowed-positions]
`deprecated` 属性可以应用于任何[项][item]、[trait 项][trait item]、[枚举变体][enum variant]、[结构体字段][struct field]、[外部块项][external block item]或[宏定义][macro definition]。它不能应用于 [trait 实现项][trait-impl]。当应用于包含其他项的项（如[模块][module]或[实现][implementation]）时，所有子项继承该弃用属性。

以下是一个示例：

```rust
#[deprecated(since = "5.2.0", note = "foo was rarely used. Users should instead use bar")]
pub fn foo() {}

pub fn bar() {}
```

[RFC][1270-deprecation.md] 包含动机和更多细节。

[1270-deprecation.md]: https://github.com/rust-lang/rfcs/blob/master/text/1270-deprecation.md

<!-- template:attributes -->
r[attributes.diagnostics.must_use]
## `must_use` 属性

r[attributes.diagnostics.must_use.intro]
*`must_use` [属性][attribute]* 标记一个应该被使用的值。

r[attributes.diagnostics.must_use.syntax]
`must_use` 属性使用 [MetaWord] 和 [MetaNameValueStr] 语法。

> [!EXAMPLE]
> ```rust
> #[must_use]
> fn use_me1() -> u8 { 0 }
>
> #[must_use = "explanation of why it should be used"]
> fn use_me2() -> u8 { 0 }
> ```

r[attributes.diagnostics.must_use.allowed-positions]
`must_use` 属性可以应用于：

- [结构体][Struct]
- [枚举][Enumeration]
- [联合体][Union]
- [函数][Function]
- [Trait]

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[attributes.diagnostics.must_use.duplicates]
`must_use` 属性在一个项上只能使用一次。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

r[attributes.diagnostics.must_use.message]
`must_use` 属性可以使用 [MetaNameValueStr] 语法包含一条消息，例如 `#[must_use = "example message"]`。该消息可能作为 lint 的一部分发出。

r[attributes.diagnostics.must_use.type]
当该属性应用于[结构体][struct]、[枚举][enumeration]或[联合体][union]时，如果[表达式语句][expression statement]的[表达式][expression]具有该类型，则使用会触发 `unused_must_use` lint。

```rust,compile_fail
#![deny(unused_must_use)]
#[must_use]
struct MustUse();
MustUse(); // 错误：必须被使用的未使用值。
```

r[attributes.diagnostics.must_use.type.uninhabited]
作为 [attributes.diagnostics.must_use.type] 的例外，当 `E` 是[无人居住的][uninhabited]或 `B` 是无人居住的时，对于 `Result<(), E>` 或 `ControlFlow<B, ()>` 不触发该 lint。来自外部 crate 的 `#[non_exhaustive]` 类型在此目的下不被视为无人居住的，因为它可能在未来获得构造函数。

```rust
#![deny(unused_must_use)]
# use core::ops::ControlFlow;
enum Empty {}
fn f1() -> Result<(), Empty> { Ok(()) }
f1(); // 正确：`Empty` 是无人居住的。
fn f2() -> ControlFlow<Empty, ()> { ControlFlow::Continue(()) }
f2(); // 正确：`Empty` 是无人居住的。
```

r[attributes.diagnostics.must_use.fn]
如果[表达式语句][expression statement]的[表达式][expression]是[调用表达式][call expression]或[方法调用表达式][method call expression]，且其函数操作数是应用了该属性的函数，则使用会触发 `unused_must_use` lint。

```rust,compile_fail
#![deny(unused_must_use)]
#[must_use]
fn f() {}
f(); // 错误：必须被使用的未使用返回值。
```

r[attributes.diagnostics.must_use.trait]
如果[表达式语句][expression statement]的[表达式][expression]是[调用表达式][call expression]或[方法调用表达式][method call expression]，且其函数操作数是一个返回 [impl trait] 或 [dyn trait] 类型的函数，而该类型的约束中有一个或多个 trait 被标记了该属性，则使用会触发 `unused_must_use` lint。

```rust,compile_fail
#![deny(unused_must_use)]
#[must_use]
trait Tr {}
impl Tr for () {}
fn f() -> impl Tr {}
f(); // 错误：必须被使用的未使用实现者。
```

r[attributes.diagnostics.must_use.trait-function]
当该属性应用于 trait 声明中的函数时，[attributes.diagnostics.must_use.fn] 中描述的规则在[调用表达式][call expression]或[方法调用表达式][method call expression]的函数操作数是该函数的实现时也适用。

```rust,compile_fail
#![deny(unused_must_use)]
trait Tr {
    #[must_use]
    fn use_me(&self);
}

impl Tr for () {
    fn use_me(&self) {}
}

().use_me(); // 错误：必须被使用的未使用返回值。
```

```rust,compile_fail
# #![deny(unused_must_use)]
# trait Tr {
#     #[must_use]
#     fn use_me(&self);
# }
#
# impl Tr for () {
#     fn use_me(&self) {}
# }
#
<() as Tr>::use_me(&());
//          ^^^^^^^^^^^ 错误：必须被使用的未使用返回值。
```

r[attributes.diagnostics.must_use.block-expr]
在针对 [attributes.diagnostics.must_use.type]、[attributes.diagnostics.must_use.fn]、[attributes.diagnostics.must_use.trait] 和 [attributes.diagnostics.must_use.trait-function] 检查[表达式语句][expression statement]的[表达式][expression]时，该 lint 会穿透[块表达式][block expression]（包括 [`unsafe` 块][`unsafe` blocks]和[带标签块表达式][labeled block expressions]）到每个块的尾部表达式。这递归地适用于嵌套块表达式。

```rust,compile_fail
#![deny(unused_must_use)]
#[must_use]
fn f() {}

{ f() };        // 错误：lint 穿透块表达式。
unsafe { f() }; // 错误：lint 穿透 `unsafe` 块。
{ { f() } };    // 错误：lint 穿透嵌套块。
```

r[attributes.diagnostics.must_use.trait-impl-function]
当用于 trait 实现中的函数时，该属性没有任何作用。

```rust
#![deny(unused_must_use)]
trait Tr {
    fn f(&self);
}

impl Tr for () {
    #[must_use] // 这没有效果。
    fn f(&self) {}
}

().f(); // 正确。
```

> [!NOTE]
> `rustc` 会对 trait 实现中的函数使用发出 lint 警告。这可能在将来成为错误。

r[attributes.diagnostics.must_use.wrapping-suppression]
> [!NOTE]
> 将 `#[must_use]` 函数的结果包装在某种表达式中可以抑制[基于函数的检查][attributes.diagnostics.must_use.fn]，因为[表达式语句][expression statement]的[表达式][expression]不是对 `#[must_use]` 函数的[调用表达式][call expression]或[方法调用表达式][method call expression]。如果整体表达式的类型是 `#[must_use]`，则[基于类型的检查][attributes.diagnostics.must_use.type]仍然适用。
>
> ```rust
> #![deny(unused_must_use)]
> #[must_use]
> fn f() {}
>
> // 基于函数的检查不会对以下任何情况触发，因为
> // 表达式语句的表达式不是对 `#[must_use]` 函数的调用。
> (f(),);                    // 表达式是元组，不是调用。
> Some(f());                 // 被调用者 `Some` 不是 `#[must_use]`。
> if true { f() } else {};   // 表达式是 `if`，不是调用。
> match true {               // 表达式是 `match`，不是调用。
>     _ => f()
> };
> ```
>
> ```rust,compile_fail
> #![deny(unused_must_use)]
> #[must_use]
> struct MustUse;
> fn g() -> MustUse { MustUse }
>
> // 尽管 `if` 表达式不是调用，基于类型的检查会触发，
> // 因为表达式的类型是 `MustUse`，该类型具有
> // `#[must_use]` 属性。
> if true { g() } else { MustUse }; // 错误：必须被使用。
> ```

r[attributes.diagnostics.must_use.underscore-idiom]
> [!NOTE]
> 当有意丢弃一个 must-used 值时，使用模式为 `_` 的 [let 语句][let statement]或[解构赋值][destructuring assignment]是惯用的。
>
> ```rust
> #![deny(unused_must_use)]
> #[must_use]
> fn f() {}
> let _ = f(); // 正确。
> _ = f(); // 正确。
> ```

r[attributes.diagnostic.namespace]
## `diagnostic` 工具属性命名空间

r[attributes.diagnostic.namespace.intro]
`#[diagnostic]` 属性命名空间是影响编译时错误消息的属性的集合。这些属性提供的提示不保证被使用。

r[attributes.diagnostic.namespace.unknown-invalid-syntax]
此命名空间中的未知属性被接受，尽管可能发出未使用属性的警告。此外，对已知属性的无效输入通常将是警告（详见属性定义）。这意味着允许在未来添加或丢弃属性和更改输入，而无需保持无意义的属性或选项正常工作。

r[attributes.diagnostic.on_unimplemented]
### `diagnostic::on_unimplemented` 属性

r[attributes.diagnostic.on_unimplemented.intro]
`#[diagnostic::on_unimplemented]` 属性是对编译器的提示，用于补充在需要 trait 但类型未实现该 trait 的情况下通常会生成的错误消息。

r[attributes.diagnostic.on_unimplemented.allowed-positions]
该属性应放置在 [trait 声明][trait declaration]上，尽管放在其他位置也不是错误。

r[attributes.diagnostic.on_unimplemented.syntax]
该属性使用 [MetaListNameValueStr] 语法来指定其输入，尽管对属性任何格式错误的输入不被视为错误，以提供向前和向后兼容性。

r[attributes.diagnostic.on_unimplemented.keys]
以下键具有给定的含义：
* `message` --- 顶层错误消息的文本。
* `label` --- 在错误消息的损坏代码中内联显示的标签文本。
* `note` --- 提供附加注释。

r[attributes.diagnostic.on_unimplemented.note-repetition]
`note` 选项可以出现多次，这将导致发出多条注释消息。

r[attributes.diagnostic.on_unimplemented.repetition]
如果任何其他选项出现多次，相关选项的第一次出现指定实际使用的值。后续出现会生成警告。

r[attributes.diagnostic.on_unimplemented.unknown-keys]
任何未知键都会生成警告。

r[attributes.diagnostic.on_unimplemented.format-string]
所有三个选项都接受字符串作为参数，使用与 [`std::fmt`] 字符串相同的格式进行解释。

r[attributes.diagnostic.on_unimplemented.format-parameters]
带有给定命名参数的格式参数将被替换为以下文本：
* `{Self}` --- 实现 trait 的类型的名称。
* `{` *GenericParameterName* `}` --- 给定泛型参数的泛型参数类型的名称。

r[attributes.diagnostic.on_unimplemented.invalid-formats]
任何其他格式参数将生成警告，但否则将按原样包含在字符串中。

r[attributes.diagnostic.on_unimplemented.invalid-string]
无效的格式字符串可能生成警告，但其他方面是允许的，但可能不会按预期显示。格式说明符可能生成警告，但其他方面被忽略。

在此示例中：

```rust,compile_fail,E0277
#[diagnostic::on_unimplemented(
    message = "My Message for `ImportantTrait<{A}>` implemented for `{Self}`",
    label = "My Label",
    note = "Note 1",
    note = "Note 2"
)]
trait ImportantTrait<A> {}

fn use_my_trait(_: impl ImportantTrait<i32>) {}

fn main() {
    use_my_trait(String::new());
}
```

编译器可能生成如下错误消息：

```text
error[E0277]: My Message for `ImportantTrait<i32>` implemented for `String`
  --> src/main.rs:14:18
   |
14 |     use_my_trait(String::new());
   |     ------------ ^^^^^^^^^^^^^ My Label
   |     |
   |     required by a bound introduced by this call
   |
   = help: the trait `ImportantTrait<i32>` is not implemented for `String`
   = note: Note 1
   = note: Note 2
```

r[attributes.diagnostic.do_not_recommend]
### `diagnostic::do_not_recommend` 属性

r[attributes.diagnostic.do_not_recommend.intro]
`#[diagnostic::do_not_recommend]` 属性是对编译器的提示，不要在诊断消息中显示标注的 trait 实现。

> [!NOTE]
> 如果你知道该推荐通常对程序员没有帮助，抑制推荐可能会有用。这通常发生在广泛的毯式 impl 上。推荐可能会将程序员引向错误的方向，或者 trait 实现可能是你不想暴露的内部细节，或者约束可能无法被程序员满足。
>
> 例如，在关于类型未实现所需 trait 的错误消息中，编译器可能会找到一个如果没有 trait 实现中的特定约束就能满足要求的 trait 实现。编译器可能会告诉用户存在一个 impl，但问题在于 trait 实现中的约束。`#[diagnostic::do_not_recommend]` 属性可以用来告诉编译器*不要*告诉用户关于该 trait 实现的信息，而是简单地告诉用户该类型未实现所需的 trait。

r[attributes.diagnostic.do_not_recommend.allowed-positions]
该属性应放置在 [trait 实现项][trait-impl]上，尽管放在其他位置也不是错误。

r[attributes.diagnostic.do_not_recommend.syntax]
该属性不接受任何参数，但意外参数不被视为错误。

在以下示例中，有一个称为 `AsExpression` 的 trait，用于将任意类型转换为 SQL 库中使用的 `Expression` 类型。有一个名为 `check` 的方法，它接受一个 `AsExpression`。

```rust,compile_fail,E0277
# pub trait Expression {
#     type SqlType;
# }
#
# pub trait AsExpression<ST> {
#     type Expression: Expression<SqlType = ST>;
# }
#
# pub struct Text;
# pub struct Integer;
#
# pub struct Bound<T>(T);
# pub struct SelectInt;
#
# impl Expression for SelectInt {
#     type SqlType = Integer;
# }
#
# impl<T> Expression for Bound<T> {
#     type SqlType = T;
# }
#
# impl AsExpression<Integer> for i32 {
#     type Expression = Bound<Integer>;
# }
#
# impl AsExpression<Text> for &'_ str {
#     type Expression = Bound<Text>;
# }
#
# impl<T> Foo for T where T: Expression {}

// 取消此行注释以更改推荐。
// #[diagnostic::do_not_recommend]
impl<T, ST> AsExpression<ST> for T
where
    T: Expression<SqlType = ST>,
{
    type Expression = T;
}

trait Foo: Expression + Sized {
    fn check<T>(&self, _: T) -> <T as AsExpression<<Self as Expression>::SqlType>>::Expression
    where
        T: AsExpression<Self::SqlType>,
    {
        todo!()
    }
}

fn main() {
    SelectInt.check("bar");
}
```

`SelectInt` 类型的 `check` 方法期望一个 `Integer` 类型。用 i32 类型调用它可以工作，因为它通过 `AsExpression` trait 被转换为 `Integer`。然而，用字符串调用它不会工作，并生成可能如下所示的错误：

```text
error[E0277]: the trait bound `&str: Expression` is not satisfied
  --> src/main.rs:53:15
   |
53 |     SelectInt.check("bar");
   |               ^^^^^ the trait `Expression` is not implemented for `&str`
   |
   = help: the following other types implement trait `Expression`:
             Bound<T>
             SelectInt
note: required for `&str` to implement `AsExpression<Integer>`
  --> src/main.rs:45:13
   |
45 | impl<T, ST> AsExpression<ST> for T
   |             ^^^^^^^^^^^^^^^^     ^
46 | where
47 |     T: Expression<SqlType = ST>,
   |        ------------------------ unsatisfied trait bound introduced here
```

通过将 `#[diagnostic::do_not_recommend]` 属性添加到 `AsExpression` 的毯式 `impl` 中，消息变为：

```text
error[E0277]: the trait bound `&str: AsExpression<Integer>` is not satisfied
  --> src/main.rs:53:15
   |
53 |     SelectInt.check("bar");
   |               ^^^^^ the trait `AsExpression<Integer>` is not implemented for `&str`
   |
   = help: the trait `AsExpression<Integer>` is not implemented for `&str`
           but trait `AsExpression<Text>` is implemented for it
   = help: for that trait implementation, expected `Text`, found `Integer`
```

第一条错误消息包含有关 `&str` 和 `Expression` 关系以及毯式 impl 中未满足的 trait 约束的有些令人困惑的错误消息。添加 `#[diagnostic::do_not_recommend]` 后，它不再为该推荐考虑毯式 impl。消息应该更清晰一些，表明字符串无法转换为 `Integer`。

[Clippy]: https://github.com/rust-lang/rust-clippy
[`Drop`]: ../special-types-and-traits.md#drop
[`unsafe` blocks]: ../expressions/block-expr.md#unsafe-blocks
[attribute]: ../attributes.md
[attributes]: ../attributes.md
[block expression]: ../expressions/block-expr.md
[call expression]: ../expressions/call-expr.md
[destructuring assignment]: expr.assign.destructure
[method call expression]: ../expressions/method-call-expr.md
[dyn trait]: ../types/trait-object.md
[enum variant]: ../items/enumerations.md
[enumeration]: ../items/enumerations.md
[expression statement]: ../statements.md#expression-statements
[expression]: ../expressions.md
[external block item]: ../items/external-blocks.md
[functions]: ../items/functions.md
[impl trait]: ../types/impl-trait.md
[implementation]: ../items/implementations.md
[item]: ../items.md
[labeled block expressions]: ../expressions/block-expr.md#labeled-block-expressions
[let statement]: ../statements.md#let-statements
[macro definition]: ../macros-by-example.md
[module]: ../items/modules.md
[rustc book]: ../../rustc/lints/index.html
[rustc-lint-caps]: ../../rustc/lints/levels.html#capping-lints
[rustc-lint-cli]: ../../rustc/lints/levels.html#via-compiler-flag
[rustdoc]: ../../rustdoc/lints.html
[struct field]: ../items/structs.md
[struct]: ../items/structs.md
[external block]: ../items/external-blocks.md
[trait declaration]: ../items/traits.md
[trait item]: ../items/traits.md
[trait-impl]: ../items/implementations.md#trait-implementations
[traits]: ../items/traits.md
[uninhabited]: glossary.uninhabited
[union]: ../items/unions.md
