---
title: "快速开始"
draft: false
weight: 11
enableToc: false
---

## 快速开始

### 步骤 1：安装 git-repo

访问 git-repo 的下载页面: [https://git-repo.info/zh_cn/download/](https://git-repo.info/zh_cn/download/)。

根据您平台的类型，下载合适的软件包。然后将下载并解压缩后的 `git-repo` 文件拷贝到可执行目录中（如 Linux 下的 `/usr/local/bin` 目录），即完成安装。


### 步骤 2：运行 git-repo

初次运行任意 git-repo 子命令，会完成一些初始化工作。例如执行下面的命令查看版本号：

    git repo version


### 步骤 3：单仓库下发起代码评审

如果工作区当前分支未关联远程分支，先执行操作和远程仓库的远程分支建立关联。例如：如下命令建立和 origin 远程仓库的 master 分支建立关联。

    git branch -u origin/master

发起代码审核，执行如下命令：

    git pr


### 步骤 4：多仓库工作流

`git-repo` 支持 Android 式的多仓库工作流。

1. 创建工作区。

        $ mkdir workspace
        $ cd workspace

2. 下载 manifest 清单仓库，初始化工作区。

        $ git repo init -u <manifest repository>

3. 按照 Manifest 清单仓库中的文件，下载各个子仓库的代码，并检出到工作区。

        $ git repo sync

4. 创建开发分支。

        $ git repo start --all <branch/name>

5. 在工作区中开发，每个仓库的改动，在各自仓库中完成提交。

6. 执行下面命令，扫描工作区所有仓库的改动，逐个向上游仓库发起代码评审。

        $ git repo upload
