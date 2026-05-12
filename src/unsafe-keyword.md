r[unsafe]
# `unsafe` 关键字

r[unsafe.intro]
`unsafe` 关键字用于创建或解除证明某事物是安全的义务。具体来说：

- 它用于标记那些*定义*了在别处必须遵守的额外安全条件的代码。
  - 这包括 `unsafe fn`、`unsafe static` 和 `unsafe trait`。
- 它用于标记程序员*断言*满足别处所定义安全条件的代码。
  - 这包括 `unsafe {}`、`unsafe impl`、不带 [`unsafe_op_in_unsafe_fn`] 的 `unsafe fn`、`unsafe extern` 和 `#[unsafe(attr)]`。

下文将逐一讨论这些情况。
请参阅[关键字文档][keyword]了解一些示例说明。

r[unsafe.positions]
`unsafe` 关键字可以出现在以下几种不同的上下文中：

- 不安全函数（`unsafe fn`）
- 不安全块（`unsafe {}`）
- 不安全 trait（`unsafe trait`）
- 不安全 trait 实现（`unsafe impl`）
- 不安全外部块（`unsafe extern`）
- 不安全外部静态变量（`unsafe static`）
- 不安全属性（`#[unsafe(attr)]`）

r[unsafe.fn]
## 不安全函数（`unsafe fn`）{#unsafe-functions-unsafe-fn}

r[unsafe.fn.intro]
不安全函数是指并非在所有上下文和/或所有可能输入下都安全的函数。
我们说它们具有*额外安全条件*，即所有调用者必须满足、而编译器不做检查的要求。
例如，[`get_unchecked`] 的额外安全条件是索引必须在边界内。
不安全函数应附带文档说明这些额外安全条件是什么。

r[unsafe.fn.safety]
此类函数必须以 `unsafe` 关键字为前缀，并且只能在 `unsafe` 块内调用，或者在未启用 [`unsafe_op_in_unsafe_fn`] lint 的情况下，在 `unsafe fn` 内部调用。

r[unsafe.block]
## 不安全块（`unsafe {}`）{#unsafe-blocks-unsafe-}

r[unsafe.block.intro]
代码块可以用 `unsafe` 关键字作为前缀，以允许使用[不安全操作][Unsafety]章节中定义的那些不安全动作，例如调用其他不安全函数或解引用裸指针。

r[unsafe.block.fn-body]
默认情况下，不安全函数体也被视为一个不安全块；
可以通过启用 [`unsafe_op_in_unsafe_fn`] lint 来改变此行为。

通过将操作放入不安全块中，程序员声明他们已经处理好满足该块内所有操作的额外安全条件。

不安全块是不安全函数的逻辑对偶：
不安全函数定义了调用者必须履行的证明义务，而不安全块则声明块内调用的所有函数或操作的相关证明义务已被解除。
解除证明义务有多种方式；
例如，可以有运行时检查或数据结构不变量来保证某些性质确实成立，或者不安全块可以位于 `unsafe fn` 内部，此时该块可以使用该函数的证明义务来解除块内产生的证明义务。

不安全块用于包装外部库、直接使用硬件或实现语言中不直接存在的特性。
例如，Rust 提供了在语言中实现内存安全并发所需的语言特性，但标准库中线程和消息传递的实现使用了不安全块。

Rust 的类型系统是对动态安全需求的保守近似，因此在某些情况下使用安全代码会有性能代价。
例如，双向链表不是树形结构，在安全代码中只能用引用计数指针来表示。
通过使用 `unsafe` 块将反向链接表示为裸指针，可以在不使用引用计数的情况下实现。
（请参阅["Learn Rust With Entirely Too Many Linked Lists"](https://rust-unofficial.github.io/too-many-lists/) 来深入探索这个具体示例。）

[Unsafety]: unsafety.md

r[unsafe.trait]
## 不安全 trait（`unsafe trait`）

r[unsafe.trait.intro]
不安全 trait 是带有*trait 实现者*必须遵守的额外安全条件的 trait。
不安全 trait 应附带文档说明这些额外安全条件是什么。

r[unsafe.trait.safety]
此类 trait 必须以 `unsafe` 关键字为前缀，并且只能由 `unsafe impl` 块来实现。

r[unsafe.impl]
## 不安全 trait 实现（`unsafe impl`）

在实现不安全 trait 时，需要以 `unsafe` 关键字作为实现的前缀。
通过写作 `unsafe impl`，程序员声明他们已经处理好满足该 trait 所要求的额外安全条件。

不安全 trait 实现是不安全 trait 的逻辑对偶：不安全 trait 定义了实现者必须履行的证明义务，而不安全实现则声明所有相关证明义务已被解除。

[keyword]: ../std/keyword.unsafe.html
[`get_unchecked`]: slice::get_unchecked
[`unsafe_op_in_unsafe_fn`]: ../rustc/lints/listing/allowed-by-default.html#unsafe-op-in-unsafe-fn

r[unsafe.extern]
## 不安全外部块（`unsafe extern`）

声明[外部块][external block]的程序员必须确保其中包含的项的签名是正确的。未能做到这一点可能导致未定义行为。通过书写 `unsafe extern` 来表明此义务已被履行。

r[unsafe.extern.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，`extern` 块允许不用 `unsafe` 修饰。

[external block]: items/external-blocks.md

r[unsafe.attribute]
## 不安全属性（`#[unsafe(attr)]`）

[不安全属性][unsafe attribute]是指具有额外安全条件、使用时必须遵守的属性。编译器无法检查这些条件是否已被满足。为了断言它们已被满足，这些属性必须包裹在 `unsafe(..)` 中，例如 `#[unsafe(no_mangle)]`。

[unsafe attribute]: attributes.md
