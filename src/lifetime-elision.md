r[lifetime-elision]
# 生命周期省略

Rust 有规则允许在编译器可以推断出合理的默认选择的各种位置省略生命周期。

r[lifetime-elision.function]
## 函数中的生命周期省略

r[lifetime-elision.function.intro]
为了使常见模式更加符合人体工程学，生命周期参数可以在[函数项][function item]、[函数指针][function pointer]和[闭包 trait][closure trait] 签名中*省略*。以下规则用于推断省略的生命周期的生命周期参数。

r[lifetime-elision.function.lifetimes-not-inferred]
省略无法推断的生命周期参数是错误的。

r[lifetime-elision.function.explicit-placeholder]
占位生命周期 `'_` 也可以用来以相同的方式推断生命周期。对于路径中的生命周期，首选使用 `'_`。

r[lifetime-elision.function.only-functions]
Trait 对象的生命周期遵循[下面](#默认-trait-对象生命周期)讨论的不同规则。

r[lifetime-elision.function.implicit-lifetime-parameters]
* 参数中的每个省略的生命周期成为一个独立的生命周期参数。

r[lifetime-elision.function.output-lifetime]
* 如果参数中恰好使用了一个生命周期（省略或未省略），则该生命周期被分配给*所有*省略的输出生命周期。

r[lifetime-elision.function.receiver-lifetime]
在方法签名中还有另一条规则

* 如果接收者（receiver）的类型是 `&Self` 或 `&mut Self`，则对 `Self` 的该引用的生命周期被分配给所有省略的输出生命周期参数。

示例：

```rust
# trait T {}
# trait ToCStr {}
# struct Thing<'a> {f: &'a i32}
# struct Command;
#
# trait Example {
fn print1(s: &str);                                   // 省略
fn print2(s: &'_ str);                                // 同样省略
fn print3<'a>(s: &'a str);                            // 展开

fn debug1(lvl: usize, s: &str);                       // 省略
fn debug2<'a>(lvl: usize, s: &'a str);                // 展开

fn substr1(s: &str, until: usize) -> &str;            // 省略
fn substr2<'a>(s: &'a str, until: usize) -> &'a str;  // 展开

fn get_mut1(&mut self) -> &mut dyn T;                 // 省略
fn get_mut2<'a>(&'a mut self) -> &'a mut dyn T;       // 展开

fn args1<T: ToCStr>(&mut self, args: &[T]) -> &mut Command;                  // 省略
fn args2<'a, 'b, T: ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command; // 展开

fn other_args1<'a>(arg: &str) -> &'a str;             // 省略
fn other_args2<'a, 'b>(arg: &'b str) -> &'a str;      // 展开

fn new1(buf: &mut [u8]) -> Thing<'_>;                 // 省略 - 首选
fn new2(buf: &mut [u8]) -> Thing;                     // 省略
fn new3<'a>(buf: &'a mut [u8]) -> Thing<'a>;          // 展开
# }

type FunPtr1 = fn(&str) -> &str;                      // 省略
type FunPtr2 = for<'a> fn(&'a str) -> &'a str;        // 展开

type FunTrait1 = dyn Fn(&str) -> &str;                // 省略
type FunTrait2 = dyn for<'a> Fn(&'a str) -> &'a str;  // 展开
```

```rust,compile_fail
// 以下示例展示了不允许省略生命周期参数的情况。

# trait Example {
// 无法推断，因为没有参数可以从中推断。
fn get_str() -> &str;                                 // 非法

// 无法推断，有歧义：是从第一个参数还是第二个参数借用的。
fn frob(s: &str, t: &str) -> &str;                    // 非法
# }
```

r[lifetime-elision.trait-object]
## 默认 trait 对象生命周期

r[lifetime-elision.trait-object.intro]
[trait 对象][trait object]持有的引用的假定生命周期称为其*默认对象生命周期约束*。这些定义在 [RFC 599] 中，并在 [RFC 1156] 中修订。

r[lifetime-elision.trait-object.explicit-bound]
当生命周期约束完全省略时，使用这些默认对象生命周期约束而不是上面定义的生命周期参数省略规则。

r[lifetime-elision.trait-object.explicit-placeholder]
如果使用 `'_` 作为生命周期约束，则该约束遵循通常的省略规则。

r[lifetime-elision.trait-object.containing-type]
如果 trait 对象用作泛型类型的类型参数，则首先使用包含类型尝试推断约束。

r[lifetime-elision.trait-object.containing-type-unique]
* 如果从包含类型有唯一的约束，则使用该约束作为默认值。

r[lifetime-elision.trait-object.containing-type-explicit]
* 如果从包含类型有多个约束，则必须指定显式约束。

r[lifetime-elision.trait-object.trait-bounds]
如果上述规则都不适用，则使用 trait 上的约束：

r[lifetime-elision.trait-object.trait-unique]
* 如果 trait 定义了单个生命周期*约束*，则使用该约束。

r[lifetime-elision.trait-object.static-lifetime]
* 如果任何生命周期约束使用了 `'static`，则使用 `'static`。

r[lifetime-elision.trait-object.default]
* 如果 trait 没有生命周期约束，则在表达式中推断生命周期，在表达式外部为 `'static`。

```rust
// 对于以下 trait……
trait Foo { }

// 这两个是相同的，因为 Box<T> 没有对 T 的生命周期约束
type T1 = Box<dyn Foo>;
type T2 = Box<dyn Foo + 'static>;

// ……这些也是相同的：
impl dyn Foo {}
impl dyn Foo + 'static {}

// ……这些也是相同的，因为 &'a T 要求 T: 'a
type T3<'a> = &'a dyn Foo;
type T4<'a> = &'a (dyn Foo + 'a);

// std::cell::Ref<'a, T> 也要求 T: 'a，因此这些是相同的
type T5<'a> = std::cell::Ref<'a, dyn Foo>;
type T6<'a> = std::cell::Ref<'a, dyn Foo + 'a>;
```

```rust,compile_fail
// 这是一个错误示例。
# trait Foo { }
struct TwoBounds<'a, 'b, T: ?Sized + 'a + 'b> {
    f1: &'a i32,
    f2: &'b i32,
    f3: T,
}
type T7<'a, 'b> = TwoBounds<'a, 'b, dyn Foo>;
//                                  ^^^^^^^
// 错误：无法从上下文推断此对象类型的生命周期约束
```

r[lifetime-elision.trait-object.innermost-type]
请注意，最内层的对象设置约束，因此 `&'a Box<dyn Foo>` 仍然是 `&'a Box<dyn Foo + 'static>`。

```rust
// 对于以下 trait……
trait Bar<'a>: 'a { }

// ……这两个是相同的：
type T1<'a> = Box<dyn Bar<'a>>;
type T2<'a> = Box<dyn Bar<'a> + 'a>;

// ……这些也是相同的：
impl<'a> dyn Bar<'a> {}
impl<'a> dyn Bar<'a> + 'a {}
```

r[lifetime-elision.const-static]
## `const` 和 `static` 省略

r[lifetime-elision.const-static.implicit-static]
引用类型的[常量][constant]和[静态][static]声明都具有*隐式*的 `'static` 生命周期，除非指定了显式生命周期。因此，上面涉及 `'static` 的常量声明可以在没有生命周期的情况下编写。

```rust
// STRING: &'static str
const STRING: &str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

// BITS_N_STRINGS: BitsNStrings<'static>
const BITS_N_STRINGS: BitsNStrings<'_> = BitsNStrings {
    mybits: [1, 2],
    mystring: STRING,
};
```

r[lifetime-elision.const-static.fn-references]
请注意，如果 `static` 或 `const` 项包含函数或闭包引用，而这些引用本身包含引用，编译器将首先尝试标准省略规则。如果无法通过其通常规则解析生命周期，则会报错。举例说明：

```rust
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc(a: &Foo, b: &Bar, c: &Baz) -> usize {42}
// 解析为 `for<'a> fn(&'a str) -> &'a str`。
const RESOLVED_SINGLE: fn(&str) -> &str = |x| x;

