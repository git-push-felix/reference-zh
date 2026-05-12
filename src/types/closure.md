r[type.closure]
# 闭包类型 {#closure-types}

r[type.closure.intro]
[闭包表达式][closure expression]生成一个闭包值，其类型是唯一的、匿名的，无法被写出。闭包类型大致等价于一个包含被捕获值的结构体。例如，以下闭包：

```rust
#[derive(Debug)]
struct Point { x: i32, y: i32 }
struct Rectangle { left_top: Point, right_bottom: Point }

fn f<F : FnOnce() -> String> (g: F) {
    println!("{}", g());
}

let mut rect = Rectangle {
    left_top: Point { x: 1, y: 1 },
    right_bottom: Point { x: 0, y: 0 }
};

let c = || {
    rect.left_top.x += 1;
    rect.right_bottom.x += 1;
    format!("{:?}", rect.left_top)
};
f(c); // 打印 "Point { x: 2, y: 1 }"。
```

生成一个大致如下的闭包类型：

<!-- ignore: simplified -->
```rust,ignore
// 注意：这不是精确的转换方式，仅用于说明。
struct Closure<'a> {
    left_top : &'a mut Point,
    right_bottom_x : &'a mut i32,
}

impl<'a> FnOnce<()> for Closure<'a> {
    type Output = String;
    extern "rust-call" fn call_once(self, args: ()) -> String {
        self.left_top.x += 1;
        *self.right_bottom_x += 1;
        format!("{:?}", self.left_top)
    }
}
```

因此对 `f` 的调用如同：

<!-- ignore: continuation of above -->
```rust,ignore
f(Closure{ left_top: &mut rect.left_top, right_bottom_x: &mut rect.right_bottom.x });
```

r[type.closure.capture]
## 捕获模式 {#capture-modes}

r[type.closure.capture.intro]
*捕获模式*决定了环境中的[位置表达式][place expression]如何被借用或移动到闭包中。捕获模式有：

