r[attributes.testing]
# 测试属性

以下[属性][attributes]用于指定执行测试的函数。在"test"模式下编译 crate 会启用测试函数的构建以及用于执行测试的测试框架。启用测试模式还会启用 [`test` 条件编译选项][`test` conditional compilation option]。

<!-- template:attributes -->
r[attributes.testing.test]
## `test` 属性 {#the-test-attribute}

r[attributes.testing.test.intro]
*`test` [属性][attributes]* 将一个函数标记为作为测试执行。

> [!EXAMPLE]
> ```rust,no_run
> # pub fn add(left: u64, right: u64) -> u64 { left + right }
> #[test]
> fn it_works() {
>     let result = add(2, 2);
>     assert_eq!(result, 4);
> }
> ```

r[attributes.testing.test.syntax]
`test` 属性使用 [MetaWord] 语法。

r[attributes.testing.test.allowed-positions]
`test` 属性只能应用于单态的、不接收参数的[自由函数][free functions]，并且其返回类型必须实现 [`Termination`] trait。

> [!NOTE]
> 实现 [`Termination`] trait 的一些类型包括：
> * `()`
> * `Result<T, E> where T: Termination, E: Debug`

r[attributes.testing.test.duplicates]
只有第一次在函数上使用 `test` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

<!-- TODO: This is a minor lie. Currently rustc warns that duplicates are ignored, but it then generates multiple test entries with the same name. I would vote for rejecting this in the future. -->

r[attributes.testing.test.stdlib]
`test` 属性从标准库预导入中导出为 [`std::prelude::v1::test`]。

r[attributes.testing.test.enabled]
这些函数仅在测试模式下编译。

> [!NOTE]
> 测试模式通过向 `rustc` 传递 `--test` 参数或使用 `cargo test` 启用。

r[attributes.testing.test.success]
测试框架调用返回值的 [`report`] 方法，根据结果 [`ExitCode`] 是否表示成功终止将测试分类为通过或失败。
特别是：
* 返回 `()` 的测试只要终止且不发生 panic 就通过。
* 返回 `Result<(), E>` 的测试只要返回 `Ok(())` 就通过。
* 返回 `ExitCode::SUCCESS` 的测试通过，返回 `ExitCode::FAILURE` 的测试失败。
* 不终止的测试既不通不过也不失败。

> [!EXAMPLE]
> ```rust,no_run
> # use std::io;
> # fn setup_the_thing() -> io::Result<i32> { Ok(1) }
> # fn do_the_thing(s: &i32) -> io::Result<()> { Ok(()) }
> #[test]
> fn test_the_thing() -> io::Result<()> {
>     let state = setup_the_thing()?; // 预期成功
>     do_the_thing(&state)?;          // 预期成功
>     Ok(())
> }
> ```

<!-- template:attributes -->
r[attributes.testing.ignore]
## `ignore` 属性 {#the-ignore-attribute}

r[attributes.testing.ignore.intro]
*`ignore` [属性][attributes]* 可以与 [`test` 属性][attributes.testing.test]一起使用，告知测试框架不要将该函数作为测试执行。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[ignore]
> fn check_thing() {
>     // …
> }
> ```

> [!NOTE]
> `rustc` 测试框架支持 `--include-ignored` 标志来强制运行被忽略的测试。

r[attributes.testing.ignore.syntax]
`ignore` 属性使用 [MetaWord] 和 [MetaNameValueStr] 语法。

r[attributes.testing.ignore.reason]
`ignore` 属性的 [MetaNameValueStr] 形式提供了一种指定测试被忽略原因的方法。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[ignore = "not yet implemented"]
> fn mytest() {
>     // …
> }
> ```

r[attributes.testing.ignore.allowed-positions]
`ignore` 属性只能应用于标注了 `test` 属性的函数。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[attributes.testing.ignore.duplicates]
只有第一次在函数上使用 `ignore` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出 lint 警告。这可能在将来成为错误。

r[attributes.testing.ignore.behavior]
被忽略的测试在测试模式下仍会被编译，但不会被运行。

<!-- template:attributes -->
r[attributes.testing.should_panic]
## `should_panic` 属性 {#the-should_panic-attribute}

r[attributes.testing.should_panic.intro]
*`should_panic` [属性][attributes]* 使测试仅在应用该属性的[测试函数][attributes.testing.test]发生 panic 时通过。

> [!EXAMPLE]
> ```rust,no_run
> #[test]
> #[should_panic(expected = "values don't match")]
> fn mytest() {
>     assert_eq!(1, 2, "values don't match");
> }
> ```

r[attributes.testing.should_panic.syntax]
`should_panic` 属性有以下形式：

- [MetaWord]
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic]
  > fn mytest() { panic!("error: some message, and more"); }
  > ```

- [MetaNameValueStr] --- 给定的字符串必须出现在 panic 消息中，测试才能通过。
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic = "some message"]
  > fn mytest() { panic!("error: some message, and more"); }
  > ```

- [MetaListNameValueStr] --- 与 [MetaNameValueStr] 语法一样，给定的字符串必须出现在 panic 消息中。
  > [!EXAMPLE]
  > ```rust,no_run
  > #[test]
  > #[should_panic(expected = "some message")]
  > fn mytest() { panic!("error: some message, and more"); }
  > ```

r[attributes.testing.should_panic.allowed-positions]
`should_panic` 属性只能应用于标注了 `test` 属性的函数。

> [!NOTE]
> `rustc` 忽略其他位置的用法但会发出 lint 警告。这可能在将来成为错误。

r[attributes.testing.should_panic.duplicates]
只有第一次在函数上使用 `should_panic` 才有效。

> [!NOTE]
> `rustc` 会对第一次之后的使用发出未来兼容性警告的 lint。这可能在将来成为错误。

r[attributes.testing.should_panic.expected]
当使用 [MetaNameValueStr] 形式或带有 `expected` 键的 [MetaListNameValueStr] 形式时，给定的字符串必须出现在 panic 消息的某处，测试才能通过。

r[attributes.testing.should_panic.return]
测试函数的返回类型必须是 `()`。

[`Termination`]: std::process::Termination
[`report`]: std::process::Termination::report
[`test` conditional compilation option]: ../conditional-compilation.md#test
[attributes]: ../attributes.md
[`ExitCode`]: std::process::ExitCode
[free functions]: ../glossary.md#free-item
