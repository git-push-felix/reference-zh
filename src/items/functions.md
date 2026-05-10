r[items.fn]
# 函数

r[items.fn.syntax]
```grammar,items
Function ->
    FunctionQualifiers `fn` IDENTIFIER GenericParams?
        `(` FunctionParameters? `)`
        FunctionReturnType? WhereClause?
        ( BlockExpression | `;` )

FunctionQualifiers -> `const`? `async`?[^async-edition] ItemSafety?[^extern-qualifiers] (`extern` Abi?)?

ItemSafety -> `safe`[^extern-safe] | `unsafe`

Abi -> STRING_LITERAL | RAW_STRING_LITERAL

FunctionParameters ->
      SelfParam `,`?
    | (SelfParam `,`)? FunctionParam (`,` FunctionParam)* `,`?

SelfParam -> OuterAttribute* ( ShorthandSelf | TypedSelf )

ShorthandSelf -> (`&` | `&` Lifetime)? `mut`? `self`

TypedSelf -> `mut`? `self` `:` Type

FunctionParam -> OuterAttribute* ( FunctionParamPattern | `...` | Type[^fn-param-2015] )

FunctionParamPattern -> PatternNoTopAlt `:` ( Type | `...` )

FunctionReturnType -> `->` Type
```

[^async-edition]: 在 2015 版本中不允许 `async` 限定符。

[^extern-safe]: `safe` 函数限定符仅在 `extern` 块中语义上允许使用。

[^extern-qualifiers]: *与早于 Rust 2024 的版本相关*：在 `extern` 块中，仅当 `extern` 被限定为 `unsafe` 时才允许 `safe` 或 `unsafe` 函数限定符。

[^fn-param-2015]: 在 2015 版本中，只有[trait 项][trait item]的关联函数才允许仅有类型的函数参数。

r[items.fn.intro]
*函数*由一个[块][block]（即函数的*主体*）以及一个名称、一组参数和输出类型组成。除名称外，所有这些都是可选的。

r[items.fn.namespace]
函数使用 `fn` 关键字声明，该关键字在函数所在模块或块的[值命名空间][value namespace]中定义给定的名称。

r[items.fn.signature]
函数可以声明一组*输入*[*变量*][variables]作为参数，调用者通过它们将实参传入函数，以及函数完成后返回给调用者的值的*输出*[*类型*][type]。

r[items.fn.implicit-return]
如果输出类型未显式声明，则为[单元类型][unit type]。

r[items.fn.fn-item-type]
当被引用时，*函数*会产生一个对应的[零大小][zero-sized]的[*函数项类型*][*function item type*]的一等*值*，调用时会对函数进行直接调用。

例如，这是一个简单函数：

```rust
fn answer_to_life_the_universe_and_everything() -> i32 {
    return 42;
}
```

r[items.fn.safety-qualifiers]
`safe` 函数在语义上仅在 [`extern` 块][`extern` block]中使用时允许。

r[items.fn.params]
## 函数参数 {#function-parameters}

r[items.fn.params.intro]
函数参数是不可反驳的[模式][patterns]，因此任何在无 `else` 的 `let` 绑定中有效的模式作为参数也是有效的：

```rust
fn first((value, _): (i32, i32)) -> i32 { value }
```

r[items.fn.params.self-pat]
如果第一个参数是 [SelfParam]，则表示该函数是一个[方法][method]。

r[items.fn.params.self-restriction]
带有 self 参数的函数只能作为 [trait] 或[实现][implementation]中的[关联函数][associated function]出现。

r[items.fn.params.varargs]
带有 `...` 标记的参数表示[可变参数函数][variadic function]，并且只能用作[外部块][external block]函数的最后一个参数。可变参数可以有可选的标识符，例如 `args: ...`。

r[items.fn.body]
## 函数体 {#function-body}

r[items.fn.body.intro]
函数的主体块在概念上被包裹在另一个块中，该块首先绑定参数模式，然后 `return` 函数主体的值。这意味着块的尾部表达式（如果被求值）最终会被返回给调用者。像往常一样，函数体内的显式 return 表达式会短路该隐式返回（如果被执行到的话）。

例如，上述函数的行为就像是这样写的：

<!-- ignore: example expansion -->
```rust,ignore
// argument_0 是调用者实际传入的第一个参数
let (value, _) = argument_0;
return {
    value
};
```

r[items.fn.body.bodyless]
没有主体块的函数以分号结尾。这种形式只能出现在 [trait] 或[外部块][external block]中。

r[items.fn.generics]
## 泛型函数 {#generic-functions}

