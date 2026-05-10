r[asm]
# 内联汇编

r[asm.intro]
内联汇编的支持通过 [`asm!`]、[`naked_asm!`] 和 [`global_asm!`] 宏提供。它可以用于在编译器生成的汇编输出中嵌入手写汇编。

[`asm!`]: core::arch::asm
[`naked_asm!`]: core::arch::naked_asm
[`global_asm!`]: core::arch::global_asm

r[asm.stable-targets]
内联汇编支持在以下架构上已稳定：
- x86 和 x86-64
- ARM
- AArch64 和 Arm64EC
- RISC-V
- LoongArch
- s390x
- PowerPC 和 PowerPC64

编译器将在不支持的平台上使用汇编宏时发出错误。

r[asm.example]
## 示例

```rust
# #[cfg(target_arch = "x86_64")] {
use std::arch::asm;

// 使用移位和加法将 x 乘以 6
let mut x: u64 = 4;
unsafe {
    asm!(
        "mov {tmp}, {x}",
        "shl {tmp}, 1",
        "shl {x}, 2",
        "add {x}, {tmp}",
        x = inout(reg) x,
        tmp = out(reg) _,
    );
}
assert_eq!(x, 4 * 6);
# }
```

r[asm.syntax]
## 语法

以下语法指定了可以传递给 `asm!`、`global_asm!` 和 `naked_asm!` 宏的参数。

```grammar,assembly
@root AsmArgs -> AsmAttrFormatString (`,` AsmAttrFormatString)* (`,` AsmAttrOperand)* `,`?

FormatString -> STRING_LITERAL | RAW_STRING_LITERAL | MacroInvocation

AsmAttrFormatString -> (OuterAttribute)* FormatString

AsmOperand ->
      ClobberAbi
    | AsmOptions
    | RegOperand

AsmAttrOperand -> (OuterAttribute)* AsmOperand

ClobberAbi -> `clobber_abi` `(` Abi (`,` Abi)* `,`? `)`

AsmOptions ->
    `options` `(` ( AsmOption (`,` AsmOption)* `,`? )? `)`

AsmOption ->
      `pure`
    | `nomem`
    | `readonly`
    | `preserves_flags`
    | `noreturn`
    | `nostack`
    | `att_syntax`
    | `raw`

RegOperand -> (ParamName `=`)?
    (
          DirSpec `(` RegSpec `)` Expression
        | DualDirSpec `(` RegSpec `)` DualDirSpecExpression
        | `sym` PathExpression
        | `const` Expression
        | `label` `{` Statements? `}`
    )

ParamName -> IDENTIFIER_OR_KEYWORD | RAW_IDENTIFIER

DualDirSpecExpression ->
      Expression
    | Expression `=>` Expression

RegSpec -> RegisterClass | ExplicitRegister

RegisterClass -> IDENTIFIER_OR_KEYWORD

ExplicitRegister -> STRING_LITERAL

DirSpec ->
      `in`
    | `out`
    | `lateout`

DualDirSpec ->
      `inout`
    | `inlateout`
```

r[asm.scope]
## 作用域

r[asm.scope.intro]
内联汇编可以以三种方式之一使用。