1. 不可变借用 (`ImmBorrow`) --- 位置表达式被捕获为[共享引用][shared reference]。
2. 唯一不可变借用 (`UniqueImmBorrow`) --- 类似于不可变借用，但必须是唯一的，如[下文](#unique-immutable-borrows-in-captures)所述。
3. 可变借用 (`MutBorrow`) --- 位置表达式被捕获为[可变引用][mutable reference]。
4. 移动 (`ByValue`) --- 位置表达式通过[移动值][moving the value]的方式被捕获到闭包中。

r[type.closure.capture.precedence]
环境中的位置表达式从第一个与闭包体内捕获值的使用方式兼容的模式开始被捕获。模式不受闭包周围代码的影响，例如所涉及的变量或字段的生命周期，或闭包自身的生命周期。

[moving the value]: ../expressions.md#moved-and-copied-types
[mutable reference]: pointer.md#mutable-references-mut
[place expression]: ../expressions.md#place-expressions-and-value-expressions
[shared reference]: pointer.md#references--and-mut

r[type.closure.capture.copy]
### `Copy` 值

实现了 [`Copy`] 并且被移动到闭包中的值以 `ImmBorrow` 模式捕获。

```rust
let x = [0; 1024];
let c = || {
    let y = x; // x 以 ImmBorrow 捕获
};
```

r[type.closure.async.input]
### 异步输入捕获

异步闭包始终捕获所有输入参数，无论它们是否在闭包体内被使用。

## 捕获精度

r[type.closure.capture.precision.capture-path]
*捕获路径*是一个序列，以环境中的变量开始，后面跟随零个或多个从该变量开始的位置投影。

r[type.closure.capture.precision.place-projection]
*位置投影*是[字段访问][field access]、[元组索引][tuple index]、[解引用][dereference]（和自动解引用）、[数组或切片索引][array or slice index]表达式，或应用于变量的[模式解构][pattern destructuring]。

> [!NOTE]
> 在 `rustc` 中，模式解构会脱糖为一系列解引用和字段或元素访问。

r[type.closure.capture.precision.intro]
闭包借用或移动捕获路径，该路径可能根据下述规则被截断。

例如：

```rust
struct SomeStruct {
    f1: (i32, i32),
}
let s = SomeStruct { f1: (1, 2) };

let c = || {
    let x = s.f1.1; // s.f1.1 以 ImmBorrow 捕获
};
c();
```

此处捕获路径是局部变量 `s`，后跟字段访问 `.f1`，再后跟元组索引 `.1`。此闭包捕获 `s.f1.1` 的不可变借用。

[field access]: ../expressions/field-expr.md
[pattern destructuring]: patterns.destructure
[tuple index]: ../expressions/tuple-expr.md#tuple-indexing-expressions
[dereference]: ../expressions/operator-expr.md#the-dereference-operator
[array or slice index]: ../expressions/array-expr.md#array-and-slice-indexing-expressions

r[type.closure.capture.precision.shared-prefix]
### 共享前缀

当一条捕获路径及其某个祖先路径同时被闭包捕获时，祖先路径将以两者中较高的捕获模式捕获，`CaptureMode = max(AncestorCaptureMode, DescendantCaptureMode)`，使用如下严格弱序：

`ImmBorrow < UniqueImmBorrow < MutBorrow < ByValue`

注意，这可能需要递归应用。

```rust
// 此例中有三条不同的捕获路径共享同一祖先：
# fn move_value<T>(_: T){}
let s = String::from("S");
let t = (s, String::from("T"));
let mut u = (t, String::from("U"));

let c = || {
    println!("{:?}", u); // u 以 ImmBorrow 捕获
    u.1.truncate(0); // u.1 以 MutBorrow 捕获
    move_value(u.0.0); // u.0.0 以 ByValue 捕获
};
c();
```

总体而言，此闭包将以 `ByValue` 捕获 `u`。

r[type.closure.capture.precision.dereference-shared]
### 最右侧共享引用截断

捕获路径在路径中最右侧的解引用处截断，前提是该解引用作用于共享引用。

允许此截断是因为通过共享引用读取的字段始终通过共享引用或复制来读取。这有助于在额外精度从借用检查角度而言没有收益时减少捕获的大小。

之所以是*最右侧*解引用，是为了帮助避免不必要的更短生命周期。考虑以下示例：

```rust
struct Int(i32);
struct B<'a>(&'a i32);

struct MyStruct<'a> {
   a: &'static Int,
   b: B<'a>,
}

fn foo<'a, 'b>(m: &'a MyStruct<'b>) -> impl FnMut() + 'static {
    let c = || drop(&m.a.0);
    c
}
```

如果此处捕获 `m`，则闭包将不再能存活超过 `'static`，因为 `m` 受 `'a` 约束。相反，它以 `ImmBorrow` 捕获 `(*(*m).a)`。

r[type.closure.capture.precision.wildcard]
### 通配符模式绑定

r[type.closure.capture.precision.wildcard.reads]
闭包仅捕获需要被读取的数据。使用[通配符模式][wildcard pattern]绑定值不会读取该值，因此该位置不会被捕获。

```rust,no_run
struct S; // 非 `Copy` 类型。
let x = S;
let c = || {
    let _ = x;  // 不捕获 `x`。
};
let c = || match x {
    _ => (), // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();
```

r[type.closure.capture.precision.wildcard.destructuring]
解构元组、结构体和单变体枚举本身不会导致读取或捕获该位置。

> [!NOTE]
> 标记有 [`#[non_exhaustive]`][attributes.type-system.non_exhaustive] 的枚举始终被视为具有多个变体。参见 *[type.closure.capture.precision.discriminants.non_exhaustive]*。

```rust,no_run
struct S; // 非 `Copy` 类型。

// 解构元组不会导致读取或捕获。
let x = (S,);
let c = || {
    let (..) = x; // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();

// 解构单元结构体不会导致读取或捕获。
let x = S;
let c = || {
    let S = x; // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();

// 解构结构体不会导致读取或捕获。
struct W<T>(T);
let x = W(S);
let c = || {
    let W(..) = x; // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();

// 解构单变体枚举不会导致读取或捕获。
enum E<T> { V(T) }
let x = E::V(S);
let c = || {
    let E::V(..) = x; // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();
```

r[type.closure.capture.precision.wildcard.fields]
匹配 [RestPattern] (`..`) 或 [StructPatternEtCetera]（也是 `..`）的字段不会被读取，这些字段不会被捕获。

```rust,no_run
struct S; // 非 `Copy` 类型。
let x = (S, S);
let c = || {
    let (x0, ..) = x;  // 以 `ByValue` 捕获 `x.0`。
};
// 只有第一个元组字段被闭包捕获。
x.1; // 正确：`x.1` 可以在这里移动。
c();
```

r[type.closure.capture.precision.wildcard.array-slice]
不支持对数组和切片的部分捕获；即使使用通配符模式匹配、索引或子切片，整个切片或数组也始终被捕获。

```rust,compile_fail,E0382
struct S; // 非 `Copy` 类型。
let mut x = [S, S];
let c = || {
    let [x0, _] = x; // 以 `ByValue` 捕获整个 `x`。
};
let _ = &mut x[1]; // 错误：借用已移动的值。
```

r[type.closure.capture.precision.wildcard.initialized]
使用通配符匹配的值仍然必须被初始化。

```rust,compile_fail,E0381
let x: u8;
let c = || {
    let _ = x; // 错误：绑定 `x` 未初始化。
};
```

[wildcard pattern]: ../patterns.md#wildcard-pattern

r[type.closure.capture.precision.discriminants]
### 判别值读取的捕获

r[type.closure.capture.precision.discriminants.reads]
如果模式匹配读取了判别值，则包含该判别值的位置以 `ImmBorrow` 被捕获。

r[type.closure.capture.precision.discriminants.multiple-variant]
匹配具有多个变体的枚举的某个变体将读取判别值，以 `ImmBorrow` 捕获该位置。

```rust,compile_fail,E0502
struct S; // 非 `Copy` 类型。
let mut x = (Some(S), S);
let c = || match x {
    (None, _) => (),
//   ^^^^
// 此模式需要读取判别值，这导致
// `x.0` 以 `ImmBorrow` 被捕获。
    _ => (),
};
let _ = &mut x.0; // 错误：无法将 `x.0` 作为可变借用。
//           ^^^
// 闭包仍然存活，因此这里 `x.0` 仍然
// 被不可变借用。
c();
```

```rust,no_run
# struct S; // 非 `Copy` 类型。
# let x = (Some(S), S);
let c = || match x { // 以 `ImmBorrow` 捕获 `x.0`。
    (None, _) => (),
    _ => (),
};
// 虽然 `x.0` 因判别值读取而被捕获，
// 但 `x.1` 没有被捕获。
x.1; // 正确：`x.1` 可以在这里移动。
c();
```

r[type.closure.capture.precision.discriminants.single-variant]
匹配单变体枚举的唯一变体不会读取判别值，也不会捕获该位置。

```rust,no_run
enum E<T> { V(T) } // 单变体枚举。
let x = E::V(());
let c = || {
    let E::V(_) = x; // 不捕获 `x`。
};
x; // 正确：`x` 可以在这里移动。
c();
```

r[type.closure.capture.precision.discriminants.non_exhaustive]
如果 [`#[non_exhaustive]`][attributes.type-system.non_exhaustive] 应用于某个枚举，就判断是否发生读取而言，该枚举被视为具有多个变体，即使它实际上只有一个变体。

r[type.closure.capture.precision.discriminants.uninhabited-variants]
即使除所匹配的变体之外的所有变体都是不可居住的，使得该模式[不可反驳][patterns.refutable]，判别值仍然会被读取（如果本来应读取的话）。

```rust,compile_fail,E0502
enum Empty {}
let mut x = Ok::<_, Empty>(42);
let c = || {
    let Ok(_) = x; // 以 `ImmBorrow` 捕获 `x`。
};
let _ = &mut x; // 错误：无法将 `x` 作为可变借用。
c();
```


r[type.closure.capture.precision.range-patterns]
### 范围模式的捕获

r[type.closure.capture.precision.range-patterns.reads]
匹配[范围模式][patterns.range]会读取被匹配的位置，即使该范围包含类型的所有可能值，也会以 `ImmBorrow` 捕获该位置。

```rust,compile_fail,E0502
let mut x = 0u8;
let c = || {
    let 0..=u8::MAX = x; // 以 `ImmBorrow` 捕获 `x`。
};
let _ = &mut x; // 错误：无法将 `x` 作为可变借用。
c();
```

r[type.closure.capture.precision.slice-patterns]
### 切片模式的捕获

r[type.closure.capture.precision.slice-patterns.slices]
将切片与除仅包含单个[剩余模式][patterns.rest]（即 `[..]`）之外的[切片模式][patterns.slice]匹配，视为从切片读取长度，并以 `ImmBorrow` 捕获该切片。

```rust,compile_fail,E0502
let x: &mut [u8] = &mut [];
let c = || match x { // 以 `ImmBorrow` 捕获 `*x`。
    &mut [] => (),
//       ^^
// 这匹配一个恰好零元素的切片。要判断被检查值是否
// 匹配，必须读取长度，导致切片被捕获。
    _ => (),
};
let _ = &mut *x; // 错误：无法将 `*x` 作为可变借用。
c();
```

```rust,no_run
let x: &mut [u8] = &mut [];
let c = || match x { // 不捕获 `*x`。
    [..] => (),
//   ^^ 剩余模式。
};
let _ = &mut *x; // 正确：`*x` 可以在这里借用。
c();
```

> [!NOTE]
> 也许令人惊讶的是，尽管长度包含在指向切片的（宽）*指针*中，但被视为读取并捕获的是*指向对象*（切片）的位置。
>
> ```rust,no_run
> fn f<'l: 's, 's>(x: &'s mut &'l [u8]) -> impl Fn() + 'l {
>     // 闭包存活期超过 `'l`，因为它捕获了 `**x`。
>     // 如果它捕获的是 `*x`，则存活时间不足以
>     // 满足 `impl Fn() + 'l` 的约束。
>     || match *x { // 以 `ImmBorrow` 捕获 `**x`。
>         &[] => (),
>         _ => (),
>     }
> }
> ```
>
> 这样，行为与在检查值中解引用到切片是一致的。
>
> ```rust,no_run
> fn f<'l: 's, 's>(x: &'s mut &'l [u8]) -> impl Fn() + 'l {
>     || match **x { // 以 `ImmBorrow` 捕获 `**x`。
>         [] => (),
>         _ => (),
>     }
> }
> ```
>
> 详细信息见 [Rust PR #138961](https://github.com/rust-lang/rust/pull/138961)。

r[type.closure.capture.precision.slice-patterns.arrays]
由于数组的长度由类型确定，将数组与切片模式匹配本身并不会捕获该位置。

```rust,no_run
let x: [u8; 1] = [0];
let c = || match x { // 不捕获 `x`。
    [_] => (), // 长度是固定的。
};
x; // 正确：`x` 可以在这里移动。
c();
```

r[type.closure.capture.precision.move-dereference]
### `move` 上下文中捕获引用

由于不允许从引用中移出字段，`move` 闭包仅会捕获到达引用第一次解引用之前（但不包括）的捕获路径前缀。引用本身将被移动到闭包中。

```rust
struct T(String, String);

