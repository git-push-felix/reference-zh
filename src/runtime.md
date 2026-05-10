r[runtime]
# Rust 运行时

本节记录了定义 Rust 运行时某些方面的特性。

<!-- template:attributes -->
r[runtime.global_allocator]
## `global_allocator` 属性

r[runtime.global_allocator.intro]
*`global_allocator` [属性][attributes]* 选择一个[内存分配器][std::alloc]。

> [!EXAMPLE]
> ```rust
> use core::alloc::{GlobalAlloc, Layout};
> use std::alloc::System;
>
> struct MyAllocator;
>
> unsafe impl GlobalAlloc for MyAllocator {
>     unsafe fn alloc(&self, layout: Layout) -> *mut u8 {
>         unsafe { System.alloc(layout) }
>     }
>     unsafe fn dealloc(&self, ptr: *mut u8, layout: Layout) {
>         unsafe { System.dealloc(ptr, layout) }
>     }
> }
>
> #[global_allocator]
> static GLOBAL: MyAllocator = MyAllocator;
> ```

r[runtime.global_allocator.syntax]
`global_allocator` 属性使用 [MetaWord] 语法。

r[runtime.global_allocator.allowed-positions]
`global_allocator` 属性只能应用于类型实现了 [`GlobalAlloc`] trait 的[静态项][static item]。

r[runtime.global_allocator.duplicates]
`global_allocator` 属性在一个项上只能使用一次。

r[runtime.global_allocator.single]
`global_allocator` 属性在 crate 图中只能使用一次。

r[runtime.global_allocator.stdlib]
`global_allocator` 属性从[标准库预导入][core::prelude::v1]中导出。

<!-- template:attributes -->
r[runtime.windows_subsystem]
## `windows_subsystem` 属性

r[runtime.windows_subsystem.intro]
*`windows_subsystem` [属性][attributes]* 在 Windows 目标上链接时设置[子系统][subsystem]。

> [!EXAMPLE]
> ```rust
> #![windows_subsystem = "windows"]
> ```

r[runtime.windows_subsystem.syntax]
`windows_subsystem` 属性使用 [MetaNameValueStr] 语法。可接受的值为 `"console"` 和 `"windows"`。

r[runtime.windows_subsystem.allowed-positions]
`windows_subsystem` 属性只能应用于 crate 根。

r[runtime.windows_subsystem.duplicates]
只有第一次使用 `windows_subsystem` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

r[runtime.windows_subsystem.ignored]
`windows_subsystem` 属性在非 Windows 目标和非 `bin` [crate 类型][crate types]上被忽略。

r[runtime.windows_subsystem.console]
`"console"` 子系统是默认的。如果控制台进程从现有控制台运行，它将附加到该控制台；否则将创建一个新的控制台窗口。

r[runtime.windows_subsystem.windows]
`"windows"` 子系统将与任何现有控制台分离运行。

> [!NOTE]
> `"windows"` 子系统通常由不想在启动时显示控制台窗口的 GUI 应用程序使用。

[`GlobalAlloc`]: alloc::alloc::GlobalAlloc
[crate types]: linkage.md
[static item]: items/static-items.md
[subsystem]: https://msdn.microsoft.com/en-us/library/fcc1zstk.aspx
