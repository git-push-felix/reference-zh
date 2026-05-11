r[asm]
# 内联汇编

r[asm.intro]
内联汇编的支持通过 [`asm!`]、[`naked_asm!`] 和 [`global_asm!`] 宏提供。它可以用于在编译器生成的汇编输出中嵌入手写汇编。

[`asm!`]: core::arch::asm
[`naked_asm!`]: core::arch::naked_asm
[`global_asm!`]: core::arch::global_asm

r[asm.stable-targets]
内联汇编的支持在以下架构上已稳定：
- x86 和 x86-64
- ARM
- AArch64 和 Arm64EC
- RISC-V
- LoongArch
- s390x
- PowerPC 和 PowerPC64

在不支持的平台上使用汇编宏时，编译器将发出错误。

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
使用 `asm!` 宏，汇编代码在函数作用域中发出并集成到编译器生成的函数汇编代码中。此汇编代码必须遵守[严格的规则](#rules-for-inline-assembly)以避免未定义行为。请注意，在某些情况下编译器可能会选择将汇编代码作为单独的函数发出并生成对其的调用。

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
unsafe { core::arch::asm!("/* {x} */"); } // ERROR: no argument named x
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
unsafe { core::arch::asm!("/* {x} */", x = const 5, "ud2"); } // ERROR: unexpected token
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.positional-first]
与格式字符串一样，位置参数必须出现在命名参数和显式[寄存器操作数](#register-operands)之前。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 命名操作数需要放在位置操作数之后
unsafe { core::arch::asm!("/* {x} {} */", x = const 5, in(reg) 5); }
// ERROR: positional arguments cannot follow named arguments or explicit register arguments
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们也不能将显式寄存器放在位置操作数之前
unsafe { core::arch::asm!("/* {} */", in("eax") 0, in(reg) 5); }
// ERROR: positional arguments cannot follow named arguments or explicit register arguments
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.register-operands]
显式寄存器操作数不能被模板字符串中的占位符使用。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 显式寄存器操作数不会被替换，在字符串中显式使用 `eax`
unsafe { core::arch::asm!("/* {} */", in("eax") 5); }
// ERROR: invalid reference to argument at index 0
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.at-least-once]
所有其他命名和位置操作数必须在模板字符串中至少出现一次，否则会生成编译器错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们必须在格式字符串中命名所有操作数
unsafe { core::arch::asm!("", in(reg) 5, x = const 5); }
// ERROR: multiple unused asm arguments
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.ts-args.opaque]
确切的汇编代码语法是平台特定的，对编译器是不透明的，除了操作数被替换到模板字符串中以形成传递给汇编器的代码的方式。

