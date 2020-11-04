---
title: "下载"
draft: false
weight: 20
---

根据您的平台类型（Linux、Mac、或是 Windows），CPU 类型（amd64 或 386）下载合适的软件包。

现代个人笔记本电脑请选择amd64下载。Linux开发机输入`uname -a`，输出包含`x86_64`选择amd64下载，输出包含`i386`选择386下载。

注：

1. sha 文件是软件包的 sha256 校验码。
2. pgp 文件是校验码文件的 PGP 签名。签名者公钥: [公钥1](/download/pubkey/key1.pub)、[公钥2](/download/pubkey/key2.pub)

## 最新版本

## 0.7.6 (2020-10-13)

### 变更

Improvements:

+ Add http proxy support in command git-repo upgrade
+ Fix http proxy support for API call of ssh-info

Bugfix:

+ go 1.15 compatible issue: not add rune with int

### 文件

+ Linux (amd64): [git-repo-0.7.6-Linux-64.tar.gz](/releases/v0.7.6/git-repo-0.7.6-Linux-64.tar.gz)
  SHA-256: 523feeeea0cb65769ad881d0bc08ab1e6ecf9a1cbeb06acacc407ab8961ee47a

+ Linux (386): [git-repo-0.7.6-Linux-32.tar.gz](/releases/v0.7.6/git-repo-0.7.6-Linux-32.tar.gz)
  SHA-256: 04705a11c314d249595c42a682d4bb176d7e9a2904076484b840ca1f638de19c

+ macOS (amd64): [git-repo-0.7.6-macOS-64.tar.gz](/releases/v0.7.6/git-repo-0.7.6-macOS-64.tar.gz)
  SHA-256: b2d11133de94badbce34ea3e738dfd4c949bc9819f0c918be8700c3e9248c8fa

+ macOS (386): [git-repo-0.7.6-macOS-32.tar.gz](/releases/v0.7.6/git-repo-0.7.6-macOS-32.tar.gz)
  SHA-256: 0bf536a9d626f6395238e86288e5dbb6d15cfbbd558d7596d2d897b9c6b5a3e5

+ Windows (amd64): [git-repo-0.7.6-Windows-64.zip](/releases/v0.7.6/git-repo-0.7.6-Windows-64.zip)
  SHA-256: 55952d53eae833fead90fb4719e60f262cd9b7b8648d2f4e669311fa42f5d77d

+ Windows (386): [git-repo-0.7.6-Windows-32.zip](/releases/v0.7.6/git-repo-0.7.6-Windows-32.zip)
  SHA-256: f79afd6e658e49ca5540049241e38827dcd0e2dbc9f4c7bac7be5d2a205be275


## 历史版本