r[items.fn.generics.intro]
*泛型函数*允许一个或多个*参数化类型*出现在其签名中。每个类型参数必须在函数名之后、用尖括号括起来的逗号分隔列表中显式声明。

```rust
// foo 对 A 和 B 是泛型的

fn foo<A, B>(x: A, y: B) {
# }
```

r[items.fn.generics.param-names]
在函数签名和函数体内，类型参数的名称可以用作类型名。

r[items.fn.generics.param-bounds]
可以为类型参数指定 [trait] 约束，以允许调用该类型的值上的方法。这通过 `where` 语法来指定：

```rust
# use std::fmt::Debug;
fn foo<T>(x: T) where T: Debug {
# }
```

r[items.fn.generics.mono]
当泛型函数被引用时，其类型会根据引用的上下文进行实例化。例如，在此处调用 `foo` 函数：

```rust
use std::fmt::Debug;

fn foo<T>(x: &[T]) where T: Debug {
    // 细节省略
}

foo(&[1, 2]);
```

将用 `i32` 实例化类型参数 `T`。

r[items.fn.generics.explicit-arguments]
类型参数也可以在函数名之后的[路径][path]尾段中显式提供。如果上下文不足以确定类型参数，这可能是必要的。例如 `mem::size_of::<u32>() == 4`。

r[items.fn.extern]
## extern 函数限定符 {#extern-function-qualifier}

r[items.fn.extern.intro]
`extern` 函数限定符允许提供可以通过特定 ABI 调用的函数*定义*：

<!-- ignore: fake ABI -->
```rust,ignore
extern "ABI" fn foo() { /* ... */ }
```

r[items.fn.extern.def]
这些通常与[外部块][external block]程序项结合使用，后者提供函数*声明*，可用于调用函数而无需提供其*定义*：

<!-- ignore: fake ABI -->
```rust,ignore
unsafe extern "ABI" {
  unsafe fn foo(); /* 没有主体 */
  safe fn bar(); /* 没有主体 */
}
unsafe { foo() };
bar();
```

r[items.fn.extern.default-abi]
当 `"extern" Abi?*` 在函数项中从 `FunctionQualifiers` 中省略时，ABI `"Rust"` 被指定。例如：

```rust
fn foo() {}
```

等价于：

```rust
extern "Rust" fn foo() {}
```

r[items.fn.extern.foreign-call]
函数可以被外部代码调用，使用与 Rust 不同的 ABI 可以（例如）提供可从其他编程语言（如 C）调用的函数：

```rust
// 声明一个具有 "C" ABI 的函数
extern "C" fn new_i32() -> i32 { 0 }

// 声明一个具有 "stdcall" ABI 的函数
# #[cfg(any(windows, target_arch = "x86"))]
extern "stdcall" fn new_i32_stdcall() -> i32 { 0 }
```

r[items.fn.extern.default-extern]
与[外部块][external block]一样，当使用 `extern` 关键字且省略 `"ABI"` 时，默认使用的 ABI 为 `"C"`。也就是说：

```rust
extern fn new_i32() -> i32 { 0 }
let fptr: extern fn() -> i32 = new_i32;
```

等价于：

```rust
extern "C" fn new_i32() -> i32 { 0 }
let fptr: extern "C" fn() -> i32 = new_i32;
```

r[items.fn.extern.unwind]
### 展开 {#unwinding}

r[items.fn.extern.unwind.intro]
大多数 ABI 字符串有两种变体，一种带有 `-unwind` 后缀，另一种不带。`Rust` ABI 始终允许展开，因此没有 `Rust-unwind` ABI。ABI 的选择与运行时的 [panic 处理器][panic handler]一起决定了展开出函数时的行为。

r[items.fn.extern.unwind.behavior]
下表指示了展开操作到达每种 ABI 边界（使用相应 ABI 字符串的函数声明或定义）时的行为。请注意，Rust 运行时不受且无法影响完全发生在其他语言运行时内部的任何展开操作的影响，即那些在不触及 Rust ABI 边界的情况下抛出和捕获的展开。

`panic`-展开列指的是通过 `panic!` 宏及类似的标准库机制进行[panic][panicking]，以及任何其他导致 panic 的 Rust 操作，例如数组越界索引或整数溢出。

"unwinding" ABI 类别指的是 `"Rust"`（未标记 `extern` 的 Rust 函数的隐式 ABI）、`"C-unwind"` 以及任何其他名称中带有 `-unwind` 的 ABI。"non-unwinding" ABI 类别指的是所有其他 ABI 字符串，包括 `"C"` 和 `"stdcall"`。