r[asm.ts-args.llvm-syntax]
目前，所有支持的目标都遵循 LLVM 内部汇编器使用的汇编代码语法，这通常对应于 GNU 汇编器 (GAS) 的语法。在 x86 上，默认使用 GAS 的 `.intel_syntax noprefix` 模式。在 ARM 上，使用 `.syntax unified` 模式。这些目标对汇编代码施加了一个额外的限制：任何汇编器状态（例如可以用 `.section` 更改的当前节）必须在 asm 字符串的末尾恢复到其原始值。不符合 GAS 语法的汇编代码将导致特定于汇编器的行为。内联汇编使用的指令的进一步约束由[指令支持](#directives-support)指示。

[format-syntax]: std::fmt#syntax
[rfc-2795]: https://github.com/rust-lang/rfcs/pull/2795

r[asm.attributes]
## 属性

r[asm.attributes.supported-attributes]
仅 [`cfg`] 和 [`cfg_attr`] 属性在内联汇编模板字符串和操作数上被语义接受。其他属性会被解析但在汇编宏展开时被拒绝。

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
> 在 `rustc` 中，汇编宏实现处理这些属性的方式与处理语言中类似属性的常规系统分开。这解释了支持的属性类型的有限性，并可能导致行为的细微差异。

r[asm.attributes.starts-with-template]
语法上，第一个操作数之前必须至少有一个模板字符串。

```rust,compile_fail
// 这被拒绝，因为 `a = out(reg) x` 不被解析为模板字符串。
core::arch::asm!(
    #[cfg(false)]
    a = out(reg) x, // ERROR。
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
  - `<path>` 必须引用一个 `fn` 或 `static`。
  - 引用该项的装饰过的符号名被替换到 asm 模板字符串中。
  - 替换后的字符串不包含任何修饰符（例如 GOT、PLT、重定位等）。
  - `<path>` 允许指向 `#[thread_local]` 静态变量，在这种情况下，汇编代码可以结合重定位（例如 `@plt`、`@TPOFF`）从线程局部数据中读取。

```rust
# #[cfg(target_arch = "x86_64")] {
extern "C" fn foo() {
    println!("Hello from inline assembly")
}
// `sym` 可用于引用函数（即使它没有我们可以直接编写的外部名称）
unsafe { core::arch::asm!("call {}", sym foo, clobber_abi("C")); }
# }
```

r[asm.operand-type.supported-operands.const]
* `const <expr>`
  - `<expr>` 必须是一个整数常量表达式。此表达式遵循与内联 `const` 块相同的规则。
  - 表达式的类型可以是任何整数类型，但默认与整数字面量一样为 `i32`。
  - 表达式的值被格式化为字符串并直接替换到 asm 模板字符串中。

```rust
# #[cfg(target_arch = "x86_64")] {
// 交换 [0, 1, 2, 3] => [3, 2, 0, 1]
const SHUFFLE: u8 = 0b01_00_10_11;
let x: core::arch::x86_64::__m128 = unsafe { core::mem::transmute([0u32, 1u32, 2u32, 3u32]) };
let y: core::arch::x86_64::__m128;
// 将常量值传入期望立即数的指令，如 `pshufd`
unsafe {
    core::arch::asm!("pshufd {xmm}, {xmm}, {shuffle}",
        xmm = inlateout(xmm_reg) x=>y,
        shuffle = const SHUFFLE
    );
}
let y: [u32; 4] = unsafe { core::mem::transmute(y) };
assert_eq!(y, [3, 2, 0, 1]);
# }
```

r[asm.operand-type.supported-operands.label]
* `label <block>`
  - 块的地址被替换到 asm 模板字符串中。汇编代码可以跳转到替换后的地址。
  - 对于区分直接跳转和间接跳转的目标（例如启用了 `cf-protection` 的 x86-64），汇编代码不能间接跳转到替换后的地址。
  - 块执行完毕后，`asm!` 表达式返回。
  - 块的类型必须是单元类型或 `!`（never 类型）。
  - 块启动一个新的安全性上下文；`label` 块内的不安全操作必须包装在内部的 `unsafe` 块中，即使整个 `asm!` 表达式已经被 `unsafe` 包装。

```rust
# #[cfg(target_arch = "x86_64")]
unsafe {
    core::arch::asm!("jmp {}", label {
        println!("Hello from inline assembly label");
    });
}
```

r[asm.operand-type.left-to-right]
操作数表达式从左到右求值，就像函数调用参数一样。在 `asm!` 执行完毕后，输出按从左到右的顺序写入。如果两个输出指向同一位置，这很重要：该位置将包含最右侧输出的值。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut y: i64;
// y 从第二个输出获取值，而不是第一个
unsafe { core::arch::asm!("mov {}, 0", "mov {}, 1", out(reg) y, out(reg) y); }
assert_eq!(y, 1);
# }
```

r[asm.operand-type.naked_asm-restriction]
由于 `naked_asm!` 定义了整个函数体而编译器无法发出任何额外代码来处理操作数，因此它只能使用 `sym` 和 `const` 操作数。

r[asm.operand-type.global_asm-restriction]
由于 `global_asm!` 存在于函数外部，因此它只能使用 `sym` 和 `const` 操作数。

```rust,compile_fail
# fn main() {}
// 不允许寄存器操作数，因为我们不在函数中
# #[cfg(target_arch = "x86_64")]
core::arch::global_asm!("", in(reg) 5);
// ERROR: the `in` operand cannot be used with `global_asm!`
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

```rust
# fn main() {}
fn foo() {}

# #[cfg(target_arch = "x86_64")]
// 然而 `const` 和 `sym` 都是允许的
core::arch::global_asm!("/* {} {} */", const 0, sym foo);
```

r[asm.register-operands]
## 寄存器操作数 {#register-operands}

r[asm.register-operands.register-or-class]
输入和输出操作数可以指定为显式寄存器或来自寄存器类别的寄存器，由寄存器分配器从中选择。显式寄存器作为字符串字面量（例如 `"eax"`）指定，而寄存器类别作为标识符（例如 `reg`）指定。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut y: i64;
// 我们可以命名 `reg` 或像 `eax` 这样的显式寄存器来获取整数寄存器
unsafe { core::arch::asm!("mov eax, {:e}", in(reg) 5, lateout("eax") y); }
assert_eq!(y, 5);
# }
```

r[asm.register-operands.equivalence-to-base-register]
请注意，显式寄存器将寄存器别名（例如 ARM 上的 `r14` 与 `lr`）和寄存器的较小视图（例如 `eax` 与 `rax`）视为等同于基寄存器。

r[asm.register-operands.error-two-operands]
将同一显式寄存器用于两个输入操作数或两个输出操作数是编译时错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们不能两次命名 eax
unsafe { core::arch::asm!("", in("eax") 5, in("eax") 4); }
// ERROR: register `eax` conflicts with register `eax`
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```
```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// ...即使使用不同的别名也不行
unsafe { core::arch::asm!("", in("ax") 5, in("rax") 4); }
// ERROR: register `rax` conflicts with register `ax`
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-operands.error-overlapping]
此外，在输入操作数或输出操作数中使用重叠寄存器（例如 ARM VFP）也是编译时错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// al 与 ax 重叠，所以我们不能同时命名两者。
unsafe { core::arch::asm!("", in("ax") 5, in("al") 4i8); }
// ERROR: register `al` conflicts with register `ax`
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-operands.allowed-types]
仅以下类型允许作为内联汇编的操作数：
- 整数（有符号和无符号）
- 浮点数
- 指针（仅瘦指针）
- 函数指针
- SIMD 向量（使用 `#[repr(simd)]` 定义且实现了 `Copy` 的结构体）。这包括在 `std::arch` 中定义的架构特定向量类型，如 `__m128`（x86）或 `int8x16_t`（ARM）。

```rust
# #[cfg(target_arch = "x86_64")] {
extern "C" fn foo() {}

// 整数是允许的……
let y: i64 = 5;
unsafe { core::arch::asm!("/* {} */", in(reg) y); }

// 还有指针……
let py = &raw const y;
unsafe { core::arch::asm!("/* {} */", in(reg) py); }

// 浮点数也是……
let f = 1.0f32;
unsafe { core::arch::asm!("/* {} */", in(xmm_reg) f); }

// 甚至函数指针和 SIMD 向量也行。
let func: extern "C" fn() = foo;
unsafe { core::arch::asm!("/* {} */", in(reg) func); }

let z = unsafe { core::arch::x86_64::_mm_set_epi64x(1, 0) };
unsafe { core::arch::asm!("/* {} */", in(xmm_reg) z); }
# }
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
struct Foo;
let x: Foo = Foo;
// 像结构体这样的复杂类型是不允许的
unsafe { core::arch::asm!("/* {} */", in(reg) x); }
// ERROR: cannot use value of type `Foo` for inline assembly
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-operands.supported-register-classes]
以下是当前支持的寄存器类别列表：

| 架构 | 寄存器类别 | 寄存器 | LLVM 约束代码 |
| ------------ | -------------- | --------- | -------------------- |
| x86 | `reg` | `ax`, `bx`, `cx`, `dx`, `si`, `di`, `bp`, `r[8-15]` (仅 x86-64) | `r` |
| x86 | `reg_abcd` | `ax`, `bx`, `cx`, `dx` | `Q` |
| x86-32 | `reg_byte` | `al`, `bl`, `cl`, `dl`, `ah`, `bh`, `ch`, `dh` | `q` |
| x86-64 | `reg_byte`\* | `al`, `bl`, `cl`, `dl`, `sil`, `dil`, `bpl`, `r[8-15]b` | `q` |
| x86 | `xmm_reg` | `xmm[0-7]` (x86) `xmm[0-15]` (x86-64) | `x` |
| x86 | `ymm_reg` | `ymm[0-7]` (x86) `ymm[0-15]` (x86-64) | `x` |
| x86 | `zmm_reg` | `zmm[0-7]` (x86) `zmm[0-31]` (x86-64) | `v` |
| x86 | `kreg` | `k[1-7]` | `Yk` |
| x86 | `kreg0` | `k0` | 仅清除器 |
| x86 | `x87_reg` | `st([0-7])` | 仅清除器 |
| x86 | `mmx_reg` | `mm[0-7]` | 仅清除器 |
| x86-64 | `tmm_reg` | `tmm[0-7]` | 仅清除器 |
| AArch64 | `reg` | `x[0-30]` | `r` |
| AArch64 | `vreg` | `v[0-31]` | `w` |
| AArch64 | `vreg_low16` | `v[0-15]` | `x` |
| AArch64 | `preg` | `p[0-15]`, `ffr` | 仅清除器 |
| Arm64EC | `reg` | `x[0-12]`, `x[15-22]`, `x[25-27]`, `x30` | `r` |
| Arm64EC | `vreg` | `v[0-15]` | `w` |
| Arm64EC | `vreg_low16` | `v[0-15]` | `x` |
| ARM (ARM/Thumb2) | `reg` | `r[0-12]`, `r14` | `r` |
| ARM (Thumb1) | `reg` | `r[0-7]` | `r` |
| ARM | `sreg` | `s[0-31]` | `t` |
| ARM | `sreg_low16` | `s[0-15]` | `x` |
| ARM | `dreg` | `d[0-31]` | `w` |
| ARM | `dreg_low16` | `d[0-15]` | `t` |
| ARM | `dreg_low8` | `d[0-8]` | `x` |
| ARM | `qreg` | `q[0-15]` | `w` |
| ARM | `qreg_low8` | `q[0-7]` | `t` |
| ARM | `qreg_low4` | `q[0-3]` | `x` |
| RISC-V | `reg` | `x1`, `x[5-7]`, `x[9-15]`, `x[16-31]` (非 RV32E) | `r` |
| RISC-V | `freg` | `f[0-31]` | `f` |
| RISC-V | `vreg` | `v[0-31]` | 仅清除器 |
| LoongArch | `reg` | `$r1`, `$r[4-20]`, `$r[23,30]` | `r` |
| LoongArch | `freg` | `$f[0-31]` | `f` |
| s390x | `reg` | `r[0-10]`, `r[12-14]` | `r` |
| s390x | `reg_addr` | `r[1-10]`, `r[12-14]` | `a` |
| s390x | `freg` | `f[0-15]` | `f` |
| s390x | `vreg` | `v[0-31]` | `v` |
| s390x | `areg` | `a[2-15]` | 仅清除器 |
| PowerPC | `reg` | `r0`, `r[3-12]`, `r[14-28]` | `r` |
| PowerPC | `reg_nonzero` | `r[3-12]`, `r[14-28]` | `b` |
| PowerPC | `spe_acc` | `spe_acc` | 仅清除器 |
| PowerPC64 | `reg` | `r0`, `r[3-12]`, `r[14-29]` | `r` |
| PowerPC64 | `reg_nonzero` | `r[3-12]`, `r[14-29]` | `b` |
| PowerPC/PowerPC64 | `freg` | `f[0-31]` | `f` |
| PowerPC/PowerPC64 | `vreg` | `v[0-31]` | `v` |
| PowerPC/PowerPC64 | `vsreg` | `vs[0-63]` | `wa` |
| PowerPC/PowerPC64 | `cr` | `cr[0-7]`, `cr` | 仅清除器 |
| PowerPC/PowerPC64 | `ctr` | `ctr` | 仅清除器 |
| PowerPC/PowerPC64 | `lr` | `lr` | 仅清除器 |
| PowerPC/PowerPC64 | `xer` | `xer` | 仅清除器 |

> [!NOTE]
> - 在 x86 上，我们将 `reg_byte` 与 `reg` 区别对待，因为编译器可以分别分配 `al` 和 `ah`，而 `reg` 会保留整个寄存器。
> - 在 x86-64 上，高字节寄存器（例如 `ah`）在 `reg_byte` 寄存器类别中不可用。
> - 某些寄存器类别标记为"仅清除器"，这意味着这些类别中的寄存器不能用于输入或输出，只能使用 `out(<explicit register>) _` 或 `lateout(<explicit register>) _` 形式的清除器。
> - `spe_acc` 寄存器仅在 PowerPC SPE 目标上可用。

r[asm.register-operands.value-type-constraints]
每个寄存器类别对其可使用的值类型有约束。这是必要的，因为将值加载到寄存器的方式取决于其类型。例如，在大端系统上，将 `i32x4` 和 `i8x16` 加载到 SIMD 寄存器可能产生不同的寄存器内容，即使两个值的字节级内存表示相同。特定寄存器类别的受支持类型的可用性可能取决于当前启用的目标特性。

| 架构 | 寄存器类别 | 目标特性 | 允许的类型 |
| ------------ | -------------- | -------------- | ------------- |
| x86-32 | `reg` | None | `i16`, `i32`, `f32` |
| x86-64 | `reg` | None | `i16`, `i32`, `f32`, `i64`, `f64` |
| x86 | `reg_byte` | None | `i8` |
| x86 | `xmm_reg` | `sse` | `i32`, `f32`, `i64`, `f64`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` |
| x86 | `ymm_reg` | `avx` | `i32`, `f32`, `i64`, `f64`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` <br> `i8x32`, `i16x16`, `i32x8`, `i64x4`, `f32x8`, `f64x4` |
| x86 | `zmm_reg` | `avx512f` | `i32`, `f32`, `i64`, `f64`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` <br> `i8x32`, `i16x16`, `i32x8`, `i64x4`, `f32x8`, `f64x4` <br> `i8x64`, `i16x32`, `i32x16`, `i64x8`, `f32x16`, `f64x8` |
| x86 | `kreg` | `avx512f` | `i8`, `i16` |
| x86 | `kreg` | `avx512bw` | `i32`, `i64` |
| x86 | `mmx_reg` | N/A | 仅清除器 |
| x86 | `x87_reg` | N/A | 仅清除器 |
| x86 | `tmm_reg` | N/A | 仅清除器 |
| AArch64 | `reg` | None | `i8`, `i16`, `i32`, `f32`, `i64`, `f64` |
| AArch64 | `vreg` | `neon` | `i8`, `i16`, `i32`, `f32`, `i64`, `f64`, <br> `i8x8`, `i16x4`, `i32x2`, `i64x1`, `f32x2`, `f64x1`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` |
| AArch64 | `preg` | N/A | 仅清除器 |
| Arm64EC | `reg` | None | `i8`, `i16`, `i32`, `f32`, `i64`, `f64` |
| Arm64EC | `vreg` | `neon` | `i8`, `i16`, `i32`, `f32`, `i64`, `f64`, <br> `i8x8`, `i16x4`, `i32x2`, `i64x1`, `f32x2`, `f64x1`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` |
| ARM | `reg` | None | `i8`, `i16`, `i32`, `f32` |
| ARM | `sreg` | `vfp2` | `i32`, `f32` |
| ARM | `dreg` | `vfp2` | `i64`, `f64`, `i8x8`, `i16x4`, `i32x2`, `i64x1`, `f32x2` |
| ARM | `qreg` | `neon` | `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4` |
| RISC-V32 | `reg` | None | `i8`, `i16`, `i32`, `f32` |
| RISC-V64 | `reg` | None | `i8`, `i16`, `i32`, `f32`, `i64`, `f64` |
| RISC-V | `freg` | `f` | `f32` |
| RISC-V | `freg` | `d` | `f64` |
| RISC-V | `vreg` | N/A | 仅清除器 |
| LoongArch32 | `reg` | None | `i8`, `i16`, `i32`, `f32` |
| LoongArch64 | `reg` | None | `i8`, `i16`, `i32`, `i64`, `f32`, `f64` |
| LoongArch | `freg` | `f` | `f32` |
| LoongArch | `freg` | `d` | `f64` |
| s390x | `reg`, `reg_addr` | None | `i8`, `i16`, `i32`, `i64` |
| s390x | `freg` | None | `f32`, `f64` |
| s390x | `vreg` | `vector` | `i32`, `f32`, `i64`, `f64`, `i128`, <br> `i8x16`, `i16x8`, `i32x4`, `i64x2`, `f32x4`, `f64x2` |
| s390x | `areg` | N/A | 仅清除器 |
| PowerPC | `spe_acc` | None | 仅清除器 |
| PowerPC/PowerPC64 | `reg` | None | `i8`, `i16`, `i32`, `i64` (仅 PowerPC64) |
| PowerPC/PowerPC64 | `reg_nonzero` | None | `i8`, `i16`, `i32`, `i64` (仅 PowerPC64) |
| PowerPC/PowerPC64 | `freg` | None | `f32`, `f64` |
| PowerPC/PowerPC64 | `vreg` | `altivec` | `i8x16`, `i16x8`, `i32x4`, `f32x4` |
| PowerPC/PowerPC64 | `vreg` | `vsx` | `f32`, `f64`, `i64x2`, `f64x2` |
| PowerPC/PowerPC64 | `vsreg` | `vsx` | vsx 和 altivec vreg 类型的并集 |
| PowerPC/PowerPC64 | `cr` | None | 仅清除器 |
| PowerPC/PowerPC64 | `ctr` | None | 仅清除器 |
| PowerPC/PowerPC64 | `lr` | None | 仅清除器 |
| PowerPC/PowerPC64 | `xer` | None | 仅清除器 |

> [!NOTE]
> 就上表而言，指针、函数指针和 `isize`/`usize` 被视为等效的整数类型（根据目标为 `i16`/`i32`/`i64`）。

```rust
# #[cfg(target_arch = "x86_64")] {
let x = 5i32;
let y = -1i8;
let z = unsafe { core::arch::x86_64::_mm_set_epi64x(1, 0) };

// reg 对 `i32` 有效，reg_byte 对 `i8` 有效，xmm_reg 对 `__m128i` 有效
// 我们不能将 `tmm0` 用作输入或输出，但可以将其清除。
unsafe { core::arch::asm!("/* {} {} {} */", in(reg) x, in(reg_byte) y, in(xmm_reg) z, out("tmm0") _); }
# }
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
let z = unsafe { core::arch::x86_64::_mm_set_epi64x(1, 0) };
// 我们不能将 `__m128i` 传递给 `reg` 输入
unsafe { core::arch::asm!("/* {} */", in(reg) z); }
// ERROR: type `__m128i` cannot be used with this register class
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-operands.smaller-value]
如果值的尺寸小于其分配的寄存器，则对于输入，该寄存器的高位比特将具有未定义的值，对于输出则会被忽略。唯一的例外是 RISC-V 上的 `freg` 寄存器类别，其中 `f32` 值按照 RISC-V 架构的要求在 `f64` 中进行 NaN 装箱。

<!--no_run, this test has a non-deterministic runtime behavior-->
```rust,no_run
# #[cfg(target_arch = "x86_64")] {
let mut x: i64;
// 将 32 位值移动到 64 位值中，糟糕。
#[allow(asm_sub_register)] // rustc 对此行为发出警告
unsafe { core::arch::asm!("mov {}, {}", lateout(reg) x, in(reg) 4i32); }
// 高 32 位是不确定的
assert_eq!(x, 4); // 此断言不保证成功
assert_eq!(x & 0xFFFFFFFF, 4); // 然而，这个断言会成功
# }
```

r[asm.register-operands.separate-input-output]
当为 `inout` 操作数指定单独的输入和输出表达式时，两个表达式必须具有相同的类型。唯一的例外是如果两个操作数都是指针或整数，在这种情况下它们只需要具有相同的尺寸。此限制存在是因为 LLVM 和 GCC 中的寄存器分配器有时无法处理具有不同类型的关联操作数。

```rust
# #[cfg(target_arch = "x86_64")] {
// 指针和整数可以混合（只要它们尺寸相同）
let x: isize = 0;
let y: *mut ();
// 使用内联汇编魔法将 `isize` 转换为 `*mut ()`
unsafe { core::arch::asm!("/*{}*/", inout(reg) x=>y); }
assert!(y.is_null()); // 极其迂回地创建空指针的方式
# }
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
let x: i32 = 0;
let y: f32;
// 但我们不能这样将 `i32` 重新解释为 `f32`
unsafe { core::arch::asm!("/* {} */", inout(reg) x=>y); }
// ERROR: incompatible types for asm inout argument
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-names]
## 寄存器名称

