r[names.preludes]
# 预导入

r[names.preludes.intro]
*预导入*是一组自动带入 crate 中每个模块作用域的名称集合。

这些预导入名称不是模块本身的一部分：它们在[名称解析][name resolution]期间被隐式查询。例如，尽管像 [`Box`] 这样的名称在每个模块中都在作用域内，你不能将其引用为 `self::Box`，因为它不是当前模块的成员。

r[names.preludes.kinds]
存在几种不同的预导入：

- [标准库预导入][Standard library prelude]
- [外部 crate 预导入][Extern prelude]
- [语言预导入][Language prelude]
- [`macro_use` 预导入][`macro_use` prelude]
- [工具预导入][Tool prelude]

r[names.preludes.std]
## 标准库预导入 {#standard-library-prelude}

r[names.preludes.std.intro]
每个 crate 都有一个标准库预导入，由来自单个标准库模块的名称组成。

r[names.preludes.std.module]
所使用的模块取决于 crate 的版次以及 [`no_std` 属性][`no_std` attribute]是否应用于 crate：

版次  | `no_std` 未应用            | `no_std` 已应用
--------| --------------------------- | ----------------------------
2015    | [`std::prelude::rust_2015`] | [`core::prelude::rust_2015`]
2018    | [`std::prelude::rust_2018`] | [`core::prelude::rust_2018`]
2021    | [`std::prelude::rust_2021`] | [`core::prelude::rust_2021`]
2024    | [`std::prelude::rust_2024`] | [`core::prelude::rust_2024`]

> [!NOTE]
> [`std::prelude::rust_2015`] 和 [`std::prelude::rust_2018`] 与 [`std::prelude::v1`] 具有相同的内容。
>
> [`core::prelude::rust_2015`] 和 [`core::prelude::rust_2018`] 与 [`core::prelude::v1`] 具有相同的内容。

> [!NOTE]
> 当 [`core::panic!`] 或 [`std::panic!`] 之一由于[标准库预导入][standard library prelude]被带入作用域，而用户编写的 [glob 导入][glob import]将另一个带入作用域时，`rustc` 目前允许使用 `panic!`，即使它是有歧义的。用户编写的 glob 导入优先以解决此歧义。
>
> 详情请参见 [names.resolution.expansion.imports.ambiguity.panic-hack]。

r[names.preludes.extern]
## 外部 crate 预导入 {#extern-prelude}

r[names.preludes.extern.intro]
在根模块中使用 [`extern crate`] 导入的或提供给编译器的（如使用 `rustc` 的 `--extern` 标志）外部 crate 被添加到*外部 crate 预导入*中。如果使用别名导入，例如 `extern crate orig_name as new_name`，则符号 `new_name` 被添加到预导入中。

r[names.preludes.extern.core]
[`core`] crate 始终被添加到外部 crate 预导入中。

r[names.preludes.extern.std]
只要在 crate 根中未指定 [`no_std` 属性][`no_std` attribute]，[`std`] crate 就会被添加。

