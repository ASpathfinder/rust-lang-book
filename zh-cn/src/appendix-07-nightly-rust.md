## 附录 G：Rust 是如何开发的与“Nightly Rust”

本附录介绍 Rust 是如何开发的，以及这会怎样影响身为 Rust 开发者的你。

### 稳定而不僵化

作为一门语言，Rust <em>非常</em>重视代码的稳定性。我们希望 Rust 成为可以依靠的坚如磐石的基础；如果一切都不断变化，就不可能做到这一点。与此同时，如果无法试验新功能，我们可能直到功能发布之后才发现重要缺陷，而那时已经无法再改变它们。

我们解决这个问题的方案称为“<em>稳定而不僵化(stability without stagnation)</em>”，其指导原则是：你永远不应该害怕升级到新的 Rust 稳定版。每次升级都应轻松无痛，同时还应带来新功能、更少的 bug 和更快的编译速度。

### 呜呜！发布通道与搭乘列车

Rust 开发采用<em>列车时刻表(train schedule)</em>：所有开发都在 Rust 仓库的 main 分支上进行。发布遵循软件发布列车模型，该模型也用于 Cisco IOS 和其他软件项目。Rust 有三个<em>发布通道(release channel)</em>：

- Nightly
- Beta
- Stable

大多数 Rust 开发者主要使用 stable 通道，但想尝试实验性新功能的人可能会使用 nightly 或 beta。

下面举例说明开发与发布流程如何工作：假设 Rust 团队正在准备 Rust 1.5 版本。这个版本实际发布于 2015 年 12 月，但能为我们提供真实的版本号。Rust 中加入一项新功能：一个新提交落入 main 分支。每晚都会生成一个新的 Rust nightly 版本。每天都是发布日，这些版本由发布基础设施自动创建。因此，随着时间推移，每晚一次的发布看起来如下：

```text
nightly: * - - * - - *
```

每六周就到了准备新版本的时候！Rust 仓库的 `beta` 分支从 nightly 所用的 main 分支分出。现在有两个版本：

```text
nightly: * - - * - - *
                     |
beta:                *
```

大多数 Rust 用户不会主动使用 beta 版本，但会在 CI 系统中针对 beta 测试，帮助 Rust 发现可能的回归。与此同时，每晚仍然会有 nightly 版本：

```text
nightly: * - - * - - * - - * - - *
                     |
beta:                *
```

假设发现了一个回归。幸好，在回归悄悄进入稳定版之前，我们有一些时间测试 beta 版本！修复会应用到 main 分支，从而修复 nightly；然后再把修复向后移植到 `beta` 分支，生成新的 beta 版本：

```text
nightly: * - - * - - * - - * - - * - - *
                     |
beta:                * - - - - - - - - *
```

创建第一个 beta 六周后，就到了发布稳定版的时候！`stable` 分支从 `beta` 分支产生：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |
beta:                * - - - - - - - - *
                                       |
stable:                                *
```

太好了！Rust 1.5 完成了！不过，我们忘了一件事：六周已经过去，还需要 Rust <em>下一个</em>版本 1.6 的新 beta。因此，在 `stable` 从 `beta` 分出后，下一版 `beta` 会再次从 `nightly` 分出：

```text
nightly: * - - * - - * - - * - - * - - * - * - *
                     |                         |
beta:                * - - - - - - - - *       *
                                       |