r[asm.register-names.supported-register-aliases]
某些寄存器有多个名称。这些都被编译器视为等同于基寄存器名称。以下是所有支持的寄存器别名列表：

| 架构 | 基寄存器 | 别名 |
| ------------ | ------------- | ------- |
| x86 | `ax` | `eax`, `rax` |
| x86 | `bx` | `ebx`, `rbx` |
| x86 | `cx` | `ecx`, `rcx` |
| x86 | `dx` | `edx`, `rdx` |
| x86 | `si` | `esi`, `rsi` |
| x86 | `di` | `edi`, `rdi` |
| x86 | `bp` | `bpl`, `ebp`, `rbp` |
| x86 | `sp` | `spl`, `esp`, `rsp` |
| x86 | `ip` | `eip`, `rip` |
| x86 | `st(0)` | `st` |
| x86 | `r[8-15]` | `r[8-15]b`, `r[8-15]w`, `r[8-15]d` |
| x86 | `xmm[0-31]` | `ymm[0-31]`, `zmm[0-31]` |
| AArch64 | `x[0-30]` | `w[0-30]` |
| AArch64 | `x29` | `fp` |
| AArch64 | `x30` | `lr` |
| AArch64 | `sp` | `wsp` |
| AArch64 | `xzr` | `wzr` |
| AArch64 | `v[0-31]` | `b[0-31]`, `h[0-31]`, `s[0-31]`, `d[0-31]`, `q[0-31]` |
| Arm64EC | `x[0-30]` | `w[0-30]` |
| Arm64EC | `x29` | `fp` |
| Arm64EC | `x30` | `lr` |
| Arm64EC | `sp` | `wsp` |
| Arm64EC | `xzr` | `wzr` |
| Arm64EC | `v[0-15]` | `b[0-15]`, `h[0-15]`, `s[0-15]`, `d[0-15]`, `q[0-15]` |
| ARM | `r[0-3]` | `a[1-4]` |
| ARM | `r[4-9]` | `v[1-6]` |
| ARM | `r9` | `rfp` |
| ARM | `r10` | `sl` |
| ARM | `r11` | `fp` |
| ARM | `r12` | `ip` |
| ARM | `r13` | `sp` |
| ARM | `r14` | `lr` |
| ARM | `r15` | `pc` |
| RISC-V | `x0` | `zero` |
| RISC-V | `x1` | `ra` |
| RISC-V | `x2` | `sp` |
| RISC-V | `x3` | `gp` |
| RISC-V | `x4` | `tp` |
| RISC-V | `x[5-7]` | `t[0-2]` |
| RISC-V | `x8` | `fp`, `s0` |
| RISC-V | `x9` | `s1` |
| RISC-V | `x[10-17]` | `a[0-7]` |
| RISC-V | `x[18-27]` | `s[2-11]` |
| RISC-V | `x[28-31]` | `t[3-6]` |
| RISC-V | `f[0-7]` | `ft[0-7]` |
| RISC-V | `f[8-9]` | `fs[0-1]` |
| RISC-V | `f[10-17]` | `fa[0-7]` |
| RISC-V | `f[18-27]` | `fs[2-11]` |
| RISC-V | `f[28-31]` | `ft[8-11]` |
| LoongArch | `$r0` | `$zero` |
| LoongArch | `$r1` | `$ra` |
| LoongArch | `$r2` | `$tp` |
| LoongArch | `$r3` | `$sp` |
| LoongArch | `$r[4-11]` | `$a[0-7]` |
| LoongArch | `$r[12-20]` | `$t[0-8]` |
| LoongArch | `$r21` | |
| LoongArch | `$r22` | `$fp`, `$s9` |
| LoongArch | `$r[23-31]` | `$s[0-8]` |
| LoongArch | `$f[0-7]` | `$fa[0-7]` |
| LoongArch | `$f[8-23]` | `$ft[0-15]` |
| LoongArch | `$f[24-31]` | `$fs[0-7]` |
| PowerPC/PowerPC64 | `r1` | `sp` |
| PowerPC/PowerPC64 | `r31` | `fp` |
| PowerPC/PowerPC64 | `r[0-31]` | `[0-31]` |
| PowerPC/PowerPC64 | `f[0-31]` | `fr[0-31]`|

