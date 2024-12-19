---
title: "使用 GitHub SSH 管理多个 GitHub 账号"
pubDatetime: 2024-12-19T08:54:56.097Z
featured: false
draft: false
tags:
  - github
  - ssh
description: 使用 GitHub SSH 管理多个 GitHub 账号
---

如果我们有多个 Github 账号，使用 ssh 的方式向不同的仓库提交代码时，需要配置多个 ssh key。

## Table of contents

## 具体步骤

1. 为两个账号生成不同的 SSH 密钥：

```bash
# 为第一个账号生成密钥
ssh-keygen -t ed25519 -C "your-first-email@example.com" -f ~/.ssh/id_ed25519_github1

# 为第二个账号生成密钥
ssh-keygen -t ed25519 -C "your-second-email@example.com" -f ~/.ssh/id_ed25519_github2
```

2. 在 ~/.ssh/config 文件中添加配置(如果没有这个文件就创建一个):
```bash
# 第一个 GitHub 账号
Host github-1
    HostName github.com
    User git                # 这里的 User 必须是 git
    IdentityFile ~/.ssh/id_ed25519_github1
    
# 第二个 GitHub 账号
Host github-2
    HostName github.com
    User git                # 这里的 User 必须是 git
    IdentityFile ~/.ssh/id_ed25519_github2
```

3. 将公钥添加到对应的 GitHub 账号中:

* 复制 ~/.ssh/id_ed25519_github1.pub 的内容到第一个 GitHub 账号
* 复制 ~/.ssh/id_ed25519_github2.pub 的内容到第二个 GitHub 账号

4. 使用时，需要修改仓库的远程 URL。例如:
```bash
# 第一个账号的仓库
git remote add origin git@github-1:username1/repo.git

# 第二个账号的仓库
git remote add origin git@github-2:username2/repo.git
```

可以用以下命令测试连接:
```bash
ssh -T github-1
ssh -T github-2
```

这样配置后，每个仓库都会使用对应的 SSH 密钥，实现两个账号的分离。

## ed25519 是什么？

ed25519 是一种现代的加密算法，它的全名是 Edwards-curve Digital Signature Algorithm (EdDSA)，使用的是 Curve25519 曲线。我来解释一下它的几个主要特点：

1. 优势：
- 更安全：提供了很高的安全性（128位安全性）
- 更快速：比传统的 RSA 和 DSA 算法运算速度更快
- 密钥更短：公钥只有32字节，签名只有64字节，比 RSA 密钥小很多
- 更难出错：算法设计使其较难因为实现不当而导致安全问题

2. 对比传统的 SSH 密钥类型：
- RSA：最古老和最常见的，需要较长密钥（2048位或4096位）才安全
- DSA：较旧的算法，现在已不推荐使用
- ECDSA：也是椭圆曲线算法，但有一些潜在的安全担忧
- Ed25519：目前最推荐的选择

如果你想用更传统的 RSA，命令也很简单：
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

不过既然是新建密钥，我建议还是使用 Ed25519，因为它既现代又安全。

## config 配置里面的 User 是什么？

config 文件中的 `User` 指定的是 SSH 连接时使用的用户名。在 GitHub 的情况下，这个值总是设置为 `git`，因为所有的 Git 操作都是通过 git 用户来进行的。

举个例子：
```
Host github-1
    HostName github.com
    User git                    # 这里的 User 必须是 git
    IdentityFile ~/.ssh/id_ed25519_github1
```

这里解释下为什么是 git：
1. 当你克隆一个 GitHub 仓库时，URL 格式是这样的：`git@github.com:username/repo.git`
2. 这里的 `git@` 就表示使用 git 用户连接到 github.com
3. 实际上你的 GitHub 用户名是通过 SSH 密钥来识别的，而不是通过这个 User 值

所以不管你有几个 GitHub 账号，config 文件中的 User 都应该设置为 git。这是 GitHub 的特定要求，而不是你的 GitHub 用户名。

