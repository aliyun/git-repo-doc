---
title: "单仓库协同概览"
draft: false
weight: 31
---

大多数用户参与的项目是一个代码仓。对于单仓库操作， `git-repo` 提供了如下命令扩展：

* git peer-review，或 git pr: 创建代码评审。
* git download: 下载评审的代码到本地。
* git abandon: 清理当前完成评审发起的分支。

使用 git-repo 针对单仓库系统的示意如下：

{{< figure src="/images/git-repo-single.gif" caption="图: git-repo 针对单仓库的操作" >}}