```rust
# #[cfg(target_arch = "x86_64")] {
let z = 0i64;
// rax 是 eax 和 ax 的别名
unsafe { core::arch::asm!("", in("rax") z); }
# }
```

r[asm.register-names.not-for-io]
某些寄存器不能用于输入或输出操作数：

| 架构 | 不支持的寄存器 | 原因 |
| ------------ | -------------------- | ------ |
| All | `sp`, `r15` (s390x), `r1` (PowerPC 和 PowerPC64) | 栈指针必须在汇编代码结束时或跳转到 `label` 块之前恢复到其原始值。 |
| All | `bp` (x86), `x29` (AArch64 和 Arm64EC), `x8` (RISC-V), `$fp` (LoongArch), `r11` (s390x), `fp` (PowerPC 和 PowerPC64) | 帧指针不能用作输入或输出。 |
| ARM | `r7` 或 `r11` | 在 ARM 上，帧指针可以是 `r7` 或 `r11`，取决于目标。帧指针不能用作输入或输出。 |
| All | `si` (x86-32), `bx` (x86-64), `r6` (ARM), `x19` (AArch64 和 Arm64EC), `x9` (RISC-V), `$s8` (LoongArch), `r29` 和 `r30` (PowerPC), `r30` (PowerPC64) | 此寄存器被 LLVM 内部用作具有复杂栈帧的函数的"基指针"。 |
| x86 | `ip` | 这是程序计数器，不是真正的寄存器。 |
| AArch64 | `xzr` | 这是一个常量零寄存器，不能被修改。 |
| AArch64 | `x18` | 在某些 AArch64 目标上这是 OS 保留的寄存器。 |
| Arm64EC | `xzr` | 这是一个常量零寄存器，不能被修改。 |
| Arm64EC | `x18` | 这是一个 OS 保留的寄存器。 |
| Arm64EC | `x13`, `x14`, `x23`, `x24`, `x28`, `v[16-31]`, `p[0-15]`, `ffr` | 这些是 Arm64EC 不支持的 AArch64 寄存器。 |
| ARM | `pc` | 这是程序计数器，不是真正的寄存器。 |
| ARM | `r9` | 在某些 ARM 目标上这是 OS 保留的寄存器。 |
| RISC-V | `x0` | 这是一个常量零寄存器，不能被修改。 |
| RISC-V | `gp`, `tp` | 这些寄存器是保留的，不能用作输入或输出。 |
| LoongArch | `$r0` 或 `$zero` | 这是一个常量零寄存器，不能被修改。 |
| LoongArch | `$r2` 或 `$tp` | 此寄存器保留用于 TLS。 |
| LoongArch | `$r21` | 此寄存器由 ABI 保留。 |
| s390x | `c[0-15]` | 由内核保留。 |
| s390x | `a[0-1]` | 保留供系统使用。 |
| PowerPC/PowerPC64 | `r2`, `r13` | 这些是系统保留的寄存器。 |
| PowerPC/PowerPC64 | `vrsave` | vrsave 寄存器不能用作输入或输出。 |

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// bp 是保留的
unsafe { core::arch::asm!("", in("bp") 5i32); }
// ERROR: invalid register `bp`: the frame pointer cannot be used as an operand for inline asm
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.register-names.fp-bp-reserved]
帧指针寄存器和基指针寄存器保留供 LLVM 内部使用。虽然 `asm!` 语句不能显式指定使用保留寄存器，但在某些情况下 LLVM 会为 `reg` 操作数分配这些保留寄存器之一。使用保留寄存器的汇编代码应谨慎，因为 `reg` 操作数可能使用相同的寄存器。

r[asm.template-modifiers]
## 模板修饰符

r[asm.template-modifiers.intro]
占位符可以通过修饰符来增强，修饰符在大括号中的 `:` 之后指定。这些修饰符不影响寄存器分配，但会改变操作数插入模板字符串时的格式化方式。

r[asm.template-modifiers.only-one]
每个模板占位符只允许一个修饰符。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// 我们不能同时指定 `r` 和 `e`。
unsafe { core::arch::asm!("/* {:er}", in(reg) 5i32); }
// ERROR: asm template modifier must be a single character
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.template-modifiers.supported-modifiers]
支持的修饰符是 LLVM（和 GCC）的 [asm 模板参数修饰符][llvm-argmod] 的子集，但不使用相同的字母代码。

| 架构 | 寄存器类别 | 修饰符 | 示例输出 | LLVM 修饰符 |
| ------------ | -------------- | -------- | -------------- | ------------- |
| x86-32 | `reg` | None | `eax` | `k` |
| x86-64 | `reg` | None | `rax` | `q` |
| x86-32 | `reg_abcd` | `l` | `al` | `b` |
| x86-64 | `reg` | `l` | `al` | `b` |
| x86 | `reg_abcd` | `h` | `ah` | `h` |
| x86 | `reg` | `x` | `ax` | `w` |
| x86 | `reg` | `e` | `eax` | `k` |
| x86-64 | `reg` | `r` | `rax` | `q` |
| x86 | `reg_byte` | None | `al` / `ah` | None |
| x86 | `xmm_reg` | None | `xmm0` | `x` |
| x86 | `ymm_reg` | None | `ymm0` | `t` |
| x86 | `zmm_reg` | None | `zmm0` | `g` |
| x86 | `*mm_reg` | `x` | `xmm0` | `x` |
| x86 | `*mm_reg` | `y` | `ymm0` | `t` |
| x86 | `*mm_reg` | `z` | `zmm0` | `g` |
| x86 | `kreg` | None | `k1` | None |
| AArch64/Arm64EC | `reg` | None | `x0` | `x` |
| AArch64/Arm64EC | `reg` | `w` | `w0` | `w` |
| AArch64/Arm64EC | `reg` | `x` | `x0` | `x` |
| AArch64/Arm64EC | `vreg` | None | `v0` | None |
| AArch64/Arm64EC | `vreg` | `v` | `v0` | None |
| AArch64/Arm64EC | `vreg` | `b` | `b0` | `b` |
| AArch64/Arm64EC | `vreg` | `h` | `h0` | `h` |
| AArch64/Arm64EC | `vreg` | `s` | `s0` | `s` |
| AArch64/Arm64EC | `vreg` | `d` | `d0` | `d` |
| AArch64/Arm64EC | `vreg` | `q` | `q0` | `q` |
| ARM | `reg` | None | `r0` | None |
| ARM | `sreg` | None | `s0` | None |
| ARM | `dreg` | None | `d0` | `P` |
| ARM | `qreg` | None | `q0` | `q` |
| ARM | `qreg` | `e` / `f` | `d0` / `d1` | `e` / `f` |
| RISC-V | `reg` | None | `x1` | None |
| RISC-V | `freg` | None | `f0` | None |
| LoongArch | `reg` | None | `$r1` | None |
| LoongArch | `freg` | None | `$f0` | None |
| s390x | `reg` | None | `%r0` | None |
| s390x | `reg_addr` | None | `%r1` | None |
| s390x | `freg` | None | `%f0` | None |
| s390x | `vreg` | None | `%v0` | None |
| PowerPC/PowerPC64 | `reg` | None | `0` | None |
| PowerPC/PowerPC64 | `reg_nonzero` | None | `3` | None |
| PowerPC/PowerPC64 | `freg` | None | `0` | None |
| PowerPC/PowerPC64 | `vreg` | None | `0` | None |
| PowerPC/PowerPC64 | `vsreg` | None | `0` | None |

