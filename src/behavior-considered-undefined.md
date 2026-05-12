r[undefined]
# 被视为未定义的行为

r[undefined.intro]
如果 Rust 代码表现出以下列表中的任何行为，则该代码是错误的。这包括 `unsafe` 块和 `unsafe` 函数中的代码。`unsafe` 仅仅意味着避免未定义行为是程序员的责任；它不改变 Rust 程序绝不能导致未定义行为这一事实。

r[undefined.soundness]
在编写 `unsafe` 代码时，程序员有责任确保任何与该 `unsafe` 代码交互的安全代码不会触发这些行为。对于任何安全客户端都能满足这一属性的 `unsafe` 代码称为*健全的*（sound）；如果 `unsafe` 代码可能被安全代码误用而表现出未定义行为，则称为*不健全的*（unsound）。

> [!WARNING]
> 以下列表并非详尽无遗；它可能会增加或减少。对于 `unsafe` 代码中允许和不允许的内容，Rust 没有形式化的语义模型，因此可能有更多行为被视为不安全。我们也保留在未来使该列表中的某些行为变为已定义的权利。换句话说，此列表并不表示任何内容*一定*在所有未来的 Rust 版本中都始终是未定义的（但我们可能在未来对某些列表项做出此类承诺）。
>
> 在编写 `unsafe` 代码之前，请阅读 [Rustonomicon]。

r[undefined.race]
* 数据竞争。

r[undefined.pointer-access]
* 访问（加载或存储）一个[悬垂][dangling]或[基于未对齐指针][based on a misaligned pointer]的位置。

