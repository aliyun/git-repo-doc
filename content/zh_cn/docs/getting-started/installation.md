---
title: "安装 git-repo"
draft: false
weight: 12
enableToc: false
---

## 依赖

git-repo 依赖 git，在安装 git-repo 之前，需要确保已经安装了 git 2.10.0 以上版本。


## 下载和安装 git-repo

1. 访问 git-repo 的下载页面: [https://git-repo.info/zh_cn/download/](https://git-repo.info/zh_cn/download/)。
   根据您的平台类型（Linux、Mac、或是 Windows），CPU 类型（amd64 或 386）下载并展开压缩包。

2. 如果是 Linux 或者 Mac OS X 平台，为下载文件设置可执行权限。例如：

        $ chmod a+x git-repo

3. 将下载的可执行文件复制到可执行目录。对于 Linux 和 Mac OS X 平台，这个目录可以是
   `/usr/local/bin` 或者其他可执行目录。对于 Windows 平台，这个目录可能是 `C:\Windows\system32`。

        $ cp git-repo /usr/local/bin/