// 解析为 `for<'a, 'b, 'c> Fn(&'a Foo, &'b Bar, &'c Baz) -> usize`。
const RESOLVED_MULTIPLE: &dyn Fn(&Foo, &Bar, &Baz) -> usize = &somefunc;
```

```rust,compile_fail
# struct Foo;
# struct Bar;
# struct Baz;
# fn somefunc<'a,'b>(a: &'a Foo, b: &'b Bar) -> &'a Baz {unimplemented!()}
// 没有足够的信息来约束返回引用生命周期相对于参数生命周期，
// 因此这是一个错误。
const RESOLVED_STATIC: &dyn Fn(&Foo, &Bar) -> &Baz = &somefunc;
//                                            ^
// 此函数的返回类型包含一个借用的值，但签名未说明它
// 是从参数 1 还是参数 2 借用的
```

[closure trait]: types/closure.md
[constant]: items/constant-items.md
[function item]: types/function-item.md
[function pointer]: types/function-pointer.md
[RFC 599]: https://github.com/rust-lang/rfcs/blob/master/text/0599-default-object-bound.md
[RFC 1156]: https://github.com/rust-lang/rfcs/blob/master/text/1156-adjust-default-object-bounds.md
[static]: items/static-items.md
[trait object]: types/trait-object.md