r[names.preludes.extern.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，外部 crate 预导入中的 crate 不能通过 [use 声明][use declarations]引用，因此通常的标准做法是包含 `extern crate` 声明将其带入作用域。
>
> 从 2018 版次开始，[use 声明][use declarations]可以引用外部 crate 预导入中的 crate，因此使用 `extern crate` 被认为是不惯用的。

> [!NOTE]
> 随 `rustc` 一起分发的其他 crate，例如 [`alloc`] 和 [`test`](mod@test)，在使用 Cargo 时不会通过 `--extern` 标志自动包含。即使在 2018 版次中，也必须使用 `extern crate` 声明将其带入作用域。
>
> ```rust
> extern crate alloc;
> use alloc::rc::Rc;
> ```
>
> 对于仅 proc-macro crate，Cargo 确实会将 `proc_macro` 引入外部 crate 预导入中。

<!--
See https://github.com/rust-lang/rust/issues/57288 for more about the alloc/test limitation.
-->

<!-- template:attributes -->
r[names.preludes.extern.no_std]
### `no_std` 属性 {#the-no_std-attribute}

r[names.preludes.extern.no_std.intro]
*`no_std` [属性][attributes]* 导致 [`std`] crate 不被自动链接，并且[标准库预导入][standard library prelude]改为使用 `core` 预导入。

> [!EXAMPLE]
> <!-- ignore: test infrastructure can't handle no_std -->
> ```rust,ignore
> #![no_std]
> ```

> [!NOTE]
> 当 crate 面向不支持标准库的平台，或有目的地不使用标准库的功能时，使用 `no_std` 是有用的。这些功能主要是动态内存分配（如 `Box` 和 `Vec`）以及文件和网络功能（如 `std::fs` 和 `std::io`）。

> [!WARNING]
> 使用 `no_std` 不会阻止标准库被链接。在 crate 或其依赖之一中写入 `extern crate std` 仍然是有效的；这将导致编译器将 `std` crate 链接到程序中。

r[names.preludes.extern.no_std.syntax]
`no_std` 属性使用 [MetaWord] 语法。

r[names.preludes.extern.no_std.allowed-positions]
`no_std` 属性只能应用于 crate 根。

r[names.preludes.extern.no_std.duplicates]
`no_std` 属性可以在一个形式上使用任意次数。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。

r[names.preludes.extern.no_std.module]
`no_std` 属性将[标准库预导入][standard library prelude]改为使用 `core` 预导入而非 `std` 预导入。

r[names.preludes.extern.no_std.edition2018]
> [!EDITION-2018]
> 在 2018 版次之前，`std` 默认被注入到 crate 根中。如果指定了 `no_std`，则改为注入 `core`。从 2018 版次开始，无论是否指定 `no_std`，两者都不会被注入到 crate 根中。

r[names.preludes.lang]
## 语言预导入 {#language-prelude}

r[names.preludes.lang.intro]
语言预导入包括语言内置的类型和属性名称。语言预导入始终在作用域中。

r[names.preludes.lang.entities]
它包括以下内容：

* [类型命名空间][Type namespace]
    * [布尔类型][Boolean type] --- `bool`
    * [`char`]
    * [`str`]
    * [整数类型][Integer types] --- `i8`、`i16`、`i32`、`i64`、`i128`、`u8`、`u16`、`u32`、`u64`、`u128`
    * [机器相关整数类型][Machine-dependent integer types] --- `usize` 和 `isize`
    * [浮点类型][floating-point types] --- `f32` 和 `f64`
* [宏命名空间][Macro namespace]
    * [内置属性][Built-in attributes]
    * [内置派生宏][attributes.derive.built-in]

r[names.preludes.macro_use]
## `macro_use` 预导入 {#macro_use-prelude}

r[names.preludes.macro_use.intro]
`macro_use` 预导入包括通过对 [`extern crate`] 应用 [`macro_use` 属性][`macro_use` attribute]从外部 crate 导入的宏。

r[names.preludes.tool]
## 工具预导入 {#tool-prelude}

r[names.preludes.tool.intro]
工具预导入包括[类型命名空间][Type namespace]中外部工具的工具名称。更多细节请参见[工具属性][tool attributes]部分。

<!-- template:attributes -->
r[names.preludes.no_implicit_prelude]
## `no_implicit_prelude` 属性 {#the-no_implicit_prelude-attribute}

r[names.preludes.no_implicit_prelude.intro]
*`no_implicit_prelude` [属性][attribute]* 用于阻止隐式预导入被带入作用域。

> [!EXAMPLE]
> ```rust
> // 该属性可以应用于 crate 根以影响所有模块。
> #![no_implicit_prelude]
>
> // 或者可以应用于一个模块以仅影响该模块及其后代。
> #[no_implicit_prelude]
> mod example {
>     // ...
> }
> ```

r[names.preludes.no_implicit_prelude.syntax]
`no_implicit_prelude` 属性使用 [MetaWord] 语法。

r[names.preludes.no_implicit_prelude.allowed-positions]
`no_implicit_prelude` 属性只能应用于 crate 或模块。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[names.preludes.no_implicit_prelude.duplicates]
`no_implicit_prelude` 属性可以在一个形式上使用任意次数。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。

r[names.preludes.no_implicit_prelude.excluded-preludes]
`no_implicit_prelude` 属性阻止[标准库预导入][standard library prelude]、[外部 crate 预导入][extern prelude]、[`macro_use` 预导入][`macro_use` prelude]和[工具预导入][tool prelude]被带入模块及其后代的作用域。

r[names.preludes.no_implicit_prelude.implicitly-imported-macros]
> [!NOTE]
> 尽管有 `#![no_implicit_prelude]`，`rustc` 目前仍隐式地将某些宏带入作用域。这些宏是：
>
> - [`assert!`]
> - [`cfg!`]
> - [`cfg_select!`]
> - [`column!`]
> - [`compile_error!`]
> - [`concat!`]
> - [`concat_bytes!`]
> - [`env!`]
> - [`file!`]
> - [`format_args!`]
> - [`include!`]
> - [`include_bytes!`]
> - [`include_str!`]
> - [`line!`]
> - [`module_path!`]
> - [`option_env!`]
> - [`panic!`]
> - [`stringify!`]
> - [`unreachable!`]
>
> 例如，这可以工作：
>
> ```rust
> #![no_implicit_prelude]
> fn main() { assert!(true); }
> ```
>
> 不要依赖此行为；它可能在未来被移除。在使用 `#![no_implicit_prelude]` 时，始终显式地将你需要的项带入作用域。
>
> 详情请参见 [Rust PR #62086](https://github.com/rust-lang/rust/pull/62086) 和 [Rust PR #139493](https://github.com/rust-lang/rust/pull/139493)。

r[names.preludes.no_implicit_prelude.lang]
`no_implicit_prelude` 属性不影响[语言预导入][language prelude]。

r[names.preludes.no_implicit_prelude.edition2018]
> [!EDITION-2018]
> 在 2015 版次中，`no_implicit_prelude` 属性不影响 [`macro_use` 预导入][`macro_use` prelude]，标准库导出的所有宏仍包含在 `macro_use` 预导入中。从 2018 版次开始，该属性确实移除了 `macro_use` 预导入。

[`char`]: ../types/char.md
[`extern crate`]: ../items/extern-crates.md
[`macro_use` attribute]: ../macros-by-example.md#the-macro_use-attribute
[`macro_use` prelude]: #macro_use-prelude
[`no_std` attribute]: #the-no_std-attribute
[`str`]: ../types/str.md
[attribute]: ../attributes.md
[Boolean type]: ../types/boolean.md
[Built-in attributes]: ../attributes.md#built-in-attributes-index
[extern prelude]: #extern-prelude
[floating-point types]: ../types/numeric.md#floating-point-types
[glob import]: items.use.glob
[Integer types]: ../types/numeric.md#integer-types
[Language prelude]: #language-prelude
[Machine-dependent integer types]: ../types/numeric.md#machine-dependent-integer-types
[Macro namespace]: namespaces.md
[name resolution]: name-resolution.md
[standard library prelude]: names.preludes.std
[tool attributes]: ../attributes.md#tool-attributes
[Tool prelude]: #tool-prelude
[Type namespace]: namespaces.md
[use declarations]: ../items/use-declarations.md