stable:                                *
```

之所以称为“<em>列车模型(train model)</em>”，是因为每六周就有一个版本“驶离车站”，但在作为稳定版到达之前，仍要经过 beta 通道的旅程。

Rust 像时钟一样每六周发布一次。如果知道某个 Rust 版本的发布日期，就能知道下一版的日期：六周以后。按六周安排发布的一个好处是，下一班列车很快就会到来。如果某项功能恰好错过某个版本，无需担心：很快就会有另一次发布！这有助于减轻临近发布截止时间时，偷偷塞入可能尚未打磨好之功能的压力。

得益于这套流程，你始终可以检出 Rust 的下一个构建版本，亲自确认升级是否轻松：如果 beta 版本没有按预期工作，可以向团队报告，使其在下一稳定版发布之前得到修复！beta 版本发生破坏的情况相对少见，但 `rustc` 仍然是软件，bug 确实存在。

### 维护时间

Rust 项目支持最新的稳定版。新稳定版发布时，旧版本会到达其<em>生命周期终点(end of life, EOL)</em>。这意味着每个版本会得到六周支持。

<a id="unstable-features"></a>

### 不稳定功能

这种发布模型还有一个要点：不稳定功能。Rust 使用名为“<em>功能标志(feature flag)</em>”的技术，确定给定版本中启用哪些功能。如果新功能正在积极开发，它会落入 main 分支，因而进入 nightly，但位于<em>功能标志</em>之后。作为用户，如果想试用开发中的功能，可以这样做；但必须使用 Rust nightly 版本，并以适当标志标注源代码，选择采用该功能。

如果使用 Rust 的 beta 或 stable 版本，就无法使用任何功能标志。正是这一点让我们可以在宣布新功能永远稳定之前，先实际使用它们。想选择最前沿体验的人可以这样做；想要坚如磐石体验的人则可以坚持使用 stable，并确信代码不会损坏。这就是稳定而不僵化。

本书只包含稳定功能的信息，因为开发中的功能仍在变化，而且从本书写作时到它们在稳定构建中启用时，肯定会有所不同。你可以在线查找仅限 nightly 功能的文档。

### Rustup 与 Rust Nightly 的作用

Rustup 可以轻松地在不同 Rust 发布通道间切换，既可以全局切换，也可以按项目切换。默认会安装 Rust stable。例如，要安装 nightly：

```console
$ rustup toolchain install nightly
```

还可以通过 `rustup` 查看已经安装的所有<em>工具链(toolchain)</em>（Rust 版本及相关组件）。下面是作者之一所用 Windows 计算机上的例子：

```powershell
> rustup toolchain list
stable-x86_64-pc-windows-msvc (default)
beta-x86_64-pc-windows-msvc
nightly-x86_64-pc-windows-msvc
```

可以看到，stable 工具链是默认选择。大多数 Rust 用户多数时候使用 stable。你可能希望平时使用 stable，但因为关注某项前沿功能，而在特定项目中使用 nightly。为此，可以在该项目目录中使用 `rustup override`，把 nightly 工具链设为进入该目录时 `rustup` 应使用的工具链：

```console
$ cd ~/projects/needs-nightly
$ rustup override set nightly
```

现在，每次在 <em>~/projects/needs-nightly</em> 中调用 `rustc` 或 `cargo`，`rustup` 都会确保使用 Rust nightly，而不是默认的 Rust stable。当你拥有许多 Rust 项目时，这很方便！

### RFC 流程与团队

那么，要如何了解这些新功能呢？Rust 的开发模型遵循<em>征求意见(Request For Comments, RFC)流程</em>。如果希望改进 Rust，可以撰写一份称为 RFC 的提案。

任何人都可以编写 RFC 来改进 Rust，提案会由 Rust 团队审查和讨论；Rust 团队由许多主题子团队组成。[Rust 网站上](https://www.rust-lang.org/governance)列出了所有团队，包括负责项目各领域的团队：语言设计、编译器实现、基础设施、文档等。相应团队会阅读提案和评论，写下自己的评论，最终就接受或拒绝该功能达成共识。

如果功能获准接受，就会在 Rust 仓库中创建一个议题，然后任何人都可以实现它。实现者完全可能不是最初提出该功能的人！实现就绪后，会像[“不稳定功能”](#unstable-features)<!-- ignore -->一节所说，在<em>功能门(feature gate)</em>之后落入 main 分支。

经过一段时间，在使用 nightly 版本的 Rust 开发者能够试用新功能后，团队成员会讨论该功能及其在 nightly 上的表现，并决定是否应让它进入 Rust stable。如果决定继续推进，就会移除功能门，该功能从此被视为稳定！它会搭乘列车进入新的 Rust 稳定版。