r[undefined.place-projection]
* 执行违反[边界内指针算术](pointer#method.offset)要求的位置投影。位置投影是[字段表达式][project-field]、[元组索引表达式][project-tuple]或[数组/切片索引表达式][project-slice]。

r[undefined.alias]
* 违反指针别名规则。确切的别名规则尚未确定，但以下是一般原则的概述：
  `&T` 必须指向在其存活期间不被修改的内存（除了 [`UnsafeCell<U>`] 内部的数据），
  而 `&mut T` 必须指向在其存活期间不被任何非派生自该引用的指针读取或写入的内存，并且在此期间没有其他引用指向该内存。
  就这些规则而言，`Box<T>` 被视作类似于 `&'static mut T`。
  确切的存活时长未指定，但存在一些界限：
  * 对于引用，存活时长以借用检查器分配的语法生命周期为上界；它不能比该生命周期存活*更长时间*。
  * 每次引用或 box 被解引用或重新借用时，它被视为存活。
  * 每次引用或 box 被传递到函数或从函数返回时，它被视为存活。
  * 当引用（但不是 `Box`！）被传递到函数时，它至少在该函数调用期间是存活的，除非 `&T` 包含 [`UnsafeCell<U>`] 则再次例外。

  当这些类型的值以复合类型的（嵌套）字段形式传递时，所有这些也适用，但通过指针间接传递时则不适用。

r[undefined.immutable]
* 修改不可变字节。
  通过[常量提升][const-promoted]表达式可达的所有字节都是不可变的，以及在 `static` 和 `const` 初始化器中已被[生命周期延长][lifetime-extended]到 `'static` 的借用可达的字节也是不可变的。
  不可变绑定或不可变 `static` 所拥有的字节是不可变的，除非这些字节属于 [`UnsafeCell<U>`] 的一部分。

  此外，共享引用[指向][pointed to]的字节，包括通过其他引用（共享的和可变的）和 `Box` 的传递，都是不可变的；传递性包括存储在复合类型字段中的那些引用。

  变更是任何写入超过 0 个字节且与任何相关字节重叠的操作（即使该写入不改变内存内容）。

r[undefined.intrinsic]
* 通过编译器内部函数调用未定义行为。

r[undefined.target-feature]
* 执行使用了当前平台不支持的平台特性编译的代码（参见 [`target_feature`]），*除非*平台明确文档说明这是安全的。

r[undefined.call]
* 使用错误的[调用 ABI][abi] 调用函数，或者展开（unwind）越过不允许展开的栈帧（例如，调用了一个以 `"C"` 函数或函数指针形式导入或 transmute 的 `"C-unwind"` 函数）。

r[undefined.invalid]
* 产生[无效值][invalid-values]。"产生"一个值发生在任何将值赋值给位置、从位置读取值、将值传递给函数/原始操作或从函数/原始操作返回值的时候。

r[undefined.asm]
* 错误地使用内联汇编。更多细节请参考编写使用内联汇编的代码时应遵循的[规则][rules]。

r[undefined.runtime]
* 违反 Rust 运行时的假设。Rust 运行时的大多数假设目前没有明确文档化。
  * 有关与展开（unwinding）特别相关的假设，请参见 [panic 文档][unwinding-ffi]。
  * 运行时假设 Rust 栈帧在未执行该栈帧拥有的局部变量的析构函数的情况下不会被释放。此假设可能被 C 函数如 `longjmp` 违反。

> [!NOTE]
> 未定义行为影响整个程序。例如，调用 C 中表现出 C 的未定义行为的函数意味着你的整个程序包含未定义行为，这也会影响 Rust 代码。反之亦然，Rust 中的未定义行为可能对通过任何 FFI 调用其他语言执行的代码产生不利影响。

r[undefined.pointed-to]
## 指向的字节 {#pointed-to-bytes}

指针或引用"指向"的字节范围由指针值和指向对象类型的大小（使用 `size_of_val`）确定。

r[undefined.misaligned]
## 基于未对齐指针的位置
[based on a misaligned pointer]: #基于未对齐指针的位置

r[undefined.misaligned.ptr]
如果一个位置在位置计算期间最后一次 `*` 投影是在一个对其类型未对齐的指针上执行的，则称该位置"基于未对齐指针"。（如果位置表达式中没有 `*` 投影，那么访问的是局部变量或 `static` 的字段，rustc 将保证正确的对齐。如果存在多个 `*` 投影，则每个投影都会从内存中加载待解引用的指针本身，每个这样的加载都受对齐约束。请注意，由于自动解引用，一些 `*` 投影可能在表面 Rust 语法中被省略；我们在此考虑的是完全展开的位置表达式。）

例如，如果 `ptr` 具有类型 `*const S`，其中 `S` 的对齐为 8，则 `ptr` 必须 8 对齐，否则 `(*ptr).f` 就是"基于未对齐指针"。即使字段 `f` 的类型是 `u8`（即对齐为 1 的类型），这也成立。换句话说，对齐要求源于被解引用的指针的类型，而*不是*被访问字段的类型。

r[undefined.misaligned.load-store]
请注意，基于未对齐指针的位置仅在对其进行加载或存储时才会导致未定义行为。

r[undefined.misaligned.raw]
允许对这种位置使用 `&raw const`/`&raw mut`。

r[undefined.misaligned.reference]
对位置使用 `&`/`&mut` 要求字段类型的对齐（否则程序将"产生无效值"），这通常比基于对齐指针的要求更宽松。

r[undefined.misaligned.packed]
在字段类型可能比包含它的类型更严格对齐的情况下，即 `repr(packed)`，获取引用将导致编译器错误。这意味着基于对齐指针总是足以确保新引用是对齐的，但不总是必需的。

r[undefined.dangling]
## 悬垂指针
[dangling]: #悬垂指针

r[undefined.dangling.def]
如果引用/指针所[指向][points to]的字节并非全部属于同一个存活分配（因此特别是它们必须全部属于*某个*分配），则该引用/指针是"悬垂的"。

r[undefined.dangling.zero-size]
如果[大小为零][zero-sized]，则指针平凡地永远不会"悬垂"（即使它是空指针）。

r[undefined.dangling.dynamic-size]
请注意，动态大小类型（如切片和字符串）指向它们的整个范围，因此长度元数据绝不能太大是重要的。

r[undefined.dangling.alloc-limit]
特别是，Rust 值的动态大小（由 `size_of_val` 确定）绝不能超过 `isize::MAX`，因为单个分配不可能大于 `isize::MAX`。

r[undefined.validity]
## 无效值 {#invalid-values}
[invalid-values]: #invalid-values

r[undefined.validity.def]
Rust 编译器假设程序执行期间产生的所有值都是"有效的"，因此产生无效值即构成即时 UB。

一个值是否有效取决于类型：

r[undefined.validity.bool]
* [`bool`] 值必须是 `false`（`0`）或 `true`（`1`）。

r[undefined.validity.fn-pointer]
* `fn` 指针值必须非空。

r[undefined.validity.char]
* `char` 值不能是代理项（surrogate）（即不能位于 `0xD800..=0xDFFF` 范围内）且必须等于或小于 `char::MAX`。

r[undefined.validity.never]
* `!` 值绝不能存在。

r[undefined.validity.scalar]
* 整数（`i*`/`u*`）、浮点值（`f*`）或原始指针必须已初始化，即不能从未初始化内存获取。

r[undefined.validity.str]
* `str` 值被视为 `[u8]`，即必须已初始化。

r[undefined.validity.enum]
* `enum` 必须具有有效的判别值，且该判别值所示的变体的所有字段在其各自类型下必须有效。

r[undefined.validity.struct]
* `struct`、元组和数组要求所有字段/元素在其各自类型下有效。

r[undefined.validity.union]
* 对于 `union`，确切的有效性要求尚未确定。
  显然，所有可以完全在安全代码中创建的值都是有效的。
  如果联合体有一个[零大小][zero-sized]字段，那么每个可能的值都是有效的。
  更多细节[仍在讨论中](https://github.com/rust-lang/unsafe-code-guidelines/issues/438)。

r[undefined.validity.reference-box]
* 引用或 [`Box<T>`] 必须对齐且非空，不能[悬垂][dangling]，并且必须指向一个有效值
  （对于动态大小类型，使用由元数据确定的指向对象的实际动态类型）。
  请注意最后一点（关于指向有效值）仍然存在一些争议。

r[undefined.validity.wide]
* 宽引用、[`Box<T>`] 或原始指针的元数据必须与无大小尾部的类型匹配：
  * `dyn Trait` 元数据必须是指向编译器生成的 `Trait` 虚表的指针。
    （对于原始指针，此要求仍存在一些争议。）
  * 切片（`[T]`）元数据必须是有效的 `usize`。
    此外，对于宽引用和 [`Box<T>`]，如果切片元数据使得指向值总大小大于 `isize::MAX`，则是无效的。

r[undefined.validity.valid-range]
* 如果一个类型有自定义的有效值范围，那么有效值必须在该范围内。
  在标准库中，这影响 [`NonNull<T>`] 和 [`NonZero<T>`]。

  > [!NOTE]
  > `rustc` 通过不稳定的 `rustc_layout_scalar_valid_range_*` 属性实现这一点。

r[undefined.validity.const-provenance]
* **在 [const 上下文][const contexts] 中**：除了上述内容外，在常量求值期间还适用与来源（provenance）相关的进一步要求。任何持有纯整数数据的值（`i*`/`u*`/`f*` 类型以及 `bool` 和 `char`、枚举判别值和切片元数据）不得携带任何来源。任何持有指针数据的值（引用、原始指针、函数指针和 `dyn Trait` 元数据）必须要么不携带来源，要么所有字节必须是同一原始指针值的片段且顺序正确。

  这意味着将指针（引用、原始指针或函数指针）transmute 或以其他方式重新解释为非指针类型（如整数）是未定义行为，如果该指针具有来源的话。

  > [!EXAMPLE]
  > 以下所有情况都是 UB：
  >
  > ```rust,compile_fail
  > # use core::mem::MaybeUninit;
  > # use core::ptr;
  > // 我们不能将有来源的指针重新解释为整数，
  > // 因为这样整数的字节将具有来源。
  > const _: usize = {
  >     let ptr = &0;
  >     unsafe { (&raw const ptr as *const usize).read() }
  > };
  >
  > // 我们不能重新排列有来源的指针的字节，
  > // 然后将它们解释为引用，因为这样的话持有指针数据的值
  > // 将具有顺序错误的指针片段。
  > const _: &i32 = {
  >     let mut ptr = &0;
  >     let ptr_bytes = &raw mut ptr as *mut MaybeUninit::<u8>;
  >     unsafe { ptr::swap(ptr_bytes.add(1), ptr_bytes.add(2)) };
  >     ptr
  > };
  > ```

r[undefined.validity.undef]
**注意：** 未初始化内存对于任何具有受限有效值集合的类型也是隐式无效的。换句话说，允许读取未初始化内存的唯一情况是在 `union` 内部和"填充"（类型的字段之间的间隙）中。

[`bool`]: types/boolean.md
[`const`]: items/constant-items.md
[abi]: items/external-blocks.md#abi
[const contexts]: const-eval.const-context
[`target_feature`]: attributes/codegen.md#the-target_feature-attribute
[`UnsafeCell<U>`]: std::cell::UnsafeCell
[Rustonomicon]: ../nomicon/index.html
[`NonNull<T>`]: core::ptr::NonNull
[`NonZero<T>`]: core::num::NonZero
[place expression context]: expressions.md#place-expressions-and-value-expressions
[rules]: inline-assembly.md#rules-for-inline-assembly
[points to]: #pointed-to-bytes
[pointed to]: #pointed-to-bytes
[project-field]: expressions/field-expr.md
[project-tuple]: expressions/tuple-expr.md#tuple-indexing-expressions
[project-slice]: expressions/array-expr.md#array-and-slice-indexing-expressions
[unwinding-ffi]: panic.md#unwinding-across-ffi-boundaries
[const-promoted]: destructors.md#constant-promotion
[lifetime-extended]: destructors.md#temporary-lifetime-extension
[zero-sized]: glossary.zst