> [!NOTE]
> - 在 ARM 上 `e` / `f`：此修饰符打印 NEON 四字（128 位）寄存器的低或高双字寄存器名称。
> - 在 x86 上：我们对无修饰符的 `reg` 的行为与 GCC 不同。GCC 会基于操作数值类型推断修饰符，而我们默认使用完整的寄存器大小。
> - 在 x86 `xmm_reg` 上：`x`、`t` 和 `g` LLVM 修饰符尚未在 LLVM 中实现（它们仅受 GCC 支持），但这应该是一个简单的更改。

```rust
# #[cfg(target_arch = "x86_64")] {
let mut x = 0x10u16;

// 使用 `xchg` 进行 u16::swap_bytes
// `{x}` 的低半部分由 `{x:l}` 引用，高半部分由 `{x:h}` 引用
unsafe { core::arch::asm!("xchg {x:l}, {x:h}", x = inout(reg_abcd) x); }
assert_eq!(x, 0x1000u16);
# }
```

r[asm.template-modifiers.smaller-value]
如前一节所述，传入小于寄存器宽度的输入值将导致寄存器的高位比特包含未定义的值。如果内联汇编仅访问寄存器的低位比特，这不成问题，可以通过使用模板修饰符在汇编代码中使用子寄存器名称（例如 `ax` 而非 `rax`）来实现。由于这是一个容易犯的错误，编译器会在适当时根据输入类型建议使用模板修饰符。如果对某个操作数的所有引用都已经有修饰符，则对该操作数的警告将被抑制。

[llvm-argmod]: http://llvm.org/docs/LangRef.html#asm-template-argument-modifiers

r[asm.abi-clobbers]
## ABI 清除器

r[asm.abi-clobbers.intro]
`clobber_abi` 关键字可用于对汇编代码应用一组默认的清除器。这将自动插入必要的清除器约束，以适应使用特定调用约定调用函数的需求：如果调用约定在跨函数调用时不完全保留寄存器的值，则 `lateout("...") _` 会隐式添加到操作数列表中（其中 `...` 替换为寄存器的名称）。

```rust
# #[cfg(target_arch = "x86_64")] {
extern "C" fn foo() -> i32 { 0 }

let z: i32;
// 要调用函数，我们必须通知编译器我们要清除被调用者保存的寄存器
unsafe { core::arch::asm!("call {}", sym foo, out("rax") z, clobber_abi("C")); }
assert_eq!(z, 0);
# }
```

r[asm.abi-clobbers.many]
`clobber_abi` 可以指定任意次数。它将为所有指定调用约定的并集中的每个唯一寄存器插入一个清除器。

```rust
# #[cfg(target_arch = "x86_64")] {
extern "sysv64" fn foo() -> i32 { 0 }
extern "win64" fn bar(x: i32) -> i32 { x + 1 }

let z: i32;
// 我们甚至可以调用具有不同约定和不同保存寄存器的多个函数
unsafe {
    core::arch::asm!(
        "call {}",
        "mov ecx, eax",
        "call {}",
        sym foo,
        sym bar,
        out("rax") z,
        clobber_abi("sysv64"),
        clobber_abi("win64"),
    );
}
assert_eq!(z, 1);
# }
```

r[asm.abi-clobbers.must-specify]
当使用 `clobber_abi` 时，编译器不允许通用寄存器类别输出：所有输出必须指定显式寄存器。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
extern "C" fn foo(x: i32) -> i32 { 0 }

let z: i32;
// 必须使用显式寄存器以避免意外重叠。
unsafe {
    core::arch::asm!(
        "mov eax, {:e}",
        "call {}",
        out(reg) z,
        sym foo,
        clobber_abi("C")
    );
    // ERROR: asm with `clobber_abi` must specify explicit registers for outputs
}
assert_eq!(z, 0);
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.abi-clobbers.explicit-have-precedence]
显式寄存器输出优先于 `clobber_abi` 插入的隐式清除器：仅当寄存器未被用作输出时才会插入清除器。

r[asm.abi-clobbers.supported-abis]
以下 ABI 可与 `clobber_abi` 一起使用：

| 架构 | ABI 名称 | 被清除的寄存器 |
| ------------ | -------- | ------------------- |
| x86-32 | `"C"`, `"system"`, `"efiapi"`, `"cdecl"`, `"stdcall"`, `"fastcall"` | `ax`, `cx`, `dx`, `xmm[0-7]`, `mm[0-7]`, `k[0-7]`, `st([0-7])` |
| x86-64 | `"C"`, `"system"` (在 Windows 上), `"efiapi"`, `"win64"` | `ax`, `cx`, `dx`, `r[8-11]`, `xmm[0-31]`, `mm[0-7]`, `k[0-7]`, `st([0-7])`, `tmm[0-7]` |
| x86-64 | `"C"`, `"system"` (在非 Windows 上), `"sysv64"` | `ax`, `cx`, `dx`, `si`, `di`, `r[8-11]`, `xmm[0-31]`, `mm[0-7]`, `k[0-7]`, `st([0-7])`, `tmm[0-7]` |
| AArch64 | `"C"`, `"system"`, `"efiapi"` | `x[0-17]`, `x18`\*, `x30`, `v[0-31]`, `p[0-15]`, `ffr` |
| Arm64EC | `"C"`, `"system"` | `x[0-12]`, `x[15-17]`, `x30`, `v[0-15]` |
| ARM | `"C"`, `"system"`, `"efiapi"`, `"aapcs"` | `r[0-3]`, `r12`, `r14`, `s[0-15]`, `d[0-7]`, `d[16-31]` |
| RISC-V | `"C"`, `"system"`, `"efiapi"` | `x1`, `x[5-7]`, `x[10-17]`\*, `x[28-31]`\*, `f[0-7]`, `f[10-17]`, `f[28-31]`, `v[0-31]` |
| LoongArch | `"C"`, `"system"` | `$r1`, `$r[4-20]`, `$f[0-23]` |
| s390x | `"C"`, `"system"` | `r[0-5]`, `r14`, `f[0-7]`, `v[0-31]`, `a[2-15]` |

> [!NOTE]
> - 在 AArch64 上，`x18` 仅当在目标上不被视为保留寄存器时才包含在清除器列表中。
> - 在 RISC-V 上，`x[16-17]` 和 `x[28-31]` 仅当在目标上不被视为保留寄存器时才包含在清除器列表中。

随着架构获得新寄存器，每种 ABI 的清除寄存器列表都会在 rustc 中更新：这确保了当 LLVM 在其生成的代码中开始使用这些新寄存器时，`asm!` 清除器将继续保持正确。

r[asm.options]
## 选项

r[asm.options.supported-options]
标志用于进一步影响内联汇编代码的行为。目前定义了以下选项：

r[asm.options.supported-options.pure]
- `pure`：汇编代码没有副作用，必须最终返回，其输出仅依赖于其直接输入（即值本身，而非它们指向的内容）或从内存读取的值（除非也设置了 `nomem` 选项）。这允许编译器减少汇编代码的执行次数（例如将其提升到循环外），甚至如果输出未被使用则完全消除它。`pure` 选项必须与 `nomem` 或 `readonly` 选项结合使用，否则会发出编译时错误。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i32 = 0;
let z: i32;
// pure 可用于通过假设汇编没有副作用来进行优化
unsafe { core::arch::asm!("inc {}", inout(reg) x => z, options(pure, nomem)); }
assert_eq!(z, 1);
# }
```

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
let x: i32 = 0;
let z: i32;
// 必须满足 nomem 或 readonly，以指示是否允许读取内存
unsafe { core::arch::asm!("inc {}", inout(reg) x => z, options(pure)); }
// ERROR: the `pure` option must be combined with either `nomem` or `readonly`
assert_eq!(z, 0);
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.options.supported-options.nomem]
- `nomem`：汇编代码不从汇编代码外部可访问的任何内存读取或写入。这允许编译器在汇编代码执行期间在寄存器中缓存已修改的全局变量的值，因为它知道它们不会被汇编代码读取或写入。编译器还假设汇编代码不执行任何类型的与其他线程的同步，例如通过屏障。

<!-- no_run: This test has unpredictable or undefined behavior at runtime -->
```rust,no_run
# #[cfg(target_arch = "x86_64")] {
let mut x = 0i32;
let z: i32;
// 当指定了 `nomem` 时，从汇编中访问外部内存是不允许的
unsafe {
    core::arch::asm!("mov {val:e}, dword ptr [{ptr}]",
        ptr = in(reg) &mut x,
        val = lateout(reg) z,
        options(nomem)
    )
}

