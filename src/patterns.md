r[patterns]
# 模式

r[patterns.syntax]
```grammar,patterns
Pattern -> `|`? PatternNoTopAlt  ( `|` PatternNoTopAlt )*

PatternNoTopAlt ->
      PatternWithoutRange
    | RangePattern

PatternWithoutRange ->
      LiteralPattern
    | IdentifierPattern
    | WildcardPattern
    | RestPattern
    | ReferencePattern
    | StructPattern
    | TupleStructPattern
    | TuplePattern
    | GroupedPattern
    | SlicePattern
    | PathPattern
    | MacroInvocation
```

r[patterns.intro]
模式用于将值与结构进行匹配，并可选择性地将变量绑定到这些结构中的值。它们也用于变量声明以及函数和闭包的参数。

下面示例中的模式做了四件事：

* 测试 `person` 是否有 `car` 字段且填充了某些内容。
* 测试 `person` 的 `age` 字段是否在 13 和 19 之间，并将其值绑定到 `person_age` 变量。
* 将对 `name` 字段的引用绑定到变量 `person_name`。
* 忽略 `person` 的其余字段。剩余的字段可以有任意值，且不绑定到任何变量。

```rust
# struct Car;
# struct Computer;
# struct Person {
#     name: String,
#     car: Option<Car>,
#     computer: Option<Computer>,
#     age: u8,
# }
# let person = Person {
#     name: String::from("John"),
#     car: Some(Car),
#     computer: None,
#     age: 15,
# };
if let
    Person {
        car: Some(_),
        age: person_age @ 13..=19,
        name: ref person_name,
        ..
    } = person
{
    println!("{} has a car and is {} years old.", person_name, person_age);
}
```

r[patterns.use]
模式用于：