let mut t = T(String::from("foo"), String::from("bar"));
let t_mut_ref = &mut t;
let mut c = move || {
    t_mut_ref.0.push_str("123"); // 以 ByValue 捕获 `t_mut_ref`
};
c();
```

r[type.closure.capture.precision.raw-pointer-dereference]
### 裸指针解引用

由于解引用裸指针是 `unsafe` 的，闭包仅会捕获到达裸指针第一次解引用之前（但不包括）的捕获路径前缀。

```rust
struct T(String, String);

let t = T(String::from("foo"), String::from("bar"));
let t_ptr = &t as *const T;

let c = || unsafe {
    println!("{}", (*t_ptr).0); // 以 ImmBorrow 捕获 `t_ptr`
};
c();
```

r[type.closure.capture.precision.union]
### 联合体字段

由于访问联合体字段是 `unsafe` 的，闭包仅会捕获到达联合体本身的捕获路径前缀。

```rust
union U {
    a: (i32, i32),
    b: bool,
}
let u = U { a: (123, 456) };

let c = || {
    let x = unsafe { u.a.0 }; // 以 ByValue 捕获 `u`
};
c();

// 这也包括写入字段。
let mut u = U { a: (123, 456) };

let mut c = || {
    u.b = true; // 以 MutBorrow 捕获 `u`
};
c();
```

r[type.closure.capture.precision.unaligned]
### 对未对齐 `struct` 的引用

由于创建对结构中未对齐字段的引用是[未定义行为][undefined behavior]，闭包仅会捕获到达使用了 [`packed` 表示法][the `packed` representation]的结构体中第一次字段访问之前（但不包括）的捕获路径前缀。这包括所有字段，即使那些是对齐的，以防将来结构体中的任何字段发生变化时产生兼容性问题。

```rust
#[repr(packed)]
struct T(i32, i32);