// 当指定了 `nomem` 时，从汇编中写入外部内存也是未定义行为
unsafe {
    core::arch::asm!("mov  dword ptr [{ptr}], {val:e}",
        ptr = in(reg) &mut x,
        val = in(reg) z,
        options(nomem)
    )
}
# }
```

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i32 = 0;
let z: i32;
// 但是，如果我们分配自己的内存，例如通过 `push`，仍然可以使用它
unsafe {
    core::arch::asm!("push {x}", "add qword ptr [rsp], 1", "pop {x}",
        x = inout(reg) x => z,
        options(nomem)
    );
}
assert_eq!(z, 1);
# }
```

r[asm.options.supported-options.readonly]
- `readonly`：汇编代码不写入汇编代码外部可访问的任何内存。这允许编译器在汇编代码执行期间在寄存器中缓存未修改的全局变量的值，因为它知道它们不会被汇编代码写入。编译器还假设此汇编代码不执行任何类型的与其他线程的同步，例如通过屏障。

<!-- no_run: This test has undefined behaviour at runtime -->
```rust,no_run
# #[cfg(target_arch = "x86_64")] {
let mut x = 0;
// 当指定了 `readonly` 时，我们不能修改外部内存
unsafe {
    core::arch::asm!("mov dword ptr[{}], 1", in(reg) &mut x, options(readonly))
}
# }
```

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64 = 0;
let z: i64;
// 不过，我们仍然可以从中读取
unsafe {
    core::arch::asm!("mov {x}, qword ptr [{x}]",
        x = inout(reg) &x => z,
        options(readonly)
    );
}
assert_eq!(z, 0);
# }
```

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i64 = 0;
let z: i64;
// 与 nomem 相同的例外情况也适用。
unsafe {
    core::arch::asm!("push {x}", "add qword ptr [rsp], 1", "pop {x}",
        x = inout(reg) x => z,
        options(readonly)
    );
}
assert_eq!(z, 1);
# }
```

r[asm.options.supported-options.preserves_flags]
- `preserves_flags`：汇编代码不修改标志寄存器（在下文规则中定义）。这允许编译器在内联汇编代码执行后避免重新计算条件标志。

r[asm.options.supported-options.noreturn]
- `noreturn`：汇编代码不会穿过结尾；如果穿过则是未定义行为。它仍然可以跳转到 `label` 块。如果任何 `label` 块返回单元类型，则 `asm!` 块将返回单元类型。否则它将返回 `!`（never 类型）。与调用不返回的函数一样，作用域内的局部变量在汇编代码执行之前不会被丢弃。

<!-- no_run: This test aborts at runtime -->
```rust,no_run
fn main() -> ! {
# #[cfg(target_arch = "x86_64")] {
    // 我们可以使用指令在 noreturn 块内捕获执行
    unsafe { core::arch::asm!("ud2", options(noreturn)); }
# }
# #[cfg(not(target_arch = "x86_64"))] panic!("no return");
}
```

<!-- no_run: Test has undefined behavior at runtime -->
```rust,no_run
# #[cfg(target_arch = "x86_64")] {
// 你有责任不穿过 noreturn asm 块的结尾
unsafe { core::arch::asm!("", options(noreturn)); }
# }
```

```rust
# #[cfg(target_arch = "x86_64")]
let _: () = unsafe {
    // 你仍然可以跳转到 `label` 块
    core::arch::asm!("jmp {}", label {
        println!();
    }, options(noreturn));
};
```

r[asm.options.supported-options.nostack]
- `nostack`：汇编代码不向栈推送数据，也不写入栈红色区域（如果目标支持）。如果*未*使用此选项，则编译器保证在汇编代码开始时栈指针已适当对齐（根据目标 ABI）以进行函数调用。

<!-- no_run: Test has undefined behavior at runtime -->
```rust,no_run
# #[cfg(target_arch = "x86_64")] {
// `push` 和 `pop` 在使用 nostack 时是未定义行为
unsafe { core::arch::asm!("push rax", "pop rax", options(nostack)); }
# }
```

r[asm.options.supported-options.att_syntax]
- `att_syntax`：此选项仅在 x86 上有效，并导致汇编器使用 GNU 汇编器的 `.att_syntax prefix` 模式。寄存器操作数在替换时会带有前导 `%`。

```rust
# #[cfg(target_arch = "x86_64")] {
let x: i32;
let y = 1i32;
// 这里我们需要使用 AT&T 语法。操作数的顺序是 src, dest
unsafe {
    core::arch::asm!("mov {y:e}, {x:e}",
        x = lateout(reg) x,
        y = in(reg) y,
        options(att_syntax)
    );
}
assert_eq!(x, y);
# }
```

r[asm.options.supported-options.raw]
- `raw`：这导致模板字符串被解析为原始汇编字符串，对 `{` 和 `}` 没有特殊处理。这主要在通过 `include_str!` 从外部文件包含原始汇编代码时很有用。

r[asm.options.checks]
编译器对选项执行一些额外的检查：

r[asm.options.checks.mutually-exclusive]
- `nomem` 和 `readonly` 选项是互斥的：同时指定两者是编译时错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// nomem 比 readonly 严格得多，它们不能一起指定
unsafe { core::arch::asm!("", options(nomem, readonly)); }
// ERROR: the `nomem` and `readonly` options are mutually exclusive
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.options.checks.pure]
- 在没有输出或仅丢弃输出（`_`）的 asm 块上指定 `pure` 是编译时错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
// pure 块需要至少一个输出
unsafe { core::arch::asm!("", options(pure)); }
// ERROR: asm with the `pure` option must have at least one output
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.options.checks.noreturn]
- 在有输出且没有标签的 asm 块上指定 `noreturn` 是编译时错误。