原生展开按目标平台定义。在支持抛出和捕获 C++ 异常的目标平台上，它指的是用于实现此功能的机制。某些平台实现了一种称为["强制展开"][forced-unwinding]的展开形式；Windows 上的 `longjmp` 和 `glibc` 中的 `pthread_exit` 以这种方式实现。强制展开被明确排除在下表的"原生展开"列之外。

| panic 运行时  | ABI           | `panic`-展开                        | 原生展开（非强制） |
| -------------- | ------------  | ------------------------------------- | -----------------------  |
| `panic=unwind` | unwinding     | 展开                                | 展开                   |
| `panic=unwind` | non-unwinding | 中止（见下文注释）               | [未定义行为][undefined behavior]     |
| `panic=abort`  | unwinding     | `panic` 中止而不展开      | 中止                    |
| `panic=abort`  | non-unwinding | `panic` 中止而不展开      | [未定义行为][undefined behavior]     |

r[items.fn.extern.abort]
当 `panic=unwind` 时，如果 `panic` 因非展开 ABI 边界而转为中止，则要么没有析构函数（`Drop` 调用）会运行，要么直到 ABI 边界的所有析构函数都会运行。这两种行为中具体发生哪一种未指定。

有关跨 FFI 边界展开的其他考虑和限制，请参阅 [panic 文档中的相关章节][panic-ffi]。

[forced-unwinding]: https://rust-lang.github.io/rfcs/2945-c-unwind-abi.html#forced-unwinding
[panic handler]: ../panic.md#the-panic_handler-attribute
[panic-ffi]: ../panic.md#unwinding-across-ffi-boundaries
[panicking]: ../panic.md
[undefined behavior]: ../behavior-considered-undefined.md

r[items.fn.const]
## const 函数 {#const-functions}

const 函数的定义请参见 [const 函数][const functions]。

r[items.fn.async]
## async 函数 {#async-functions}

r[items.fn.async.intro]
函数可以被限定为 async，这也可以与 `unsafe` 限定符结合使用：

```rust
async fn regular_example() { }
async unsafe fn unsafe_example() { }
```

r[items.fn.async.future]
Async 函数在调用时不执行任何工作：相反，它们将参数捕获到一个 future 中。当被轮询时，该 future 将执行函数的主体。

r[items.fn.async.desugar-brief]
Async 函数大致等价于一个返回 [`impl Future`] 并以 [`async move` 块][async-blocks]作为主体的函数：

```rust
// 源代码
async fn example(x: &str) -> usize {
    x.len()
}
```

大致等价于：

```rust
# use std::future::Future;
// 脱糖后
fn example<'a>(x: &'a str) -> impl Future<Output = usize> + 'a {
    async move { x.len() }
}
```

r[items.fn.async.desugar]
实际的脱糖更为复杂：

r[items.fn.async.lifetime-capture]
- 脱糖中的返回类型假设会捕获来自 `async fn` 声明的所有生命周期参数。这可以从上面的脱糖示例中看到，它显式地存活（并因此捕获）`'a`。

r[items.fn.async.param-capture]
- 主体中的 [`async move` 块][async-blocks] 捕获所有函数参数，包括那些未使用或绑定到 `_` 模式的参数。这确保了函数参数的释放顺序与函数不是 async 时相同，只是释放发生在返回的 future 被完全 await 之后。

关于 async 效果的更多信息，请参见 [`async` 块][async-blocks]。

[async-blocks]: ../expressions/block-expr.md#async-blocks
[`impl Future`]: ../types/impl-trait.md

r[items.fn.async.edition2018]
> [!EDITION-2018]
> Async 函数仅从 Rust 2018 开始可用。

r[items.fn.async.safety]
### 结合 `async` 和 `unsafe` {#combining-async-and-unsafe}

r[items.fn.async.safety.intro]
声明一个既是 async 又是 unsafe 的函数是合法的。生成的函数调用时是不安全的，并且（像任何 async 函数一样）返回一个 future。这个 future 只是一个普通的 future，因此不需要 `unsafe` 上下文来"await"它：

```rust
// 返回一个 future，当被 await 时会解引用 `x`。
//
// 安全性条件：`x` 必须安全可解引用，直到
// 返回的 future 完成。
async unsafe fn unsafe_example(x: *const i32) -> i32 {
  *x
}

async fn safe_example() {
    // 初始调用函数需要 `unsafe` 块：
    let p = 22;
    let future = unsafe { unsafe_example(&p) };

    // 但这里不需要 `unsafe` 块。这将
    // 读取 `p` 的值：
    let q = future.await;
}
```

