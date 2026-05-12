r[expr.call]
# 调用表达式

r[expr.call.syntax]
```grammar,expressions
CallExpression -> Expression `(` CallParams? `)`

CallParams -> Expression ( `,` Expression )* `,`?
```

r[expr.call.intro]
*调用表达式*调用一个函数。调用表达式的语法是一个表达式（称为*函数操作数*），后跟一个用括号括起来的、以逗号分隔的表达式列表（称为*参数操作数*）。

r[expr.call.convergence]
如果函数最终返回，则表达式完成。

r[expr.call.trait]
对于[非函数类型][non-function types]，表达式 `f(...)` 根据函数操作数使用以下 trait 之一上的方法：

- [`Fn`] 或 [`AsyncFn`] --- 共享引用。
- [`FnMut`] 或 [`AsyncFnMut`] --- 可变引用。
- [`FnOnce`] 或 [`AsyncFnOnce`] --- 值。

r[expr.call.autoref-deref]
如果需要，会自动进行借用。函数操作数也会根据需要被[自动解引用][automatically dereferenced]。

一些调用表达式的示例：

```rust
# fn add(x: i32, y: i32) -> i32 { 0 }
let three: i32 = add(1i32, 2i32);
let name: &'static str = (|| "Rust")();
```

r[expr.call.desugar]
## 消歧函数调用 {#disambiguating-function-calls}

r[expr.call.desugar.fully-qualified]
所有函数调用都是更显式的[完全限定语法][fully-qualified syntax]的语法糖。

r[expr.call.desugar.ambiguity]
函数调用可能需要完全限定，这取决于调用的歧义性以及作用域内的项。

> [!NOTE]
> 过去，术语"无歧义函数调用语法"、"通用函数调用语法"或"UFCS"曾被用于文档、议题、RFC 和其他社区著作中。然而，这些术语缺乏描述力，并且可能混淆手头的问题。我们在此提及它们是为了便于搜索。

r[expr.call.desugar.limits]
经常出现的一些情况会导致方法或关联函数调用的接收者或被引用者的歧义。这些情况可能包括：

* 多个作用域内的 trait 为相同类型定义了同名方法
* 不希望自动 `deref`；例如，区分智能指针本身的方法和指针所指对象的方法
* 不接受参数的方法，如 [`default()`]，以及返回类型属性的方法，如 [`size_of()`]

r[expr.call.desugar.explicit-path]
为了解决歧义，程序员可以使用更具体的路径、类型或 trait 来引用所需的方法或函数。

例如：

```rust
trait Pretty {
    fn print(&self);
}

trait Ugly {
    fn print(&self);
}

struct Foo;
impl Pretty for Foo {
    fn print(&self) {}
}

struct Bar;
impl Pretty for Bar {
    fn print(&self) {}
}
impl Ugly for Bar {
    fn print(&self) {}
}

fn main() {
    let f = Foo;
    let b = Bar;

    // 我们可以这样做，因为对 `Foo` 只有一个名为 `print` 的项
    f.print();
    // 更显式，并且在 `Foo` 的情况下不必要
    Foo::print(&f);
    // 如果你不热衷于简洁性
    <Foo as Pretty>::print(&f);

    // b.print(); // 错误：找到多个 'print'
    // Bar::print(&b); // 仍然是错误：找到多个 `print`

    // 由于作用域内的项定义了 `print`，这是必要的
    <Bar as Pretty>::print(&b);
}
```

有关更多细节和动机，请参阅 [RFC 132]。

[RFC 132]: https://github.com/rust-lang/rfcs/blob/master/text/0132-ufcs.md
[`default()`]: std::default::Default::default
[`size_of()`]: std::mem::size_of
[automatically dereferenced]: field-expr.md#automatic-dereferencing
[fully-qualified syntax]: ../paths.md#qualified-paths
[non-function types]: ../types/function-item.md
