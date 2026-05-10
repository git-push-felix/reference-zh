r[items.static]
# 静态项

r[items.static.syntax]
```grammar,items
StaticItem ->
    ItemSafety?[^extern-safety] `static` `mut`? IDENTIFIER `:` Type ( `=` Expression )? `;`
```

[^extern-safety]: `safe` 和 `unsafe` 函数限定符仅在 `extern` 块中语义上允许使用。

r[items.static.intro]
*静态项*类似于[常量][constant]，不同之处在于它表示程序中的一个分配，该分配由初始化器表达式初始化。对静态项的所有引用和原始指针指向同一个分配。

r[items.static.lifetime]
静态项具有 `static` 生命周期，它比 Rust 程序中的所有其他生命周期都长。静态项在程序结束时不会调用 [`drop`]。

r[items.static.storage-disjointness]
如果 `static` 的大小至少为 1 字节，则此分配与所有其他此类 `static` 分配以及堆分配和栈分配变量不相交。但是，不可变 `static` 项的存储可以与本身没有唯一地址的分配（例如[提升项][promoteds]和 [`const` 项][constant]）重叠。

r[items.static.namespace]
静态声明在其所在模块或块的[值命名空间][value namespace]中定义静态值。

r[items.static.init]
静态初始化器是在编译时求值的[常量表达式][constant expression]。静态初始化器可以引用和读取其他静态项。当读取可变静态项时，它们读取该静态项的初始值。

r[items.static.read-only]
不包含[内部可变][interior mutable]类型的非 `mut` 静态项可以被放置在只读内存中。

r[items.static.safety]
对静态项的所有访问都是安全的，但对静态项有一些限制：

r[items.static.sync]
* 类型必须具有 [`Sync`](std::marker::Sync) trait 约束以允许线程安全访问。

r[items.static.init.omission]
初始化器表达式在[外部块][external block]中必须省略，而对于自由静态项则必须提供。

r[items.static.safety-qualifiers]
`safe` 和 `unsafe` 限定符在语义上仅允许在[外部块][external block]中使用。

r[items.static.generics]
## 静态项与泛型 {#statics--generics}

在泛型作用域（例如在 blanket 实现或默认实现中）中定义的静态项将导致恰好定义一个静态项，就像静态定义被从当前作用域提取到模块中一样。*不会*对每个单态化产生一个程序项。

以下代码：

```rust
use std::sync::atomic::{AtomicUsize, Ordering};

trait Tr {
    fn default_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("default_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }

    fn blanket_impl();
}

struct Ty1 {}
struct Ty2 {}

impl<T> Tr for T {
    fn blanket_impl() {
        static COUNTER: AtomicUsize = AtomicUsize::new(0);
        println!("blanket_impl: counter was {}", COUNTER.fetch_add(1, Ordering::Relaxed));
    }
}

fn main() {
    <Ty1 as Tr>::default_impl();
    <Ty2 as Tr>::default_impl();
    <Ty1 as Tr>::blanket_impl();
    <Ty2 as Tr>::blanket_impl();
}
```

打印

```text
default_impl: counter was 0
default_impl: counter was 1
blanket_impl: counter was 0
blanket_impl: counter was 1
```

r[items.static.mut]
## 可变静态项 {#mutable-statics}

r[items.static.mut.intro]
如果静态项使用 `mut` 关键字声明，则程序可以修改它。Rust 的目标之一是使并发 bug 难以出现，而这显然是一个非常大的竞态条件或其他 bug 的来源。

r[items.static.mut.safety]
因此，读取或写入可变静态变量时需要 `unsafe` 块。应注意确保对可变静态项的修改对在同一进程中运行的其他线程是安全的。

r[items.static.mut.extern]
然而，可变静态项仍然非常有用。它们可以与 C 库一起使用，也可以在 `extern` 块中从 C 库绑定。

```rust
# fn atomic_add(_: *mut u32, _: u32) -> u32 { 2 }

static mut LEVELS: u32 = 0;

// 这违反了无共享状态的理念，并且内部没有
// 针对竞态的保护，所以此函数是 `unsafe` 的
unsafe fn bump_levels_unsafe() -> u32 {
    unsafe {
        let ret = LEVELS;
        LEVELS += 1;
        return ret;
    }
}

// 作为 `bump_levels_unsafe` 的替代方案，此函数是安全的，
// 假设我们有一个返回旧值的 atomic_add 函数。此
// 函数只有在没有其他代码以非原子方式访问该静态项时才是安全的。
// 如果此类访问是可能的（例如 `bump_levels_unsafe` 中），
// 那么这需要是 `unsafe` 的，以向调用者表明他们
// 仍然必须防范并发访问。
fn bump_levels_safe() -> u32 {
    unsafe {
        return atomic_add(&raw mut LEVELS, 1);
    }
}
```

r[items.static.mut.sync]
可变静态项具有与普通静态项相同的限制，只是类型不必实现 `Sync` trait。

r[items.static.alternate]
## 使用静态项还是常量 {#using-statics-or-consts}

是否应该使用常量项还是静态项可能会令人困惑。通常应该优先使用常量而不是静态项，除非以下情况之一为真：

* 存储大量数据。
* 需要静态项的单地址属性。
* 需要内部可变性。

[constant]: constant-items.md
[`drop`]: ../destructors.md
[constant expression]: ../const_eval.md#constant-expressions
[external block]: external-blocks.md
[interior mutable]: ../interior-mutability.md
[value namespace]: ../names/namespaces.md
[promoteds]: ../destructors.md#constant-promotion
