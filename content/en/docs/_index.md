---
title: "git-repo 文档"
---

作为 Git 和 Linux 的创建者，Linus 在 Git 十周年的一次访谈中，道出了 Git 成功的奥秘：

> The big thing about distributed source control is that it makes one of the main issues with SCM’s go away – the politics around “who can make changes.”

Git 是怎样实现让更多的人参与代码贡献的呢？是通过多样的工作流。一个好的工作流要能够在保障项目质量的前提下，让更多人、更方便地参与到代码开发中。GitHub 的 "Fork + Pull Request" 开发模式是其中最经典、使用最为广泛的一种。开发者不是将提交直接写入仓库分支，而是将提交写入个人派生出来的完全自主可控的派生仓库中，之后再通过 Web 界面发起代码评审（Pull Request）向项目贡献代码。

## AGit-Flow 工作流

有没有更加方便的协同方式呢？

在 Gerrit（使用 JGit）工作模式的启发下，我们对服务端的 Git 源码做了扩展，使得基于 Git（cgit）的代码平台可以很容易引入新的集中式工作流。这种集中式工作流首先应用在阿里巴巴内部代码平台上，我们称之为 AGit-Flow。不需要派生仓库，也不需要创建一个一个的特性分支，用户使用原生的 `git push` 命令，就可以实现从命令行直接创建代码评审任务。代码评审任务经过构建、测试等自动化操作及人工评审后，合入仓库即完成了对仓库代码的更改。

AGit-Flow 的核心代码即将贡献到 Git 社区，任何代码平台都能够以此为基础，实现兼容的集中式工作流。参考：[AGit-Flow](posts/agit-flow-introduction)。

## git-repo 命令行工具

作为一款命令行工具 `git-repo`，对原生 `git` 命令做了封装，简化 AGit-Flow 等集中式工作流下用户的输入。`git-repo` 可以支持 Gerrit 以及 AGit-Flow 兼容的代码平台。

下图是用户使用 `git pr` 命令（`git-repo` 的一个别名命令）创建代码评审的演示：

{{< figure src="/images/git-pr-demo.gif" caption="git-pr 创建 Pull Request 演示图" >}}

`git-repo` 兼容 Android 的 `repo` 工具，支持对多仓库的协同管理。参考：[git-repo 实现多仓库协同](multi-repos/overview/)。

`git-repo` 开源在 GitHub 上，参见：[https://github.com/aliyun/git-repo-go](https://github.com/aliyun/git-repo-go)。

## git-repo 与 Android repo

Android 采用 Gerrit 提供代码评审服务，并且开发了一个客户端工具 `repo`，实现多仓库管理。`git-repo` 实现和安卓 `repo` 使用习惯上的兼容，两者的差异如下：

1. Android repo 只支持 Gerrit 服务器。`git-repo` 采用了一套新的服务发现协议，支持 AGit-Flow 及其兼容的工作流，也支持 Gerrit 工作流。

2. Android repo 只支持 manifests 仓库管理下的多仓库，而 `git-repo` 在此基础上，还支持单一 Git 代码仓的集中式工作流协同，并提供快捷的别名命令 `git peer-review`，或 `git pr`。

3. Android repo 使用 Python 脚本语言开发，`git-repo` 使用 Go 语言开发，并提供了完整的单元测试和集成测试用例。

4. `git-repo` 命令的命名方式采用 `git-<subcmd>` 格式，扩展了 git 的命令集，即为 git 增加了一个名为 repo 的子命令。而且可以用 git 别名为 `git-repo` 设置别名命令，如 `git peer-review` 命令即是 `git repo upload --single` 的别名命令。