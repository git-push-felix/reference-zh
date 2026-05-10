r[expr.operator]
# 运算符表达式

r[expr.operator.syntax]
```grammar,expressions
OperatorExpression ->
      BorrowExpression
    | DereferenceExpression
    | TryPropagationExpression
    | NegationExpression
    | ArithmeticOrLogicalExpression
    | ComparisonExpression
    | LazyBooleanExpression
    | TypeCastExpression
    | AssignmentExpression
    | CompoundAssignmentExpression
```

r[expr.operator.intro]
运算符由 Rust 语言为内置类型定义。

r[expr.operator.trait]
以下许多运算符也可以通过使用 `std::ops` 或 `std::cmp` 中的 trait 进行重载。

r[expr.operator.int-overflow]
## 溢出

r[expr.operator.int-overflow.intro]
整数运算符在调试模式下编译时，溢出时会 panic。`-C debug-assertions` 和 `-C overflow-checks` 编译器标志可用于更直接地控制此行为。以下情况被视为溢出：

r[expr.operator.int-overflow.binary-arith]
* 当 `+`、`*` 或二元 `-` 创建的值大于可存储的最大值，或小于可存储的最小值时。

r[expr.operator.int-overflow.unary-neg]
* 将一元 `-` 应用于任何有符号整数类型的最负值，除非操作数是[字面量表达式][literal expression]（或单独出现在一个或多个[分组表达式][grouped expression]内部的字面量表达式）。

r[expr.operator.int-overflow.div]
* 使用 `/` 或 `%`，其中左侧参数是有符号整数类型的最小整数，右侧参数是 `-1`。由于历史原因，即使禁用 `-C overflow-checks` 也会执行这些检查。

r[expr.operator.int-overflow.shift]
* 使用 `<<` 或 `>>`，其中右侧参数大于或等于左侧参数类型中的位数，或者为负数。

> [!NOTE]
> 一元 `-` 后字面量表达式的例外意味着像 `-128_i8` 或 `let j: i8 = -(128)` 这样的形式永远不会导致 panic，并且具有预期的值 -128。
>
> 在这些情况下，字面量表达式已经具有其类型的最负值（例如，`128_i8` 的值为 -128），因为根据[整数字面量表达式][literal expression]中的描述，整数字面量会被截断到其类型。
>
> 由于二进制补码溢出惯例，对这些最负值的取反操作保持值不变。
>
> 在 `rustc` 中，这些最负表达式也会被 `overflowing_literals` lint 检查忽略。

r[expr.operator.borrow]
## 借用运算符

r[expr.operator.borrow.syntax]
```grammar,expressions
BorrowExpression ->
      (`&`|`&&`) Expression
    | (`&`|`&&`) `mut` Expression
    | (`&`|`&&`) `raw` `const` Expression
    | (`&`|`&&`) `raw` `mut` Expression
```

r[expr.operator.borrow.intro]
`&`（共享借用）和 `&mut`（可变借用）运算符是一元前缀运算符。

r[expr.operator.borrow.result]
当应用于[位置表达式][place expression]时，此表达式产生一个指向该值所指位置的引用（指针）。

r[expr.operator.borrow.lifetime]
该内存位置也被置于借用状态，持续时间为引用的生命周期。对于共享借用（`&`），这意味着该位置不能被修改，但可以被读取或再次共享。对于可变借用（`&mut`），在借用到期之前，不能以任何方式访问该位置。

r[expr.operator.borrow.mut]
`&mut` 在可变位置表达式上下文中求值其操作数。

r[expr.operator.borrow.temporary]
如果 `&` 或 `&mut` 运算符应用于[值表达式][value expression]，则会创建[临时值][temporary value]。

这些运算符不能被重载。

```rust
{
    // 创建一个值为 7 的临时值，其生命周期为此作用域。
    let shared_reference = &7;
}
let mut array = [-2, 3, 9];
{
    // 在此作用域内可变借用 `array`。
    // `array` 只能通过 `mutable_reference` 使用。
    let mutable_reference = &mut array;
}
```

