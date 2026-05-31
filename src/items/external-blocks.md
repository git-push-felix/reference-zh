r[items.extern]
# 外部块

r[items.extern.syntax]
```grammar,items
ExternBlock ->
    `unsafe`?[^unsafe-2024] `extern` Abi? `{`
        InnerAttribute*
        ExternalItem*
    `}`

ExternalItem ->
    OuterAttribute* (
        MacroInvocationSemi
      | Visibility? StaticItem
      | Visibility? Function
    )
```

[^unsafe-2024]: 从 2024 版本开始，`unsafe` 关键字在语义上是必需的。

r[items.extern.intro]
外部块提供不在当前 crate 中*定义*的程序项的*声明*，是 Rust 外部函数接口的基础。这类似于未检查的导入。

r[items.extern.allowed-kinds]
外部块中允许两种程序项*声明*：[函数][functions]和[静态项][statics]。

r[items.extern.safety]
调用在外部块中声明的 unsafe 函数或访问 unsafe 静态项只允许在 [`unsafe` 上下文][`unsafe` context]中进行。

r[items.extern.namespace]
外部块在其所在模块或块的[值命名空间][value namespace]中定义其函数和静态项。

r[items.extern.unsafe-required]
在语义上，`unsafe` 关键字必须出现在外部块的 `extern` 关键字之前。

r[items.extern.edition2024]
> [!EDITION-2024]
> 在 2024 版本之前，`unsafe` 关键字是可选的。只有当外部块本身被标记为 `unsafe` 时，才允许使用 `safe` 和 `unsafe` 程序项限定符。

r[items.extern.fn]
## 函数 {#functions}

r[items.extern.fn.body]
外部块中的函数声明方式与其他 Rust 函数相同，不同之处在于它们不能有主体，而是以分号结束。

r[items.extern.fn.param-patterns]
参数中不允许使用模式，只能使用 [IDENTIFIER] 或 `_`。

r[items.extern.fn.qualifiers]
允许使用 `safe` 和 `unsafe` 函数限定符，但不允许使用其他函数限定符（例如 `const`、`async`、`extern`）。

r[items.extern.fn.foreign-abi]
外部块中的函数可以被 Rust 代码调用，就像在 Rust 中定义的函数一样。Rust 编译器会自动在 Rust ABI 和外部 ABI 之间进行翻译。

r[items.extern.fn.safety]
在 extern 块中声明的函数隐式是 `unsafe` 的，除非存在 `safe` 函数限定符。

r[items.extern.fn.fn-ptr]
当强制转换为函数指针时，extern 块中声明的函数具有类型 `for<'l1, ..., 'lm> extern "abi" fn(A1, ..., An) -> R`，其中 `'l1`、...`'lm` 是其生命周期参数，`A1`、...、`An` 是其参数的声明类型，`R` 是声明的返回类型。

r[items.extern.static]
## 静态项 {#statics}

r[items.extern.static.intro]
外部块中的静态项声明方式与外部块外的[静态项][statics]相同，不同之处在于没有初始化其值的表达式。

r[items.extern.static.safety]
除非在 extern 块中声明的静态项被限定为 `safe`，否则访问该程序项是 `unsafe` 的，无论其是否可变，因为无法保证该静态项内存中的位模式对于其声明的类型是有效的，因为某个任意的（例如 C）代码负责初始化该静态项。

r[items.extern.static.mut]
extern 静态项可以像外部块外的[静态项][statics]一样是不可变的或可变的。

r[items.extern.static.read-only]
不可变静态项*必须*在任何 Rust 代码执行之前被初始化。仅在该静态项在 Rust 代码读取它之前被初始化是不够的。一旦 Rust 代码运行，修改不可变静态项（从 Rust 内部或外部）是 UB，除非修改发生在 `UnsafeCell` 内部的字节上。

r[items.extern.abi]
## ABI {#abi}

