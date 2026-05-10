r[type.fn-pointer]
# 函数指针类型

r[type.fn-pointer.syntax]
```grammar,types
BareFunctionType ->
    ForLifetimes? FunctionTypeQualifiers `fn`
       `(` FunctionParametersMaybeNamedVariadic? `)` BareFunctionReturnType?

FunctionTypeQualifiers -> `unsafe`? (`extern` Abi?)?

BareFunctionReturnType -> `->` TypeNoBounds

FunctionParametersMaybeNamedVariadic ->
    MaybeNamedFunctionParameters | MaybeNamedFunctionParametersVariadic

MaybeNamedFunctionParameters ->
    MaybeNamedParam ( `,` MaybeNamedParam )* `,`?

MaybeNamedParam ->
    OuterAttribute* ( ( IDENTIFIER | `_` ) `:` )? Type

MaybeNamedFunctionParametersVariadic ->
    ( MaybeNamedParam `,` )* MaybeNamedParam `,` OuterAttribute* `...`
```

r[type.fn-pointer.intro]
函数指针类型使用 `fn` 关键字编写，指向一个在编译时不一定知道其标识的函数。

以下示例中 `Binop` 被定义为函数指针类型：

```rust
fn add(x: i32, y: i32) -> i32 {
    x + y
}

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

r[type.fn-pointer.coercion]
函数指针可以通过从[函数项][function items]和非捕获、非异步[闭包][closures]强制转换来创建。

r[type.fn-pointer.qualifiers]
`unsafe` 限定符表示该类型的值是一个[不安全函数][unsafe function]，`extern` 限定符表示它是一个[外部函数][extern function]。

r[type.fn-pointer.constraint-variadic]
要使函数为可变参数函数，其 `extern` ABI 必须是 [items.extern.variadic.conventions] 中列出的之一。

r[type.fn-pointer.attributes]
## 函数指针参数上的属性

函数指针参数上的属性遵循与[常规函数参数][regular function parameters]相同的规则和限制。

[`extern`]: ../items/external-blocks.md
[closures]: closure.md
[extern function]: ../items/functions.md#extern-function-qualifier
[function items]: function-item.md
[unsafe function]: ../unsafe-keyword.md
[regular function parameters]: ../items/functions.md#attributes-on-function-parameters