let t = T(2, 5);
let c = || {
    let a = t.0; // 以 ImmBorrow 捕获 `t`
};
// 从 `t` 复制是可以的。
let (a, b) = (t.0, t.1);
c();
```

类似地，获取未对齐字段的地址也会捕获整个结构体：

```rust,compile_fail,E0505
#[repr(packed)]
struct T(String, String);

let mut t = T(String::new(), String::new());
let c = || {
    let a = std::ptr::addr_of!(t.1); // 以 ImmBorrow 捕获 `t`
};
let a = t.0; // 错误：无法移出 `t.0`，因为它已被借用
c();
```

但如果不是 packed 的就可行，因为它精确地捕获了字段：

```rust
struct T(String, String);

let mut t = T(String::new(), String::new());
let c = || {
    let a = std::ptr::addr_of!(t.1); // 以 ImmBorrow 捕获 `t.1`
};
// 这里允许移动。
let a = t.0;
c();
```

[undefined behavior]: ../behavior-considered-undefined.md
[the `packed` representation]: ../type-layout.md#the-alignment-modifiers

r[type.closure.capture.precision.box-deref]
### `Box` 与其他 `Deref` 实现

[`Box`] 的 [`Deref`] trait 实现与其他 `Deref` 实现的处理方式不同，因为它被视为特殊实体。

例如，我们来看涉及 `Rc` 和 `Box` 的示例。`*rc` 脱糖为调用 `Rc` 上定义的 trait 方法 `deref`，但由于 `*box` 被特殊处理，可以对 `Box` 的内容进行精确捕获。

[`Box`]: ../special-types-and-traits.md#boxt
[`Deref`]: ../special-types-and-traits.md#deref-and-derefmut

r[type.closure.capture.precision.box-non-move.not-moved]
#### 非 `move` 闭包中的 `Box`

在非 `move` 闭包中，如果 `Box` 的内容没有被移动到闭包体内，则 `Box` 的内容会被精确捕获。

```rust
struct S(String);

