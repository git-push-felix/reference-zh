r[expr]
# 表达式

r[expr.syntax]
```grammar,expressions
Expression ->
      ExpressionWithoutBlock
    | ExpressionWithBlock

ExpressionWithoutBlock ->
    OuterAttribute* ExpressionWithoutBlockNoAttrs

ExpressionWithoutBlockNoAttrs ->
      LiteralExpression
    | PathExpression
    | OperatorExpression
    | GroupedExpression
    | ArrayExpression
    | AwaitExpression
    | IndexExpression
    | TupleExpression
    | TupleIndexingExpression
    | StructExpression
    | CallExpression
    | MethodCallExpression
    | FieldExpression
    | ClosureExpression
    | AsyncBlockExpression
    | ContinueExpression
    | BreakExpression
    | RangeExpression
    | ReturnExpression
    | UnderscoreExpression
    | MacroInvocation

ExpressionWithBlock ->
    OuterAttribute* ExpressionWithBlockNoAttrs

ExpressionWithBlockNoAttrs ->
      BlockExpression
    | ConstBlockExpression
    | UnsafeBlockExpression
    | LoopExpression
    | IfExpression
    | MatchExpression
```

r[expr.intro]
表达式可以有两种角色：它总是产生一个*值*，并且可能具有*效果*（也称为"副作用"）。

r[expr.evaluation]
表达式*求值为*一个值，并在*求值*期间产生效果。

r[expr.operands]
许多表达式包含子表达式，称为表达式的*操作数*。

r[expr.behavior]
每种表达式的含义决定了以下几件事：

* 在求值表达式时是否要求值其操作数
* 求值操作数的顺序
* 如何组合操作数的值以获得表达式的值

r[expr.structure]
这样，表达式的结构就决定了执行的结构。块只是另一种表达式，因此块、语句、表达式和块可以相互递归嵌套，达到任意深度。

> [!NOTE]
> 我们为表达式的操作数命名以便讨论，但这些名称并不稳定，可能会发生变化。

r[expr.precedence]
## 表达式优先级

Rust 运算符和表达式的优先级按从强到弱的顺序排列如下。处于同一优先级的二元运算符按其结合性给定的顺序分组。