r[asm.scope.asm]
使用 `asm!` 宏，汇编代码在函数作用域中发出并集成到编译器生成的函数汇编代码中。此汇编代码必须遵守[严格的规则](#内联汇编的规则)以避免未定义行为。请注意，在某些情况下编译器可能会选择将汇编代码作为单独的函数发出并生成对其的调用。

```rust
# #[cfg(target_arch = "x86_64")] {
unsafe { core::arch::asm!("/* {} */", in(reg) 0); }
# }
```

r[asm.scope.naked_asm]
使用 `naked_asm!` 宏，汇编代码在函数作用域中发出并构成函数的完整汇编代码。`naked_asm!` 宏仅允许在[裸函数](attributes/codegen.md#the-naked-attribute)中使用。

```rust
# #[cfg(target_arch = "x86_64")] {
# #[unsafe(naked)]
# extern "C" fn wrapper() {
core::arch::naked_asm!("/* {} */", const 0);
# }
# }
```

r[asm.scope.global_asm]
使用 `global_asm!` 宏，汇编代码在全局作用域中发出，位于函数外部。这可用于使用汇编代码手写整个函数，并且通常提供更大的自由度来使用任意寄存器和汇编器指令。

```rust
# fn main() {}
# #[cfg(target_arch = "x86_64")]
core::arch::global_asm!("/* {} */", const 0);
```

r[asm.ts-args]
## 模板字符串参数

r[asm.ts-args.syntax]
汇编器模板使用与[格式字符串][format-syntax]相同的语法（即占位符由大括号指定）。

r[asm.ts-args.order]
相应的参数按顺序、按索引或按名称访问。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64;
let y: i64;
let z: i64;
// 这种方式
unsafe { core::arch::asm!("mov {}, {}", out(reg) x, in(reg) 5); }
// ...这种方式
unsafe { core::arch::asm!("mov {0}, {1}", out(reg) y, in(reg) 5); }
// ...和这种方式
unsafe { core::arch::asm!("mov {out}, {in}", out = out(reg) z, in = in(reg) 5); }
// 都具有相同的行为
assert_eq!(x, y);
assert_eq!(y, z);
# }
```

r[asm.ts-args.no-implicit]
但是，不支持隐式命名参数（由 [RFC #2795][rfc-2795] 引入）。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
let x = 5;
// 我们不能从作用域直接引用 `x`，我们需要一个像 `in(reg) x` 这样的操作数
unsafe { core::arch::asm!("/* {x} */"); } // 错误：没有名为 x 的参数
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.one-or-more]
一次 `asm!` 调用可以有一个或多个模板字符串参数；具有多个模板字符串参数的 `asm!` 被视作所有字符串之间用 `\n` 连接。预期用途是每个模板字符串参数对应一行汇编代码。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64;
let y: i64;
// 我们可以将多个字符串分开，就像它们写在一起一样
unsafe { core::arch::asm!("mov eax, 5", "mov ecx, eax", out("rax") x, out("rcx") y); }
assert_eq!(x, y);
# }
```

r[asm.ts-args.before-other-args]
所有模板字符串参数必须出现在任何其他参数之前。

```rust,compile_fail
let x = 5;
# #[cfg(target_arch = "x86_64")] {
// 模板字符串需要首先出现在 asm 调用中
unsafe { core::arch::asm!("/* {x} */", x = const 5, "ud2"); } // 错误：意外的 token
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.positional-first]
与格式字符串一样，位置参数必须出现在命名参数和显式[寄存器操作数](#寄存器操作数)之前。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 命名操作数需要放在位置操作数之后
unsafe { core::arch::asm!("/* {x} {} */", x = const 5, in(reg) 5); }
// 错误：位置参数不能跟在命名参数或显式寄存器参数之后
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们也不能将显式寄存器放在位置操作数之前
unsafe { core::arch::asm!("/* {} */", in("eax") 0, in(reg) 5); }
// 错误：位置参数不能跟在命名参数或显式寄存器参数之后
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.register-operands]
显式寄存器操作数不能被模板字符串中的占位符使用。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 显式寄存器操作数不会被替换，在字符串中显式使用 `eax`
unsafe { core::arch::asm!("/* {} */", in("eax") 5); }
// 错误：对索引 0 处的参数无效引用
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.at-least-once]
所有其他命名和位置操作数必须在模板字符串中至少出现一次，否则会生成编译器错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们必须在格式字符串中命名所有操作数
unsafe { core::arch::asm!("", in(reg) 5, x = const 5); }
// 错误：多个未使用的 asm 参数
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.opaque]
确切的汇编代码语法是平台特定的，对编译器是不透明的，除了操作数被替换到模板字符串中以形成传递给汇编器的代码的方式。

r[asm.ts-args.llvm-syntax]
目前，所有支持的目标都遵循 LLVM 内部汇编器使用的汇编代码语法，这通常对应于 GNU 汇编器 (GAS) 的语法。在 x86 上，默认使用 GAS 的 `.intel_syntax noprefix` 模式。在 ARM 上，使用 `.syntax unified` 模式。这些目标对汇编代码施加了一个额外的限制：任何汇编器状态（例如可以用 `.section` 更改的当前节）必须在 asm 字符串的末尾恢复到其原始值。不符合 GAS 语法的汇编代码将导致特定于汇编器的行为。内联汇编使用的指令的进一步约束由[指令支持](#指令支持)指示。

[format-syntax]: std::fmt#syntax
[rfc-2795]: https://github.com/rust-lang/rfcs/pull/2795

r[asm.attributes]
## 属性

r[asm.attributes.supported-attributes]
仅 [`cfg`] 和 [`cfg_attr`] 属性在内联汇编模板字符串和操作数上被语义接受。其他属性被解析但在汇编宏展开时被拒绝。

```rust
# fn main() {}
# #[cfg(target_arch = "x86_64")]
core::arch::global_asm!(
    #[cfg(not(panic = "abort"))]
    ".cfi_startproc",
    // ...
    "ret",
    #[cfg(not(panic = "abort"))]
    ".cfi_endproc",
);
```

> [!NOTE]
> 在 `rustc` 中，汇编宏实现处理这些属性的方式与处理语言中类似属性的常规系统分开。这解释了支持的属性类型的有限性，并可能导致行为的微妙差异。

r[asm.attributes.starts-with-template]
在语法上，在第一个操作数之前必须至少有一个模板字符串。

```rust,compile_fail
// 这被拒绝，因为 `a = out(reg) x` 不被解析为模板字符串。
core::arch::asm!(
    #[cfg(false)]
    a = out(reg) x, // 错误。
    "",
);
```

[`cfg`]: conditional-compilation.md#the-cfg-attribute
[`cfg_attr`]: conditional-compilation.md#the-cfg_attr-attribute

r[asm.operand-type]
## 操作数类型

r[asm.operand-type.supported-operands]
支持几种类型的操作数：

r[asm.operand-type.supported-operands.in]
* `in(<reg>) <expr>`
  - `<reg>` 可以引用寄存器类别或显式寄存器。分配的寄存器名称被替换到 asm 模板字符串中。
  - 分配的寄存器将在汇编代码开始时包含 `<expr>` 的值。
  - 分配的寄存器必须在汇编代码结束时包含相同的值（除非为同一寄存器分配了 `lateout`）。

```rust
# #[cfg(target_arch = "x86_64")] {
// ``in` 可用于将值传递到内联汇编中……
unsafe { core::arch::asm!("/* {} */", in(reg) 5); }
# }
```

> [!NOTE]
> 如果值的类型小于寄存器，高位比特的值是平台特定的。某些目标将高位清零，而其他目标则保持不变。

r[asm.operand-type.supported-operands.out]
* `out(<reg>) <expr>`
  - `<reg>` 可以引用寄存器类别或显式寄存器。分配的寄存器名称被替换到 asm 模板字符串中。
  - 分配的寄存器将在汇编代码开始时包含一个未定义的值。
  - `<expr>` 必须是一个（可能未初始化的）位置表达式，分配的寄存器的内容在汇编代码结束时被写入该表达式。
  - 可以指定下划线（`_`）代替表达式，这将导致寄存器的内容在汇编代码结束时被丢弃（有效地充当清除器）。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64;
// `out` 可用于将值传回给 Rust。
unsafe { core::arch::asm!("/* {} */", out(reg) x); }
# }
```

r[asm.operand-type.supported-operands.lateout]
* `lateout(<reg>) <expr>`
  - 与 `out` 相同，除了寄存器分配器可以重用分配给 `in` 的寄存器。
  - 你只应在所有输入都被读取后才写入寄存器，否则可能会清除一个输入。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64;
// `lateout` 与 `out` 相同
// 但编译器知道在覆盖时我们不关心任何输入的值。
unsafe { core::arch::asm!("mov {}, 5", lateout(reg) x); }
assert_eq!(x, 5)
# }
```

r[asm.operand-type.supported-operands.inout]
* `inout(<reg>) <expr>`
  - `<reg>` 可以引用寄存器类别或显式寄存器。分配的寄存器名称被替换到 asm 模板字符串中。
  - 分配的寄存器将在汇编代码开始时包含 `<expr>` 的值。
  - `<expr>` 必须是一个可变的已初始化位置表达式，分配的寄存器的内容在汇编代码结束时被写入该表达式。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut x: i64 = 4;
// `inout` 可用于在寄存器中修改值
unsafe { core::arch::asm!("inc {}", inout(reg) x); }
assert_eq!(x, 5);
# }
```

r[asm.operand-type.supported-operands.inout-arrow]
* `inout(<reg>) <in expr> => <out expr>`
  - 与 `inout` 相同，除了寄存器的初始值取自 `<in expr>` 的值。
  - `<out expr>` 必须是一个（可能未初始化的）位置表达式，分配的寄存器的内容在汇编代码结束时被写入该表达式。
  - 可以为 `<out expr>` 指定下划线（`_`）代替表达式，这将导致寄存器的内容在汇编代码结束时被丢弃（有效地充当清除器）。
  - `<in expr>` 和 `<out expr>` 可以具有不同的类型。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64;
// `inout` 也可以将值移动到不同的位置
unsafe { core::arch::asm!("inc {}", inout(reg) 4u64=>x); }
assert_eq!(x, 5);
# }
```

r[asm.operand-type.supported-operands.inlateout]
* `inlateout(<reg>) <expr>` / `inlateout(<reg>) <in expr> => <out expr>`
  - 与 `inout` 相同，除了寄存器分配器可以重用分配给 `in` 的寄存器（如果编译器知道 `in` 与 `inlateout` 具有相同的初始值，这可能会发生）。
  - 你只应在所有输入都被读取后才写入寄存器，否则可能会清除一个输入。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut x: i64 = 4;
// `inlateout` 是使用 `lateout` 语义的 `inout`
unsafe { core::arch::asm!("inc {}", inlateout(reg) x); }
assert_eq!(x, 5);
# }
```

r[asm.operand-type.supported-operands.sym]
* `sym <path>`
  - `<path>` 是一个指向函数或静态变量的路径。
  - 将使用等价于 `path` 地址的 32 位或 64 位整数值替换 `<path>` 的（混淆过的）符号名称。
  - 替换的值包括立即数（即不是地址），因此不会对内存产生任何影响。
  - `path` 必须指向一个最终被链接到最终二进制文件的函数或静态变量。

r[asm.operand-type.supported-operands.const]
* `const <expr>`
  - `<expr>` 是一个整数常量表达式。
  - 使用与表达式求值结果相对应的格式化整数替换常量表达式。

```rust
# #[cfg(target_arch = "x86_64")] {
const M: u64 = 8;
unsafe { core::arch::asm!("mov {}, {n}", out(reg) _, n = const M); }
# }
```

r[asm.operand-type.supported-operands.label]
* `label <block>`
  - 将块的标签替换到 asm 模板字符串中。有关更多信息，请参见[标签](#标签)。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
unsafe { core::arch::asm!("0: jmp {} 0b", label { }); }
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.registers]
## 寄存器操作数

输入和输出操作数可以指定为显式寄存器或来自寄存器类别的寄存器，由寄存器分配器分配。显式寄存器作为字符串字面量（例如 `"eax"`）提供，而寄存器类别作为标识符（例如 `reg`）提供。

请注意，显式寄存器将寄存器名称视为汇编器不透明的 token，因此必须与目标架构的汇编器使用的名称匹配。

请注意，显式寄存器操作数不能在模板字符串中使用占位符：在模板字符串中显式使用寄存器名称。所有其他操作数必须在模板字符串中至少出现一次。

有关所有架构的可用寄存器类别和显式寄存器的列表，请参见下文。

r[asm.clobber_abi]
## `clobber_abi`

`clobber_abi` 是一个特殊操作数，用于提供有关哪些 ABI 的调用约定被内联汇编更改或遵守的信息。它接受由逗号分隔的 ABI 字符串列表。尽管 `clobber_abi` 操作数是一个标识符，但在格式化字符串中不能使用占位符。（注意：这是未来的兼容性规定。）

在发出内联汇编之前，编译器将根据指定的 ABI 保存所有需要保存的寄存器。然后，在发出汇编后，编译器将恢复这些寄存器。指定的 ABI 还确定了为汇编块内的标签生成的任何符号的名称修饰和调用约定。

```rust
# #[cfg(target_arch = "x86_64")] {
// 使用 C 调用约定调用 `foo` 函数
unsafe { core::arch::asm!("call {}", sym foo, clobber_abi("C")) }
# }
```

如果没有指定 `clobber_abi`，则认为汇编没有改变任何寄存器，并且编译器将不会保存或恢复任何寄存器。

r[asm.options]
## 选项

可以使用可选的第三个参数 `options` 来指定更多选项。这些选项在 `asm!` 调用的末尾指定。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut x: u64 = 4;
unsafe {
    asm!("inc {}", inout(reg) x, options(nostack, nomem, preserves_flags));
}
assert_eq!(x, 5);
# }
```

`options` 参数是一个以逗号分隔的标志列表。选项位提供的标志定义如下：

r[asm.options.supported-options]
支持以下选项：

r[asm.options.supported-options.pure]
- `pure` --- 汇编块除了其输出之外没有其他可观察到的副作用。这允许编译器在可能的情况下避免重新执行内联汇编。与 `nomem` 不同，`pure` 允许汇编块在未修改的内存上执行操作（例如读取），但不能写入内存。标志纯粹性检查与 C++ 中的 `__attribute__((pure))` 相同。

r[asm.options.supported-options.nomem]
- `nomem` --- 汇编块不读取或写入任何内存。这允许编译器缓存修改后的寄存器值，或将此汇编块移动到其他汇编代码之后（如果需要的寄存器不受影响）。标志纯粹性检查与 C++ 中的 `__attribute__((const))` 相同。

r[asm.options.supported-options.readonly]
- `readonly` --- 汇编块不写入任何内存。这允许编译器缓存未修改的寄存器值，或将此汇编块移动到其他汇编代码之后（如果需要的寄存器不受影响）。它还允许编译器推断此汇编块只读取其参数值，并且所有其他值保持不变。

r[asm.options.supported-options.preserves_flags]
- `preserves_flags` --- 汇编块不修改标志寄存器（在下文定义的[规则](#内联汇编的规则)中）。这允许编译器在汇编块之后避免重新计算条件标志。

r[asm.options.supported-options.noreturn]
- `noreturn` --- 汇编块永远不会返回，其行为像 `!`（never 类型）。行为是未定义的，如果确实返回的话。`noreturn` 汇编块的行为就像它没有输出的函数一样。

r[asm.options.supported-options.nostack]
- `nostack` --- 汇编块不将数据推入栈中，也不写入栈的红色区域（如果目标定义了红色区域）。这允许编译器使用栈红色区域跨 asm 块存储从寄存器溢出的值。

r[asm.options.supported-options.att_syntax]
- `att_syntax` --- 此选项仅在 x86 上有效，并导致汇编器使用 GNU 汇编器的 `.att_syntax prefix` 模式。寄存器操作数在使用时会自动添加前导 `%` 前缀。

r[asm.options.supported-options.raw]
- `raw` --- 这导致模板字符串被解析为原始汇编字符串，对 `{` 和 `}` 没有特殊处理。这在你想要在原始汇编字符串中包含与 Rust 字符串中使用的相同符号时主要是有用的。

r[asm.rules]
## 内联汇编的规则

r[asm.rules.intro]
为了避免未定义行为，必须遵守这些规则：

r[asm.rules.registers]
- 任何未在输出中指定的寄存器都不能被内联汇编更改，除非指定了 `clobber_abi` 或显式指定了清除器。当使用 `in` 操作数时，其值不能通过内联汇编更改。对于 `inout` 和 `inlateout` 操作数，可以更改值。编译器在这些限制下执行寄存器分配。

r[asm.rules.unsafe]
- 包含 `asm!` 宏的 `unsafe` 块或函数不能开始展开；应使用 `nomem` 选项或 `clobber_abi("C")`。有关更多信息，请参见[展开][unwinding-asm]。

r[asm.rules.flags]
- 除非指定了 `preserves_flags` 选项，否则标志寄存器在汇编块中被假定为已更改。对于某些目标，标志寄存器始终包含在[清除器列表](#寄存器清除器)中：这些目标具有作为所有 ABI 调用约定的一部分保留的标志寄存器。

r[asm.rules.framepointer]
- 在 x86 上，帧指针寄存器（`ebp` 或 `rbp`）不能用作输出或清除器，因为它由编译器维护以进行栈帧展开。

r[asm.rules.stack-pointer]
- 栈指针寄存器（在 x86 上为 `esp` 或 `rsp`）必须始终恢复为其原始值。

r[asm.rules.forward-jumps]
- 任何标签向前引用都可能引起混淆。我们不保证 ASM 块的任何预期内存排序。

r[asm.rules.mem-exclusive]
- 除非指定了 `nomem`、`readonly` 或 `pure`，否则编译器假定汇编块可以读取或写入任何可访问的内存地址。

r[asm.rules.clobber_abi-compat]
- 除非指定了 `clobber_abi`，否则假定汇编块遵循目标的外部调用约定。（例如，它不更改被调用者保存的寄存器。）

r[asm.rules.noreturn-compat]
- 除非指定了 `noreturn`，否则假定汇编块在汇编块的终止处将控制流传给下一条指令。未能使控制流正确传播是未定义行为。

r[asm.rules.noreturn-rust]
- 对于 `noreturn` 汇编块，控制流永远不会越过汇编块的尾部。

r[asm.rules.nomem-exclusive]
- 如果指定了 `nomem`，且未指定 `noreturn`，则假定除了修改输出寄存器（或其内容）外，该代码没有其他效果。

r[asm.rules.pure-exclusive]
- 如果指定了 `pure`，且未指定 `noreturn`，则假定该代码除了修改输出寄存器（或其内容）外，没有任何改变任何可观察状态的效果。

r[asm.rules.nostack-exclusive]
- 如果指定了 `nostack`，则假定没有任何可观察的内存地址包含来自汇编块内部写入的栈指针的相对地址。

r[asm.rules.readonly-exclusive]
- 如果指定了 `readonly`，则假定汇编块不修改任何可观察的内存。

r[asm.rules.overlapping-registers]
- 在不同寄存器类别中重叠的寄存器可以用来提供对不同大小寄存器的访问。但是，如果两个操作数最终使用重叠的寄存器，则会导致编译时错误。

r[asm.rules.template-must-be-valid]
- 汇编模板字符串必须是块的位置有效的汇编代码。不能跳入或跳出 asm 块。模板字符串必须由平衡的寄存器保存/恢复操作对组成。

r[asm.rules.global_asm-must-be-valid]
- `global_asm!` 宏中的汇编模板字符串也禁止使用某些指令。例如，在 macOS 上使用 `__thread` 指令会导致错误。

r[asm.rules.unwind]
- 在任何内联汇编执行之前，编译器会对受影响的寄存器执行保存和恢复。此过程可能会根据汇编宏的约束展开栈。

r[asm.rules.clobber_abi]
- 如果指定了 `clobber_abi`，则在引导进入汇编或退出汇编时，所需的保存和恢复过程将像具有该给定 ABI 的调用一样执行。对指定为输出操作数的寄存器的分配可能与 `clobber_abi` 的调用约定所需的保存/恢复冲突。如果发生这种情况，将产生编译时错误。

r[asm.rules.unwind-constraint]
- 编译器可能需要在 `clobber_abi` 不是 `"C"` 时生成附加的栈展开信息。如果无法生成展开信息，将产生编译时错误。

r[asm.rules.arm-special-registers]
- 在 ARM 目标上，某些寄存器和寄存器类别仅在使用 `clobber_abi` 指定兼容的调用约定时可用。特别是 `sp`、`pc`、`r7`、`fp`、`lr` 和部分浮点寄存器受限。

r[asm.rules.asm-template-consistency]
- 内联汇编中使用的标签数量可能与任何指定为 `label` 操作数的块数量不同。因此，`label` 操作数不能用于创建指向 ASM 块自身的跳转。

r[asm.labels]
## 标签

`label` 操作数允许在内联汇编代码块中指定局部标签。此操作数不与模板字符串中的占位符结合使用；而是将仅由该操作数的名称（或数值位置）引用的标签插入到模板字符串中。这允许内联汇编代码引用程序的其他部分（或自身），而不必担心标签名称冲突。

`label` 操作数的值是运行与该操作数关联的块。汇编宏跳转到块所在的位置，然后运行该块的代码。该块必须是 `unsafe` 块。

`label` 操作数与 `sym` 操作数具有相同的限制：它们必须引用一个函数或静态变量，该函数或静态变量最终会被链接到最终二进制文件。向前跳转（跳转到尚未看到的标签）是被允许的；向后跳转不被禁止。如果跳转到尚未被任何其他代码引用的`label` 操作数，则可能不会发出标签。

```rust
# #[cfg(target_arch = "x86_64")] {
use std::arch::asm;

// 跳转到标签
unsafe {
    asm!(
        "jmp {lbl}",
        lbl = label {
            println!("Jumped to label");
        }
    );
}
# }
```

[unwinding-asm]: #r-asm.rules.unwind

<!-- The rest of this file contains extensive target-specific register tables and directives support sections.
     These tables are kept identical to the English source as they contain register names, ABIs,
     and technical specifications that should not be translated. -->
