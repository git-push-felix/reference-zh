r[type.bool]
# 布尔类型

```rust
let b: bool = true;
```

r[type.bool.intro]
*布尔类型*或 *bool* 是一种原始数据类型，可以取两个值之一，称为 *true* 和 *false*。

r[type.bool.literal]
此类型的值可以使用[字面量表达式][literal expression]通过关键字 `true` 和 `false` 来创建，分别对应同名的值。

r[type.bool.namespace]
此类型属于[语言预导入][language prelude]的一部分，其[名称][name]为 `bool`。

r[type.bool.layout]
布尔类型的对象具有1的[大小和对齐][size and alignment]。

r[type.bool.repr]
值 false 的位模式为 `0x00`，值 true 的位模式为 `0x01`。布尔类型的对象具有任何其它位模式是[未定义行为][undefined behavior]。

r[type.bool.use]
布尔类型是多种[表达式][expressions]中许多操作数的类型：

r[type.bool.use-in-condition]
* [if 表达式][if expressions]和 [while 表达式][while expressions]中的条件操作数

r[type.bool.use-in-lazy-operator]
* [惰性布尔运算符表达式][lazy]中的操作数

> [!NOTE]
> 布尔类型的行为类似于[枚举类型][enumerated type]，但不是枚举类型。在实践中，这主要意味着构造函数不与类型关联（例如 `bool::true`）。

r[type.bool.traits]
与所有原始类型一样，布尔类型[实现][p-impl]了 [trait][p-traits] [`Clone`][p-clone]、[`Copy`][p-copy]、[`Sized`][p-sized]、[`Send`][p-send] 和 [`Sync`][p-sync]。

> [!NOTE]
> 有关库操作，请参阅[标准库文档](bool)。

r[type.bool.expr]
## 布尔值的运算

当使用某些运算符表达式对布尔类型的操作数进行运算时，它们按照[布尔逻辑][boolean logic]的规则求值。

r[type.bool.expr.not]
### 逻辑非 {#logical-not}

| `b` | [`!b`][op-not] |
|- | - |
| `true` | `false` |
| `false` | `true` |

r[type.bool.expr.or]
### 逻辑或 {#logical-or}

| `a` | `b` | [`a \| b`][op-or] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

r[type.bool.expr.and]
### 逻辑与 {#logical-and}

| `a` | `b` | [`a & b`][op-and] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

r[type.bool.expr.xor]
### 逻辑异或 {#logical-xor}

| `a` | `b` | [`a ^ b`][op-xor] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `true` |
| `false` | `false` | `false` |

r[type.bool.expr.cmp]
### 比较

r[type.bool.expr.cmp.eq]
| `a` | `b` | [`a == b`][op-compare] |
|- | - | - |
| `true` | `true` | `true` |
| `true` | `false` | `false` |
| `false` | `true` | `false` |
| `false` | `false` | `true` |

r[type.bool.expr.cmp.greater]
| `a` | `b` | [`a > b`][op-compare] |
|- | - | - |
| `true` | `true` | `false` |
| `true` | `false` | `true` |
| `false` | `true` | `false` |
| `false` | `false` | `false` |

r[type.bool.expr.cmp.not-eq]
* `a != b` 与 `!(a == b)` 相同

r[type.bool.expr.cmp.greater-eq]
* `a >= b` 与 `a == b | a > b` 相同

r[type.bool.expr.cmp.less]
* `a < b` 与 `!(a >= b)` 相同

r[type.bool.expr.cmp.less-eq]
* `a <= b` 与 `a == b | a < b` 相同

r[type.bool.validity]
## 位有效性

`bool` 的单个字节保证被初始化（换句话说，`transmute::<bool, u8>(...)` 始终是可靠的------但由于某些位模式是无效的 `bool`，反过来并不总是可靠的）。

[boolean logic]: https://en.wikipedia.org/wiki/Boolean_algebra
[enumerated type]: enum.md
[expressions]: ../expressions.md
[if expressions]: ../expressions/if-expr.md#if-expressions
[language prelude]: ../names/preludes.md#language-prelude
[lazy]: ../expressions/operator-expr.md#lazy-boolean-operators
[literal expression]: ../expressions/literal-expr.md
[name]: ../names.md
[op-and]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[op-compare]: ../expressions/operator-expr.md#comparison-operators
[op-not]: ../expressions/operator-expr.md#negation-operators
[op-or]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[op-xor]: ../expressions/operator-expr.md#arithmetic-and-logical-binary-operators
[p-clone]: ../special-types-and-traits.md#clone
[p-copy]: ../special-types-and-traits.md#copy
[p-impl]: ../items/implementations.md
[p-send]: ../special-types-and-traits.md#send
[p-sized]: ../special-types-and-traits.md#sized
[p-sync]: ../special-types-and-traits.md#sync
[p-traits]: ../items/traits.md
[size and alignment]: ../type-layout.md#size-and-alignment
[undefined behavior]: ../behavior-considered-undefined.md
[while expressions]: ../expressions/loop-expr.md#predicate-loops
