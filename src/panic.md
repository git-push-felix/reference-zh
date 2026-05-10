r[panic]
# Panic

r[panic.intro]
Rust 提供了一种机制来阻止函数正常返回，而是"panic"，这是对通常不期望在遇到错误上下文中可恢复的错误条件的响应。

r[panic.lang-ops]
某些语言构造，例如越界[数组索引][array indexing]，会自动 panic。

r[panic.control]
还有一些语言特性提供对 panic 行为的一定程度的控制：

* [_panic 处理器_][panic handler] 定义了 panic 的行为。
* [FFI ABI](items/functions.md#unwinding) 可以改变 panic 的行为方式。

> [!NOTE]
> 标准库提供了通过 [`panic!` 宏][panic!]显式 panic 的能力。

r[panic.panic_handler]
## `panic_handler` 属性

r[panic.panic_handler.intro]
*`panic_handler` 属性*可以应用于一个函数以定义 panic 的行为。

r[panic.panic_handler.allowed-positions]
`panic_handler` 属性只能应用于签名为 `fn(&PanicInfo) -> !` 的函数。

> [!NOTE]
> [`PanicInfo`] 结构体包含有关 panic 位置的信息。

r[panic.panic_handler.unique]
依赖图中必须有一个单一的 `panic_handler` 函数。

下面展示了一个 `panic_handler` 函数，该函数记录 panic 消息然后暂停线程。

<!-- ignore: test infrastructure can't handle no_std -->
```rust,ignore
#![no_std]

use core::fmt::{self, Write};
use core::panic::PanicInfo;

struct Sink {
    // ..
#    _0: (),
}
#
# impl Sink {
#     fn new() -> Sink { Sink { _0: () }}
# }
#
# impl fmt::Write for Sink {
#     fn write_str(&mut self, _: &str) -> fmt::Result { Ok(()) }
# }

#[panic_handler]
fn panic(info: &PanicInfo) -> ! {
    let mut sink = Sink::new();

    // 将 "panicked at '$reason', src/main.rs:27:4" 记录到某个 `sink`
    let _ = writeln!(sink, "{}", info);

    loop {}
}
```

r[panic.panic_handler.std]
### 标准行为

r[panic.panic_handler.std.kinds]
`std` 提供了两种不同的 panic 处理器：

* `unwind` --- 展开调用栈，可能可恢复。
* `abort` ---- 中止进程，不可恢复。

并非所有目标都提供 `unwind` 处理器。

> [!NOTE]
> 链接 `std` 时使用的 panic 处理器可以通过 [`-C panic`] CLI 标志设置。大多数目标的默认值是 `unwind`。
>
> 标准库的 panic 行为可以在运行时通过 [`std::panic::set_hook`] 函数修改。

r[panic.panic_handler.std.no_std]
链接 [`no_std`] 二进制文件、dylib、cdylib 或 staticlib 将需要指定你自己的 panic 处理器。

r[panic.strategy]
## Panic 策略

r[panic.strategy.intro]
*panic 策略* 定义了 crate 构建所支持的 panic 行为类型。

> [!NOTE]
> panic 策略可以通过 `rustc` 的 [`-C panic`] CLI 标志选择。
>
> 在生成二进制文件、dynamic library、cdylib 或 staticlib 并链接 `std` 时，`-C panic` CLI 标志也影响使用哪个 [panic 处理器][panic handler]。

> [!NOTE]
> 当使用 `abort` panic 策略编译代码时，优化器可以假设跨越 Rust 帧的展开是不可能的，这可以带来代码大小和运行时速度的改进。

> [!NOTE]
> 有关链接具有不同 panic 策略的 crate 的限制，请参见 [link.unwinding]。一个推论是，使用 `unwind` 策略构建的 crate 可以使用 `abort` panic 处理器，但 `abort` 策略不能使用 `unwind` panic 处理器。

r[panic.unwind]
## 展开（Unwinding）

r[panic.unwind.intro]
Panic 可以是可恢复的或不可恢复的，尽管可以通过选择非展开 panic 处理器将其配置为始终不可恢复。（反过来不成立：`unwind` 处理器不保证所有 panic 都是可恢复的，只保证通过 `panic!` 宏和类似的标准库机制的 panic 是可恢复的。）

r[panic.unwind.destruction]
当 panic 发生时，`unwind` 处理器"展开"Rust 帧，就像 C++ 的 `throw` 展开 C++ 帧一样，直到 panic 到达恢复点（例如在线程边界）。这意味着当 panic 遍历 Rust 帧时，这些帧中的[实现了 `Drop`][destructors] 的活动对象将调用它们的 `drop` 方法。因此，当正常执行恢复时，不再可访问的对象将已经被"清理"，就像它们正常地离开作用域一样。

> [!NOTE]
> 只要保留了这种资源清理的保证，"展开"可以在不实际使用目标平台 C++ 使用的机制的情况下实现。

> [!NOTE]
> 标准库提供了两种从 panic 恢复的机制，[`std::panic::catch_unwind`]（允许在 panic 线程内恢复）和 [`std::thread::spawn`]（自动为生成的线程设置 panic 恢复，以便其他线程可以继续运行）。

r[panic.unwind.ffi]
### 跨 FFI 边界的展开

r[panic.unwind.ffi.intro]
可以使用[适当的 ABI 声明][unwind-abi]跨 FFI 边界展开。虽然在某些情况下有用，但这为未定义行为创造了独特的机会，特别是在涉及多个语言运行时的情况下。

r[panic.unwind.ffi.undefined]
使用错误的 ABI 展开是未定义行为：

* 从通过非展开 ABI（如 `"C"`、`"system"` 等）声明的函数声明或指针调用的外部函数导致展开进入 Rust 代码。（例如，当这样用 C++ 编写的函数抛出一个未被捕获并传播到 Rust 的异常时会发生这种情况。）
* 从不支持展开的代码中调用会展开的 Rust `extern` 函数（使用 `extern "C-unwind"` 或其他允许展开的 ABI），例如使用 GCC 或 Clang 的 `-fno-exceptions` 编译的代码

r[panic.unwind.ffi.catch-foreign]
使用 [`std::panic::catch_unwind`]、[`std::thread::JoinHandle::join`] 捕获外部展开操作（如 C++ 异常），或让它传播越过 Rust `main()` 函数或线程根，将具有以下两种行为之一，且未指定哪种会发生：

* 进程中止。
* 函数返回包含不透明类型的 [`Result::Err`]。

> [!NOTE]
> 使用不同实例的 Rust 标准库编译或链接的 Rust 代码在此保证的目的下被视为"外部异常"。因此，一个使用 `panic!` 并链接到一个版本的 Rust 标准库的库，从使用不同版本标准库的应用程序调用，可能导致整个应用程序中止，即使该库仅在子线程中使用。

r[panic.unwind.ffi.dispose-panic]
目前没有关于外部运行时尝试处置或重新抛出 Rust `panic` 负载时会发生什么的保证。换句话说，源自 Rust 运行时的展开必须导致进程终止或被同一运行时捕获。

[`-C panic`]: ../rustc/codegen-options/index.html#panic
[`no_std`]: names/preludes.md#the-no_std-attribute
[`PanicInfo`]: core::panic::PanicInfo
[array indexing]: expressions/array-expr.md#array-and-slice-indexing-expressions
[attribute]: attributes.md
[destructors]: destructors.md
[panic handler]: #the-panic_handler-attribute
[runtime]: runtime.md
[unwind-abi]: items/functions.md#unwinding