请注意，这种行为是脱糖为返回 `impl Future` 的函数的结果——在这种情况下，我们脱糖得到的函数是一个 `unsafe` 函数，但返回值保持不变。

Unsafe 在 async 函数上的使用方式与在其他函数上的使用方式完全相同：它表示函数对调用者施加了一些额外的义务以确保安全性。与任何其他 unsafe 函数一样，这些条件可能超出初始调用本身——例如，在上面的代码片段中，`unsafe_example` 函数接受一个指针 `x` 作为参数，然后（在被 await 时）解引用该指针。这意味着 `x` 必须有效直到 future 完成执行，而调用者有责任确保这一点。

r[items.fn.attributes]
## 函数上的属性 {#attributes-on-functions}

r[items.fn.attributes.intro]
函数上允许使用[外部属性][attributes]。[内部属性][attributes] 允许直接在其主体[块][block]的 `{` 之后使用。

此示例展示了函数上的内部属性。该函数仅用单词 "Example" 进行文档化。

```rust
fn documented() {
    #![doc = "Example"]
}
```

> [!NOTE]
> 除了 lint 之外，在函数项上习惯上只使用外部属性。

r[items.fn.attributes.builtin-attributes]
对函数有意义的属性有：

- [`cfg_attr`]
- [`cfg`]
- [`cold`]
- [`deprecated`]
- [`doc`]
- [`export_name`]
- [`inline`]
- [`link_section`]
- [`must_use`]
- [`no_mangle`]
- [Lint 检查属性][lint check attributes]
- [过程宏属性][procedural macro attributes]
- [测试属性][testing attributes]

r[items.fn.param-attributes]
## 函数参数上的属性 {#attributes-on-function-parameters}

r[items.fn.param-attributes.intro]
函数参数上允许使用[外部属性][attributes]，允许的[内置属性][built-in attributes]仅限于 `cfg`、`cfg_attr`、`allow`、`warn`、`deny` 和 `forbid`。

```rust
fn len(
    #[cfg(windows)] slice: &[u16],
    #[cfg(not(windows))] slice: &[u8],
) -> usize {
    slice.len()
}
```

r[items.fn.param-attributes.parsed-attributes]
应用于程序项的过程宏属性所使用的惰性辅助属性也是允许的，但要注意不要将这些惰性属性包含在最终的 `TokenStream` 中。

例如，以下代码定义了一个惰性 `some_inert_attribute` 属性，该属性未在任何地方正式定义，而 `some_proc_macro_attribute` 过程宏负责检测其存在并将其从输出 token 流中移除。

<!-- ignore: requires proc macro -->
```rust,ignore
#[some_proc_macro_attribute]
fn foo_oof(#[some_inert_attribute] arg: u8) {
}
```

[const contexts]: ../const_eval.md#const-context
[const functions]: ../const_eval.md#const-functions
[external block]: external-blocks.md
[path]: ../paths.md
[block]: ../expressions/block-expr.md
[variables]: ../variables.md
[type]: ../types.md#type-expressions
[unit type]: ../types/tuple.md
[*function item type*]: ../types/function-item.md
[Trait]: traits.md
[attributes]: ../attributes.md
[`cfg`]: ../conditional-compilation.md#the-cfg-attribute
[`cfg_attr`]: ../conditional-compilation.md#the-cfg_attr-attribute
[lint check attributes]: ../attributes/diagnostics.md#lint-check-attributes
[procedural macro attributes]: macro.proc.attribute
[testing attributes]: ../attributes/testing.md
[`cold`]: ../attributes/codegen.md#the-cold-attribute
[`inline`]: ../attributes/codegen.md#the-inline-attribute
[`deprecated`]: ../attributes/diagnostics.md#the-deprecated-attribute
[`doc`]: ../../rustdoc/the-doc-attribute.html
[`must_use`]: ../attributes/diagnostics.md#the-must_use-attribute
[patterns]: ../patterns.md
[`export_name`]: ../abi.md#the-export_name-attribute
[`link_section`]: ../abi.md#the-link_section-attribute
[`no_mangle`]: ../abi.md#the-no_mangle-attribute
[built-in attributes]: ../attributes.md#built-in-attributes-index
[trait item]: traits.md
[method]: associated-items.md#methods
[associated function]: associated-items.md#associated-functions-and-methods
[implementation]: implementations.md
[value namespace]: ../names/namespaces.md
[variadic function]: external-blocks.md#variadic-functions
[`extern` block]: external-blocks.md
[zero-sized]: glossary.zst