r[patterns.let]
* [`let` 声明](statements.md#let-statements)

r[patterns.param]
* [函数](items/functions.md)和[闭包](expressions/closure-expr.md)参数

r[patterns.match]
* [`match` 表达式](expressions/match-expr.md)

r[patterns.if-let]
* [`if let` 表达式](expressions/if-expr.md)

r[patterns.while-let]
* [`while let` 表达式](expressions/loop-expr.md#while-let-patterns)

r[patterns.for]
* [`for` 表达式](expressions/loop-expr.md#iterator-loops)

r[patterns.destructure]
## 解构

r[patterns.destructure.intro]
模式可用于*解构* [structs]、[enums] 和 [tuples]。解构将一个值分解为其组成部分。使用的语法几乎与创建此类值时的语法相同。

r[patterns.destructure.wildcard]
在[被检查值][scrutinee]表达式具有 `struct`、`enum` 或 `tuple` 类型的模式中，[通配符模式](#wildcard-pattern)（`_`）代表*单个*数据字段，而[省略模式][grammar-StructPatternEtCetera]或[剩余模式][patterns.rest]（`..`）代表特定变体的*所有*剩余字段。

r[patterns.destructure.named-field-shorthand]
在解构具有命名（而非编号）字段的数据结构时，允许使用 `fieldname` 作为 `fieldname: fieldname` 的简写形式。

```rust
# enum Message {
#     Quit,
#     WriteString(String),
#     Move { x: i32, y: i32 },
#     ChangeColor(u8, u8, u8),
# }
# let message = Message::Quit;
match message {
    Message::Quit => println!("Quit"),
    Message::WriteString(write) => println!("{}", &write),
    Message::Move{ x, y: 0 } => println!("move {} horizontally", x),
    Message::Move{ .. } => println!("other move"),
    Message::ChangeColor { 0: red, 1: green, 2: _ } => {
        println!("color change, red: {}, green: {}", red, green);
    }
};
```

r[patterns.refutable]
## 可反驳性

当一个模式有可能不被所要匹配的值匹配时，称该模式是*可反驳的*（refutable）。而*不可反驳*的模式则始终与其匹配的值匹配。示例：

```rust
let (x, y) = (1, 2);               // "(x, y)" 是不可反驳的模式

if let (a, 3) = (1, 2) {           // "(a, 3)" 是可反驳的，不会匹配
    panic!("Shouldn't reach here");
} else if let (a, 4) = (3, 4) {    // "(a, 4)" 是可反驳的，会匹配
    println!("Matched ({}, 4)", a);
}
```

r[patterns.literal]
## 字面量模式

r[patterns.literal.syntax]
```grammar,patterns
LiteralPattern -> `-`? LiteralExpression
```

r[patterns.literal.intro]
*字面量模式*精确匹配与字面量创建的值相同的值。由于负数不是[字面量][literals]，模式中的字面量可以带有可选的前导负号，相当于取反运算符。

> [!WARNING]
> C 字符串和原生 C 字符串字面量在字面量模式中被接受，但 `&CStr` 没有实现结构相等性（`#[derive(Eq, PartialEq)]`），因此任何对 `&CStr` 的此类 `match` 都会因类型错误而被拒绝。

r[patterns.literal.refutable]
字面量模式始终是可反驳的。

示例：

```rust
for i in -2..5 {
    match i {
        -1 => println!("It's minus one"),
        1 => println!("It's a one"),
        2|4 => println!("It's either a two or a four"),
        _ => println!("Matched none of the arms"),
    }
}
```

r[patterns.ident]
## 标识符模式

r[patterns.ident.syntax]
```grammar,patterns
IdentifierPattern -> `ref`? `mut`? IDENTIFIER ( `@` PatternNoTopAlt )?
```

r[patterns.ident.intro]
标识符模式将其匹配的值绑定到[值命名空间][value namespace]中的一个变量。

r[patterns.ident.unique]
标识符在模式内必须唯一。

r[patterns.ident.scope]
该变量将遮蔽作用域中任何同名的变量。新绑定的[作用域][scope]取决于使用该模式的上下文（如 `let` 绑定或 `match` 分支）。

r[patterns.ident.bare]
仅由一个标识符组成的模式（可能带有 `mut`）匹配任何值并将其绑定到该标识符。这是变量声明以及函数和闭包参数中最常用的模式。

```rust
let mut variable = 10;
fn sum(x: i32, y: i32) -> i32 {
#    x + y
# }
```

r[patterns.ident.scrutinized]
要将匹配到的值绑定到一个变量，可以使用语法 `variable @ subpattern`。例如，以下代码将值 2 绑定到 `e`（不是整个范围：这里的范围是一个范围子模式）。

```rust
let x = 2;

match x {
    e @ 1 ..= 5 => println!("got a range element {}", e),
    _ => println!("anything"),
}
```

r[patterns.ident.move]
默认情况下，标识符模式将变量绑定为匹配值的副本或从匹配值移动，这取决于匹配值是否实现 [`Copy`]。

r[patterns.ident.ref]
可以通过使用 `ref` 关键字将其改为绑定到引用，或使用 `ref mut` 绑定到可变引用。例如：

```rust
# let a = Some(10);
match a {
    None => (),
    Some(value) => (),
}

match a {
    None => (),
    Some(ref value) => (),
}
```

在第一个 match 表达式中，值被复制（或移动）。在第二个 match 中，对同一内存位置的引用被绑定到变量 value。这种语法是必要的，因为在解构子模式中，`&` 运算符不能应用于值的字段。例如，以下代码是无效的：

```rust,compile_fail
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: &person_name, age: 18..=150 } = value { }
```

要使其有效，请这样写：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let value = Person { name: String::from("John"), age: 23 };
if let Person { name: ref person_name, age: 18..=150 } = value { }
```

r[patterns.ident.ref-ignored]
因此，`ref` 不是某种被匹配的东西。它的目标仅仅是使匹配的绑定成为引用，而不是潜在地复制或移动匹配到的内容。

r[patterns.ident.precedent]
[路径模式](#path-patterns)优先于标识符模式。

> [!NOTE]
> 当模式是单段标识符时，语法存在歧义：它是 [IdentifierPattern] 还是 [PathPattern]。此歧义只能在[名称解析][name resolution]之后解决。
>
> ```rust
> const EXPECTED_VALUE: u8 = 42;
> //    ^^^^^^^^^^^^^^ 这个常量在作用域中会影响下面模式的处理方式。
>
> fn check_value(x: u8) -> Result<u8, u8> {
>     match x {
>         EXPECTED_VALUE => Ok(x),
>     //  ^^^^^^^^^^^^^^ 解析为 `PathPattern`，解析到常量 `42`。
>         other_value => Err(x),
>     //  ^^^^^^^^^^^ 解析为 `IdentifierPattern`。
>     }
> }
>
> // 如果 `EXPECTED_VALUE` 在上面被当作 `IdentifierPattern`，
> // 那么该模式将始终匹配，使得函数无论输入如何都返回 `Ok(_)`。
> assert_eq!(check_value(42), Ok(42));
> assert_eq!(check_value(43), Err(43));
> ```

r[patterns.ident.constraint]
如果指定了 `ref` 或 `ref mut` 且标识符遮蔽了一个常量，则这是一个错误。

r[patterns.ident.refutable]
如果 `@` 子模式不可反驳或未指定子模式，则标识符模式是不可反驳的。

r[patterns.ident.binding]
### 绑定模式

r[patterns.ident.binding.intro]
为了提供更好的人体工学，模式在不同的*绑定模式*下运作，以便更容易地将引用绑定到值。当引用值被非引用模式匹配时，它将被自动处理为 `ref` 或 `ref mut` 绑定。示例：

```rust
let x: &Option<i32> = &Some(3);
if let Some(y) = x {
    // y 被转换为 `ref y`，其类型为 &i32
}
```

r[patterns.ident.binding.non-reference]
*非引用模式*包括除绑定、[通配符模式](#wildcard-pattern)（`_`）、引用类型的 [`const` 模式](#path-patterns)以及[引用模式](#reference-patterns)之外的所有模式。

r[patterns.ident.binding.default-mode]
如果绑定模式没有显式地带有 `ref`、`ref mut` 或 `mut`，则它使用*默认绑定模式*来决定变量如何绑定。

r[patterns.ident.binding.move]
默认绑定模式从 "move" 模式开始，即使用移动语义。

r[patterns.ident.binding.top-down]
在匹配模式时，编译器从模式的外部开始向内工作。

r[patterns.ident.binding.auto-deref]
每次一个引用被非引用模式匹配时，它会自动解引用该值并更新默认绑定模式。

r[patterns.ident.binding.ref]
引用会将默认绑定模式设置为 `ref`。

r[patterns.ident.binding.ref-mut]
可变引用会将模式设置为 `ref mut`，除非模式已经是 `ref`，此时它保持为 `ref`。

r[patterns.ident.binding.nested-references]
如果自动解引用后的值仍然是一个引用，则继续解引用，此过程会重复。

r[patterns.ident.binding.mode-limitations-binding]
只有当默认绑定模式为 "move" 时，绑定模式才能显式指定 `ref` 或 `ref mut` 绑定模式，或用 `mut` 指定可变性。例如，以下代码不被接受：

```rust,edition2024,compile_fail
let [mut x] = &[()]; //~ 错误
let [ref x] = &[()]; //~ 错误
let [ref mut x] = &mut [()]; //~ 错误
```

r[patterns.ident.binding.mode-limitations.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，即使默认绑定模式不是 "move"，绑定也可以显式指定 `ref` 或 `ref mut` 绑定模式，并且可以在此类绑定上用 `mut` 指定可变性。在这些版次中，在绑定上指定 `mut` 会将绑定模式设置为 "move"，无论当前的默认绑定模式是什么。

r[patterns.ident.binding.mode-limitations-reference]
类似地，引用模式只能在默认绑定模式为 "move" 时出现。例如，以下代码不被接受：

```rust,edition2024,compile_fail
let [&x] = &[&()]; //~ 错误
```

r[patterns.ident.binding.mode-limitations-reference.edition2024]
> [!EDITION-2024]
> 在 2024 版之前，即使默认绑定模式不是 "move"，引用模式也可以出现，并且同时具有匹配被检查值的效果和将默认绑定模式重置为 "move" 的效果。

r[patterns.ident.binding.mixed]
移动绑定和引用绑定可以在同一模式中混用。这样做会导致对所绑定对象的部分移动，且该对象在此之后不能使用。这仅适用于类型不能复制的情况。

在下面的示例中，`name` 从 `person` 移出。尝试将 `person` 整体或 `person.name` 使用将导致错误，因为发生了*部分移动*。

示例：

```rust
# struct Person {
#    name: String,
#    age: u8,
# }
# let person = Person{ name: String::from("John"), age: 23 };
// `name` 从 person 移动，`age` 被引用
let Person { name, ref age } = person;
```

r[patterns.wildcard]
## 通配符模式

r[patterns.wildcard.syntax]
```grammar,patterns
WildcardPattern -> `_`
```

r[patterns.wildcard.intro]
*通配符模式*（下划线符号）匹配任何值。用于在值无关紧要时忽略它们。

r[patterns.wildcard.struct-matcher]
在其他模式内部，它匹配单个数据字段（与 `..` 相反，后者匹配剩余字段）。

r[patterns.wildcard.no-binding]
与标识符模式不同，它不会复制、移动或借用它所匹配的值。

示例：

```rust
# let x = 20;
let (a, _) = (10, x);   // x 总是被 _ 匹配
# assert_eq!(a, 10);

// 忽略函数/闭包参数
let real_part = |a: f64, _: f64| { a };

// 忽略结构体的一个字段
# struct RGBA {
#    r: f32,
#    g: f32,
#    b: f32,
#    a: f32,
# }
# let color = RGBA{r: 0.4, g: 0.1, b: 0.9, a: 0.5};
let RGBA{r: red, g: green, b: blue, a: _} = color;
# assert_eq!(color.r, red);
# assert_eq!(color.g, green);
# assert_eq!(color.b, blue);

// 接受任何 Some，其中包含任意值
# let x = Some(10);
if let Some(_) = x {}
```

r[patterns.wildcard.refutable]
通配符模式始终是不可反驳的。

r[patterns.rest]
## 剩余模式

r[patterns.rest.syntax]
```grammar,patterns
RestPattern -> `..`
```

r[patterns.rest.intro]
*剩余模式*（`..` token）充当可变长度模式，匹配零个或多个之前和之后尚未被匹配的元素。

r[patterns.rest.allowed-patterns]
它只能用于[元组](#tuple-patterns)、[元组结构体](#tuple-struct-patterns)和[切片](#slice-patterns)模式，并且在这些模式中只能作为元素之一出现一次。它也允许在仅用于[切片模式](#slice-patterns)的[标识符模式](#identifier-patterns)中。

r[patterns.rest.refutable]
剩余模式始终是不可反驳的。

示例：

```rust
# let words = vec!["a", "b", "c"];
# let slice = &words[..];
match slice {
    [] => println!("slice is empty"),
    [one] => println!("single element {}", one),
    [head, tail @ ..] => println!("head={} tail={:?}", head, tail),
}

match slice {
    // 忽略除最后一个元素之外的所有内容，最后一个元素必须是 "!"。
    [.., "!"] => println!("!!!"),

    // `start` 是除最后一个元素（必须是 "z"）之外的所有内容的切片。
    [start @ .., "z"] => println!("starts with: {:?}", start),

    // `end` 是除第一个元素（必须是 "a"）之外的所有内容的切片。
    ["a", end @ ..] => println!("ends with: {:?}", end),

    // 'whole' 是整个切片，`last` 是最终元素
    whole @ [.., last] => println!("the last element of {:?} is {}", whole, last),

    rest => println!("{:?}", rest),
}

if let [.., penultimate, _] = slice {
    println!("next to last is {}", penultimate);
}

# let tuple = (1, 2, 3, 4, 5);
// 剩余模式也可以用于元组和元组结构体模式。
match tuple {
    (1, .., y, z) => println!("y={} z={}", y, z),
    (.., 5) => println!("tail must be 5"),
    (..) => println!("matches everything else"),
}
```

r[patterns.range]
## 范围模式

r[patterns.range.syntax]
```grammar,patterns
RangePattern ->
      RangeExclusivePattern
    | RangeInclusivePattern
    | RangeFromPattern
    | RangeToExclusivePattern
    | RangeToInclusivePattern
    | ObsoleteRangePattern[^obsolete-range-edition]

RangeExclusivePattern ->
      RangePatternBound `..` RangePatternBound

RangeInclusivePattern ->
      RangePatternBound `..=` RangePatternBound

RangeFromPattern ->
      RangePatternBound `..`

RangeToExclusivePattern ->
      `..` RangePatternBound

RangeToInclusivePattern ->
      `..=` RangePatternBound

ObsoleteRangePattern ->
    RangePatternBound `...` RangePatternBound

RangePatternBound ->
      LiteralPattern
    | PathExpression
```

[^obsolete-range-edition]: [ObsoleteRangePattern] 语法已在 2021 版中移除。

r[patterns.range.intro]
*范围模式*匹配由其边界定义的范围内的标量值。它们由*标记符号*（`..` 或 `..=`）以及一侧或两侧的边界组成。

标记符号左侧的边界称为*下界*。右侧的边界称为*上界*。

r[patterns.range.exclusive]
*排除范围模式*匹配从下界开始到上界（但不包含上界）的所有值。它的书写形式是下界，后跟 `..`，再跟在上界。

例如，模式 `'m'..'p'` 仅匹配 `'m'`、`'n'` 和 `'o'`，特别地**不**包括 `'p'`。

r[patterns.range.inclusive]
*包含范围模式*匹配从下界开始到上界（包含上界）的所有值。它的书写形式是下界，后跟 `..=`，再跟上界。

例如，模式 `'m'..='p'` 仅匹配值 `'m'`、`'n'`、`'o'` 和 `'p'`。

r[patterns.range.from]
*从范围模式*匹配所有大于等于下界的值。它的书写形式是下界后跟 `..`。

例如，`1..` 将匹配任何大于等于 1 的整数，如 1、9 或 9001，或 9007199254740991（如果其大小合适），但不匹配 0，对于有符号整数也不匹配负数。

r[patterns.range.to-exclusive]
*到排除范围模式*匹配所有小于上界的值。它的书写形式是 `..` 后跟上界。

例如，`..10` 将匹配任何小于 10 的整数，如 9、1、0，对于有符号整数类型还包括所有负数。

r[patterns.range.to-inclusive]
*到包含范围模式*匹配所有小于等于上界的值。它的书写形式是 `..=` 后跟上界。

例如，`..=10` 将匹配任何小于等于 10 的整数，如 10、1、0，对于有符号整数类型还包括所有负数。

r[patterns.range.constraint-nonempty]
范围模式必须非空；它必须在其类型的可能值集合中跨越至少一个值。换句话说：

* 在 `a..=b` 中，必须满足 a &le; b。例如，范围模式 `10..=0` 是错误的，但 `10..=10` 是允许的。
* 在 `a..b` 中，必须满足 a &lt; b。例如，范围模式 `10..0` 或 `10..10` 是错误的。
* 在 `..b` 中，b 不能是其类型的最小值。例如，范围模式 `..-128i8` 或 `..f64::NEG_INFINITY` 是错误的。

r[patterns.range.bound]
边界书写为以下之一：

* 字符、字节、整数或浮点字面量。
* 一个 `-` 后跟整数或浮点字面量。
* 一个[路径][path]。

> [!NOTE]
>
> 我们在语法上接受的比 *[RangePatternBound]* 更多。我们稍后会在语义上拒绝其他内容。

r[patterns.range.constraint-bound-path]
如果边界书写为路径，在宏解析之后，该路径必须解析为类型为 `char`、整数类型或浮点类型的常量项。

r[patterns.range.type]
范围模式匹配其上界和下界的类型，两者必须是相同的类型。

r[patterns.range.path-value]
如果边界是一个[路径][path]，则边界匹配该路径解析到的[常量][constant]的类型并取其值。

r[patterns.range.literal-value]
如果边界是一个字面量，则边界匹配相应[字面量表达式][literal expression]的类型并取其值。

r[patterns.range.negation]
如果边界是一个带有前导 `-` 的字面量，则边界匹配与相应[字面量表达式][literal expression]相同的类型，并取对相应字面量表达式值[取反][negating]后的值。

r[patterns.range.float-restriction]
对于浮点范围模式，常量不能是 `NaN`。

示例：

```rust
# let c = 'f';
let valid_variable = match c {
    'a'..='z' => true,
    'A'..='Z' => true,
    'α'..='ω' => true,
    _ => false,
};

# let ph = 10;
println!("{}", match ph {
    0..7 => "acid",
    7 => "neutral",
    8..=14 => "base",
    _ => unreachable!(),
});

# let uint: u32 = 5;
match uint {
    0 => "zero!",
    1.. => "positive number!",
};

// 使用常量路径：
# const TROPOSPHERE_MIN : u8 = 6;
# const TROPOSPHERE_MAX : u8 = 20;
#
# const STRATOSPHERE_MIN : u8 = TROPOSPHERE_MAX + 1;
# const STRATOSPHERE_MAX : u8 = 50;
#
# const MESOSPHERE_MIN : u8 = STRATOSPHERE_MAX + 1;
# const MESOSPHERE_MAX : u8 = 85;
#
# let altitude = 70;
#
println!("{}", match altitude {
    TROPOSPHERE_MIN..=TROPOSPHERE_MAX => "troposphere",
    STRATOSPHERE_MIN..=STRATOSPHERE_MAX => "stratosphere",
    MESOSPHERE_MIN..=MESOSPHERE_MAX => "mesosphere",
    _ => "outer space, maybe",
});

# pub mod binary {
#     pub const MEGA : u64 = 1024*1024;
#     pub const GIGA : u64 = 1024*1024*1024;
# }
# let n_items = 20_832_425;
# let bytes_per_item = 12;
if let size @ binary::MEGA..=binary::GIGA = n_items * bytes_per_item {
    println!("It fits and occupies {} bytes", size);
}

# trait MaxValue {
#     const MAX: u64;
# }
# impl MaxValue for u8 {
#     const MAX: u64 = (1 << 8) - 1;
# }
# impl MaxValue for u16 {
#     const MAX: u64 = (1 << 16) - 1;
# }
# impl MaxValue for u32 {
#     const MAX: u64 = (1 << 32) - 1;
# }
// 使用限定路径：
println!("{}", match 0xfacade {
    0 ..= <u8 as MaxValue>::MAX => "fits in a u8",
    0 ..= <u16 as MaxValue>::MAX => "fits in a u16",
    0 ..= <u32 as MaxValue>::MAX => "fits in a u32",
    _ => "too big",
});
```

r[patterns.range.refutable]
当固定宽度整数和 `char` 类型的范围模式跨越其类型的整个可能值集合时，它们是不可反驳的。例如，`0u8..=255u8` 是不可反驳的。

r[patterns.range.refutable-integer]
整数类型的值范围是从其最小值到最大值的闭区间。

r[patterns.range.refutable-char]
`char` 类型的值范围恰好是包含所有 Unicode 标量值的范围：`'\u{0000}'..='\u{D7FF}'` 和 `'\u{E000}'..='\u{10FFFF}'`。

r[patterns.range.constraint-slice]
[RangeFromPattern] 不能用作[切片模式](#slice-patterns)中子模式的顶层模式。例如，模式 `[1.., _]` 不是有效的模式。

r[patterns.range.edition2021]
> [!EDITION-2021]
> 在 2021 版之前，同时具有下界和上界的范围模式也可以使用 `...` 代替 `..=` 来书写，含义相同。

r[patterns.ref]
## 引用模式

r[patterns.ref.syntax]
```grammar,patterns
ReferencePattern -> (`&`|`&&`) `mut`? PatternWithoutRange
```

r[patterns.ref.intro]
引用模式解引用正在匹配的指针，从而借用它们。

例如，下面两个对 `x: &i32` 的匹配是等价的：

```rust
let int_reference = &3;

let a = match *int_reference { 0 => "zero", _ => "some" };
let b = match int_reference { &0 => "zero", _ => "some" };

assert_eq!(a, b);
```

r[patterns.ref.ref-ref]
引用模式的文法产生式必须匹配 token `&&` 以匹配引用的引用，因为它本身就是一个 token，而不是两个 `&` token。

r[patterns.ref.mut]
添加 `mut` 关键字会解引用可变引用。可变性必须与引用的可变性匹配。

r[patterns.ref.refutable]
引用模式始终是不可反驳的。

r[patterns.struct]
## 结构体模式

r[patterns.struct.syntax]
```grammar,patterns
StructPattern ->
    PathInExpression `{`
        StructPatternElements?
    `}`

StructPatternElements ->
      StructPatternFields (`,` | `,` StructPatternEtCetera)?
    | StructPatternEtCetera

StructPatternFields ->
    StructPatternField (`,` StructPatternField)*

StructPatternField ->
    OuterAttribute*
    (
        TUPLE_INDEX `:` Pattern
      | IDENTIFIER `:` Pattern
      | `ref`? `mut`? IDENTIFIER
    )

StructPatternEtCetera -> `..`
```

r[patterns.struct.intro]
结构体模式匹配满足其子模式定义的所有条件的结构体、枚举和联合体值。它们也用于[解构](#destructuring)结构体、枚举或联合体值。

r[patterns.struct.ignore-rest]
在结构体模式中，字段可以通过名称、索引（对于元组结构体）来引用，或通过 `..` 忽略：

```rust
# struct Point {
#     x: u32,
#     y: u32,
# }
# let s = Point {x: 1, y: 1};
#
match s {
    Point {x: 10, y: 20} => (),
    Point {y: 10, x: 20} => (),    // 顺序不重要
    Point {x: 10, ..} => (),
    Point {..} => (),
}

# struct PointTuple (
#     u32,
#     u32,
# );
# let t = PointTuple(1, 2);
#
match t {
    PointTuple {0: 10, 1: 20} => (),
    PointTuple {1: 10, 0: 20} => (),   // 顺序不重要
    PointTuple {0: 10, ..} => (),
    PointTuple {..} => (),
}

# enum Message {
#     Quit,
#     Move { x: i32, y: i32 },
# }
# let m = Message::Quit;
#
match m {
    Message::Quit => (),
    Message::Move {x: 10, y: 20} => (),
    Message::Move {..} => (),
}
```

r[patterns.struct.constraint-struct]
如果未使用 `..`，则用于匹配结构体的结构体模式必须指定所有字段：

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let mut struct_value = Struct{a: 10, b: 'X', c: false};
#
match struct_value {
    Struct{a: 10, b: 'X', c: false} => (),
    Struct{a: 10, b: 'X', ref c} => (),
    Struct{a: 10, b: 'X', ref mut c} => (),
    Struct{a: 10, b: 'X', c: _} => (),
    Struct{a: _, b: _, c: _} => (),
}
```

r[patterns.struct.constraint-union]
用于匹配联合体的结构体模式必须恰好指定一个字段（参见[联合体上的模式匹配][Pattern matching on unions]）。

r[patterns.struct.binding-shorthand]
[IDENTIFIER] 语法匹配任何值并将其绑定到与给定字段同名的变量。它是 `fieldname: fieldname` 的简写形式。可以包含 `ref` 和 `mut` 限定符，其行为如 [patterns.ident.ref] 所述。

```rust
# struct Struct {
#    a: i32,
#    b: char,
#    c: bool,
# }
# let struct_value = Struct{a: 10, b: 'X', c: false};
#
let Struct { a, b, c } = struct_value;
```

r[patterns.struct.refutable]
如果 [PathInExpression] 解析为具有多个变体的枚举的构造器，或其某个子模式是可反驳的，则结构体模式是可反驳的。

r[patterns.struct.namespace]
结构体模式在[类型命名空间][type namespace]中匹配其构造器由 [PathInExpression] 解析到的结构体、联合体或枚举变体。更多细节请参见 [patterns.tuple-struct.namespace]。

r[patterns.tuple-struct]
## 元组结构体模式

r[patterns.tuple-struct.syntax]
```grammar,patterns
TupleStructPattern -> PathInExpression `(` TupleStructItems? `)`

TupleStructItems -> Pattern ( `,` Pattern )* `,`?
```

r[patterns.tuple-struct.intro]
元组结构体模式匹配满足其子模式定义的所有条件的元组结构体和枚举值。它们也用于[解构](#destructuring)元组结构体或枚举值。

r[patterns.tuple-struct.refutable]
如果 [PathInExpression] 解析为具有多个变体的枚举的构造器，或其某个子模式是可反驳的，则元组结构体模式是可反驳的。

r[patterns.tuple-struct.namespace]
元组结构体模式在[值命名空间][value namespace]中匹配其构造器由 [PathInExpression] 解析到的元组结构体或[类元组枚举变体][tuple-like enum variant]。

> [!NOTE]
> 反之，用于元组结构体或[类元组枚举变体][tuple-like enum variant]的结构体模式，例如 `S { 0: _ }`，在[类型命名空间][type namespace]中匹配其构造器被解析到的元组结构体或变体。
>
> ```rust,no_run
> enum E1 { V(u16) }
> enum E2 { V(u32) }
>
> // 仅从类型命名空间导入 `E1::V`。
> mod _0 {
>     const V: () = (); // 用于命名空间掩蔽。
>     pub(super) use super::E1::*;
> }
> use _0::*;
>
> // 仅从值命名空间导入 `E2::V`。
> mod _1 {
>     struct V {} // 用于命名空间掩蔽。
>     pub(super) use super::E2::*;
> }
> use _1::*;
>
> fn f() {
>     // 此结构体模式匹配其构造器在类型命名空间中
>     // 找到的类元组枚举变体。
>     let V { 0: ..=u16::MAX } = (loop {}) else { loop {} };
>     // 此元组结构体模式匹配其构造器在值命名空间中
>     // 找到的类元组枚举变体。
>     let V(..=u32::MAX) = (loop {}) else { loop {} };
> }
> # // Required due to the odd behavior of `super` within functions.
> # fn main() {}
> ```
>
> 语言团队已经做出了某些决策，例如 [PR #138458] 中的决策，这引发了对以这种方式在模式中使用值命名空间的合理性的质疑，如 [PR #140593] 中所述。在你的代码中有意依赖这种细微差别可能是不明智的。

r[patterns.tuple]
## 元组模式

r[patterns.tuple.syntax]
```grammar,patterns
TuplePattern -> `(` TuplePatternItems? `)`

TuplePatternItems ->
      Pattern `,`
    | RestPattern
    | Pattern (`,` Pattern)+ `,`?
```

r[patterns.tuple.intro]
元组模式匹配满足其子模式定义的所有条件的元组值。它们也用于[解构](#destructuring)元组。

r[patterns.tuple.rest-syntax]
带有单个 [RestPattern] 的形式 `(..)` 是一种特殊形式，不需要逗号，并且匹配任意大小的元组。

r[patterns.tuple.refutable]
当其某个子模式是可反驳的时，元组模式是可反驳的。

使用元组模式的示例：

```rust
let pair = (10, "ten");
let (a, b) = pair;

assert_eq!(a, 10);
assert_eq!(b, "ten");
```

r[patterns.paren]
## 分组模式

r[patterns.paren.syntax]
```grammar,patterns
GroupedPattern -> `(` Pattern `)`
```

r[patterns.paren.intro]
将模式括在括号中可用于显式控制复合模式的优先级。例如，紧邻范围模式的引用模式（如 `&0..=5`）是歧义的且不被允许，但可以用括号来表达。

```rust
let int_reference = &3;
match int_reference {
    &(0..=5) => (),
    _ => (),
}
```

r[patterns.slice]
## 切片模式

r[patterns.slice.syntax]
```grammar,patterns
SlicePattern -> `[` SlicePatternItems? `]`

SlicePatternItems -> Pattern (`,` Pattern)* `,`?
```

r[patterns.slice.intro]
切片模式既可以匹配固定大小的数组，也可以匹配动态大小的切片。

```rust
// 固定大小
let arr = [1, 2, 3];
match arr {
    [1, _, _] => "starts with one",
    [a, b, c] => "starts with something else",
};
```
```rust
// 动态大小
let v = vec![1, 2, 3];
match v[..] {
    [a, b] => { /* 此分支不会应用，因为长度不匹配 */ }
    [a, b, c] => { /* 此分支会应用 */ }
    _ => { /* 需要此通配符，因为长度在静态时未知 */ }
};
```

r[patterns.slice.refutable-array]
当匹配数组时，如果每个元素都是不可反驳的，则切片模式是不可反驳的。

r[patterns.slice.refutable-slice]
当匹配切片时，仅当形式是单个 `..` [剩余模式][patterns.rest]或带有 `..` 剩余模式作为子模式的[标识符模式](#identifier-patterns)时，它才是不可反驳的。

r[patterns.slice.restriction]
在切片内部，没有同时指定下界和上界的范围模式必须括在括号中，如 `(a..)`，以明确其意图是匹配单个切片元素。同时具有下界和上界的范围模式，如 `a..=b`，不需要括在括号中。

r[patterns.path]
## 路径模式

r[patterns.path.syntax]
```grammar,patterns
PathPattern -> PathExpression
```

r[patterns.path.intro]
*路径模式*是引用常量值或没有字段的结构体或枚举变体的模式。

r[patterns.path.unqualified]
非限定路径模式可以引用：

* 枚举变体
* 结构体
* 常量
* 关联常量

r[patterns.path.qualified]
限定路径模式只能引用关联常量。

r[patterns.path.refutable]
当路径模式引用结构体或只有一个变体的枚举变体，或引用类型不可反驳的常量时，它们是不可反驳的。当引用可反驳的常量或具有多个变体的枚举变体时，它们是可反驳的。

r[patterns.const]
### 常量模式

r[patterns.const.partial-eq]
当类型为 `T` 的常量 `C` 被用作模式时，我们首先检查 `T: PartialEq`。

r[patterns.const.structural-equality]
此外我们要求 `C` 的值*具有（递归）结构相等性*，递归定义如下：

r[patterns.const.primitive]
- 整数以及 `str`、`bool` 和 `char` 值始终具有结构相等性。

r[patterns.const.builtin-aggregate]
- 如果元组、数组和切片的所有字段/元素都具有结构相等性，则它们也具有结构相等性。（特别地，`()` 和 `[]` 始终具有结构相等性。）

r[patterns.const.ref]
- 如果引用指向的值具有结构相等性，则该引用具有结构相等性。

r[patterns.const.aggregate]
- 如果 `struct` 或 `enum` 类型的 `PartialEq` 实例是通过 `#[derive(PartialEq)]` 派生得到的，并且所有字段（对于枚举：活跃变体的字段）具有结构相等性，则该类型的值具有结构相等性。

r[patterns.const.pointer]
- 如果裸指针被定义为常量整数（然后被 cast/transmute），则该裸指针具有结构相等性。

r[patterns.const.float]
- 如果浮点值不是 `NaN`，则它具有结构相等性。

r[patterns.const.exhaustive]
- 其他任何内容都不具有结构相等性。

r[patterns.const.generic]
特别地，`C` 的值必须在模式构建时（即单态化之前）已知。这意味着涉及泛型参数的关联常量不能用作模式。

r[patterns.const.immutable]
`C` 的值不能包含任何对可变静态变量（`static mut` 项或内部可变的 `static` 项）或 `extern` 静态变量的引用。

r[patterns.const.translation]
在确保满足所有条件后，常量值被转换为模式，并且现在其行为完全如同直接书写了该模式一样。特别是，它完全参与穷尽性检查。（对于裸指针，常量是书写此类模式的唯一方式。对于这些类型，只有 `_` 被认为是有穷尽性的。）

r[patterns.or]
## 或模式

*或模式*是匹配两个或多个子模式之一的模式（例如 `A | B | C`）。它们可以任意嵌套。在语法上，或模式允许出现在任何允许其他模式的地方（由 [Pattern] 产生式表示），但 `let` 绑定以及函数和闭包参数除外（由 [PatternNoTopAlt] 产生式表示）。

r[patterns.constraints]
### 静态语义

r[patterns.constraints.pattern]
1. 给定某个深度处的模式 `p | q`，对于任意模式 `p` 和 `q`，如果满足以下条件，则该模式被认为是病态的：

   + 为 `p` 推断的类型与为 `q` 推断的类型不统一，或
   + `p` 和 `q` 中没有引入相同的绑定集合，或
   + `p` 和 `q` 中任何两个具有相同名称的绑定的类型在类型或绑定模式方面不统一。

   在所有上述情况下，类型的统一是精确的，不适用隐式[类型自动强转][type coercions]。

r[patterns.constraints.match-type-check]
2. 在对表达式 `match e_s { a_1 => e_1, ... a_n => e_n }` 进行类型检查时，对于包含形式为 `p_i | q_i` 的模式的每个 match 分支 `a_i`，如果在其存在的深度 `d` 处，深度 `d` 处的 `e_s` 的片段的类型不与 `p_i | q_i` 统一，则该模式 `p_i | q_i` 被认为是病态的。

r[patterns.constraints.exhaustiveness-or-pattern]
3. 关于穷尽性检查，模式 `p | q` 被认为覆盖 `p` 以及 `q`。对于某个构造器 `c(x, ..)`，分配律适用，使得 `c(p | q, ..rest)` 覆盖与 `c(p, ..rest) | c(q, ..rest)` 相同的值集合。这可以递归应用，直到除了存在于顶层之外没有更多形式为 `p | q` 的嵌套模式。

   注意，这里所说的*"构造器"*并非指元组结构体模式，而是指任何乘积类型的模式。这包括枚举变体、元组结构体、具有命名字段的结构体、数组、元组和切片。

r[patterns.behavior]
### 动态语义

r[patterns.behavior.nested-or-patterns]
1. 在深度 `d` 处，将被检查值表达式 `e_s` 与模式 `c(p | q, ..rest)` 进行模式匹配的动态语义（其中 `c` 是某个构造器，`p` 和 `q` 是任意模式，`rest` 可选地是 `c` 中的任何剩余潜在因子）被定义为与 `c(p, ..rest) | c(q, ..rest)` 的动态语义相同。

r[patterns.precedence]
### 与其他无分隔模式之间的优先级

如本章其他部分所示，有几种语法上无分隔的模式类型，包括标识符模式、引用模式和或模式。或模式始终具有最低优先级。这使我们能够为将来可能的类型标注特性预留语法空间，并减少歧义。例如，`x @ A(..) | B(..)` 将导致错误，因为 `x` 并未在所有模式中被绑定。`&A(x) | B(x)` 将导致不同子模式中 `x` 的类型不匹配。

[PR #138458]: https://github.com/rust-lang/rust/pull/138458
[PR #140593]: https://github.com/rust-lang/rust/pull/140593#issuecomment-2972338457
[`Copy`]: special-types-and-traits.md#copy
[constant]: items/constant-items.md
[enums]: items/enumerations.md
[literals]: expressions/literal-expr.md
[literal expression]: expressions/literal-expr.md
[name resolution]: names/name-resolution.md
[negating]: expressions/operator-expr.md#negation-operators
[path]: expressions/path-expr.md
[pattern matching on unions]: items/unions.md#pattern-matching-on-unions
[range expressions]: expressions/range-expr.md
[scope]: names/scopes.md
[structs]: items/structs.md
[tuples]: types/tuple.md
[scrutinee]: glossary.md#scrutinee
[tuple-like enum variant]: items.enum.tuple-expr
[type coercions]: type-coercions.md
[type namespace]: names.namespaces.kinds
[value namespace]: names.namespaces.kinds
