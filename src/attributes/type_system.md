r[attributes.type-system]
# 类型系统属性

以下[属性][attributes]用于改变类型的使用方式。

r[attributes.type-system.non_exhaustive]
## `non_exhaustive` 属性 {#the-non_exhaustive-attribute}

r[attributes.type-system.non_exhaustive.intro]
*`non_exhaustive` 属性*指示一个类型或变体可能在将来添加更多字段或变体。

r[attributes.type-system.non_exhaustive.allowed-positions]
它可以应用于 [`struct`][struct]、[`enum`][enum] 和 `enum` 变体。

r[attributes.type-system.non_exhaustive.syntax]
`non_exhaustive` 属性使用 [MetaWord] 语法，因此不接受任何输入。

r[attributes.type-system.non_exhaustive.same-crate]
在定义的 crate 内部，`non_exhaustive` 没有效果。

```rust
#[non_exhaustive]
pub struct Config {
    pub window_width: u16,
    pub window_height: u16,
}

#[non_exhaustive]
pub struct Token;

#[non_exhaustive]
pub struct Id(pub u64);

#[non_exhaustive]
pub enum Error {
    Message(String),
    Other,
}

pub enum Message {
    #[non_exhaustive] Send { from: u32, to: u32, contents: String },
    #[non_exhaustive] Reaction(u32),
    #[non_exhaustive] Quit,
}

// 非穷尽结构体在定义的 crate 内部可以正常构造。
let config = Config { window_width: 640, window_height: 480 };
let token = Token;
let id = Id(4);

// 非穷尽结构体在定义的 crate 内部可以穷尽地模式匹配。
let Config { window_width, window_height } = config;
let Token = token;
let Id(id_number) = id;

let error = Error::Other;
let message = Message::Reaction(3);

// 非穷尽枚举在定义的 crate 内部可以穷尽地模式匹配。
match error {
    Error::Message(ref s) => { },
    Error::Other => { },
}

match message {
    // 非穷尽变体在定义的 crate 内部可以穷尽地模式匹配。
    Message::Send { from, to, contents } => { },
    Message::Reaction(id) => { },
    Message::Quit => { },
}
```

r[attributes.type-system.non_exhaustive.external-crate]
在定义的 crate 外部，标注了 `non_exhaustive` 的类型有一些限制，以保持添加新字段或变体时的向后兼容性。

r[attributes.type-system.non_exhaustive.construction]
非穷尽类型不能在定义的 crate 外部构造：

- 非穷尽变体（[`struct`][struct] 或 [`enum` variant][enum]）不能使用 [StructExpression] \(包括[功能更新语法][functional update syntax]\) 构造。
- [单元结构体][struct]隐式定义的同名常量，或[元组结构体][struct]的同名构造函数，其[可见性][visibility]不超过 `pub(crate)`。
  也就是说，如果结构体的可见性是 `pub`，则常量或构造函数的可见性是 `pub(crate)`，否则两者的可见性相同（与没有 `#[non_exhaustive]` 的情况一样）。
- [`enum`][enum] 实例可以构造。

以下构造示例在定义的 crate 外部无法编译：

<!-- ignore: requires external crates -->
```rust,ignore
// 这些是在上游 crate 中定义并标注了
// `#[non_exhaustive]` 的类型。
use upstream::{Config, Token, Id, Error, Message};

// 无法构造 `Config` 的实例；如果在 `upstream` 的新版本中添加了新字段，
// 则此代码将无法编译，因此不允许。
let config = Config { window_width: 640, window_height: 480 };

// 无法构造 `Token` 的实例；如果添加了新字段，它将不再是单元结构体，
// 因此由其作为单元结构体创建的同名常量在 crate 外部不是 public 的；
// 此代码无法编译。
let token = Token;

// 无法构造 `Id` 的实例；如果添加了新字段，其构造函数签名将改变，
// 因此其构造函数在 crate 外部不是 public 的；此代码无法编译。
let id = Id(5);

// 可以构造 `Error` 的实例；引入新变体不会导致此代码编译失败。
let error = Error::Message("foo".to_string());

// 无法构造 `Message::Send` 或 `Message::Reaction` 的实例；
// 如果在 `upstream` 的新版本中添加了新字段，则此代码将无法编译，因此不允许。
let message = Message::Send { from: 0, to: 1, contents: "foo".to_string(), };
let message = Message::Reaction(0);

// 无法构造 `Message::Quit` 的实例；如果将其转换为元组枚举变体，
// 则此代码将无法编译。
let message = Message::Quit;
```

r[attributes.type-system.non_exhaustive.match]
在定义的 crate 外部对非穷尽类型进行模式匹配时存在限制：

- 在对非穷尽变体（[`struct`][struct] 或 [`enum` variant][enum]）进行模式匹配时，必须使用包含 `..` 的 [StructPattern]。元组枚举变体的构造函数的[可见性][visibility]降低到不超过 `pub(crate)`。
- 在对非穷尽 [`enum`][enum] 进行模式匹配时，匹配一个变体不会贡献于分支的穷尽性。以下匹配示例在定义的 crate 外部无法编译：

<!-- ignore: requires external crates -->
```rust, ignore
// 这些是在上游 crate 中定义并标注了
// `#[non_exhaustive]` 的类型。
use upstream::{Config, Token, Id, Error, Message};

// 不能在没有包含通配分支的情况下匹配非穷尽枚举。
match error {
  Error::Message(ref s) => { },
  Error::Other => { },
  // 添加 `_ => {},` 则可以编译
}

// 不能在没有通配符的情况下匹配非穷尽结构体。
if let Ok(Config { window_width, window_height }) = config {
    // 添加 `..` 则可以编译
}

// 除非使用带通配符的花括号结构体语法，否则无法匹配非穷尽单元或元组结构体。
// 这可以编译为 `let Token { .. } = token;`
let Token = token;
// 这可以编译为 `let Id { 0: id_number, .. } = id;`
let Id(id_number) = id;

match message {
  // 不能在没有包含通配符的情况下匹配非穷尽结构体枚举变体。
  Message::Send { from, to, contents } => { },
  // 不能匹配非穷尽元组或单元枚举变体。
  Message::Reaction(type) => { },
  Message::Quit => { },
}
```

也不允许对包含任何非穷尽变体的枚举使用数值强制转换（`as`）。

例如，以下枚举可以进行强制转换，因为它不包含任何非穷尽变体：

```rust
#[non_exhaustive]
pub enum Example {
    First,
    Second,
}
```

但是，如果枚举包含哪怕一个非穷尽变体，强制转换将导致错误。考虑此枚举的修改版本：

```rust
#[non_exhaustive]
pub enum EnumWithNonExhaustiveVariants {
    First,
    #[non_exhaustive]
    Second,
}
```

<!-- ignore: needs multiple crates -->
```rust,ignore
use othercrate::EnumWithNonExhaustiveVariants;

// 错误：当枚举在另一个 crate 中定义时，不能对包含非穷尽变体的枚举进行强制转换
let _ = EnumWithNonExhaustiveVariants::First as u8;
```

非穷尽类型在下游 crate 中始终被视为有人居住的。

[`match`]: ../expressions/match-expr.md
[attributes]: ../attributes.md
[enum]: ../items/enumerations.md
[functional update syntax]: ../expressions/struct-expr.md#functional-update-syntax
[struct]: ../items/structs.md
[visibility]: ../visibility-and-privacy.md