```rust,compile_fail
# #[cfg(target_arch = "x86_64")] {
let z: i32;
// noreturn 不能有输出
unsafe { core::arch::asm!("mov {:e}, 1", out(reg) z, options(noreturn)); }
// ERROR: asm outputs are not allowed with the `noreturn` option
# }
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.options.checks.label-with-outputs]
- 在有输出的 asm 块中有任何 `label` 块是编译时错误。

r[asm.options.naked_asm-restriction]
`naked_asm!` 仅支持 `att_syntax` 和 `raw` 选项。其余选项没有意义，因为内联汇编定义了整个函数体。

r[asm.options.global_asm-restriction]
`global_asm!` 仅支持 `att_syntax` 和 `raw` 选项。其余选项对于全局作用域的内联汇编没有意义。

```rust,compile_fail
# fn main() {}
# #[cfg(target_arch = "x86_64")]
// nomem 在 global_asm! 上毫无用处
core::arch::global_asm!("", options(nomem));
# #[cfg(not(target_arch = "x86_64"))] core::compile_error!("Test not supported on this arch");
```

r[asm.rules]
## 内联汇编的规则 {#rules-for-inline-assembly}

r[asm.rules.intro]
为避免未定义行为，在使用函数作用域内联汇编（`asm!`）时必须遵循以下规则：

r[asm.rules.reg-not-input]
- 任何未指定为输入的寄存器在进入汇编代码时将包含未定义的值。
  - 内联汇编语境中的"未定义值"意味着该寄存器可以（非确定性地）具有架构允许的任何可能值之一。值得注意的是，它与 LLVM 的 `undef` 不同，后者每次读取时可能具有不同的值（因为汇编代码中不存在这样的概念）。

r[asm.rules.reg-not-output]
- 任何未指定为输出的寄存器在退出汇编代码时必须具有与进入时相同的值，否则行为是未定义的。
  - 这仅适用于可以指定为输入或输出的寄存器。其他寄存器遵循特定于目标的规则。
  - 请注意，`lateout` 可能被分配到与 `in` 相同的寄存器，在这种情况下此规则不适用。但是，代码不应依赖于此，因为它取决于寄存器分配的结果。

r[asm.rules.unwind]
- 如果执行从汇编代码中展开，行为是未定义的。
  - 如果汇编代码调用一个函数，然后该函数发生展开，这也适用。

r[asm.rules.mem-same-as-ffi]
- 汇编代码允许读写的内存位置集合与 FFI 函数允许的相同。
  - 如果设置了 `readonly` 选项，则仅允许内存读取。
  - 如果设置了 `nomem` 选项，则不允许读取或写入内存。
  - 这些规则不适用于汇编代码私有的内存，例如在其中分配的栈空间。

r[asm.rules.black-box]
- 编译器不能假设汇编代码中的指令就是最终实际执行的指令。
  - 这实际上意味着编译器必须将汇编代码视为黑盒，只考虑接口规范，而非指令本身。
  - 通过特定于目标的机制允许运行时代码补丁。
  - 然而，不能保证源代码中的每个汇编代码块都直接对应于目标文件中的单个指令实例；编译器可以自由地复制或去重 `asm!` 块中的汇编代码。

r[asm.rules.stack-below-sp]
- 除非设置了 `nostack` 选项，否则汇编代码允许使用栈指针以下的栈空间。
  - 在进入汇编代码时，栈指针保证已适当对齐（根据目标 ABI）以进行函数调用。
  - 你有责任确保不会溢出栈（例如使用栈探测确保碰到保护页）。
  - 你应根据目标 ABI 的要求在分配栈内存时调整栈指针。
  - 在离开汇编代码之前，栈指针必须恢复到其原始值。

r[asm.rules.stack-above-sp]
- 除非设置了 `nostack` 选项，否则当目标 ABI 要求在调用者的帧中存储某些值（例如在 PowerPC64 上保存 `lr` 时），汇编代码允许修改调用者的栈帧。

r[asm.rules.noreturn]
- 如果设置了 `noreturn` 选项，则执行穿过汇编代码的结尾是未定义行为。

r[asm.rules.pure]
- 如果设置了 `pure` 选项，则如果 `asm!` 除了其直接输出之外还有副作用，行为是未定义的。如果两次执行具有相同输入的 `asm!` 代码产生不同的输出，行为也是未定义的。
  - 当与 `nomem` 选项一起使用时，"输入"仅指 `asm!` 的直接输入。
  - 当与 `readonly` 选项一起使用时，"输入"包括汇编代码的直接输入以及允许读取的任何内存。

r[asm.rules.preserved-registers]
- 如果设置了 `preserves_flags` 选项，则以下标志寄存器必须在退出汇编代码时恢复：
  - x86
    - `EFLAGS` 中的状态标志（CF, PF, AF, ZF, SF, OF）。
    - 浮点状态字（全部）。
    - `MXCSR` 中的浮点异常标志（PE, UE, OE, ZE, DE, IE）。
  - ARM
    - `CPSR` 中的条件标志（N, Z, C, V）
    - `CPSR` 中的饱和标志（Q）
    - `CPSR` 中的大于等于标志（GE）。
    - `FPSCR` 中的条件标志（N, Z, C, V）
    - `FPSCR` 中的饱和标志（QC）
    - `FPSCR` 中的浮点异常标志（IDC, IXC, UFC, OFC, DZC, IOC）。
  - AArch64 和 Arm64EC
    - 条件标志（`NZCV` 寄存器）。
    - 浮点状态（`FPSR` 寄存器）。
  - RISC-V
    - `fcsr` 中的浮点异常标志（`fflags`）。
    - 向量扩展状态（`vtype`, `vl`, `vxsat` 和 `vxrm`）。
  - LoongArch
    - `$fcc[0-7]` 中的浮点条件标志。
  - PowerPC/PowerPC64
    - `fpscr` 中的浮点状态和粘滞位（DRN, VE, OE, UE, ZE, XE, NI 或 RN 以外的任何字段）。
    - `vscr` 中的向量状态和粘滞位（NJ 以外的任何字段）。
  - PowerPC SPE
    - `spefscr` 的粘滞位和状态位（FINXE, FINVE, FDBZE, FUNFE, FOVFE 或 FRMC 以外的任何字段）。
  - s390x
    - 条件码寄存器 `cc`。

r[asm.rules.x86-df]
- 在 x86 上，方向标志（`EFLAGS` 中的 DF）在进入汇编代码时是清除的，并且必须在退出时也是清除的。
  - 退出汇编代码时如果方向标志被设置，则行为是未定义的。

r[asm.rules.x86-x87]
- 在 x86 上，x87 浮点寄存器栈必须保持不变，除非所有 `st([0-7])` 寄存器都已使用 `out("st(0)") _, out("st(1)") _, ...` 标记为清除。
  - 如果所有 x87 寄存器都被清除，则 x87 寄存器栈在进入汇编代码时保证为空。汇编代码必须确保在退出汇编代码时 x87 寄存器栈也为空。

```rust
# #[cfg(target_arch = "x86_64")]
pub fn fadd(x: f64, y: f64) -> f64 {
  let mut out = 0f64;
  let mut top = 0u16;
  // 如果清除了整个 x87 栈，我们可以用 x87 做复杂的事情
  unsafe { core::arch::asm!(
    "fld qword ptr [{x}]",
    "fld qword ptr [{y}])",
    "faddp",
    "fstp qword ptr [{out}]",
    "xor eax, eax",
    "fstsw ax",
    "shl eax, 11",
    x = in(reg) &x,
    y = in(reg) &y,
    out = in(reg) &mut out,
    out("st(0)") _, out("st(1)") _, out("st(2)") _, out("st(3)") _,
    out("st(4)") _, out("st(5)") _, out("st(6)") _, out("st(7)") _,
    out("eax") top
  );}

  assert_eq!(top & 0x7, 0);
  out
}

pub fn main() {
# #[cfg(target_arch = "x86_64")]{
  assert_eq!(fadd(1.0, 1.0), 2.0);
# }
}
```

r[asm.rules.arm64ec]
- 在 arm64ec 上，在调用函数时，[带有适当转换块的调用检查器](https://learn.microsoft.com/en-us/windows/arm/arm64ec-abi#authoring-arm64ec-in-assembly)是强制性的。

r[asm.rules.only-on-exit]
- 恢复栈指针和非输出寄存器到其原始值的要求仅在退出汇编代码时适用。
  - 这意味着不穿过结尾且不跳转到任何 `label` 块的汇编代码，即使未标记为 `noreturn`，也不需要保留这些寄存器。
  - 当返回到与你进入时不同的 `asm!` 块的汇编代码时（例如用于上下文切换），这些寄存器必须包含你进入你正在*退出*的 `asm!` 块时它们具有的值。
    - 你不能退出来曾进入的 `asm!` 块的汇编代码。你也不能退出其汇编代码已经退出过的 `asm!` 块的汇编代码（而不先再次进入它）。
    - 你有责任切换任何特定于目标的状态（例如线程局部存储、栈边界）。
    - 你不能从一个 `asm!` 块中的地址跳转到另一个 `asm!` 块中的地址，即使在同一函数或块内，除非将它们上下文视为可能不同且需要上下文切换。你不能假设这些上下文中的任何特定值（例如当前栈指针或栈指针以下的临时值）在两个 `asm!` 块之间保持不变。
    - 你可以访问的内存位置集合是你进入和退出的 `asm!` 块所允许的那些集合的交集。

r[asm.rules.not-successive]
- 你不能假设源代码中相邻的两个 `asm!` 块（即使它们之间没有任何其他代码）最终会在二进制中连续排列，中间没有任何其他指令。

r[asm.rules.not-exactly-once]
- 你不能假设一个 `asm!` 块在输出二进制中恰好出现一次。编译器可以实例化 `asm!` 块的多个副本，例如当包含它的函数在多个位置被内联时。

r[asm.rules.x86-prefix-restriction]
- 在 x86 上，内联汇编不能以会应用于编译器生成的指令的指令前缀（如 `LOCK`）结尾。
  - 由于内联汇编的编译方式，编译器目前无法检测到这一点，但未来可能会捕获并拒绝这种情况。

r[asm.rules.preserves_flags]
> [!NOTE]
> 作为一般规则，`preserves_flags` 涵盖的标志是那些在执行函数调用时*不*被保留的标志。

r[asm.naked-rules]
## 裸内联汇编的规则

r[asm.naked-rules.intro]
为避免未定义行为，在裸函数中使用函数作用域内联汇编（`naked_asm!`）时必须遵循以下规则：

r[asm.naked-rules.reg-not-input]
- 任何未根据调用约定和函数签名用于函数输入的寄存器在进入 `naked_asm!` 块时将包含未定义的值。
  - 内联汇编语境中的"未定义值"意味着该寄存器可以（非确定性地）具有架构允许的任何可能值之一。值得注意的是，它与 LLVM 的 `undef` 不同，后者每次读取时可能具有不同的值（因为汇编代码中不存在这样的概念）。

r[asm.naked-rules.callee-saved-registers]
- 所有被调用者保存的寄存器在返回时必须具有与进入时相同的值。

r[asm.naked-rules.caller-saved-registers]
- 调用者保存的寄存器可以自由使用。

r[asm.naked-rules.noreturn]
- 如果执行穿过汇编代码的结尾，行为是未定义的。
  - 汇编代码中的每条路径都期望以返回指令终止或发散。

r[asm.naked-rules.mem-same-as-ffi]
- 汇编代码允许读写的内存位置集合与 FFI 函数允许的相同。

r[asm.naked-rules.black-box]
- 编译器不能假设 `naked_asm!` 块中的指令就是最终实际执行的指令。
  - 这实际上意味着编译器必须将 `naked_asm!` 视为黑盒，只考虑接口规范，而非指令本身。
  - 通过特定于目标的机制允许运行时代码补丁。

r[asm.naked-rules.unwind]
- 允许从 `naked_asm!` 块中展开。
  - 为获得正确行为，必须使用适当的发出展开元数据的汇编器指令。

```rust
# #[cfg(target_arch = "x86_64")] {
#[unsafe(naked)]
extern "sysv64-unwind" fn unwinding_naked() {
    core::arch::naked_asm!(
        // "CFI" 在此代表 "call frame information"（调用帧信息）。
        ".cfi_startproc",
        // CFA（规范帧地址）是 `call` 之前的 `rsp` 值，
        // 即在返回地址 `rip` 被推入 `rsp` 之前，
        // 因此在函数进入时（在 `rip` 被推入之后）
        // 它比 `rsp` 在内存中高八个字节。
        //
        // 这是默认值，所以我们不必写它。
        //".cfi_def_cfa rsp, 8",
        //
        // 传统的做法是保留基指针，所以我们照做。
        "push rbp",
        // 由于我们现在已将栈向下扩展了 8 个字节，
        // 需要将 `rsp` 到 CFA 的偏移量再调整 8 个字节。
        ".cfi_adjust_cfa_offset 8",
        // 我们还标注了调用者的 `rbp` 值存储的位置，
        // 相对于 CFA，以便在展开到调用者时能够找到它，
        // 以防我们需要它来计算调用者相对于它的 CFA。
        //
        // 这里，我们将调用者的 `rbp` 存储在 CFA 下方 16 字节处。
        // 即，从 CFA 开始，首先是 `rip`（从 CFA 下方 8 字节开始一直延伸到 CFA），
        // 然后是我们刚刚推送的调用者的 `rbp`。
        ".cfi_offset rbp, -16",
        // 按照传统，我们将基指针设置为栈指针的值。
        // 这样，基指针在整个函数体中保持不变。
        "mov rbp, rsp",
        // 现在我们可以通过基指针来跟踪 CFA 的偏移量。
        // 这意味着在结束之前我们不需要做进一步调整，因为我们不会改变 `rbp`。
        ".cfi_def_cfa_register rbp",
        // 现在我们可以调用一个可能 panic 的函数。
        "call {f}",
        // 返回时，我们恢复 `rbp` 为自身返回做准备。
        "pop rbp",
        // 既然我们已经恢复了 `rbp`，必须根据 `rsp` 重新指定 CFA 的偏移量。
        ".cfi_def_cfa rsp, 8",
        // 现在我们可以返回了。
        "ret",
        ".cfi_endproc",
        f = sym may_panic,
    )
}