let b = Box::new(S(String::new()));
let c_box = || {
    let x = &(*b).0; // 以 ImmBorrow 捕获 `(*b).0`
};
c_box();

// 将 `Box` 与另一个实现了 Deref 的类型对比：
let r = std::rc::Rc::new(S(String::new()));
let c_rc = || {
    let x = &(*r).0; // 以 ImmBorrow 捕获 `r`
};
c_rc();
```

r[type.closure.capture.precision.box-non-move.moved]
但是，如果 `Box` 的内容被移动到闭包中，则整个 box 被捕获。这样做是为了最小化需要移动到闭包中的数据量。

```rust
// 与上例相同，只是闭包移动值而不是获取引用。

struct S(String);

let b = Box::new(S(String::new()));
let c_box = || {
    let x = (*b).0; // 以 ByValue 捕获 `b`
};
c_box();
```

r[type.closure.capture.precision.box-move.read]
#### `move` 闭包中的 `Box`

与非 `move` 闭包中移动 `Box` 内容类似，在 `move` 闭包中读取 `Box` 的内容将会整体捕获该 `Box`。

```rust
struct S(i32);

let b = Box::new(S(10));
let c_box = move || {
    let x = (*b).0; // 以 ByValue 捕获 `b`
};
```

r[type.closure.unique-immutable]
## 唯一不可变借用于捕获 {#unique-immutable-borrows-in-captures}

捕获可以通过一种特殊的借用发生，称为*唯一不可变借用*，它不能在语言的其他任何地方使用，也无法显式写出。当修改可变引用的所指对象时会发生这种借用，如下例所示：

```rust
let mut b = false;
let x = &mut b;
let mut c = || {
    // 对 `x` 的 ImmBorrow 和 MutBorrow。
    let a = &x;
    *x = true; // `x` 以 UniqueImmBorrow 捕获
};
// 下面这行会出错：
// let y = &x;
c();
// 然而下面这行没问题。
let z = &x;
```

在这种情况下，可变借用 `x` 是不可能的，因为 `x` 不是 `mut`。但同时，不可变借用 `x` 会使赋值非法，因为 `& &mut` 引用可能不是唯一的，因此不能安全地用于修改值。所以使用了唯一不可变借用：它不可变地借用 `x`，但像可变借用一样，它必须是唯一的。

在上述示例中，取消 `y` 声明的注释将产生错误，因为这会违反闭包对 `x` 的借用的唯一性；`z` 的声明是有效的，因为闭包的生命周期已在块结束时到期，释放了借用。

r[type.closure.call]
## 调用 trait 与强制转换 {#call-traits-and-coercions}

r[type.closure.call.intro]
闭包类型都实现了 [`FnOnce`]，表示它们可通过消耗闭包所有权被调用一次。此外，某些闭包实现了更具体的调用 trait：

r[type.closure.call.fn-mut]
* 不移动出任何被捕获变量的闭包实现了 [`FnMut`]，表示它可以通过可变引用被调用。

r[type.closure.call.fn]
* 不修改也不移动出任何被捕获变量的闭包实现了 [`Fn`]，表示它可以通过共享引用被调用。

> [!NOTE]
> `move` 闭包仍然可能实现 [`Fn`] 或 [`FnMut`]，即使它们通过移动捕获变量。这是因为闭包类型实现的 trait 取决于闭包对捕获值所做的操作，而不是它如何捕获它们。

r[type.closure.non-capturing]
*非捕获闭包*是不从环境中捕获任何内容的闭包。非异步、非捕获闭包可以强制转换为具有匹配签名的函数指针（例如 `fn()`）。

```rust
let add = |x, y| x + y;