r[expr.borrow.and-and-syntax]
尽管 `&&` 是单个记号（[惰性 'and' 运算符](#lazy-boolean-operators)），但在借用表达式的上下文中使用时，它相当于两次借用：

```rust
// 相同的含义：
let a = &&  10;
let a = & & 10;

// 相同的含义：
let a = &&&&  mut 10;
let a = && && mut 10;
let a = & & & & mut 10;
```

r[expr.borrow.raw]
### 裸借用运算符

r[expr.borrow.raw.intro]
`&raw const` 和 `&raw mut` 是*裸借用运算符*。

r[expr.borrow.raw.place]
这些运算符的操作数表达式在位置表达式上下文中求值。

r[expr.borrow.raw.result]
`&raw const expr` 创建一个类型为 `*const T` 的指向给定位置的 const 裸指针，`&raw mut expr` 创建一个类型为 `*mut T` 的可变裸指针。

r[expr.borrow.raw.invalid-ref]
每当位置表达式可能求值为未正确对齐的位置或未按类型存储有效值的位置时，或者每当创建引用会引入不正确的别名假设时，必须使用裸借用运算符而不是借用运算符。在这些情况下，使用借用运算符会因创建无效引用而导致[未定义行为][undefined behavior]，但裸指针仍然可以被构造。

以下是通过 `packed` 结构体创建指向未对齐位置的裸指针的示例：

```rust
#[repr(packed)]
struct Packed {
    f1: u8,
    f2: u16,
}

let packed = Packed { f1: 1, f2: 2 };
// `&packed.f2` 会创建一个未对齐的引用，因此是未定义行为！
let raw_f2 = &raw const packed.f2;
assert_eq!(unsafe { raw_f2.read_unaligned() }, 2);
```

以下是创建指向不包含有效值的位置的裸指针的示例：

```rust
use std::mem::MaybeUninit;

struct Demo {
    field: bool,
}

let mut uninit = MaybeUninit::<Demo>::uninit();
// `&uninit.as_mut().field` 会创建一个指向未初始化 `bool` 的引用，
// 因此是未定义行为！
let f1_ptr = unsafe { &raw mut (*uninit.as_mut_ptr()).field };
unsafe { f1_ptr.write(true); }
let init = unsafe { uninit.assume_init() };
```

r[expr.deref]
## 解引用运算符

r[expr.deref.syntax]
```grammar,expressions
DereferenceExpression -> `*` Expression
```

r[expr.deref.intro]
`*`（解引用）运算符也是一元前缀运算符。

r[expr.deref.result]
当应用于[指针](../types/pointer.md)或 [`Box`] 时，它表示指向的位置。

r[expr.deref.mut]
如果表达式的类型为 `&mut T`、`*mut T` 或 `Box<T>`，并且是局部变量、局部变量的（嵌套）字段或可变的[位置表达式][place expression]，则可以赋值给结果内存位置。

r[expr.deref.box]
当应用于 [`Box`] 时，结果位置可以被[移出][moved from]。

r[expr.deref.safety]
解引用裸指针需要 `unsafe`。

r[expr.deref.traits]
对于非指针类型，`*x` 等价于[不可变位置表达式上下文](../expressions.md#mutability)中的 `*std::ops::Deref::deref(&x)` 和可变位置表达式上下文中的 `*std::ops::DerefMut::deref_mut(&mut x)`，不同之处在于当 `*x` 经历[临时值生命周期延长][temporary lifetime extension]时，被解引用的表达式 `x` 的[临时值作用域][temporary scope]也会被延长。

```rust
# struct NoCopy;
let a = &7;
assert_eq!(*a, 7);
let b = &mut 9;
*b = 11;
assert_eq!(*b, 11);
let c = Box::new(NoCopy);
let d: NoCopy = *c;
```

```rust
// 持有 `String::new()` 结果的临时值被延长到块的末尾，
// 因此 `x` 可以在后续语句中使用。
let x = &*String::new();
# x;
```

```rust,compile_fail,E0716
// 持有 `String::new()` 结果的临时值在语句末尾被丢弃，
// 因此之后使用 `y` 是错误的。
let y = &*std::ops::Deref::deref(&String::new()); // 错误
# y;
```

r[expr.try]
## try 传播表达式

r[expr.try.syntax]
```grammar,expressions
TryPropagationExpression -> Expression `?`
```

r[expr.try.intro]
try 传播表达式使用内部表达式的值和 [`Try`] trait 来决定是否产生一个值，如果要产生，则产生什么值，或者是否向调用者返回一个值，如果要返回，则返回什么值。

> [!EXAMPLE]
> ```rust
> # use std::num::ParseIntError;
> fn try_to_parse() -> Result<i32, ParseIntError> {
>     let x: i32 = "123".parse()?; // `x` 是 `123`。
>     let y: i32 = "24a".parse()?; // 立即返回 `Err()`。
>     Ok(x + y)                    // 不会运行。
> }
>
> let res = try_to_parse();
> println!("{res:?}");
> # assert!(res.is_err())
> ```
>
> ```rust
> fn try_option_some() -> Option<u8> {
>     let val = Some(1)?;
>     Some(val)
> }
> assert_eq!(try_option_some(), Some(1));
>
> fn try_option_none() -> Option<u8> {
>     let val = None?;
>     Some(val)
> }
> assert_eq!(try_option_none(), None);
> ```
>
> ```rust
> use std::ops::ControlFlow;
>
> pub struct TreeNode<T> {
>     value: T,
>     left: Option<Box<TreeNode<T>>>,
>     right: Option<Box<TreeNode<T>>>,
> }
>
> impl<T> TreeNode<T> {
>     pub fn traverse_inorder<B>(&self, f: &mut impl FnMut(&T) -> ControlFlow<B>) -> ControlFlow<B> {
>         if let Some(left) = &self.left {
>             left.traverse_inorder(f)?;
>         }
>         f(&self.value)?;
>         if let Some(right) = &self.right {
>             right.traverse_inorder(f)?;
>         }
>         ControlFlow::Continue(())
>     }
> }
> #
> # fn main() {
> #     let n = TreeNode {
> #         value: 1,
> #         left: Some(Box::new(TreeNode{value: 2, left: None, right: None})),
> #         right: None,
> #     };
> #     let v = n.traverse_inorder(&mut |t| {
> #         if *t == 2 {
> #             ControlFlow::Break("found")
> #         } else {
> #             ControlFlow::Continue(())
> #         }
> #     });
> #     assert_eq!(v, ControlFlow::Break("found"));
> # }
> ```

> [!NOTE]
> [`Try`] trait 目前是不稳定的，因此不能为用户类型实现。
>
> try 传播表达式目前大致等价于：
>
> ```rust
> # #![ feature(try_trait_v2) ]
> # fn example() -> Result<(), ()> {
> # let expr = Ok(());
> match core::ops::Try::branch(expr) {
>     core::ops::ControlFlow::Continue(val) => val,
>     core::ops::ControlFlow::Break(residual) =>
>         return core::ops::FromResidual::from_residual(residual),
> }
> # Ok(())
> # }
> ```

> [!NOTE]
> try 传播运算符有时被称为*问号运算符*、*`?` 运算符*或*try 运算符*。

r[expr.try.restricted-types]
try 传播运算符可以应用于以下类型的表达式：

- [`Result<T, E>`]
    - `Result::Ok(val)` 求值为 `val`。
    - `Result::Err(e)` 返回 `Result::Err(From::from(e))`。
- [`Option<T>`]
    - `Option::Some(val)` 求值为 `val`。
    - `Option::None` 返回 `Option::None`。
- [`ControlFlow<B, C>`][core::ops::ControlFlow]
    - `ControlFlow::Continue(c)` 求值为 `c`。
    - `ControlFlow::Break(b)` 返回 `ControlFlow::Break(b)`。
- [`Poll<Result<T, E>>`][core::task::Poll]
    - `Poll::Ready(Ok(val))` 求值为 `Poll::Ready(val)`。
    - `Poll::Ready(Err(e))` 返回 `Poll::Ready(Err(From::from(e)))`。
    - `Poll::Pending` 求值为 `Poll::Pending`。
- [`Poll<Option<Result<T, E>>>`][`core::task::Poll`]
    - `Poll::Ready(Some(Ok(val)))` 求值为 `Poll::Ready(Some(val))`。
    - `Poll::Ready(Some(Err(e)))` 返回 `Poll::Ready(Some(Err(From::from(e))))`。
    - `Poll::Ready(None)` 求值为 `Poll::Ready(None)`。
    - `Poll::Pending` 求值为 `Poll::Pending`。

r[expr.negate]
## 取反运算符

r[expr.negate.syntax]
```grammar,expressions
NegationExpression ->
      `-` Expression
    | `!` Expression
```

r[expr.negate.intro]
这是最后两个一元运算符。

r[expr.negate.results]
此表总结了它们在原始类型上的行为以及用于为其他类型重载这些运算符的 trait。请记住，有符号整数始终使用二进制补码表示。所有这些运算符的操作数都在[值表达式上下文][value expression]中求值，因此会被移动或复制。

| 符号 | 整数           | `bool`         | 浮点数       | 重载 Trait         |
|------|----------------|----------------|--------------|--------------------|
| `-`  | 取反*          |                | 取反         | `std::ops::Neg`    |
| `!`  | 按位 NOT       | [逻辑 NOT][Logical NOT] |         | `std::ops::Not`    |

\* 仅对有符号整数类型。

以下是这些运算符的一些示例

```rust
let x = 6;
assert_eq!(-x, -6);
assert_eq!(!x, -7);
assert_eq!(true, !false);
```

r[expr.arith-logic]
## 算术和逻辑二元运算符

r[expr.arith-logic.syntax]
```grammar,expressions
ArithmeticOrLogicalExpression ->
      Expression `+` Expression
    | Expression `-` Expression
    | Expression `*` Expression
    | Expression `/` Expression
    | Expression `%` Expression
    | Expression `&` Expression
    | Expression `|` Expression
    | Expression `^` Expression
    | Expression `<<` Expression
    | Expression `>>` Expression
```

r[expr.arith-logic.intro]
二元运算符表达式都使用中缀表示法书写。

r[expr.arith-logic.behavior]
此表总结了算术和逻辑二元运算符在原始类型上的行为以及用于为其他类型重载这些运算符的 trait。请记住，有符号整数始终使用二进制补码表示。所有这些运算符的操作数都在[值表达式上下文][value expression]中求值，因此会被移动或复制。

| 符号 | 整数                       | `bool`         | 浮点数       | 重载 Trait           | 重载复合赋值 Trait                    |
|------|----------------------------|----------------|--------------|----------------------| ------------------------------------- |
| `+`  | 加法                       |                | 加法         | `std::ops::Add`      | `std::ops::AddAssign`                 |
| `-`  | 减法                       |                | 减法         | `std::ops::Sub`      | `std::ops::SubAssign`                 |
| `*`  | 乘法                       |                | 乘法         | `std::ops::Mul`      | `std::ops::MulAssign`                 |
| `/`  | 除法*†                     |                | 除法         | `std::ops::Div`      | `std::ops::DivAssign`                 |
| `%`  | 取余**†                    |                | 取余         | `std::ops::Rem`      | `std::ops::RemAssign`                 |
| `&`  | 按位 AND                   | [逻辑 AND][Logical AND]  |         | `std::ops::BitAnd`   | `std::ops::BitAndAssign`              |
| `\|` | 按位 OR                    | [逻辑 OR][Logical OR]   |         | `std::ops::BitOr`    | `std::ops::BitOrAssign`               |
| `^`  | 按位 XOR                   | [逻辑 XOR][Logical XOR]  |         | `std::ops::BitXor`   | `std::ops::BitXorAssign`              |
| `<<` | 左移                       |                |              | `std::ops::Shl`      | `std::ops::ShlAssign`                 |
| `>>` | 右移***                    |                |              | `std::ops::Shr`      |  `std::ops::ShrAssign`                |

\* 整数除法向零舍入。

\*\* Rust 使用的取余定义为[截断除法](https://en.wikipedia.org/wiki/Modulo_operation#Variants_of_the_definition)。给定 `remainder = dividend % divisor`，余数将与被除数具有相同的符号。

\*\*\* 对有符号整数类型进行算术右移，对无符号整数类型进行逻辑右移。

† 对于整数类型，除以零会 panic。

以下是在使用这些运算符的示例。

```rust
assert_eq!(3 + 6, 9);
assert_eq!(5.5 - 1.25, 4.25);
assert_eq!(-5 * 14, -70);
assert_eq!(14 / 3, 4);
assert_eq!(100 % 7, 2);
assert_eq!(0b1010 & 0b1100, 0b1000);
assert_eq!(0b1010 | 0b1100, 0b1110);
assert_eq!(0b1010 ^ 0b1100, 0b110);
assert_eq!(13 << 3, 104);
assert_eq!(-10 >> 2, -3);
```

r[expr.cmp]
## 比较运算符

r[expr.cmp.syntax]
```grammar,expressions
ComparisonExpression ->
      Expression `==` Expression
    | Expression `!=` Expression
    | Expression `>` Expression
    | Expression `<` Expression
    | Expression `>=` Expression
    | Expression `<=` Expression
```

r[expr.cmp.intro]
比较运算符也是既为原始类型定义，也为标准库中的许多类型定义。

r[expr.cmp.paren-chaining]
链接比较运算符时需要使用括号。例如，表达式 `a == b == c` 无效，可以写为 `(a == b) == c`。

r[expr.cmp.trait]
与算术和逻辑运算符不同，用于重载这些运算符的 trait 更普遍地用于展示类型之间如何进行比较，并且很可能被使用这些 trait 作为约束的函数假定为定义了实际的比较操作。标准库中的许多函数和宏可以利用该假设（尽管不能确保安全性）。

r[expr.cmp.place]
与上面的算术和逻辑运算符不同，这些运算符隐式获取其操作数的共享借用，在[位置表达式上下文][place expression]中求值它们：

```rust
# let a = 1;
# let b = 1;
a == b;
// 等价于
::std::cmp::PartialEq::eq(&a, &b);
```

这意味着操作数不必被移出。

r[expr.cmp.behavior]

| 符号 | 含义          | 重载方法                    |
|------|---------------|-----------------------------|
| `==` | 等于          | `std::cmp::PartialEq::eq`   |
| `!=` | 不等于        | `std::cmp::PartialEq::ne`   |
| `>`  | 大于          | `std::cmp::PartialOrd::gt`  |
| `<`  | 小于          | `std::cmp::PartialOrd::lt`  |
| `>=` | 大于等于      | `std::cmp::PartialOrd::ge`  |
| `<=` | 小于等于      | `std::cmp::PartialOrd::le`  |

以下是在使用比较运算符的示例。

```rust
assert!(123 == 123);
assert!(23 != -12);
assert!(12.5 > 12.2);
assert!([1, 2, 3] < [1, 3, 4]);
assert!('A' <= 'B');
assert!("World" >= "Hello");
```

r[expr.bool-logic]
## 惰性布尔运算符

r[expr.bool-logic.syntax]
```grammar,expressions
LazyBooleanExpression ->
      Expression `||` Expression
    | Expression `&&` Expression
```

r[expr.bool-logic.intro]
运算符 `||` 和 `&&` 可以应用于布尔类型的操作数。`||` 运算符表示逻辑"或"，`&&` 运算符表示逻辑"与"。

r[expr.bool-logic.conditional-evaluation]
它们与 `|` 和 `&` 的不同之处在于，只有当左侧操作数尚未确定表达式的结果时，才求值右侧操作数。即，只有当左侧操作数求值为 `false` 时，`||` 才求值其右侧操作数；只有当左侧操作数求值为 `true` 时，`&&` 才求值其右侧操作数。

```rust
let x = false || true; // true
let y = false && panic!(); // false，不求值 `panic!()`
```

r[expr.as]
## 类型转换表达式

r[expr.as.syntax]
```grammar,expressions
TypeCastExpression -> Expression `as` TypeNoBounds
```

r[expr.as.intro]
类型转换表达式用二元运算符 `as` 表示。

r[expr.as.result]
执行 `as` 表达式将左侧的值转换为右侧的类型。

`as` 表达式示例：

```rust
# fn sum(values: &[f64]) -> f64 { 0.0 }
# fn len(values: &[f64]) -> i32 { 0 }
fn average(values: &[f64]) -> f64 {
    let sum: f64 = sum(values);
    let size: f64 = len(values) as f64;
    sum / size
}
```

r[expr.as.coercions]
`as` 可用于显式执行[强制转换](../type-coercions.md)，以及以下附加的转换。任何不符合强制转换规则或表中条目的转换都是编译错误。此处 `*T` 表示 `*const T` 或 `*mut T`。`m` 表示引用类型中的可选 `mut` 以及指针类型中的 `mut` 或 `const`。

| `e` 的类型            | `U`                        | `e as U` 执行的转换                               |
|-----------------------|----------------------------|---------------------------------------------------|
| 整数或浮点类型        | 整数或浮点类型             | [数值转换][expr.as.numeric]                       |
| 枚举                  | 整数类型                   | [枚举转换][expr.as.enum]                          |
| `bool` 或 `char`      | 整数类型                   | [原始类型到整数转换][expr.as.bool-char-as-int]    |
| `u8`                  | `char`                     | [`u8` 到 `char` 转换][expr.as.u8-as-char]         |
| `*T`                  | `*V`（当[兼容][expr.as.pointer]时） | [指针到指针转换][expr.as.pointer]         |
| `*T`，其中 `T: Sized` | 整数类型                   | [指针到地址转换][expr.as.pointer-as-int]          |
| 整数类型              | `*V`，其中 `V: Sized`      | [地址到指针转换][expr.as.int-as-pointer]          |
| `&m₁ [T; n]`          | `*m₂ T` [^lessmut]         | 数组到指针转换                                    |
| `*m₁ [T; n]`          | `*m₂ T` [^lessmut]         | 数组到指针转换                                    |
| [函数项][Function item]              | [函数指针][Function pointer]                 | 函数项到函数指针转换                              |
| [函数项][Function item]              | `*V`，其中 `V: Sized`      | 函数项到指针转换                                  |
| [函数项][Function item]              | 整数                       | 函数项到地址转换                                  |
| [函数指针][Function pointer]            | `*V`，其中 `V: Sized`      | 函数指针到指针转换                                |
| [函数指针][Function pointer]            | 整数                       | 函数指针到地址转换                                |
| 闭包 [^no-capture]    | 函数指针                   | 闭包到函数指针转换                                |

[^lessmut]: 仅当 `m₁` 为 `mut` 或 `m₂` 为 `const` 时。允许将 `mut` 引用/指针转换为 `const` 指针。

[^no-capture]: 只有不捕获（封闭）任何局部变量的闭包才能转换为函数指针。

### 语义

r[expr.as.numeric]
#### 数值转换

r[expr.as.numeric.int-same-size]
* 相同大小的两个整数之间的转换（例如 i32 -> u32）是无操作（Rust 使用二进制补码处理固定整数的负值）

  ```rust
  assert_eq!(42i8 as u8, 42u8);
  assert_eq!(-1i8 as u8, 255u8);
  assert_eq!(255u8 as i8, -1i8);
  assert_eq!(-1i16 as u16, 65535u16);
  ```

r[expr.as.numeric.int-truncation]
* 从较大整数到较小整数的转换（例如 u32 -> u8）将截断

  ```rust
  assert_eq!(42u16 as u8, 42u8);
  assert_eq!(1234u16 as u8, 210u8);
  assert_eq!(0xabcdu16 as u8, 0xcdu8);

  assert_eq!(-42i16 as i8, -42i8);
  assert_eq!(1234u16 as i8, -46i8);
  assert_eq!(0xabcdi32 as i8, -51i8);
  ```

r[expr.as.numeric.int-extension]
* 从较小整数到较大整数的转换（例如 u8 -> u32）将
    * 如果源是无符号的，则零扩展
    * 如果源是有符号的，则符号扩展

  ```rust
  assert_eq!(42i8 as i16, 42i16);
  assert_eq!(-17i8 as i16, -17i16);
  assert_eq!(0b1000_1010u8 as u16, 0b0000_0000_1000_1010u16, "零扩展");
  assert_eq!(0b0000_1010i8 as i16, 0b0000_0000_0000_1010i16, "符号扩展 0");
  assert_eq!(0b1000_1010u8 as i8 as i16, 0b1111_1111_1000_1010u16 as i16, "符号扩展 1");
  ```

r[expr.as.numeric.float-as-int]
* 从浮点数到整数的转换将向零舍入
    * `NaN` 将返回 `0`
    * 大于最大整数值的值（包括 `INFINITY`）将饱和到整数类型的最大值。
    * 小于最小整数值的值（包括 `NEG_INFINITY`）将饱和到整数类型的最小值。

  ```rust
  assert_eq!(42.9f32 as i32, 42);
  assert_eq!(-42.9f32 as i32, -42);
  assert_eq!(42_000_000f32 as i32, 42_000_000);
  assert_eq!(std::f32::NAN as i32, 0);
  assert_eq!(1_000_000_000_000_000f32 as i32, 0x7fffffffi32);
  assert_eq!(std::f32::NEG_INFINITY as i32, -0x80000000i32);
  ```

r[expr.as.numeric.int-as-float]
* 从整数到浮点数的转换将产生最接近的可能浮点数 \*
    * 如有必要，舍入根据 `roundTiesToEven` 模式进行 \*\*\*
    * 溢出时，产生无穷大（与输入符号相同）
    * 注意：在当前数值类型集合中，溢出仅在 `u128 as f32` 中发生，当值大于或等于 `f32::MAX + (0.5 ULP)` 时

  ```rust
  assert_eq!(1337i32 as f32, 1337f32);
  assert_eq!(123_456_789i32 as f32, 123_456_790f32, "已舍入");
  assert_eq!(0xffffffff_ffffffff_ffffffff_ffffffff_u128 as f32, std::f32::INFINITY);
  ```

r[expr.as.numeric.float-widening]
* 从 f32 到 f64 的转换是完美且无损失的

  ```rust
  assert_eq!(1_234.5f32 as f64, 1_234.5f64);
  assert_eq!(std::f32::INFINITY as f64, std::f64::INFINITY);
  assert!((std::f32::NAN as f64).is_nan());
  ```

r[expr.as.numeric.float-narrowing]
* 从 f64 到 f32 的转换将产生最接近的可能 f32 \*\*
    * 如有必要，舍入根据 `roundTiesToEven` 模式进行 \*\*\*
    * 溢出时，产生无穷大（与输入符号相同）

  ```rust
  assert_eq!(1_234.5f64 as f32, 1_234.5f32);
  assert_eq!(1_234_567_891.123f64 as f32, 1_234_567_890f32, "已舍入");
  assert_eq!(std::f64::INFINITY as f32, std::f32::INFINITY);
  assert!((std::f64::NAN as f32).is_nan());
  ```

\* 如果硬件本身不支持具有此舍入模式和溢出行为的整数到浮点数转换，这些转换可能会比预期慢。

\*\* 如果硬件本身不支持具有此舍入模式和溢出行为的 f64 到 f32 转换，这些转换可能会比预期慢。

\*\*\* 如 IEEE 754-2008 §4.3.1 定义：选择最接近的浮点数，如果恰好处于两个浮点数中间，则选择最低有效位为偶数的那个。

r[expr.as.enum]
#### 枚举转换

r[expr.as.enum.discriminant]
将枚举转换为其判别值，然后根据需要应用数值转换。转换仅限于以下类型的枚举：

* [仅单元变体枚举][Unit-only enums]
* 没有[显式判别值][explicit discriminants]的[无字段枚举][field-less enums]，或其中仅有单元变体具有显式判别值

```rust
enum Enum { A, B, C }
assert_eq!(Enum::A as i32, 0);
assert_eq!(Enum::B as i32, 1);
assert_eq!(Enum::C as i32, 2);
```

r[expr.as.enum.no-drop]
如果枚举实现了 [`Drop`]，则不允许转换。

r[expr.as.bool-char-as-int]
#### 原始类型到整数转换

* `false` 转换为 `0`，`true` 转换为 `1`
* `char` 转换为码点值，然后根据需要应用数值转换。

```rust
assert_eq!(false as i32, 0);
assert_eq!(true as i32, 1);
assert_eq!('A' as i32, 65);
assert_eq!('Ö' as i32, 214);
```

r[expr.as.u8-as-char]
#### `u8` 到 `char` 转换

转换为具有对应码点的 `char`。

```rust
assert_eq!(65u8 as char, 'A');
assert_eq!(214u8 as char, 'Ö');
```

r[expr.as.pointer-as-int]
#### 指针到地址转换

从裸指针到整数的转换产生所引用内存的机器地址。如果整数类型小于指针类型，地址可能被截断；使用 `usize` 可避免此问题。

r[expr.as.int-as-pointer]
#### 地址到指针转换

从整数到裸指针的转换将整数解释为内存地址，并产生一个引用该内存的指针。

> [!WARNING]
> 这与 Rust 内存模型交互，该模型仍在开发中。
> 从此转换获得的指针可能受到额外限制，即使它在位级别上与有效指针相等。
> 如果不遵循别名规则，解引用此类指针可能是[未定义行为][undefined behavior]。

一个合理的地址算术简单示例：

```rust
let mut values: [i32; 2] = [1, 2];
let p1: *mut i32 = values.as_mut_ptr();
let first_address = p1 as usize;
let second_address = first_address + 4; // 4 == size_of::<i32>()
let p2 = second_address as *mut i32;
unsafe {
    *p2 += 1;
}
assert_eq!(values[1], 3);
```

r[expr.as.pointer]
#### 指针到指针转换

r[expr.as.pointer.behavior]
`*const T` / `*mut T` 可以转换为 `*const U` / `*mut U`，具有以下行为：

r[expr.as.pointer.sized]
- 如果 `T` 和 `U` 都是固定大小的，则指针原样返回不变。

  > [!EXAMPLE]
  > ```rust
  > let x: i32 = 42;
  > let p1: *const i32 = &x;
  > let p2: *const u8 = p1 as *const u8;
  > // 指针地址保持不变。
  > assert_eq!(p1 as usize, p2 as usize);
  > ```

r[expr.as.pointer.discard-metadata]
- 如果 `T` 是非固定大小的而 `U` 是固定大小的，该转换会丢弃完成宽指针 `T` 的所有元数据，并产生一个由非固定大小指针的数据部分组成的瘦指针 `U`。

  > [!EXAMPLE]
  > ```rust
  > let slice: &[i32] = &[1, 2, 3];
  > let ptr: *const [i32] = slice as *const [i32];
  > // 从宽指针 (*const [i32]) 到瘦指针 (*const i32) 的转换
  > // 丢弃长度元数据。
  > let data_ptr: *const i32 = ptr as *const i32;
  > assert_eq!(unsafe { *data_ptr }, 1);
  > ```

r[expr.as.pointer.unsized.unchanged]
- 如果 `T` 和 `U` 都是非固定大小的，指针也原样返回不变。特别地，元数据被精确保留。只有根据以下规则元数据兼容时才能执行转换：

r[expr.as.pointer.unsized.slice]
- 当 `T` 和 `U` 是具有切片元数据的非固定大小类型时，它们始终兼容。切片的元数据是元素数量，因此将 `*[u16] -> *[u8]` 转换是合法的，但会导致字节数减半。

  > [!EXAMPLE]
  > ```rust
  > let slice: &[u16] = &[1, 2, 3];
  > let ptr: *const [u16] = slice as *const [u16];
  > let byte_ptr: *const [u8] = ptr as *const [u8];
  > assert_eq!(byte_ptr.len(), 3);
  > ```

r[expr.as.pointer.unsized.trait]
- 当 `T` 和 `U` 是具有 trait 对象元数据的非固定大小类型时，仅当满足以下所有条件时元数据才兼容：
  1. 主 trait 必须相同。

     > [!EXAMPLE]
     > ```rust,compile_fail,E0606
     > trait Foo {}
     > trait Bar {}
     > impl Foo for i32 {}
     > impl Bar for i32 {}
     >
     > let x: i32 = 42;
     > let ptr_foo: *const dyn Foo = &x as *const dyn Foo;
     > // 不能转换到不同的主 trait。
     > let ptr_bar: *const dyn Bar = ptr_foo as *const dyn Bar; // 错误
     > ```


  2. 自动 trait 可以被移除。

     > [!EXAMPLE]
     > ```rust
     > trait Foo {}
     > struct S;
     > impl Foo for S {}
     > unsafe impl Send for S {}
     >
     > let s = S;
     > let ptr_send: *const (dyn Foo + Send) = &s;
     > // 移除自动 trait。
     > let ptr_no_send: *const dyn Foo = ptr_send as *const dyn Foo;
     > ```


  3. 自动 trait 仅当它们是主 trait 的超级 trait 时才可以被添加。

     > [!EXAMPLE]
     > ```rust
     > trait Foo: Send {}
     > struct S;
     > impl Foo for S {}
     > unsafe impl Send for S {}
     >
     > let s = S;
     > let ptr_no_send: *const dyn Foo = &s;
     > // 添加自动 trait。
     > let ptr_send: *const (dyn Foo + Send) = ptr_no_send as *const (dyn Foo + Send);
     > ```
     >
     > ```rust,compile_fail,E0804
     > trait Foo {}
     > # struct S;
     > # impl Foo for S {}
     > # unsafe impl Send for S {}
     > #
     > # let s = S;
     > # let ptr_no_send: *const dyn Foo = &s;
     > // 同上，除了 trait Foo 没有 Send 作为超级 trait。
     > let ptr_send: *const (dyn Foo + Send) = ptr_no_send as *const (dyn Foo + Send); // 错误
     > ```


  4. 尾部生命周期只能被缩短。

     > [!EXAMPLE]
     > ```rust
     > trait Foo {}
     >
     > fn shorten_lifetime<'long: 'short, 'short>(
     >     ptr: *const (dyn Foo + 'long),
     > ) -> *const (dyn Foo + 'short) {
     >     // 缩短生命周期是允许的。
     >     ptr as *const (dyn Foo + 'short)
     > }
     > ```
     >
     > ```rust,compile_fail
     > trait Foo {}
     >
     > fn lengthen_lifetime<'long: 'short, 'short>(
     >     ptr: *const (dyn Foo + 'short),
     > ) -> *const (dyn Foo + 'long) {
     >     // 不允许转换到更长的生命周期。
     >     ptr as *const (dyn Foo + 'long) // 错误
     > }
     > ```

  5. 泛型（包括生命周期）和关联类型必须完全匹配。

     > [!EXAMPLE]
     > ```rust,compile_fail,E0606
     > trait Generic<T> {}
     > impl Generic<i32> for () {}
     > impl Generic<u32> for () {}
     >
     > let x = ();
     > let ptr_i32: *const dyn Generic<i32> = &x;
     > // 不能转换到不同的泛型参数。
     > let ptr_u32: *const dyn Generic<u32> = ptr_i32 as *const dyn Generic<u32>; // 错误
     > ```
     >
     > ```rust
     > trait HasType {
     >     type Output;
     > }
     >
     > trait Generic<'x, T> {}
     >
     > fn cast_via_associated<'a, 'b, A, B>(
     >     ptr: *const dyn Generic<'a, A::Output>,
     > ) -> *const dyn Generic<'b, B::Output>
     > where
     >     'a: 'b,
     >     'b: 'a,
     >     A: HasType,
     >     B: HasType<Output = A::Output>, // 强制相等
     > {
     >     ptr as *const dyn Generic<'b, B::Output>
     > }
     > ```



r[expr.as.pointer.unsized.compound]
- 当 `T` 或 `U` 是其最后一个字段为非固定大小的结构体或元组类型时，它具有与其最后一个字段相同的元数据和兼容性规则。

  > [!EXAMPLE]
  > ```rust
  > struct Wrapper(u32, [u8]);
  >
  > let slice: &[u8] = &[1, 2, 3];
  > let ptr: *const [u8] = slice;
  >
  > // 转换为最后一个字段为非固定大小类型 `[u8]` 的
  > // 结构体时，元数据（长度 3）被保留。
  > let wrapper_ptr: *const Wrapper = ptr as *const Wrapper;
  >
  > // 转换回来时也被保留。
  > let ptr_back: *const [u8] = wrapper_ptr as *const [u8];
  > assert_eq!(ptr_back.len(), 3);
  > ```

r[expr.assign]
## 赋值表达式

r[expr.assign.syntax]
```grammar,expressions
AssignmentExpression -> Expression `=` Expression
```

r[expr.assign.intro]
*赋值表达式*将一个值移动到指定位置。

r[expr.assign.assignee]
赋值表达式由一个[可变][mutable]的[赋值目标表达式][assignee expression]（*赋值目标操作数*）、后跟一个等号（`=`）和一个[值表达式][value expression]（*所赋值的值操作数*）组成。

r[expr.assign.behavior-basic]
在其最基本的形式中，赋值目标表达式是[位置表达式][place expression]，我们首先讨论这种情况。

r[expr.assign.behavior-destructuring]
下面将讨论解构赋值的更通用情况，但这种情况下总是分解为对位置表达式的顺序赋值，后者可以被视为更基本的情况。

r[expr.assign.basic]
### 基本赋值

r[expr.assign.evaluation-order]
求值赋值表达式从求值其操作数开始。所赋值的值操作数首先被求值，然后求值赋值目标表达式。

r[expr.assign.destructuring-order]
对于解构赋值，赋值目标表达式的子表达式从左到右求值。

> [!NOTE]
> 这与其他表达式不同，因为右侧操作数在左侧之前求值。

r[expr.assign.drop-target]
然后它的效果是首先[丢弃][dropping]赋值位置上的值，除非该位置是未初始化的局部变量或局部变量的未初始化字段。

r[expr.assign.behavior]
接下来它将所赋值的值[复制或移动][copies or moves]到赋值位置。

r[expr.assign.result]
赋值表达式始终产生[单元值][unit]。

示例：

```rust
let mut x = 0;
let y = 0;
x = y;
```

r[expr.assign.destructure]
### 解构赋值

r[expr.assign.destructure.intro]
解构赋值是变量声明的解构模式匹配的对应物，允许赋值给复杂值，例如元组或结构体。例如，我们可以交换两个可变变量：

```rust
let (mut a, mut b) = (0, 1);
// 使用解构赋值交换 `a` 和 `b`。
(b, a) = (a, b);
```

r[expr.assign.destructure.assignee]
与使用 `let` 的解构声明不同，由于语法歧义，模式不能出现在赋值的左侧。相反，一组对应于模式的表达式被指定为[赋值目标表达式][assignee expression][assignee expression]，并允许出现在赋值的左侧。赋值目标表达式随后被脱糖为模式匹配后跟顺序赋值。

r[expr.assign.destructure.irrefutable]
脱糖后的模式必须是不可反驳的：特别是，这意味着只有长度在编译时已知的切片模式和简单切片 `[..]` 才允许用于解构赋值。

脱糖方法很直接，最好通过示例来说明。

```rust
# struct Struct { x: u32, y: u32 }
# let (mut a, mut b) = (0, 0);
(a, b) = (3, 4);

[a, b] = [3, 4];

Struct { x: a, y: b } = Struct { x: 3, y: 4};

// 脱糖为：

{
    let (_a, _b) = (3, 4);
    a = _a;
    b = _b;
}

{
    let [_a, _b] = [3, 4];
    a = _a;
    b = _b;
}

{
    let Struct { x: _a, y: _b } = Struct { x: 3, y: 4};
    a = _a;
    b = _b;
}
```

r[expr.assign.destructure.repeat-ident]
不禁止标识符在单个赋值目标表达式中多次使用。

r[expr.assign.destructure.discard-value]
[下划线表达式][Underscore expressions]和空[区间表达式][range expressions]可用于忽略某些值，而不绑定它们。

r[expr.assign.destructure.default-binding]
注意，默认绑定模式不适用于脱糖后的表达式。

r[expr.assign.destructure.tmp-scopes]
> [!NOTE]
> 脱糖限制了赋值值操作数（RHS）的[临时值作用域][temporary scope]。
>
> 在基本赋值中，[临时值][temporary value]在外围临时值作用域的末尾被丢弃。下面是语句的末尾。因此，赋值和使用是允许的。
>
> ```rust
> # fn temp() {}
> fn f<T>(x: T) -> T { x }
> let x;
> (x = f(&temp()), x); // OK
> ```
>
> 反之，在解构赋值中，临时值在脱糖中 `let` 语句的末尾被丢弃。由于这发生在我们尝试赋值给 `x` 之前，以下代码失败。
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # fn f<T>(x: T) -> T { x }
> # let x;
> [x] = [f(&temp())]; // 错误
> ```
>
> 这会脱糖为：
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # fn f<T>(x: T) -> T { x }
> # let x;
> {
>     let [_x] = [f(&temp())];
>     //                     ^
>     //      临时值在此处被丢弃。
>     x = _x; // 错误
> }
> ```

r[expr.assign.destructure.tmp-ext]
> [!NOTE]
> 由于脱糖，解构赋值的赋值值操作数（RHS）是新引入块内的一个[延长表达式][extending expression]。
>
> 下面，由于[临时值作用域][temporary scope]被延长到此引入块的末尾，赋值是允许的。
>
> ```rust
> # fn temp() {}
> # let x;
> [x] = [&temp()]; // OK
> ```
>
> 这会脱糖为：
>
> ```rust
> # fn temp() {}
> # let x;
> { let [_x] = [&temp()]; x = _x; } // OK
> ```
>
> 然而，如果我们尝试使用 `x`，即使在同一条语句内，也会得到错误，因为[临时值][temporary value]在此引入块的末尾被丢弃。
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # let x;
> ([x] = [&temp()], x); // 错误
> ```
>
> 这会脱糖为：
>
> ```rust,compile_fail,E0716
> # fn temp() {}
> # let x;
> (
>     {
>         let [_x] = [&temp()];
>         x = _x;
>     }, // <-- 临时值在此处被丢弃。
>     x, // 错误
> );
> ```

r[expr.compound-assign]
## 复合赋值表达式

r[expr.compound-assign.syntax]
```grammar,expressions
CompoundAssignmentExpression ->
      Expression `+=` Expression
    | Expression `-=` Expression
    | Expression `*=` Expression
    | Expression `/=` Expression
    | Expression `%=` Expression
    | Expression `&=` Expression
    | Expression `|=` Expression
    | Expression `^=` Expression
    | Expression `<<=` Expression
    | Expression `>>=` Expression
```

r[expr.compound-assign.intro]
*复合赋值表达式*将算术和逻辑二元运算符与赋值表达式结合起来。

例如：

```rust
let mut x = 5;
x += 1;
assert!(x == 6);
```

复合赋值的语法是一个[可变][mutable]的[位置表达式][place expression]（*被赋值操作数*），后跟一个运算符和一个 `=` 作为一个整体记号（无空白），再后跟一个[值表达式][value expression]（*修改操作数*）。

r[expr.compound-assign.place]
与其他位置操作数不同，被赋值的位置操作数必须是位置表达式。

r[expr.compound-assign.no-value]
尝试使用值表达式是编译错误，而不是将其提升为临时值。

r[expr.compound-assign.operand-order]
复合赋值表达式的求值取决于操作数的类型。

r[expr.compound-assign.primitives]
如果在单态化之前已知两个操作数的类型都是原始类型，则首先求值右侧，然后求值左侧，通过将运算符应用于两侧的值来修改左侧求值给出的位置。

```rust
# use core::{num::Wrapping, ops::AddAssign};
#
trait Equate {}
impl<T> Equate for (T, T) {}

fn f1(x: (u8,)) {
    let mut order = vec![];
    // 先求值 RHS，因为两个操作数都是原始类型。
    { order.push(2); x }.0 += { order.push(1); x }.0;
    assert!(order.is_sorted());
}

fn f2(x: (Wrapping<u8>,)) {
    let mut order = vec![];
    // 先求值 LHS，因为 `Wrapping<_>` 不是原始类型。
    { order.push(1); x }.0 += { order.push(2); (0u8,) }.0;
    assert!(order.is_sorted());
}

fn f3<T: AddAssign<u8> + Copy>(x: (T,)) where (T, u8): Equate {
    let mut order = vec![];
    // 先求值 LHS，因为操作数之一是泛型参数，即使该泛型参数
    // 可以因 where 子句约束而与原始类型统一。
    { order.push(1); x }.0 += { order.push(2); (0u8,) }.0;
    assert!(order.is_sorted());
}

fn main() {
    f1((0u8,));
    f2((Wrapping(0u8),));
    // 我们提供原始类型作为泛型参数，但这不影响
    // 单态化时 `f3` 中的求值顺序。
    f3::<u8>((0u8,));
}
```

> [!NOTE]
> 这是不寻常的。在其他地方，从左到右求值是常态。
>
> 有关更多示例，请参见[求值顺序测试][eval order test]。

r[expr.compound-assign.trait]
否则，此表达式是使用运算符对应 trait 的语法糖（参见 [expr.arith-logic.behavior]），并以左侧作为[接收者][receiver]、右侧作为下一个参数来调用其方法。

例如，以下两条语句是等价的：

```rust
# use std::ops::AddAssign;
fn f<T: AddAssign + Copy>(mut x: T, y: T) {
    x += y; // 语句 1。
    x.add_assign(y); // 语句 2。
}
```

> [!NOTE]
> 令人惊讶的是，将其进一步脱糖为完全限定方法调用是不等价的，因为当通过[自动引用][autoref]获取对第一个操作数的可变引用时，借检查器有特殊行为。
>
> ```rust
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 这里我们将 `x` 同时用作 LHS 和 RHS。因为调用 trait 方法
>     // 所需的 LHS 可变引用是通过自动引用隐式获取的，所以这是可以的。
>     x += x; //~ OK
>     x.add_assign(x); //~ OK
> }
> ```
>
> ```rust,compile_fail,E0503
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 我们不能将上述代码脱糖为以下代码，因为一旦我们获取了 `x`
>     // 的可变引用来传递第一个参数，我们就不能在第二个参数中
>     // 按值传递 `x`，因为可变引用仍然存活。
>     <T as AddAssign>::add_assign(&mut x, x);
>     //~^ 错误：无法使用 `x`，因为它已被可变借用
> }
> ```
>
> ```rust,compile_fail,E0503
> # use std::ops::AddAssign;
> fn f<T: AddAssign + Copy>(mut x: T) {
>     // 同上。
>     (&mut x).add_assign(x);
>     //~^ 错误：无法使用 `x`，因为它已被可变借用
> }
> ```

r[expr.compound-assign.result]
与普通赋值表达式一样，复合赋值表达式始终产生[单元值][unit]。

> [!WARNING]
> 避免编写依赖于复合赋值中操作数求值顺序的代码，因为它可能不寻常且令人惊讶。

[`Box`]: ../special-types-and-traits.md#boxt
[`Try`]: core::ops::Try
[autoref]: expr.method.candidate-receivers-refs
[copies or moves]: ../expressions.md#moved-and-copied-types
[dropping]: ../destructors.md
[eval order test]: https://github.com/rust-lang/rust/blob/1.58.0/src/test/ui/expr/compound-assignment/eval-order.rs
[explicit discriminants]: ../items/enumerations.md#explicit-discriminants
[extending expression]: destructors.scope.lifetime-extension.exprs
[field-less enums]: ../items/enumerations.md#field-less-enum
[grouped expression]: grouped-expr.md
[literal expression]: literal-expr.md#integer-literal-expressions
[logical and]: ../types/boolean.md#logical-and
[logical not]: ../types/boolean.md#logical-not
[logical or]: ../types/boolean.md#logical-or
[logical xor]: ../types/boolean.md#logical-xor
[moved from]: expr.move.movable-place
[mutable]: ../expressions.md#mutability
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[assignee expression]: ../expressions.md#place-expressions-and-value-expressions
[undefined behavior]: ../behavior-considered-undefined.md
[unit]: ../types/tuple.md
[Unit-only enums]: ../items/enumerations.md#unit-only-enum
[value expression]: ../expressions.md#place-expressions-and-value-expressions
[temporary lifetime extension]: destructors.scope.lifetime-extension
[temporary scope]: destructors.scope.temporary
[temporary value]: ../expressions.md#temporaries
[float-float]: https://github.com/rust-lang/rust/issues/15536
[Function pointer]: ../types/function-pointer.md
[Function item]: ../types/function-item.md
[receiver]: expr.method.intro
[temporary]: expr.temporary
[undefined behavior]: ../behavior-considered-undefined.md
[Underscore expressions]: ./underscore-expr.md
[range expressions]: ./range-expr.md
