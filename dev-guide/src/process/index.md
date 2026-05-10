# 贡献流程

## 贡献前

对于非平凡的更改，我们鼓励人们在提交 PR 之前进行讨论。这使参考手册团队有机会更好地理解你的想法，并确保其符合参考手册的预期方向。通常，你应该在提交 pull request 之前提交 issue 或在 [Zulip]（[Zulip](#zulip)）上发起讨论。

## 贡献流程概述

贡献的一般流程如下：

1. [检出源代码。](../tooling/building.md#checking-out-the-source)
2. [安装 mdbook。](../tooling/building.md#installing-mdbook)
3. [学习在本地构建本书。](../tooling/building.md#running-mdbook)
4. 对源文件进行更改。请务必遵循本书中关于风格、约定等的所有指南。
5. [运行测试。](../tests.md)
6. [提交 pull request](#submitting-a-pull-request)
7. PR 将进入评审流程。
   - 关于可能经历的评审类型，请参见 **[评审策略](../review-policy.md)**。
   - 这可能需要一些时间，因为团队的时间有限。
8. 一旦获得批准，团队成员将合并更改。
   - 团队可能会在合并前进行编辑性修改。
   - 更改可能需要几周时间才会出现在[ nightly 网站](https://doc.rust-lang.org/nightly/reference/)上。更多细节请参见 **[发布](../publishing.md)**。

## 办公时间

lang-docs 团队在周二 [美国东部时间下午 3:30](https://dateful.com/convert/est-edt-eastern-time?t=330pm) 举行办公时间。我们在 [Jitsi Meet](https://meet.jit.si/rust-t-lang-docs) 上会面。请查看 [Zulip]（[Zulip](#zulip)）频道以获取最新状态和可用性。

## Zulip

Zulip 上有用于讨论参考手册的频道：

- [`#t-lang-docs`](https://rust-lang.zulipchat.com/#narrow/channel/237824-t-lang-docs) --- 由 lang docs 团队使用。
- [`#t-lang-docs/reference`](https://rust-lang.zulipchat.com/#narrow/channel/520709-t-lang-docs.2Freference) --- 专门讨论参考手册。

## 处理 issue

当 issue 被标记为 [Help Wanted] 时，团队正在寻求贡献来帮助解决它。

如果你想处理某个 issue，可以通过评论 `@rustbot claim` 来分配给自己。更多信息请参见 **[Issue 分配][issue assignment]**。

[Help Wanted]: https://github.com/rust-lang/reference/issues?q=state%3Aopen%20label%3A%22Help%20Wanted%22
[issue assignment]: https://forge.rust-lang.org/triagebot/issue-assignment.html

## 新特性

关于如何为新增特性编写文档，请参见 **[稳定化][stabilization]**。

[stabilization]: stabilization.md

## 小改动

小改动 --- 例如小的更正、措辞清理和格式修正 --- 可以直接提交 PR 来完成。

## 大改动

大改动 --- 例如大规模重写、重新组织和新增章节 --- 应首先与参考手册团队讨论并获得批准。提交一个 issue（如果还没有的话）来讨论你感兴趣的更改类型。当参考手册团队能够提供帮助时，他们将与你合作来批准或对更改给出反馈。

## 提交 pull request

提交 pull request 时，请遵循以下指南：

- 包含对更改内容及其原因的清晰描述。
- 保持干净的 git 历史记录；每个提交都应解释更改的原因。
- 在描述中使用 [GitHub 关键词][GitHub’s keywords] 来自动将 PR 链接到 issue。例如，写上 `Closes #1234` 会将 issue #1234 链接到该 PR。当 PR 被合并时，GitHub 将自动关闭该 issue。

当你的 PR 提交后，GitHub 会自动运行所有测试。GitHub 界面会显示绿色勾号（表示通过）或红色 X（表示失败）。PR 页面上有日志链接用于诊断任何问题。

[GitHub’s keywords]: https://docs.github.com/en/github/managing-your-work-on-github/linking-a-pull-request-to-an-issue

### PR 标签

PR 会用[标签][labels]标记，例如 [`S-waiting-on-review`] 和 [`S-waiting-on-author`]，以指示其状态。任何人都可以使用 [`@rustbot`] 机器人来调整标签。如果 PR 被标记为 `S-waiting-on-author`，而你已经推送了希望被评审的新更改，你可以在 PR 上评论 `@rustbot ready`。机器人将切换 PR 上的标签。

有关这些命令的更多信息，请参见[快捷方式文档][shortcuts documentation]。

[`@rustbot`]: https://github.com/rustbot
[`S-waiting-on-author`]: https://github.com/rust-lang/reference/labels/S-waiting-on-author
[`S-waiting-on-review`]: https://github.com/rust-lang/reference/labels/S-waiting-on-review
[labels]: https://github.com/rust-lang/reference/labels
[shortcuts documentation]: https://forge.rust-lang.org/triagebot/shortcuts.html
