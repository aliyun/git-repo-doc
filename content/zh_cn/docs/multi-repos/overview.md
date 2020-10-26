---
title: "多仓库协同概览"
draft: false
weight: 41
---

在实践中，Git 大仓库存在着诸多问题：下载慢、不能对授权进行精细控制。解决大仓库的一个有效方案是化整为零：将大仓库拆分为多仓库。

为了实现多仓库像一个仓库一样工作，Git提供一个原生的 submodule 方案，但是这个方案问题很多。例如：子模组固定在某个版本上，而不能实现随上游仓库分支更新而更新。多人修改子模组仓库，冲突从文件级别放大到仓库级别，让子模组的冲突不易解决。

Android 项目创造了一个新的方案。即使用 XML 格式文件（manifest 清单文件）定义一个项目的多仓库关联，然后用 `repo` 客户端工具操作多仓库。

git-repo 使用 go 语言重新实现了 Android repo 工具多仓库管理功能，目标是实现和 Android repo 工具的 100% 兼容。

git repo 命令行格式如下：

    git repo <子命令> <参数>

查看 git-repo 的帮助，使用命令：

    git repo help
    git repo help <子命令>


使用 git-repo 管理多个代码仓库的典型使用场景，如下图所示：

{{< figure src="/images/git-repo-manifest.gif" caption="图: git-repo 针对多仓库的操作" >}}

上面的演示场景中，主要步骤如下：

1. 创建一个空目录，作为工作区。

        $ mkdir workspace
        $ cd workspace

2. 通过克隆 manifest 清单仓库，完成工作区的初始化。

        $ git repo init -u <manifest-url>

    例如:

        $ git repo init -u https://codeup.aliyun.com/git-repo/manifests.git

3. 下载相关仓库代码

        $ git repo sync

4. 创建开发分支

        $ git repo start --all <topic-branch>

    例如：

        $ git repo start --all jx/topic1

5. 在工作区中开发，每个仓库的改动单独完成本地提交。

6. 执行下面命令，会扫描工作区所有仓库的改动，逐个向上游仓库发起代码评审。

        $ git repo upload


