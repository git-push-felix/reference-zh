r[link]
# 链接

> [!NOTE]
> 本节更多从编译器角度描述，而非语言角度。

r[link.intro]
编译器支持多种将 crate 静态和动态链接在一起的方法。本节将探讨将 crate 链接在一起的各种方法，有关原生库的更多信息见[本书的 FFI 部分][ffi]。

[ffi]: ../book/ch20-01-unsafe-rust.html#using-extern-functions-to-call-external-code

r[link.type]
在一次编译会话中，编译器可以通过使用命令行标志或 `crate_type` 属性生成多个产物。如果指定了一个或多个命令行标志，所有 `crate_type` 属性将被忽略，仅构建命令行指定的产物。

r[link.bin]
* `--crate-type=bin`、`#![crate_type = "bin"]` - 将生成一个可运行的可执行文件。这要求 crate 中有一个 `main` 函数，当程序开始执行时将运行该函数。这将链接所有 Rust 和原生依赖，生成一个可分发二进制文件。这是默认的 crate 类型。

r[link.lib]
* `--crate-type=lib`、`#![crate_type = "lib"]` - 将生成一个 Rust 库。这是一个模糊的概念，因为库可以表现为几种形式。此通用 `lib` 选项的目的是生成"编译器推荐"风格的库。输出库始终可被 rustc 使用，但实际的库类型可能随时间变化。其余的产物类型都是不同风格的库，`lib` 类型可以看作是其中之一的别名（但具体是哪一个由编译器定义）。

r[link.dylib]
* `--crate-type=dylib`、`#![crate_type = "dylib"]` - 将生成一个动态 Rust 库。这与 `lib` 产物类型不同，因为它强制生成动态库。生成的动态库可以用作其他库和/或可执行文件的依赖。此产物类型将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

r[link.staticlib]
* `--crate-type=staticlib`、`#![crate_type = "staticlib"]` - 将生成一个静态系统库。这与其他库产物不同，因为编译器不会尝试链接到 `staticlib` 产物。此产物类型的目的是创建一个包含所有本地 crate 代码以及所有上游依赖的静态库。此产物类型将在 Linux、macOS 和 Windows (MinGW) 上创建 `*.a` 文件，在 Windows (MSVC) 上创建 `*.lib` 文件。此格式推荐用于将 Rust 代码链接到现有非 Rust 应用程序的情况，因为它不会对其他 Rust 代码有动态依赖。

  请注意，当从某处链接该静态库时，静态库可能具有的任何动态依赖（如对系统库的依赖，或对编译为动态库的 Rust 库的依赖）必须手动指定。`--print=native-static-libs` 标志可能对此有帮助。

  请注意，由于生成的静态库包含所有依赖的代码，包括标准库，并导出它们的所有公共符号，将静态库链接到可执行文件或共享库可能需要特别小心。对于共享库，导出的符号列表必须通过例如链接器或符号版本脚本、导出符号列表（macOS）或模块定义文件（Windows）来限制。此外，可以移除未使用的段以移除所有实际未使用的依赖代码（例如 mac 上的 `--gc-sections` 或 `-dead_strip`）。

r[link.cdylib]
* `--crate-type=cdylib`、`#![crate_type = "cdylib"]` - 将生成一个动态系统库。这用于编译要从其他语言加载的动态库。此产物类型将在 Linux 上创建 `*.so` 文件，在 macOS 上创建 `*.dylib` 文件，在 Windows 上创建 `*.dll` 文件。

r[link.rlib]
* `--crate-type=rlib`、`#![crate_type = "rlib"]` - 将生成一个"Rust 库"文件。这用作中间产物，可以认为是"静态 Rust 库"。与 `staticlib` 文件不同，这些 `rlib` 文件在未来的链接中由编译器解释。这本质上意味着 `rustc` 将在 `rlib` 文件中查找元数据，就像在动态库中查找元数据一样。此形式的输出用于生成静态链接的可执行文件以及 `staticlib` 产物。

r[link.proc-macro]
* `--crate-type=proc-macro`、`#![crate_type = "proc-macro"]` - 产生的输出未指定，但如果提供了 `-L` 路径，编译器会将输出产物识别为宏，并且可以加载给程序使用。使用此 crate 类型编译的 crate 必须仅导出[过程宏][procedural macros]。编译器将自动设置 `proc_macro` [配置选项][configuration option]。这些 crate 始终以编译器本身构建的目标进行编译。例如，如果你在 Linux 上使用 `x86_64` CPU 执行编译器，即使该 crate 是另一个为不同目标构建的 crate 的依赖，目标也将是 `x86_64-unknown-linux-gnu`。