let mut x = add(5,7);

type Binop = fn(i32, i32) -> i32;
let bo: Binop = add;
x = bo(5,7);
```

r[type.closure.async.traits]
### 异步闭包 trait

r[type.closure.async.traits.fn-family]
异步闭包在是否实现 [`FnMut`] 或 [`Fn`] 方面有进一步的限制。

异步闭包返回的 [`Future`] 具有与闭包类似的捕获特征。它根据捕获值在异步闭包中的使用方式，从异步闭包中捕获位置表达式。如果异步闭包具有以下任一属性，则称其*借出*给其 [`Future`]：

- `Future` 包含可变捕获。
- 异步闭包按值捕获，除非该值通过解引用投影访问。

如果异步闭包借出给其 `Future`，则 [`FnMut`] 和 [`Fn`] *不*被实现。[`FnOnce`] 总是被实现。

> **示例**：可变捕获的第一个条款可以用以下示例说明：
>
> ```rust,compile_fail
> fn takes_callback<Fut: Future>(c: impl FnMut() -> Fut) {}
>
> fn f() {
>     let mut x = 1i32;
>     let c = async || {
>         x = 2;  // x 以 MutBorrow 捕获
>     };
>     takes_callback(c);  // 错误：异步闭包未实现 `FnMut`
> }
> ```
>
> 常规值捕获的第二个条款可以用以下示例说明：
>
> ```rust,compile_fail
> fn takes_callback<Fut: Future>(c: impl Fn() -> Fut) {}
>
> fn f() {
>     let x = &1i32;
>     let c = async move || {
>         let a = x + 2;  // x 以 ByValue 捕获
>     };
>     takes_callback(c);  // 错误：异步闭包未实现 `Fn`
> }
> ```
>
> 第二个条款的例外可以通过使用解引用来说明，这样确实允许实现 `Fn` 和 `FnMut`：
>
> ```rust
> fn takes_callback<Fut: Future>(c: impl Fn() -> Fut) {}
>
> fn f() {
>     let x = &1i32;
>     let c = async move || {
>         let a = *x + 2;
>     };
>     takes_callback(c);  // 正确：实现了 `Fn`
> }
> ```

r[type.closure.async.traits.async-family]
异步闭包以类似于常规闭包实现 [`Fn`]、[`FnMut`] 和 [`FnOnce`] 的方式实现 [`AsyncFn`]、[`AsyncFnMut`] 和 [`AsyncFnOnce`]；即，取决于闭包体内对捕获变量的使用方式。

r[type.closure.traits]
### 其他 trait

r[type.closure.traits.intro]
所有闭包类型都实现了 [`Sized`]。此外，闭包类型在其存储的捕获类型允许时实现以下 trait：

* [`Clone`]
* [`Copy`]
* [`Sync`]
* [`Send`]

r[type.closure.traits.behavior]
[`Send`] 和 [`Sync`] 的规则与普通结构体类型相同，而 [`Clone`] 和 [`Copy`] 的行为如同[派生][derived]一样。对于 [`Clone`]，捕获值的克隆顺序未指定。

由于捕获通常是通过引用进行的，因此产生以下一般规则：

* 如果所有捕获的值都是 [`Sync`] 的，则闭包是 [`Sync`] 的。
* 如果所有通过非唯一不可变引用捕获的值都是 [`Sync`] 的，并且所有通过唯一不可变或可变引用、复制或移动捕获的值都是 [`Send`] 的，则闭包是 [`Send`] 的。
* 如果闭包没有通过唯一不可变或可变引用捕获任何值，并且它通过复制或移动捕获的所有值分别是 [`Clone`] 或 [`Copy`] 的，则闭包是 [`Clone`] 或 [`Copy`] 的。

[`Clone`]: ../special-types-and-traits.md#clone
[`Copy`]: ../special-types-and-traits.md#copy
[`Send`]: ../special-types-and-traits.md#send
[`Sized`]: ../special-types-and-traits.md#sized
[`Sync`]: ../special-types-and-traits.md#sync
[closure expression]: ../expressions/closure-expr.md
[derived]: ../attributes/derive.md

r[type.closure.drop-order]
## 丢弃顺序

如果闭包按值捕获了复合类型（例如结构体、元组和枚举）的字段，则该字段的生命周期现在将绑定到闭包。因此，复合类型的不相交字段可能在不同时间被丢弃。

```rust
{
    let tuple =
      (String::from("foo"), String::from("bar")); // --+
    { //                                               |
        let c = || { // ----------------------------+  |
            // tuple.0 被捕获到闭包中              |  |
            drop(tuple.0); //                       |  |
        }; //                                       |  |
    } // 'c' 和 'tuple.0' 在此处丢弃 --------------+  |
} // tuple.1 在此处丢弃 ------------------------------+
```

r[type.closure.capture.precision.edition2018.entirety]
## 2018 及更早版本 {#edition-2018-and-before}

### 闭包类型差异

在 2018 版本及之前，闭包始终整体捕获一个变量，不带其精确捕获路径。这意味着对于[闭包类型](#closure-types)一节中使用的示例，生成的闭包类型将类似如下：

<!-- ignore: simplified -->
```rust,ignore
struct Closure<'a> {
    rect : &'a mut Rectangle,
}

