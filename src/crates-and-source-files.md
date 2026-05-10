r[crate]
# crate 与源文件

r[crate.syntax]
```grammar,items
@root Crate ->
    InnerAttribute*
    Item*
```

> [!NOTE]
> 尽管 Rust 像其他任何语言一样可以由解释器和编译器实现，但唯一的现有实现是编译器，并且该语言始终被设计为可编译的。因此，本节假设使用编译器。

r[crate.compile-time]
Rust 的语义遵循编译期和运行时之间的*阶段区分*。[^phase-distinction]具有*静态解释*的语义规则决定编译的成功或失败，而具有*动态解释*的语义规则决定程序在运行时的行为。

r[crate.unit]
编译模型以称为 _crate_ 的工件为中心。每次编译处理一个源形式的 crate，如果成功，则生成一个二进制形式的 crate：可以是可执行文件或某种库。[^cratesourcefile]

r[crate.module]
_crate_ 是编译和链接的单位，也是版本控制、分发和运行时加载的单位。一个 crate 包含一个嵌套[模块][module]作用域的*树*。这棵树的顶层是一个匿名模块（从模块内路径的角度来看），crate 中的任何项都有一个规范的[模块路径][module path]，表示其在 crate 模块树中的位置。

r[crate.input-source]
Rust 编译器总是以单个源文件作为输入被调用，并且总是生成单个输出 crate。对该源文件的处理可能导致其他源文件作为模块被加载。源文件具有 `.rs` 扩展名。

r[crate.module-def]
一个 Rust 源文件描述了一个模块，其名称和位置——在当前 crate 的模块树中——是从源文件外部定义的：要么通过引用源文件中的显式 [Module][grammar-Module] 项，要么通过 crate 本身的名称。

r[crate.inline-module]
每个源文件都是一个模块，但并非每个模块都需要自己的源文件：[模块定义][module]可以嵌套在一个文件内。

r[crate.items]
每个源文件包含零个或多个[项][Item]定义的序列，并且可以选择性地以任意数量的[属性][attributes]开头，这些属性适用于包含它的模块，其中大多数会影响编译器的行为。

r[crate.attributes]
匿名 crate 模块可以具有适用于整个 crate 的附加属性。

> [!NOTE]
> 文件的内容前面可以有一个 [shebang]。

```rust
// 指定 crate 名称。
#![crate_name = "projx"]

// 指定输出工件的类型。
#![crate_type = "lib"]

// 开启一个警告。
// 这可以在任何模块中完成，不仅仅是匿名 crate 模块。
#![warn(non_camel_case_types)]
```

r[crate.main]
## main 函数

r[crate.main.executable]
包含 `main` [函数][function]的 crate 可以编译为可执行文件。

r[crate.main.restriction]
如果存在 `main` 函数，它必须不接受任何参数，不能声明任何 [trait 约束或生命周期约束][trait or lifetime bounds]，不能有任何 [where 子句][where clauses]，并且其返回类型必须实现 [`Termination`] trait。

```rust
fn main() {}
```
```rust
fn main() -> ! {
    std::process::exit(0);
}
```
```rust
fn main() -> impl std::process::Termination {
    std::process::ExitCode::SUCCESS
}
```

r[crate.main.import]
`main` 函数可以是一个导入，例如来自外部 crate 或当前 crate。

```rust
mod foo {
    pub fn bar() {
        println!("Hello, world!");
    }
}
use foo::bar as main;
```

> [!NOTE]
> 标准库中实现了 [`Termination`] 的类型包括：
>
> * `()`
> * [`!`]
> * [`Infallible`]
> * [`ExitCode`]
> * `Result<T, E> where T: Termination, E: Debug`

<!-- If the previous section needs updating (from "must take no arguments"
  onwards, also update it in the testing.md file -->

r[crate.uncaught-foreign-unwinding]
### 未捕获的外部展开

当"外部"展开（例如，C++ 代码抛出的异常，或使用不同 panic 处理器的 Rust 代码中的 `panic!`）传播到 `main` 函数之外时，进程将被安全终止。这可能采取中止的形式，在这种情况下不能保证任何 `Drop` 调用会被执行，并且错误输出可能比由"原生" Rust `panic` 终止运行时提供的信息更少。

更多信息，请参见 [panic 文档][panic-docs]。

r[crate.no_main]
### `no_main` 属性

*`no_main` [属性][attribute]* 可以应用于 crate 级别，以禁用为可执行二进制文件发出 `main` 符号。当某个其他被链接的对象定义了 `main` 时，这很有用。

r[crate.crate_name]
## `crate_name` 属性 {#the-crate_name-attribute}

r[crate.crate_name.general]
*`crate_name` [属性][attribute]* 可以应用于 crate 级别，以使用 [MetaNameValueStr] 语法指定 crate 的名称。

```rust
#![crate_name = "mycrate"]
```

r[crate.crate_name.restriction]
crate 名称不能为空，且只能包含 [Unicode 字母数字][Unicode alphanumeric]或 `_`（U+005F）字符。

[^phase-distinction]: 这种区分在解释器中也会存在。像语法分析、类型检查和 lint 这样的静态检查应该在程序执行之前发生，无论何时执行。

[^cratesourcefile]: 一个 crate 在某种程度上类似于 ECMA-335 CLI 模型中的*程序集*，SML/NJ 编译管理器中的*库*，Owens 和 Flatt 模块系统中的*单元*，或者 Mesa 中的*配置*。

[Unicode alphanumeric]: char::is_alphanumeric
[`!`]: types/never.md
[`ExitCode`]: std::process::ExitCode
[`Infallible`]: std::convert::Infallible
[`Termination`]: std::process::Termination
[attribute]: attributes.md
[attributes]: attributes.md
[function]: items/functions.md
[module]: items/modules.md
[module path]: paths.md
[panic-docs]: panic.md#unwinding-across-ffi-boundaries
[shebang]: shebang.md
[trait or lifetime bounds]: trait-bounds.md
[where clauses]: items/generics.md#where-clauses
