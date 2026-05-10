# rustc 测试注解

<https://github.com/rust-lang/rust> 中的测试可以链接到参考手册中的规则。规则将包含测试的链接，还有一个[附录][appendix]跟踪规则当前的链接状态。

`tests` 目录中的测试可以用 `//@ reference: x.y.z` 头部注解来链接到规则。如果一个文件覆盖多个规则，可以多次指定该头部。

编译器开发者不需要在测试中添加 `reference` 注解。但如果他们愿意提供帮助，他们的合作是受欢迎的。参考手册的作者和编辑负责确保每条规则都有相关的测试。

这些测试有助于评审者看到规则的行为。它们也对希望查看特定行为示例的读者有帮助。添加新规则时，你应该等到参考手册端被批准后才向 `rust-lang/rust` 提交 PR（以避免在我们决定使用不同名称时产生不必要的修改）。

始终使用可用的最具体的规则名称进行注解。例如，使用 `asm.rules.reg-not-input` 而不是更宽泛的 `asm.rules`。

完全覆盖是目标，但目前尚未达到预期。

[appendix]: https://doc.rust-lang.org/nightly/reference/test-summary.html
