---
title: "AGit-Flow 阿里巴巴集中式 Git 工作流"
slug: "agit-flow-and-git-repo"
date: 2020-03-04T08:00:00+08:00
draft: false
---

## Git Merge 2020

Git 的全球盛会 [Git Merge 2020](https://git-merge.com) 于 2020年3月4日在美国洛杉矶召开，因为新冠病毒引发的疫情，作为演讲嘉宾的我，虽然早早买好了北京和洛杉矶之间的往返机票，最终还是没能成行。

这次会议我的议题是 “AGit-Flow 和 git-repo”，这也是我在 2019年云栖大会演讲“[Go Git：面向未来的代码平台](https://developer.alibaba.com/article/720615)”做出的承诺，将阿里巴巴代码平台的技术开放给全世界。本文就是原计划在 Git Merge 2020 上的演讲稿，文中介绍的相关核心软件均已开源。


## 为什么 Git 能成功？

Git 成为了源代码管理的标准和基础设施。为什么 Git 能够成功？

Linus 作为 Git 和 Linux 的创建者，在 Git 十周年的一次采访中，道出了其中的奥秘：

> The big thing about distributed source control is that it makes one of the main issues with SCM’s go away – the politics around “who can make changes.”

其中的关键词是“politics”（政治）。传统的集中式版本控制系统只能针对核心用户开放写授权，而把大量的潜在的代码贡献者拒之门外，参与项目贡献的门槛高，而这对项目的发展不利。Git 作为分布式版本控制工具，拥有更加灵活的工作流，不仅仅是项目的核心成员，只读用户也可以使用更加优雅的方式参与代码开发。


## 最为常用的两种 Git 工作流

当前两种主流的 Git 协同方式分别是 GitHub 和 Gerrit 提出和普及的。这两种协同模式能够做到：

1. 仓库的授权模型简单。无需为项目设置复杂的授权，只读用户亦可参与代码贡献。

2. 通过代码评审提升代码质量。参与者不是将代码直接推送的分支上，而是创建代码评审。

3. 仓库的分支模型简单。仓库中没有多余的分支，即不需要创建特性分支。

{{< figure src="/images/agit-flow/github-gerrit-comparisons-zh.png" width="750" caption="图: GitHub、Gerrit 协同模式比较" >}}

这两种协同模型的差异非常明显：

1. 代码评审的模式不同。

    GitHub 的代码评审称为 “pull request”，每个特性生成一次代码评审。

    Gerrit 的代码评审称为 “Change”，每个提交生成一个单独的代码评审。

2. 工作流的类型不同。

    GitHub 的工作流属于分布式，代码贡献者的代码先提交到自己完全自主可控的派生仓中。

    Gerrit 的工作流是集中式，所有用户工作在统一管控的集中式仓库中。

3. 实现细节不一样。

    GitHub 模式是仓库派生和创建 pull request。GitHub 底层采用的是原生的 Git（即 CGit）。

    Gerrit 要求用户在本地克隆仓库中安装一个 "commit-msg" 钩子，以便在生成的提交中插入唯一的“Change-Id”，向服务器推送要使用特殊的 `git push` 命令。Gerrit 采用的是 JGit（Java 的 Git 实现）。

4. 各自的优势：

    GitHub 工作流使用标准 Git 操作，使用简单。派生仓自主可控，不受上游项目影响。项目复用、全球开发者大协同，形成了最大的开源社区。

    Gerrit 项目管控更严格；采用 manifest 仓库管理多仓库协同，相比 git-submodule 更加实用。

5. 各自的劣势：

    GitHub 使用派生仓库的工作模式，对于一次性参与项目贡献显得太重了，而且对于多仓库项目难于管理。很难想象在 GitHub 上如何使用派生工作流来管理类似 Android 的多仓库类型的项目。

    Gerrit 需要集中管控，由管理员负责创建项目，而普通用户不能创建项目，这就使得一个 Gerrit 实例通常只管理一个项目或一个组织内的项目，难以在项目之间形成代码复用，也很难汇集跨项目的开发者组成开发者社区。


## AGit-Flow 的使用

### 什么是 AGit-Flow ？

我们能否将 GitHub 和 Gerrit 两种工作流结合到一起（不是 GerritHub 的那种集成模式），兼具他们的优点？

在 Gerrit 的启发下，我们在 CGit 的基础上创建了一个集中式 Git 工作流，其中包含对 Git 核心做的少量修改和几个服务端API。我们称之为 “AGit-Flow”，最早在阿里巴巴的内部代码平台中实现。使用 AGit-Flow 工作流，无需创建派生仓库，也无需在仓库中创建特性分支，只读用户就可以通过 `git push` 命令创建代码评审。

{{< figure src="/images/agit-flow/agit-flow-overview-zh.png" width="750" caption="图: AGit-Flow 功能概览" >}}

在阿里巴巴，我们喜欢 pull request、CGit，喜欢在命令行直接创建代码评审的集中式工作流，喜欢开放的开发者社区。我们不喜欢 "commit-msg" 钩子方式关联提交的代码评审，我们不喜欢一个一个分散的代码平台。

我们还开发了配套的命令行工具 “git-repo”，既能在单仓库下工作，又支持类似 Android 的多仓库项目协同。


### AGit-Flow 工作流

单仓库下 AGit-Flow 工作流如下图所示：

{{< figure src="/images/agit-flow/agit-flow-diagram-zh.png" width="600" caption="图: 单仓库 AGit-Flow 协作流程图" >}}

图中的两个角色，一个是开发者，另外一个是评审者。

开发者通过如下操作，创建和更新 pull request：

1. 开发者克隆仓库。

2. 本地仓库内开发，创建提交。

3. 工作区中执行 `git pr` 命令，推送本地提交到服务器。

4. 服务器自动创建新的代码评审（例如：pull request #123）。

5. 开发者根据评审意见，在本地工作区继续开发，新增或修改提交。

6. 工作区中再次执行 `git pr` 命令，推送本地提交到服务器。

7. 服务器发现目标分支上已经存在来自同一用户、同一本地分支的 pull request，因此用户此次推送没有创建新的 pull request，而是更新已经存在的 pull request。


代码评审者，不但可以给出评审意见，也可以直接发起对评审代码的修改，更新 pull request：

8. 代码评审者执行 `git download 123` 下载编号为 123 的 pull request 到本地仓库。

9. 代码评审者本地修改代码后，执行 `git pr --change 123` 命令，将本地修改推送到服务端。

10. 服务端接收到代码评审者的特殊 `git push` 命令，更新之前由开发者创建的 pull request。

11. 项目管理者通过点击 pull request 评审界面的合并按钮，将 pull request 合入 master 分支。master 分支被更新，同时关闭 pull request。


下面是单仓库下 AGit-Flow 工作流的演示，代码平台使用阿里巴巴·云效（[https://codeup.aliyun.com/](https://codeup.aliyun.com/)）。

{{< figure src="/images/git-repo-single.gif" width="750" caption="图: AGit-Flow 单仓库操作演示" >}}


## AGit-Flow 服务端实现

客户端使用特殊的 `git push` 命令向服务端发起代码推送请求，触发 AGit-Flow 工作流。

这个 `git push` 命令的特殊之处主要在于特殊的引用表达式：

    $ git push origin HEAD:refs/for/<target-branch>/<session>

即：

1. 引用表达式的目标分支包含特殊的前缀 "refs/for/"，用于向远程仓库特定分支 "&lt;target-branch&gt;" 发起代码评审。其中的 "&lt;session&gt;" 通常使用客户端工作区本地分支名。多次 `git push` 请求，如果是相同用户、相同的目标分支、相同的 "&lt;session&gt;"，则对应用同一个 pull request。

2. AGit-Flow 中还有 "refs/drafts/"、"refs/for-review/" 等特殊前缀。

    前缀 "refs/drafts/" 的格式和 "refs/for/" 类似，也是针对目标分支创建或者更新 pull request，区别在于创建的 pull request 处于草稿状态，只能发表评审意见，不能合入。

    前缀 "refs/for-review/" 后面跟指定的 pull request ID，用于更新指定的 pull request。

客户端触发 AGit-Flow 工作流，服务端各个模块及其处理流程示意如下：

{{< figure src="/images/agit-flow/agit-flow-impl-zh.png" width="750" caption="图: AGit-Flow 服务端实现" >}}


### 前端授权模块

前端授权模块在处理用户的推送请求时，要检查用户是否拥有写授权。只读用户是不允许执行 `git push` 命令向服务器推送的。而一个“polictis”正确的集中式工作流，要允许只读用户向仓库贡献代码，如何才能实现呢？

如果既要保持对常规 `git push` 推送命令采用严格的授权模型，又要对 AGit-Flow 的特殊推送降低授权要求，允许来自只读用户的推送操作，是否可以做到呢？

我们使用的方法是传递特殊的环境变量（SSH协议）或者特殊的HTTP头（HTTP协议），如下图所示：

{{< figure src="/images/agit-flow/impl-1-front-end-zh.png" width="700" caption="图: AGit-Flow 前端授权模块" >}}

说明如下：

+ 对于 HTTP 协议，客户端发起的 `git push` 推送命令要通过 `-c http.extraHeader=AGIT-FLOW: <agent-version>` 参数设置 git 配置变量，使得 git 在向服务端发送请求时，设置指定的 HTTP HEADER。

+ 对于 SSH 协议，使用 "GIT_SSH_COMMAND" 环境变量设置使用指定的 SSH 客户端命令。这个 SSH 客户端命令中包含特殊的参数 `-o SendEnv=AGIT_FLOW`，这样使用 SSH 协议时，就能将环境变量 "AGIT_FLOW" 传递给服务器端。

+ 当服务器前端接收到特殊的环境变量（SSH协议）或者特殊的HTTP头（HTTP协议），就会识别出 AGit-Flow 模式的推送指令，采用特殊的授权检查。

+ 注意：为防止用户通过设置特殊环境变量方式越权推送，还需要在 "pre-receive" 钩子脚本中对授权做进一步检查。

### Git 核心改造和 post-receive 钩子

接下来客户端请求传递给 "git-receive-pack"。原生的 "git-receive-pack" 工作流如下图所示：

{{< figure src="/images/agit-flow/impl-2-git-core.png" width="320" caption="图: 原生的 git-receive-pack 工作流" >}}

1. 客户端请求分为两个部分 "commands" 和 "packfile" 依次发送到服务端的 "git-receive-pack" 进程。

2. "packfile" 会进入到隔离区（quarantine），而 "commands" 被解析后，先传递给 "pre-receive" 钩子脚本。

3. 如果 "pre-receive" 钩子脚本失败，则删除隔离区，并返回错误信息，终止推送命令的执行。

4. 如果 "pre-receive" 钩子脚本执行成功，则隔离区中的 "packfile" 移动到仓库的对象库中。

5. 而命令 "commands" 传递给内置的 `execute_commands` 函数，执行 commands（实现分支的创建、更新、删除等操作）。

6. 最后执行 "post-receive" 等钩子脚本，完成事件通知等。


AGit-Flow 对 "git-receive-pack" 的源码做了改动，相关改动已经贡献到 Git 社区，参见：

* [https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/](https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/)。

新的流程如下图所示：

{{< figure src="/images/agit-flow/impl-2-git-core-patched.png" width="600" caption="图: AGit-Flow 对 Git 核心的改动" >}}

为支持 AGit-Flow，我们对 "git-receive-pack" 做了如下修改：

1. 在 "git-receive-pack" 的入口增加了一个命令过滤器。

2. 过滤器将 "commands" 分作两组，一组执行原生的 "git-receive-pack" 流程，另外一组 "commands" 不执行内部的 `execute_commands` 函数，而是调用一个新的外部钩子 "proc-receive" 执行 "commands"。

3. "proc-receive" 钩子将执行结果报告给 "receive-pack"，并由 "receive-pack" 通知客户端（调用 `report()` 函数）。

具体参见下面的介绍。


#### 配置变量：receive.procReceiveRefs

{{< figure src="/images/agit-flow/proc-receive-1.png" width="320" >}}

客户端的推送请求通过标准输入传递给服务端（git-receive-pack），每个命令一行，格式为：

    <旧的oid> <新的oid> <引用名称>

常规 `git push` 推送命令的引用名称是以 "refs/heads/" 或 "refs/tags/" 作为前缀。而 AGit-Flow 模式的推送命令的引用名称中使用不同的前缀。

我们为 Git 引入了一个新的配置变量 "receive.procReceiveRefs"，用于区分 AGit-Flow 模式的引用前缀名称。这个配置变量是一个多值变量。例如在阿里巴巴的代码平台，我们会进行如下的设置：

    git config --system --add receive.procReceiveRefs refs/for
    git config --system --add receive.procReceiveRefs refs/drafts
    git config --system --add receive.procReceiveRefs refs/for-review

上面的指令为该配置变量设置了三个值，来自客户端的 command 指令中的引用名称如果和这三个值任意一个相匹配，则 command 会打上特殊的标记，在后面的执行中会选择另外的处理逻辑。


#### 新钩子 proc-receive

被打上了特殊标记的 的命令，不再通过内置的 `execute_commands` 函数执行，而是调用外部的钩子来执行命令、更新引用。

"receive-pack" 和 钩子 "proc-receive" 之间通过 pkt-line 格式的协议进行交互。如下图所示：

{{< figure src="/images/agit-flow/proc-receive-2.png" width="800" >}}

1. "receive-pack" 和 "proc-receive" 进行版本协商。

   "receive-pack" 首先通过 pkt-line 编码发送协议版本号和能力（capabilities）给钩子，钩子回复自己所支持的协议版本号和能力。当前协议版本号为1，从服务端向客户端传递的能力有 "push-options", "atomic" 等。

2. "receive-pack" 向 "proc-receive" 发送命令和 push-options。

   命令每个一行，格式为 `<old-oid> <new-oid> <reference>`，使用 pkt-line 编码，以 flush-pkt 结束。

   只有在版本协商阶段双方都支持 "push-options"，"receive-pack" 才向 "proc-receive" 发送 push-options。

3. "proc-receive" 钩子调用外部 API 执行用户推送的命令。在阿里巴巴，这个被 "proc-receive" 钩子调用的 API 用于创建或者更新代码评审（pull request）。

4. "proc-receive" 钩子执行完毕，向 "receive-pack" 报告执行结果。支持如下格式的报告：

    + `ok <ref>`

      引用 `<ref>` 更新成功。

    + `ng <ref> <reason>`

      引用 `<ref>` 更新失败，原因由 `<reason>` 提供。

    + `alt <ref> [<alt-ref>] [old-oid=<oid>] [new-oid=<oid>] [forced-update]`

      预期更新 `<ref>`，但实际更新的引用为 `<alt-ref>`，可选参数可以用于设置 `old-oid`、`new-oid`、强制更新模式。

    + `ft <ref>`

      交由 "receive-pack" 执行该命令。`ft` 为 fallthrough 的简写。


#### 向客户端报告

"proc-receive" 向标准错误的输出信息，直接显示给客户端。例如 "proc-receive" 可以用此方法将创建好的 pull request 地址通知给用户。

命令 `git push` 执行完毕，客户端会显示命令执行结果。我们对 Git 报告机制做了扩展，可以显示实际更新的引用。

{{< figure src="/images/agit-flow/proc-receive-3.png" width="800" >}}

例如执行下面命令：

    $ git push origin HEAD:refs/for/master/topic

新版本 Git 显示的执行结果中可以包含实际更新的引用，而非 `refs/for/master/topic`。示例如下：

    To <URL/of/upstream.git>
     + 263ea37...e5a9ada  HEAD -> refs/pull/123/head (forced update)


### Public API: ssh-info

Gerrit HTTP 服务提供了 `ssh_info` API 接口，返回 Gerrit 的 SSH 服务器的 IP 和端口。这样像 repo 这样的命令行客户端就可以使用 SSH 协议进行推送操作，免除口令认证的麻烦等。

AGit-Flow 对 `ssh_info` API 进行了拓展，返回值可以是 JSON 格式，内容包含协议类型和版本等。拓展后的 `ssh_info` 可以视为 "Smart Submit Handler information" 的缩写。AGit-Flow 不但在 HTTP 服务中提供该 API，还在 SSH 服务上也提供 `ssh_info` 命令，用于判断服务端是否支持集中式评审、协议类型和版本等。

下图是 Gerrit、Agit-Flow 服务的 `ssh_info` API 的返回值。不同的协议，`git push` 命令格式和代码评审获取的引用名称各不相同。未来如果有其它的 AGit-Flow 兼容协议，也会有不同的 `ssh_info` 输出，有不同的 `git push` 命令和不同的 pull request 引用名称。

{{< figure src="/images/agit-flow/impl-4-ssh-info-zh.png" width="750" caption="图: `ssh_info` API" >}}

## git-repo

git-repo 是阿里巴巴开源的一款命令行工具，对原生 Git 命令做了封装，简化了使用 AGit-Flow 等集中式工作流时稍嫌繁琐的 Git 命令。git-repo 可以支持 AGit-Flow 兼容的代码平台以及 Gerrit。

git-repo 使用 Golang 开发，在使用上兼容 Android 的 repo，并且运行时除 Git 外不依赖其他软件。除了具备 Android repo 的多仓库管理能力外，还可以对单独的代码仓库进行操作。

+ 网址：[https://git-repo.info](https://git-repo.info/)

+ 源代码：[https://github.com/alibaba/git-repo-go](https://github.com/alibaba/git-repo-go/) 


### 安装

访问 git-repo 的下载页面：[https://git-repo.info/zh_cn/download/](https://git-repo.info/zh_cn/download/)。

根据您平台的类型，下载合适的软件包。然后将下载并解压缩后的 git-repo 文件移动到可执行目录中（如 Linux 下的 `/usr/local/bin` 目录），即完成安装。

### 运行

初次运行任意 git-repo 子命令，会完成一些初始化工作。例如执行下面的命令查看版本号：

    git repo version

下面的这些针对单仓库的别名命令，就是通过 git-repo 初始化安装的 Git 配置文件扩展实现的：

{{< figure src="/images/agit-flow/git-repo-aliases-zh.png" width="450" caption="图: git-repo 别名命令" >}}


### 单仓库下工作

如果工作区当前分支未关联远程分支，先执行操作和远程仓库的远程分支建立关联。例如：如下命令建立和远程仓库 origin 的 master 分支建立关联。

    git branch -u origin/master

然后执行如下命令，从命令行发起代码评审：

    git pr

参见下面的演示：

{{< figure src="/images/git-pr-demo.gif" width="750" caption="图: git pr 命令演示" >}}


### 多仓库下工作

git-repo 支持 Android 模式的多仓库工作流。

1. 创建工作区。

        $ mkdir workspace
        $ cd workspace

2. 下载 Manifest 清单仓库，初始化工作区。

        $ git repo init -u <manifest repository>

3. 按照 Manifest 清单仓库中的文件，下载各个子仓库的代码，并检出到工作区。

        $ git repo sync

4. 创建开发分支。

        $ git repo start --all <branch/name>

5. 在工作区中开发，在各自独立的仓库中修改和完成提交。

6. 执行下面命令，扫描工作区所有仓库的改动，逐个向上游仓库发起代码评审。

        $ git repo upload

参见下面的多仓库工作流演示：

{{< figure src="/images/git-repo-manifest.gif" width="750" caption="图: git-repo 多仓库操作演示" >}}


### 扩展 git-repo

git-repo 不仅支持 Gerrit 和 AGit-Flow 服务，还支持其他与 AGit-Flow 兼容的服务。添加一个新的服务可以通过两种方式实现：

+ 方法一：在 git-repo 的 `helper` 目录中添加新的 protocol helper，实现 `ProtoHelper` 接口。这种方法适于那些提供对外服务的 Git 代码平台。

+ 方法二：提供外部 helper 程序，通过 `--upload`、`--download` 等参数返回 `git push` 命令、代码评审引用名称等信息。这种方法适用于那些私有的代码平台。

{{< figure src="/images/agit-flow/impl-git-repo-extension-zh.png" width="750" caption="图: git-repo 的协议扩展" >}}


## 总结

### 开源地址

AGit-Flow 是受 Gerrit 启发重新设计和开发的 Git 集中式协同方案。底层基于 CGit、不需要仓库派生、使用 pull request 进行代码评审。

git-repo 是使用 Go 语言开发的，与 Android repo 兼容的 AGit-Flow 客户端。

相关代码已经开源：

* https://github.com/alibaba/git-repo-go （欢迎加 ⭐️）

* https://github.com/alibaba/git-repo-go-doc （网站 https://git-repo.info 代码）

* [https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/](https://public-inbox.org/git/20200304113312.34229-1-zhiyou.jx@alibaba-inc.com/) （Git核心的代码修改）


### 未来已来，抢先体验 AGit-Flow

AGit-Flow 是一套开放的协议，已经在如下代码平台提供服务。

* 阿里巴巴·云效2.0 : https://codeup.aliyun.com/


### 为即将到来的 Git 2.27，实现你自己的“AGit-Flow”

* 为你的 AGit-Flow 兼容协议起个名字。

* 修改前端授权，以便允许只读用户执行特殊的推送操作。

* 服务器上安装带有 AGit-Flow 补丁的 Git 核心，并设置相关的 Git 配置变量以开启相关功能。

* 开发 "proc-receive" 钩子和内部创建代码评审（pull request）的 API。

* 在 HTTP 和 SSH 服务中开发对外服务的 `ssh_info` API，返回 JSON 格式数据，实现服务发现。

* 在 git-repo 中添加内置 helper 或者外部 helper 程序扩展，以支持你的“AGit-Flow”。


----

编辑记录：

+ 2020/4/27: 钩子重命名，从 "execute-commands" 改名为 "proc-receive"。
+ 2020/4/27: "receive-pack" 和 "proc-receive" 钩子之间使用 pkt-line 协议。
