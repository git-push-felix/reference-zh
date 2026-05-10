r[items.const]
# 常量项

r[items.const.syntax]
```grammar,items
ConstantItem ->
    `const` ( IDENTIFIER | `_` ) `:` Type ( `=` Expression )? `;`
```

r[items.const.intro]
*常量项*是一个可选的带名称的*[常量值][constant value]*，不与程序中的特定内存位置关联。

r[items.const.behavior]
常量本质上在使用的每个地方都会被内联，这意味着在使用时它们会被直接复制到相关上下文中。这包括使用来自外部 crate 的常量，以及非 [`Copy`] 类型。对同一常量的引用不一定保证引用相同的内存地址。

r[items.const.namespace]
常量声明在其所在模块或块的[值命名空间][value namespace]中定义常量值。

r[items.const.static]
常量必须显式指定类型。该类型必须具有 `'static` 生命周期：初始化器中的任何引用都必须具有 `'static` 生命周期。常量类型中的引用默认为 `'static` 生命周期；请参见[静态生命周期省略][static lifetime elision]。

r[items.const.static-temporary]
如果常量值符合[提升][promotion]的条件，对常量的引用将具有 `'static` 生命周期；否则会创建一个临时值。

```rust
const BIT1: u32 = 1 << 0;
const BIT2: u32 = 1 << 1;

const BITS: [u32; 2] = [BIT1, BIT2];
const STRING: &'static str = "bitstring";

struct BitsNStrings<'a> {
    mybits: [u32; 2],
    mystring: &'a str,
}

const BITS_N_STRINGS: BitsNStrings<'static> = BitsNStrings {
    mybits: BITS,
    mystring: STRING,
};
```

r[items.const.expr-omission]
常量表达式只能在 [trait 定义][trait definition]中省略。

r[items.const.destructor]
## 带析构函数的常量 {#constants-with-destructors}

常量可以包含析构函数。析构函数在值离开作用域时运行。

```rust
struct TypeWithDestructor(i32);

impl Drop for TypeWithDestructor {
    fn drop(&mut self) {
        println!("Dropped. Held {}.", self.0);
    }
}

const ZERO_WITH_DESTRUCTOR: TypeWithDestructor = TypeWithDestructor(0);

fn create_and_drop_zero_with_destructor() {
    let x = ZERO_WITH_DESTRUCTOR;
    // x 在函数末尾被释放，调用 drop。
    // 打印 "Dropped. Held 0."。
}
```

r[items.const.unnamed]
## 未命名常量 {#unnamed-constant}

r[items.const.unnamed.intro]
与[关联常量][associated constant]不同，[自由][free]常量可以通过使用下划线代替名称来取消命名。例如：

```rust
const _: () =  { struct _SameNameTwice; };

// OK 尽管名称与上面相同：
const _: () =  { struct _SameNameTwice; };
```

r[items.const.unnamed.repetition]
与[下划线导入][underscore imports]一样，宏可以安全地在同一作用域中多次发出相同的未命名常量。例如，以下不应产生错误：

```rust
macro_rules! m {
    ($item: item) => { $item $item }
}

m!(const _: () = (););
// 这会展开为：
// const _: () = ();
// const _: () = ();
```

r[items.const.eval]
## 求值 {#evaluation}

[自由][free]常量总是在编译时被[求值][const_eval]以显示 panic。即使在未使用的函数中也会发生：

```rust,compile_fail
// 编译时 panic
const PANIC: () = std::unimplemented!();

fn unused_generic_function<T>() {
    // 一个失败的编译时断言
    const _: () = assert!(usize::BITS == 0);
}
```

[const_eval]: ../const_eval.md
[associated constant]: ../items/associated-items.md#associated-constants
[constant value]: ../const_eval.md#constant-expressions
[free]: ../glossary.md#free-item
[static lifetime elision]: ../lifetime-elision.md#const-and-static-elision
[trait definition]: traits.md
[underscore imports]: use-declarations.md#underscore-imports
[`Copy`]: ../special-types-and-traits.md#copy
[value namespace]: ../names/namespaces.md
[promotion]: destructors.scope.const-promotion