| 运算符/表达式                  | 结合性              |
|--------------------------------|---------------------|
| [路径][expr.path]              |                     |
| [方法调用][expr.method]        |                     |
| [字段表达式][expr.field]       | 从左到右            |
| [函数调用][expr.call]、[数组索引][expr.array.index] |       |
| [`?`][expr.try]                |                     |
| 一元 [`-`][expr.negate] [`!`][expr.negate] [`*`][expr.deref] [借用][expr.operator.borrow] | |
| [`as`][expr.as]                | 从左到右            |
| [`*`][expr.arith-logic] [`/`][expr.arith-logic] [`%`][expr.arith-logic] | 从左到右    |
| [`+`][expr.arith-logic] [`-`][expr.arith-logic] | 从左到右      |
| [`<<`][expr.arith-logic] [`>>`][expr.arith-logic] | 从左到右    |
| [`&`][expr.arith-logic]        | 从左到右            |
| [`^`][expr.arith-logic]        | 从左到右            |
| [<code>&#124;</code>][expr.arith-logic] | 从左到右     |
| [`==`][expr.cmp] [`!=`][expr.cmp] [`<`][expr.cmp] [`>`][expr.cmp] [`<=`][expr.cmp] [`>=`][expr.cmp] | 需要括号 |
| [`&&`][expr.bool-logic]        | 从左到右            |
| [<code>&#124;&#124;</code>][expr.bool-logic] | 从左到右     |
| [`..`][expr.range] [`..=`][expr.range] | 需要括号        |
| [`=`][expr.assign] [`+=`][expr.compound-assign] [`-=`][expr.compound-assign] [`*=`][expr.compound-assign] [`/=`][expr.compound-assign] [`%=`][expr.compound-assign] <br> [`&=`][expr.compound-assign] [<code>&#124;=</code>][expr.compound-assign] [`^=`][expr.compound-assign] [`<<=`][expr.compound-assign] [`>>=`][expr.compound-assign] | 从右到左 |
| [`return`][expr.return] [`break`][expr.loop.break] [闭包][expr.closure]  | |

r[expr.operand-order]
## 操作数的求值顺序

r[expr.operand-order.default]
以下列表中的表达式都以相同的方式求值其操作数，如下文所述。其他表达式要么不接收操作数，要么根据其各自页面的描述有条件地求值。

* 解引用表达式
* 错误传播表达式
* 取反表达式
* 算术和逻辑二元运算符
* 比较运算符
* 类型转换表达式
* 分组表达式
* 数组表达式
* Await 表达式
* 索引表达式
* 元组表达式
* 元组索引表达式
* 结构体表达式
* 调用表达式
* 方法调用表达式
* 字段表达式
* Break 表达式
* 区间表达式
* Return 表达式

r[expr.operand-order.operands-before-primary]
这些表达式的操作数在应用表达式效果之前被求值。接受多个操作数的表达式按源代码中从左到右的顺序求值。

> [!NOTE]
> 哪些子表达式是表达式的操作数由前一节中的表达式优先级决定。

例如，两个 `next` 方法调用将始终以相同的顺序被调用：

```rust
# // 使用 vec 而不是数组以避免引用
# // 因为在编写此示例时尚无稳定的自有数组迭代器
# // 在编写此示例时。
let mut one_two = vec![1, 2].into_iter();
assert_eq!(
    (1, 2),
    (one_two.next().unwrap(), one_two.next().unwrap())
);
```

> [!NOTE]
> 由于这是递归应用的，这些表达式也从最内层到最外层求值，忽略兄弟节点，直到没有内部子表达式为止。

r[expr.place-value]
## 位置表达式和值表达式

r[expr.place-value.intro]
表达式分为两大类：位置表达式和值表达式；还有第三类次要的表达式，称为赋值目标表达式。在每个表达式内部，操作数同样可以出现在位置上下文或值上下文中。表达式的求值取决于其自身的类别以及它所处的上下文。

r[expr.place-value.place-memory-location]
*位置表达式*是表示内存位置的表达式。

r[expr.place-value.place-expr-kinds]
这些表达式包括引用局部变量的[路径]、[静态变量]、[解引用][deref]（`*expr`）、[数组索引]表达式（`expr[expr]`）、[字段]引用（`expr.f`）和带括号的位置表达式。

r[expr.place-value.value-expr-kinds]
所有其他表达式都是值表达式。

r[expr.place-value.value-result]
*值表达式*是表示实际值的表达式。

r[expr.place-value.place-context]
以下上下文是*位置表达式*上下文：

* [复合赋值]表达式的左操作数。
* 一元[借用]、[裸借用]或[解引用][deref]运算符的操作数。
* [字段表达式]的操作数。
* [数组索引表达式]的索引操作数。
* [元组索引表达式]的元组操作数。
* 任何[隐式借用]的操作数。
* [let 语句]的初始化器。
* [`if let`]、[`match`][match] 或 [`while let`] 表达式的[受检者]。
* [函数式更新]结构体表达式的基值。

> [!NOTE]
> 历史上，位置表达式曾被称为*lvalues*，值表达式曾被称为*rvalues*。

r[expr.place-value.assignee]
*赋值目标表达式*是出现在[赋值][assign]表达式左操作数中的表达式。具体来说，赋值目标表达式包括：

- 位置表达式。
- [下划线]。
- 由赋值目标表达式构成的[元组]。
- 由赋值目标表达式构成的[切片][expr.array.index]。
- 由赋值目标表达式构成的[元组结构体]。
- 由赋值目标表达式构成的[结构体]（可带命名字段）。
- [单元结构体]

r[expr.place-value.parenthesis]
在赋值目标表达式内部允许任意加括号。

r[expr.move]
### 移动和复制类型

r[expr.move.intro]
当位置表达式在值表达式上下文中求值，或在模式中以值绑定时，它表示该内存位置中*持有的*值。

r[expr.move.copy]
如果该值的类型实现了 [`Copy`]，则该值将被复制。

r[expr.move.requires-sized]
在其余情况下，如果该类型是 [`Sized`]，则可能可以移动该值。

r[expr.move.movable-place]
只有以下位置表达式可以被移出：

* 当前未被借用的[变量]。
* [临时值](#temporaries)。
* 可以被移出且未实现 [`Drop`] 的位置表达式的[字段][field]。
* 对类型为 [`Box<T>`] 且也可以被移出的表达式的[解引用][deref]结果。

r[expr.move.deinitialization]
从求值为局部变量的位置表达式中移出后，该位置被反初始化，在重新初始化之前不能再被读取。

r[expr.move.place-invalid]
在所有其他情况下，尝试在值表达式上下文中使用位置表达式是错误的。

r[expr.mut]
### 可变性

r[expr.mut.intro]
要使一个位置表达式能够被[赋值][assign]、可变[借用][borrow]、[隐式可变借用]或绑定到包含 `ref mut` 的模式，它必须是*可变的*。我们称这些为*可变位置表达式*。相反，其他位置表达式称为*不可变位置表达式*。

r[expr.mut.valid-places]
以下表达式可以是可变位置表达式上下文：

* 当前未被借用的可变[变量]。
* [可变 `static` 项]。
* [临时值]。
* [字段][field]：这在可变位置表达式上下文中求值子表达式。
* 对 `*mut T` 指针的[解引用][deref]。
* 对类型为 `&mut T` 的变量或变量的字段的解引用。注意：这是下一条规则要求的例外。
* 对实现了 `DerefMut` 的类型的解引用：这要求被解引用的值在可变位置表达式上下文中求值。
* 对实现了 `IndexMut` 的类型的[数组索引]：这在可变位置表达式上下文中求值被索引的值，但不求值索引。

r[expr.temporary]
### 临时值

在大多数位置表达式上下文中使用值表达式时，会创建一个临时的无名内存位置并初始化为该值。表达式求值为该位置，除非被[提升]为 `static`。临时值的[丢弃作用域]通常是包围语句的末尾。

r[expr.super-macros]
### 超级宏

r[expr.super-macros.intro]
某些内置宏可能会创建[临时值]，其[作用域][temporary scopes]可以被[延长]。这些临时值是*超级临时值*，这些宏是*超级宏*。这些宏的[调用][macro invocations]是*超级宏调用表达式*。这些宏的参数可以是*超级操作数*。

> [!NOTE]
> 当超级宏调用表达式是[延长表达式]时，其超级操作数是[延长表达式]，并且超级临时值的[作用域][temporary scopes]被[延长]。参见 [destructors.scope.lifetime-extension.exprs]。

r[expr.super-macros.format_args]
#### `format_args!`

r[expr.super-macros.format_args.super-operands]
除格式字符串参数外，传递给 [`format_args!`] 的所有参数都是*超级操作数*。

```rust,edition2024
# fn temp() -> String { String::from("") }
// 由于该调用是延长表达式且参数是超级操作数，
// 内部块是延长表达式，因此在其尾部表达式中创建的
// 临时值的作用域被延长。
let _ = format_args!("{}", { &temp() }); // OK
```

r[expr.super-macros.format_args.super-temporaries]
[`format_args!`] 的超级操作数被[隐式借用]，因此是[位置表达式上下文]。当传递[值表达式]作为参数时，它创建一个*超级临时值*。

```rust
# fn temp() -> String { String::from("") }
let x = format_args!("{}", temp());
x; // <-- 临时值被延长了，允许在此处使用。
```

[`format_args!`] 调用的展开有时会创建其他内部的*超级临时值*。

```rust,compile_fail,E0716
let x = {
    // 此调用创建一个内部临时值。
    let x = format_args!("{:?}", 0);
    x // <-- 临时值被延长了，允许在此处使用。
}; // <-- 临时值在此处被丢弃。
x; // 错误
```

```rust
// 此调用不创建内部临时值。
let x = { let x = format_args!("{}", 0); x };
x; // OK
```

> [!NOTE]
> [`format_args!`] 何时创建或不创建内部临时值的细节目前尚未规定。

r[expr.super-macros.pin]
#### `pin!`

r[expr.super-macros.pin.super-operands]
[`pin!`] 的参数是*超级操作数*。

```rust,edition2024
# use core::pin::pin;
# fn temp() {}
// 与上面的 `format_args!` 相同。
let _ = pin!({ &temp() }); // OK
```

r[expr.super-macros.pin.super-temporaries]
[`pin!`] 的参数是[值表达式上下文]，并创建一个*超级临时值*。

```rust
# use core::pin::pin;
# fn temp() {}
// 参数被求值为一个超级临时值。
let x = pin!(temp());
// 临时值被延长了，允许在此处使用。
x; // OK
```

r[expr.implicit-borrow]
### 隐式借用

r[expr.implicit-borrow-intro]
某些表达式会将一个表达式视为位置表达式，通过隐式借用它。例如，可以直接比较两个非固定大小的[切片][slice]是否相等，因为 `==` 运算符会隐式借用其操作数：

```rust
# let c = [1, 2, 3];
# let d = vec![1, 2, 3];
let a: &[i32];
let b: &[i32];
# a = &c;
# b = &d;
// ...
*a == *b;
// 等价形式：
::std::cmp::PartialEq::eq(&*a, &*b);
```

r[expr.implicit-borrow.application]
隐式借用可能在以下表达式中发生：

* [方法调用][method-call]表达式中的左操作数。
* [字段][field]表达式中的左操作数。
* [调用表达式]中的左操作数。
* [数组索引]表达式中的左操作数。
* [解引用运算符][deref]（`*`）的操作数。
* [比较]的操作数。
* [复合赋值]的左操作数。
* 传递给 [`format_args!`] 的参数（格式字符串除外）。

r[expr.overload]
## 重载 trait

以下许多运算符和表达式也可以使用 `std::ops` 或 `std::cmp` 中的 trait 为其他类型进行重载。这些 trait 也以相同的名称存在于 `core::ops` 和 `core::cmp` 中。

r[expr.attr]
## 表达式属性

r[expr.attr.restriction]
表达式前允许[外部属性]的情况仅限于以下几种：

* 用作[语句]的表达式之前。
* [数组表达式]、[元组表达式]、[调用表达式]和元组式[结构体]表达式的元素。
* [块表达式]的尾部表达式。
<!-- Keep list in sync with block-expr.md -->

r[expr.attr.never-before]
绝不允许在以下表达式之前：
* [区间][Range]表达式。
* 二元运算符表达式（[ArithmeticOrLogicalExpression]、[ComparisonExpression]、[LazyBooleanExpression]、[TypeCastExpression]、[AssignmentExpression]、[CompoundAssignmentExpression]）。

[`Box<T>`]:             special-types-and-traits.md#boxt
[`Copy`]:               special-types-and-traits.md#copy
[`Drop`]:               special-types-and-traits.md#drop
[`if let`]:             expressions/if-expr.md#if-let-patterns
[`format_args!`]:       core::format_args
[`pin!`]:               core::pin::pin
[`Sized`]:              special-types-and-traits.md#sized
[`while let`]:          expressions/loop-expr.md#while-let-patterns
[array expressions]:    expressions/array-expr.md
[array indexing]:       expressions/array-expr.md#array-and-slice-indexing-expressions
[array indexing expression]: expr.array.index
[assign]:               expressions/operator-expr.md#assignment-expressions
[block expressions]:    expressions/block-expr.md
[borrow]:               expressions/operator-expr.md#borrow-operators
[call expressions]:     expressions/call-expr.md
[comparison]:           expressions/operator-expr.md#comparison-operators
[compound assignment]:  expressions/operator-expr.md#compound-assignment-expressions
[deref]:                expressions/operator-expr.md#the-dereference-operator
[destructors]:          destructors.md
[drop scope]:           destructors.md#drop-scopes
[extended]:             destructors.scope.lifetime-extension
[extending expression]: destructors.scope.lifetime-extension.exprs
[extending expressions]: destructors.scope.lifetime-extension.exprs
[field]:                expressions/field-expr.md
[field expression]:     expr.field
[functional update]:    expressions/struct-expr.md#functional-update-syntax
[implicit borrow]:      #implicit-borrows
[implicitly borrowed]:  expr.implicit-borrow
[implicitly mutably borrowed]: #implicit-borrows
[interior mutability]:  interior-mutability.md
[let statement]:        statements.md#let-statements
[macro invocations]:    macro.invocation
[match]:                expressions/match-expr.md
[method-call]:          expressions/method-call-expr.md
[Mutable `static` items]: items/static-items.md#mutable-statics
[Outer attributes]:     attributes.md
[paths]:                expressions/path-expr.md
[place expression contexts]: expr.place-value
[promoted]:             destructors.md#constant-promotion
[Range]:                expressions/range-expr.md
[raw borrow]:           expressions/operator-expr.md#raw-borrow-operators
[scrutinee]:            glossary.md#scrutinee
[slice]:                types/slice.md
[statement]:            statements.md
[static variables]:     items/static-items.md
[struct]:               expressions/struct-expr.md
[Structs]:              expr.struct
[temporaries]:          expr.temporary
[temporary scopes]:     destructors.scope.temporary
[Temporary values]:     #temporaries
[tuple expressions]:    expressions/tuple-expr.md
[tuple indexing expression]: expr.tuple-index
[Tuple structs]:        items.struct.tuple
[Tuples]:               expressions/tuple-expr.md
[Underscores]:          expressions/underscore-expr.md
[Unit structs]:         items.struct.unit
[value expression context]: expr.place-value
[value expression]:     expr.place-value
[Variables]:            variables.md