r[link.repetition]
请注意，这些产出是可叠加的，如果指定了多个，编译器将生成每种形式的输出而无需重新编译。然而，这仅适用于通过相同方法指定的产出。如果只指定了 `crate_type` 属性，则它们都将被构建，但如果指定了一个或多个 `--crate-type` 命令行标志，则只会构建那些输出。

r[link.dependency]
对于所有这些不同类型的输出，如果 crate A 依赖 crate B，编译器可以在整个系统中以各种形式找到 B。然而，编译器查找的形式只有 `rlib` 格式和动态库格式。对于依赖库的这两种选项，编译器必须在某个时候在这两种格式之间做出选择。鉴于此，编译器在确定将使用什么格式的依赖时遵循以下规则：

r[link.dependency-staticlib]
1. 如果正在生成静态库，所有上游依赖必须以 `rlib` 格式可用。此要求源于动态库不能转换为静态格式的原因。

   请注意，不可能将原生动态依赖链接到静态库中，在这种情况下将打印关于所有未链接的原生动态依赖的警告。

r[link.dependency-rlib]
2. 如果正在生成 `rlib` 文件，则对上游依赖可用的格式没有限制。仅要求所有上游依赖可供读取元数据。

   原因在于 `rlib` 文件不包含其任何上游依赖。如果所有 `rlib` 文件都包含 `libstd.rlib` 的副本，那将非常低效！

r[link.dependency-prefer-dynamic]
3. 如果正在生成可执行文件且未指定 `-C prefer-dynamic` 标志，则首先尝试以 `rlib` 格式查找依赖。如果某些依赖在 rlib 格式中不可用，则尝试动态链接（见下文）。

r[link.dependency-dynamic]
4. 如果正在生成动态库或正在动态链接的可执行文件，编译器将尝试协调 rlib 或 dylib 格式的可用依赖以创建最终产物。

   编译器的一个主要目标是确保库在任何产物中出现不超过一次。例如，如果动态库 B 和 C 各自静态链接到库 A，则 crate 无法同时链接到 B 和 C，因为会有两个 A 的副本。编译器允许混合 rlib 和 dylib 格式，但必须满足此限制。

   编译器目前没有实现提示库应以什么格式链接的方法。在动态链接时，编译器将尝试最大化动态依赖，同时仍允许通过 rlib 链接某些依赖。

   对于大多数情况，如果动态链接，推荐将所有库作为 dylib 可用。对于其他情况，如果编译器无法确定每个库使用哪种格式链接，它将发出警告。

总的来说，`--crate-type=bin` 或 `--crate-type=lib` 应该足以满足所有编译需求，其他选项仅在需要更精细地控制 crate 的输出格式时才可用。

r[link.crt]
## 静态和动态 C 运行时 {#static-and-dynamic-c-runtimes}

r[link.crt.intro]
标准库通常努力为目标平台适当支持静态链接和动态链接的 C 运行时。例如，`x86_64-pc-windows-msvc` 和 `x86_64-unknown-linux-musl` 目标通常附带两种运行时，用户可以选择他们喜欢的运行时。编译器中的所有目标都有链接到 C 运行时的默认模式。通常目标是默认动态链接的，但有一些例外默认是静态链接的，例如：

* `arm-unknown-linux-musleabi`
* `arm-unknown-linux-musleabihf`
* `armv7-unknown-linux-musleabihf`
* `i686-unknown-linux-musl`
* `x86_64-unknown-linux-musl`

r[link.crt.crt-static]
C 运行时的链接配置为遵循 `crt-static` 目标特性。这些目标特性通常通过编译器本身的命令行标志进行配置。例如，要启用静态运行时，你可以执行：

```sh
rustc -C target-feature=+crt-static foo.rs
```

而要动态链接到 C 运行时，你可以执行：

```sh
rustc -C target-feature=-crt-static foo.rs
```

r[link.crt.ineffective]
不支持在 C 运行时链接方式之间切换的目标将忽略此标志。建议在编译器成功编译后检查生成的二进制文件，确保其按预期链接。

r[link.crt.target_feature]
Crate 也可以了解 C 运行时的链接方式。例如，MSVC 上的代码需要根据链接的运行时以不同方式编译（例如，使用 `/MT` 或 `/MD`）。这目前通过 [`cfg` 属性 `target_feature` 选项][`cfg` attribute `target_feature` option]导出：