r[items.extern.abi.intro]
`extern` 关键字后可以跟一个可选的 [ABI] 字符串。ABI 指定块中函数的调用约定。调用约定定义了函数的低级接口，例如参数如何放置在寄存器或栈上、返回值如何传递以及谁负责清理栈。

> [!EXAMPLE]
> ```rust
> // Windows API 的接口。
> unsafe extern "system" { /* ... */ }
> ```

r[items.extern.abi.default]
如果未指定 ABI 字符串，则默认为 `"C"`。

> [!NOTE]
> 不带显式 ABI 的 `extern` 语法正被逐步淘汰，因此最好始终显式地写出 ABI。
>
> 详情请参见 [Rust issue #134986](https://github.com/rust-lang/rust/issues/134986)。

r[items.extern.abi.standard]
以下 ABI 字符串在所有平台上都受支持：

r[items.extern.abi.rust]
* `unsafe extern "Rust"` --- Rust 函数和闭包的本机调用约定。这是声明函数而不使用 [`extern fn`] 时的默认值。Rust ABI 不提供稳定性保证。

r[items.extern.abi.c]
* `unsafe extern "C"` --- "C" ABI 与目标平台的占主导地位的 C 编译器选择的默认 ABI 相匹配。

r[items.extern.abi.system]
* `unsafe extern "system"` --- 这等价于 `extern "C"`，但在 Windows x86_32 上等价于非可变参数函数的 `"stdcall"`，对于可变参数函数等价于 `"C"`。

  > [!NOTE]
  > 由于 Windows 上正确的底层 ABI 是目标平台特定的，因此在尝试链接不使用显式定义 ABI 的 Windows API 函数时，最好使用 `extern "system"`。

r[items.extern.abi.unwind]
* `extern "C-unwind"` 和 `extern "system-unwind"` --- 分别与 `"C"` 和 `"system"` 相同，但在被调用方展开（通过 panic 或抛出 C++ 风格的异常）时具有[不同的行为][unwind-behavior]。

r[items.extern.abi.platform]
还有一些特定于平台的 ABI 字符串：

r[items.extern.abi.cdecl]
* `unsafe extern "cdecl"` --- 通常与 x86_32 C 代码一起使用的调用约定。
  * 仅在 x86_32 目标上可用。
  * 对应于 MSVC 的 `__cdecl` 以及 GCC 和 clang 的 `__attribute__((cdecl))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/cdecl>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#cdecl>

r[items.extern.abi.stdcall]
* `unsafe extern "stdcall"` --- x86_32 上的 [Win32 API] 通常使用的调用约定。
  * 仅在 x86_32 目标上可用。
  * 对应于 MSVC 的 `__stdcall` 以及 GCC 和 clang 的 `__attribute__((stdcall))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/stdcall>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#stdcall>

r[items.extern.abi.win64]
* `unsafe extern "win64"` --- Windows x64 ABI。
  * 仅在 x86_64 目标上可用。
  * "win64" 与 Windows x86_64 目标上的 "C" ABI 相同。
  * 对应于 GCC 和 clang 的 `__attribute__((ms_abi))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://learn.microsoft.com/en-us/cpp/build/x64-software-conventions>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#Microsoft_x64_calling_convention>

r[items.extern.abi.sysv64]
* `unsafe extern "sysv64"` --- System V ABI。
  * 仅在 x86_64 目标上可用。
  * "sysv64" 与非 Windows x86_64 目标上的 "C" ABI 相同。
  * 对应于 GCC 和 clang 的 `__attribute__((sysv_abi))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://wiki.osdev.org/System_V_ABI>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#System_V_AMD64_ABI>

r[items.extern.abi.aapcs]
* `unsafe extern "aapcs"` --- ARM 的 soft-float ABI。
  * 仅在 ARM32 目标上可用。
  * "aapcs" 与 soft-float ARM32 上的 "C" ABI 相同。
  * 对应于 clang 的 `__attribute__((pcs("aapcs")))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - [Arm Procedure Call Standard](https://developer.arm.com/documentation/107656/0101/Getting-started-with-Armv8-M-based-systems/Procedure-Call-Standard-for-Arm-Architecture--AAPCS-)

r[items.extern.abi.fastcall]
* `unsafe extern "fastcall"` --- stdcall 的一种"快速"变体，通过寄存器传递某些参数。
  * 仅在 x86_32 目标上可用。
  * 对应于 MSVC 的 `__fastcall` 以及 GCC 和 clang 的 `__attribute__((fastcall))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://learn.microsoft.com/en-us/cpp/cpp/fastcall>
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#Microsoft_fastcall>

r[items.extern.abi.thiscall]
* `unsafe extern "thiscall"` --- x86_32 MSVC 上 C++ 类成员函数通常使用的调用约定。
  * 仅在 x86_32 目标上可用。
  * 对应于 MSVC 的 `__thiscall` 以及 GCC 和 clang 的 `__attribute__((thiscall))`。

  > [!NOTE]
  > 详情请参见：
  >
  > - <https://en.wikipedia.org/wiki/X86_calling_conventions#thiscall>
  > - <https://learn.microsoft.com/en-us/cpp/cpp/thiscall>

r[items.extern.abi.efiapi]
* `unsafe extern "efiapi"` --- 用于 [UEFI] 函数的 ABI。
  * 仅在 x86 和 ARM 目标（32 位和 64 位）上可用。

r[items.extern.abi.platform-unwind-variants]
与 `"C"` 和 `"system"` 一样，大多数特定于平台的 ABI 字符串也有[相应的 `-unwind` 变体][unwind-behavior]；具体来说，这些是：

* `"aapcs-unwind"`
* `"cdecl-unwind"`
* `"fastcall-unwind"`
* `"stdcall-unwind"`
* `"sysv64-unwind"`
* `"thiscall-unwind"`
* `"win64-unwind"`

r[items.extern.variadic]
## 可变参数函数 {#variadic-functions}

外部块中的函数可以通过将 `...` 指定为最后一个参数来成为可变参数函数。可变参数可以有选择地指定一个标识符。

```rust
unsafe extern "C" {
    unsafe fn foo(...);
    unsafe fn bar(x: i32, ...);
    unsafe fn with_name(format: *const u8, args: ...);
    // 安全性：此函数保证不会访问
    // 可变参数。
    safe fn ignores_variadic_arguments(x: i32, ...);
}
```

> [!WARNING]
> 不应在 `extern` 块中的函数上使用 `safe` 限定符，除非该函数保证完全不会访问可变参数。向可变参数函数传递意外数量的参数或意外类型的参数可能导致[未定义行为][undefined]。

r[items.extern.variadic.conventions]
可变参数只能在具有以下 ABI 字符串或其相应 [`-unwind` 变体][items.fn.extern.unwind]的 `extern` 块中指定：

- `"aapcs"`
- `"C"`
- `"cdecl"`
- `"efiapi"`
- `"system"`
- `"sysv64"`
- `"win64"`

r[items.extern.attributes]
## 外部块上的属性 {#attributes-on-extern-blocks}

r[items.extern.attributes.intro]
以下[属性][attributes]控制外部块的行为。

r[items.extern.attributes.link]
### `link` 属性 {#the-link-attribute}

r[items.extern.attributes.link.intro]
*`link` 属性*指定编译器应为 `extern` 块中的程序项与之链接的本地库的名称。

r[items.extern.attributes.link.syntax]
它使用 [MetaListNameValueStr] 语法来指定其输入。`name` 键是要链接的本地库的名称。`kind` 键是一个可选值，用于指定库的种类，可能的值如下：

r[items.extern.attributes.link.dylib]
- `dylib` --- 表示动态库。如果未指定 `kind`，这是默认值。

r[items.extern.attributes.link.static]
- `static` --- 表示静态库。

r[items.extern.attributes.link.framework]
- `framework` --- 表示 macOS 框架。仅对 macOS 目标有效。

r[items.extern.attributes.link.raw-dylib]
- `raw-dylib` --- 表示动态库，编译器将生成导入库进行链接（详见下文的 [`dylib` 与 `raw-dylib`][`dylib` versus `raw-dylib`]）。仅对 Windows 目标有效。

r[items.extern.attributes.link.name-requirement]
如果指定了 `kind`，则必须包含 `name` 键。

r[items.extern.attributes.link.modifiers]
可选的 `modifiers` 参数提供了一种为要链接的库指定链接修饰符的方式。

r[items.extern.attributes.link.modifiers.syntax]
修饰符以逗号分隔的字符串形式指定，每个修饰符以 `+` 或 `-` 为前缀，分别表示该修饰符是启用还是禁用。

r[items.extern.attributes.link.modifiers.multiple]
目前不支持在单个 `link` 属性中指定多个 `modifiers` 参数，或在同一个 `modifiers` 参数中指定多个相同的修饰符。示例：`#[link(name = "mylib", kind = "static", modifiers = "+whole-archive")]`。

r[items.extern.attributes.link.wasm_import_module]
`wasm_import_module` 键可用于在从宿主环境导入符号时指定 `extern` 块中程序项的 [WebAssembly 模块][WebAssembly module]名称。如果未指定 `wasm_import_module`，则默认模块名为 `env`。

<!-- ignore: requires extern linking -->
```rust,ignore
#[link(name = "crypto")]
unsafe extern {
    // …
}

#[link(name = "CoreFoundation", kind = "framework")]
unsafe extern {
    // …
}

#[link(wasm_import_module = "foo")]
unsafe extern {
    // …
}
```

r[items.extern.attributes.link.empty-block]
在空外部块上添加 `link` 属性是有效的。你可以使用这种方式来满足代码中其他地方（包括上游 crate）的外部块的链接要求，而不是在每个外部块上都添加属性。

r[items.extern.attributes.link.modifiers.bundle]
#### 链接修饰符：`bundle` {#linking-modifiers-bundle}

r[items.extern.attributes.link.modifiers.bundle.allowed-kinds]
此修饰符仅与 `static` 链接种类兼容。使用任何其他种类将导致编译器错误。

r[items.extern.attributes.link.modifiers.bundle.behavior]
在构建 rlib 或 staticlib 时，`+bundle` 表示本地静态库将被打包到 rlib 或 staticlib 归档中，然后在最终二进制文件的链接过程中从那里取出。

r[items.extern.attributes.link.modifiers.bundle.behavior-negative]
在构建 rlib 时，`-bundle` 表示本地静态库按名称注册为该 rlib 的依赖项，其中的目标文件仅在最终二进制文件的链接过程中被包含，按名称文件搜索也在最终链接时执行。在构建 staticlib 时，`-bundle` 表示本地静态库根本不包含在归档中，需要某个更高级的构建系统在最终二进制文件的链接过程中稍后添加它。

r[items.extern.attributes.link.modifiers.bundle.no-effect]
此修饰符在构建其他目标（如可执行文件或动态库）时无效。

r[items.extern.attributes.link.modifiers.bundle.default]
此修饰符的默认值为 `+bundle`。

关于此修饰符的更多实现细节可以在 [`bundle` 的 rustc 文档][`bundle` documentation for rustc]中找到。

r[items.extern.attributes.link.modifiers.whole-archive]
#### 链接修饰符：`whole-archive` {#linking-modifiers-whole-archive}

r[items.extern.attributes.link.modifiers.whole-archive.allowed-kinds]
此修饰符仅与 `static` 链接种类兼容。使用任何其他种类将导致编译器错误。

r[items.extern.attributes.link.modifiers.whole-archive.behavior]
`+whole-archive` 表示将静态库作为完整归档链接，不丢弃任何目标文件。

r[items.extern.attributes.link.modifiers.whole-archive.default]
此修饰符的默认值为 `-whole-archive`。

关于此修饰符的更多实现细节可以在 [`whole-archive` 的 rustc 文档][`whole-archive` documentation for rustc]中找到。

r[items.extern.attributes.link.modifiers.verbatim]
### 链接修饰符：`verbatim` {#linking-modifiers-verbatim}

r[items.extern.attributes.link.modifiers.verbatim.allowed-kinds]
此修饰符与所有链接种类兼容。

r[items.extern.attributes.link.modifiers.verbatim.behavior]
`+verbatim` 表示 rustc 本身不会向库名称添加任何目标平台指定的库前缀或后缀（如 `lib` 或 `.a`），并会尽量要求链接器也这样做。

r[items.extern.attributes.link.modifiers.verbatim.behavior-negative]
`-verbatim` 表示 rustc 在将库名称传递给链接器之前会添加目标平台特定的前缀和后缀，或者不会阻止链接器隐式添加它们。

r[items.extern.attributes.link.modifiers.verbatim.default]
此修饰符的默认值为 `-verbatim`。

关于此修饰符的更多实现细节可以在 [`verbatim` 的 rustc 文档][`verbatim` documentation for rustc]中找到。

r[items.extern.attributes.link.kind-raw-dylib]
#### `dylib` 与 `raw-dylib` {#dylib-versus-raw-dylib}

r[items.extern.attributes.link.kind-raw-dylib.intro]
在 Windows 上，链接动态库需要向链接器提供一个导入库：这是一个特殊的静态库，它声明了动态库导出的所有符号，以使得链接器知道它们必须在运行时动态加载。

r[items.extern.attributes.link.kind-raw-dylib.import]
指定 `kind = "dylib"` 指示 Rust 编译器根据 `name` 键链接一个导入库。然后链接器将使用其正常的库解析逻辑来查找该导入库。或者，指定 `kind = "raw-dylib"` 指示编译器在编译期间生成一个导入库并将其提供给链接器。

r[items.extern.attributes.link.kind-raw-dylib.platform-specific]
`raw-dylib` 仅在 Windows 上受支持。在面向其他平台时使用它将导致编译器错误。

r[items.extern.attributes.link.import_name_type]
#### `import_name_type` 键 {#the-import_name_type-key}

r[items.extern.attributes.link.import_name_type.intro]
在 x86 Windows 上，函数名称会被"修饰"（即添加特定的前缀和/或后缀）以指示其调用约定。例如，一个名为 `fn1` 的 `stdcall` 调用约定函数（没有参数）会被修饰为 `_fn1@0`。然而，[PE 格式][PE Format]也允许名称不带前缀或不被修饰。此外，MSVC 和 GNU 工具链对相同的调用约定使用不同的修饰，这意味着默认情况下，某些 Win32 函数无法通过 GNU 工具链使用 `raw-dylib` 链接种类进行调用。

r[items.extern.attributes.link.import_name_type.values]
为了应对这些差异，在使用 `raw-dylib` 链接种类时，你还可以指定 `import_name_type` 键，其值可以是以下之一，用于更改生成的导入库中函数的命名方式：

* `decorated`：函数名将使用 MSVC 工具链格式进行完全修饰。
* `noprefix`：函数名将使用 MSVC 工具链格式进行修饰，但跳过前导的 `?`、`@` 或可选的 `_`。
* `undecorated`：函数名将不被修饰。

r[items.extern.attributes.link.import_name_type.default]
如果未指定 `import_name_type` 键，则函数名将使用目标工具链的格式进行完全修饰。

r[items.extern.attributes.link.import_name_type.variables]
变量永远不会被修饰，因此 `import_name_type` 键对它们在生成的导入库中的命名方式没有影响。

r[items.extern.attributes.link.import_name_type.platform-specific]
`import_name_type` 键仅在 x86 Windows 上受支持。在面向其他平台时使用它将导致编译器错误。

<!-- template:attributes -->
r[items.extern.attributes.link_name]
### `link_name` 属性 {#the-link_name-attribute}

r[items.extern.attributes.link_name.intro]
*`link_name` [属性][attributes]* 可以应用于 `extern` 块中的声明，以指定为给定函数或静态项导入的符号。

> [!EXAMPLE]
> ```rust
> unsafe extern "C" {
>     #[link_name = "actual_symbol_name"]
>     safe fn name_in_rust();
> }
> ```

r[items.extern.attributes.link_name.syntax]
`link_name` 属性使用 [MetaNameValueStr] 语法。

r[items.extern.attributes.link_name.invalid-names]
符号名称不能是空字符串，也不能包含任何 `U+0000` (NUL) 字节。

r[items.extern.attributes.link_name.allowed-positions]
`link_name` 属性只能应用于 `extern` 块中的函数或静态项。

> [!NOTE]
> `rustc` 会忽略在其他位置的使用但会给出 lint 警告。这将来可能变成错误。

r[items.extern.attributes.link_name.duplicates]
只有程序项上首次使用的 `link_name` 才会生效。

> [!NOTE]
> `rustc` 会对首次之后的任何使用以未来兼容性警告的形式给出 lint 警告。这将来可能变成错误。

r[items.extern.attributes.link_name.link_ordinal]
`link_name` 属性不能与 [`link_ordinal`] 属性一起使用。

r[items.extern.attributes.link_ordinal]
### `link_ordinal` 属性 {#the-link_ordinal-attribute}

r[items.extern.attributes.link_ordinal.intro]
*`link_ordinal` 属性*可以应用于 `extern` 块中的声明，以指示在生成导入库进行链接时要使用的数字序号。序号是 Windows 上动态库中每个导出符号的唯一编号，可以在加载库时使用，以查找该符号，而不必按名称查找。

> [!WARNING]
> `link_ordinal` 应仅在已知符号的序号是稳定的情况下使用：如果符号的序号在构建其所在的二进制文件时未显式设置，则会自动分配一个序号，并且该分配的序号可能在二进制文件的构建之间发生变化。

```rust
# #[cfg(all(windows, target_arch = "x86"))]
#[link(name = "exporter", kind = "raw-dylib")]
unsafe extern "stdcall" {
    #[link_ordinal(15)]
    safe fn imported_function_stdcall(i: i32);
}
```

r[items.extern.attributes.link_ordinal.allowed-kinds]
此属性仅与 `raw-dylib` 链接种类一起使用。使用任何其他种类将导致编译器错误。

r[items.extern.attributes.link_ordinal.exclusive]
将此属性与 `link_name` 属性一起使用将导致编译器错误。

r[items.extern.attributes.fn-parameters]
### 函数参数上的属性 {#attributes-on-function-parameters}

extern 函数参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[ABI]: glossary.abi
[PE Format]: https://learn.microsoft.com/windows/win32/debug/pe-format#import-name-type
[UEFI]: https://uefi.org/specifications
[WebAssembly module]: https://webassembly.github.io/spec/core/syntax/modules.html
[`bundle` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-bundle
[`dylib` versus `raw-dylib`]: #dylib-versus-raw-dylib
[`extern fn`]: items.fn.extern
[`unsafe` context]: ../unsafe-keyword.md
[`verbatim` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-verbatim
[`whole-archive` documentation for rustc]: ../../rustc/command-line-arguments.html#linking-modifiers-whole-archive
[attributes]: ../attributes.md
[functions]: functions.md
[regular function parameters]: functions.md#attributes-on-function-parameters
[statics]: static-items.md
[unwind-behavior]: functions.md#unwinding
[value namespace]: ../names/namespaces.md
[win32 api]: https://learn.microsoft.com/en-us/windows/win32/api/
[`link_ordinal`]: items.extern.attributes.link_ordinal