extern "sysv64-unwind" fn may_panic() {
    panic!("unwind");
}
# }
```

> [!NOTE]
>
> 有关上述 `cfi` 汇编器指令的更多信息，请参阅以下资源：
>
> - [Using `as` - CFI directives](https://sourceware.org/binutils/docs/as/CFI-directives.html)
> - [DWARF Debugging Information Format Version 5](https://dwarfstd.org/doc/DWARF5.pdf)
> - [ImperialViolet - CFI directives in assembly files](https://www.imperialviolet.org/2017/01/18/cfi.html)

r[asm.validity]
### 正确性与有效性

r[asm.validity.necessary-but-not-sufficient]
除上述所有规则外，传递给 `asm!` 的字符串参数最终必须---在所有其他参数被求值、格式化被执行且操作数被翻译之后---成为对目标架构既语法正确又语义有效的汇编代码。格式化规则允许编译器生成语法正确的汇编代码。有关操作数的规则允许将 Rust 操作数有效地翻译进出汇编代码。遵守这些规则是必要的，但不足以使最终展开的汇编代码既正确又有效。例如：

- 参数可能被放置在格式化后语法上不正确的位置
- 一条指令可能被正确编写，但被赋予了在架构上无效的操作数
- 一条架构上未指定的指令可能被汇编成未定义的代码
- 一组各自正确且有效的指令，如果被紧接放置可能会引起未定义行为

r[asm.validity.non-exhaustive]
因此，这些规则是*非穷尽的*。编译器不需要检查初始字符串的正确性和有效性，也不需要检查最终生成的汇编的正确性和有效性。汇编器可能检查正确性和有效性，但不要求这样做。在使用 `asm!` 时，一个排版错误可能就足以使程序不健全，而汇编的规则可能包含数千页的架构参考手册。程序员应谨慎行事，因为调用此 `unsafe` 功能意味着承担不违反编译器或架构规则的责任。

r[asm.directives]
### 指令支持 {#directives-support}

r[asm.directives.subset-supported]
内联汇编支持 GNU AS 和 LLVM 内部汇编器都支持的指令子集，如下所示。使用其他指令的结果是特定于汇编器的（可能引起错误，也可能按原样接受）。

r[asm.directives.stateful]
如果内联汇编包含任何修改后续汇编处理方式的"有状态"指令，则汇编代码必须在内联汇编结束之前撤销这些指令的影响。

r[asm.directives.supported-directives]
以下指令保证受汇编器支持：

- `.2byte`
- `.4byte`
- `.8byte`
- `.align`
- `.alt_entry`
- `.ascii`
- `.asciz`
- `.balign`
- `.balignl`
- `.balignw`
- `.bss`
- `.byte`
- `.comm`
- `.data`
- `.def`
- `.double`
- `.endef`
- `.equ`
- `.equiv`
- `.eqv`
- `.fill`
- `.float`
- `.global`
- `.globl`
- `.inst`
- `.insn`
- `.lcomm`
- `.long`
- `.octa`
- `.option`
- `.p2align`
- `.popsection`
- `.private_extern`
- `.pushsection`
- `.quad`
- `.scl`
- `.section`
- `.set`
- `.short`
- `.size`
- `.skip`
- `.sleb128`
- `.space`
- `.string`
- `.text`
- `.type`
- `.uleb128`
- `.word`

```rust
# #[cfg(target_arch = "x86_64")] {
let bytes: *const u8;
let len: usize;
unsafe {
    core::arch::asm!(
        "jmp 3f", "2: .ascii \"Hello World!\"",
        "3: lea {bytes}, [2b+rip]",
        "mov {len}, 12",
        bytes = out(reg) bytes,
        len = out(reg) len
    );
}

let s = unsafe { core::str::from_utf8_unchecked(core::slice::from_raw_parts(bytes, len)) };

assert_eq!(s, "Hello World!");
# }
```

r[asm.target-specific-directives]
#### 目标特定指令支持

r[asm.target-specific-directives.dwarf-unwinding]
##### DWARF 展开

以下指令在支持 DWARF 展开信息的 ELF 目标上受支持：

- `.cfi_adjust_cfa_offset`
- `.cfi_def_cfa`
- `.cfi_def_cfa_offset`
- `.cfi_def_cfa_register`
- `.cfi_endproc`
- `.cfi_escape`
- `.cfi_lsda`
- `.cfi_offset`
- `.cfi_personality`
- `.cfi_register`
- `.cfi_rel_offset`
- `.cfi_remember_state`
- `.cfi_restore`
- `.cfi_restore_state`
- `.cfi_return_column`
- `.cfi_same_value`
- `.cfi_sections`
- `.cfi_signal_frame`
- `.cfi_startproc`
- `.cfi_undefined`
- `.cfi_window_save`

r[asm.target-specific-directives.structured-exception-handling]
##### 结构化异常处理

在具有结构化异常处理的目标上，以下附加指令保证受支持：

- `.seh_endproc`
- `.seh_endprologue`
- `.seh_proc`
- `.seh_pushreg`
- `.seh_savereg`
- `.seh_setframe`
- `.seh_stackalloc`

r[asm.target-specific-directives.x86]
##### x86（32 位和 64 位）

在 x86 目标（32 位和 64 位）上，以下附加指令保证受支持：
- `.nops`
- `.code16`
- `.code32`
- `.code64`

仅当在退出汇编代码之前将状态重置为默认值时，才支持使用 `.code16`、`.code32` 和 `.code64` 指令。32 位 x86 默认使用 `.code32`，x86_64 默认使用 `.code64`。

r[asm.target-specific-directives.arm-32-bit]
##### ARM（32 位）

在 ARM 上，以下附加指令保证受支持：

- `.even`
- `.fnstart`
- `.fnend`
- `.save`
- `.movsp`
- `.code`
- `.thumb`
- `.thumb_func`