```rust
#[cfg(target_feature = "crt-static")]
fn foo() {
    println!("the C runtime should be statically linked");
}

#[cfg(not(target_feature = "crt-static"))]
fn foo() {
    println!("the C runtime should be dynamically linked");
}
```

另请注意，Cargo 构建脚本可以通过[环境变量][cargo]了解此特性。在构建脚本中，你可以通过以下方式检测链接方式：

```rust
use std::env;

fn main() {
    let linkage = env::var("CARGO_CFG_TARGET_FEATURE").unwrap_or(String::new());

    if linkage.contains("crt-static") {
        println!("the C runtime will be statically linked");
    } else {
        println!("the C runtime will be dynamically linked");
    }
}
```

要本地使用此特性，你通常会使用 `RUSTFLAGS` 环境变量通过 Cargo 指定编译器标志。例如要在 MSVC 上编译静态链接的二进制文件，你可以执行：

```sh
RUSTFLAGS='-C target-feature=+crt-static' cargo build --target x86_64-pc-windows-msvc
```

r[link.foreign-code]
## 混合 Rust 和外部代码库

r[link.foreign-code.foreign-linkers]
如果你正在将 Rust 与外部代码（如 C、C++）混合使用，并希望创建包含两种类型代码的单个二进制文件，你有两种方法进行最终的二进制链接：

* 使用 `rustc`。使用 `-L <directory>` 和 `-l<library>` rustc 参数以及/或 Rust 代码中的 `#[link]` 指令传递任何非 Rust 库。如果需要链接 `.o` 文件，你可以使用 `-Clink-arg=file.o`。
* 使用外部链接器。在这种情况下，你首先需要生成一个 Rust `staticlib` 目标并将其传递给你的外部链接器调用。如果你需要链接多个 Rust 子系统，你需要生成一个*单一*的 `staticlib`，可能使用大量 `extern crate` 语句来包含多个 Rust `rlib`。多个 Rust `staticlib` 文件可能会冲突。

目前不支持直接将 `rlib` 传递给你的外部链接器。

> [!NOTE]
> 使用 Rust 运行时的不同实例编译或链接的 Rust 代码在本节的目的下被视为"外部代码"。

r[link.unwinding]
### 禁止的链接和展开

r[link.unwinding.intro]
只有在二进制文件根据以下规则一致构建时，才能使用 panic 展开。

r[link.unwinding.potential]
如果满足以下任一条件，则 Rust 产物被称为*可能展开的*：
- 该产物使用 [`unwind` panic 处理器][panic.panic_handler]。
- 该产物包含以 `unwind` [panic 策略][panic strategy]构建的 crate，该 crate 调用了一个使用 `-unwind` ABI 的函数。
- 该产物发起 `"Rust"` ABI 调用到运行在另一个 Rust 产物中的代码，该产物具有 Rust 运行时的单独副本，并且该另一个产物是可能展开的。

> [!NOTE]
> 此定义描述了 Rust 产物中的 `"Rust"` ABI 调用是否可能展开。

r[link.unwinding.prohibited]
如果 Rust 产物是可能展开的，则其所有 crate 必须使用 `unwind` [panic 策略][panic strategy]构建。否则，展开可能导致未定义行为。

> [!NOTE]
> 如果你使用 `rustc` 链接，这些规则会自动强制执行。如果你*不*使用 `rustc` 链接，你必须注意确保整个二进制文件中的展开处理一致。不使用 `rustc` 链接包括使用 `dlopen` 或类似设施的情况，其中链接由系统运行时完成，而不涉及 `rustc`。这仅在混合使用不同 [`-C panic`] 标志的代码时才会发生，因此大多数用户不必关注这一点。

> [!NOTE]
> 为了保证库在链接时无论使用何种 panic 运行时都将是健全的（并且可与 `rustc` 链接），可以使用 [`ffi_unwind_calls` lint]。该 lint 标记对 `-unwind` 外部函数或函数指针的任何调用。

[`cfg` attribute `target_feature` option]: conditional-compilation.md#target_feature
[`ffi_unwind_calls` lint]: ../rustc/lints/listing/allowed-by-default.html#ffi-unwind-calls
[configuration option]: conditional-compilation.md
[procedural macros]: procedural-macros.md
[panic strategy]: panic.md#panic-strategy
[`-C panic`]: ../rustc/codegen-options/index.html#panic
[cargo]: ../cargo/reference/environment-variables.html#environment-variables-cargo-sets-for-build-scripts