impl<'a> FnOnce<()> for Closure<'a> {
    type Output = String;
    extern "rust-call" fn call_once(self, args: ()) -> String {
        self.rect.left_top.x += 1;
        self.rect.right_bottom.x += 1;
        format!("{:?}", self.rect.left_top)
    }
}
```

对 `f` 的调用将如下工作：

<!-- ignore: continuation of above -->
```rust,ignore
f(Closure { rect: rect });
```

r[type.closure.capture.precision.edition2018.composite]
### 捕获精度差异

复合类型（如结构体、元组和枚举）始终被整体捕获，而不是按单个字段捕获。因此，可能需要借用到局部变量才能捕获单个字段：

```rust
# use std::collections::HashSet;
#
struct SetVec {
    set: HashSet<u32>,
    vec: Vec<u32>
}

impl SetVec {
    fn populate(&mut self) {
        let vec = &mut self.vec;
        self.set.iter().for_each(|&n| {
            vec.push(n);
        })
    }
}
```

如果闭包直接使用 `self.vec`，则它会尝试以可变引用捕获 `self`。但由于 `self.set` 已经被借用以进行迭代，代码将无法编译。

r[type.closure.capture.precision.edition2018.move]
如果使用了 `move` 关键字，则所有捕获都是通过移动或（对于 `Copy` 类型）复制进行的，无论借用是否可行。`move` 关键字通常用于允许闭包在捕获值之后继续存活，例如在返回闭包或使用它来生成新线程时。

r[type.closure.capture.precision.edition2018.wildcard]
无论闭包是否会读取数据（即在通配符模式的情况下），如果在闭包内提到了闭包外部定义的变量，该变量将被整体捕获。

r[type.closure.capture.precision.edition2018.drop-order]
### 丢弃顺序差异

由于复合类型被整体捕获，按值捕获这些复合类型之一的闭包将在闭包被丢弃时同时丢弃整个被捕获的变量。

```rust
{
    let tuple =
      (String::from("foo"), String::from("bar"));
    {
        let c = || { // --------------------------+
            // tuple 被捕获到闭包中              |
            drop(tuple.0); //                     |
        }; //                                     |
    } // 'c' 和 'tuple' 在此处丢弃 --------------+
}
```
