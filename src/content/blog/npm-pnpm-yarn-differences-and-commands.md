---
title: "npm vs pnpm vs yarn: 全面对比和命令指南"
pubDatetime: 2024-08-13T11:12:44.065Z
modDatetime: 2024-08-14T06:12:45.664Z
featured: false
draft: false
tags:
  - npm
  - pnpm
  - yarn
  - nodejs
description: 深入对比npm、pnpm和yarn的特性与命令，探讨它们在依赖管理、性能和使用体验上的差异
---

# npm vs yarn vs pnpm: 全面对比和命令指南

在 Node.js 生态系统中，包管理器扮演着至关重要的角色。npm（Node Package Manager）作为最早且最广泛使用的选择，yarn 作为一个强大的替代品，以及新兴的 pnpm（performant npm），都为开发者提供了管理项目依赖的强大工具。本文将深入探讨这三个包管理器之间的区别，并对比它们的常用命令。

## Table of contents

## npm、pnpm 和 yarn 的主要区别

### 1. 依赖存储方式

- **npm**: 使用扁平的依赖结构，所有包都安装在项目根目录的 `node_modules` 文件夹中。
- **pnpm**: 使用内容寻址存储，所有包都存储在一个集中的位置，通过符号链接连接到项目。
- **yarn**: 默认使用扁平的依赖结构，类似于 npm。但在 yarn 2+（Berry）中，引入了 Plug'n'Play（PnP）模式，不再使用 `node_modules`。

### 2. 性能和磁盘空间使用

- **npm**: 安装速度相对较慢，可能导致重复安装和较大的磁盘空间占用。
- **pnpm**: 安装速度通常最快，大幅减少磁盘空间使用。
- **yarn**: 通常比 npm 快，并提供了离线模式。yarn 2+ 的 PnP 模式进一步提高了性能和减少了磁盘使用。

### 3. 锁文件

- **npm**: 使用 `package-lock.json`
- **pnpm**: 使用 `pnpm-lock.yaml`
- **yarn**: 使用 `yarn.lock`（yarn 1.x）或 `.yarnrc.yml`（yarn 2+）

### 4. "幽灵依赖"问题

- **npm** 和 **yarn 1.x**: 扁平结构可能导致访问未在 `package.json` 中声明的依赖。
- **pnpm**: 严格遵守依赖声明，只允许访问明确声明的依赖。
- **yarn 2+ (PnP)**: 严格限制只能访问直接依赖。

### 5. 工作区（Workspaces）支持

- **npm**: 从版本 7 开始支持工作区。
- **pnpm**: 对工作区的支持成熟且高效。
- **yarn**: excellent 对工作区的支持，特别是在 monorepo 项目中。

## 命令对比

以下是 npm、pnpm 和 yarn 在常见操作上的命令对比:

| 操作 | npm | pnpm | yarn |
|------|-----|------|------|
| 创建项目 | `npm create package-name` | `pnpm create package-name` | `yarn create package-name` |
| 安装所有依赖 | `npm install` | `pnpm install` | `yarn` 或 `yarn install` |
| 安装特定依赖 | `npm install package-name` | `pnpm add package-name` | `yarn add package-name` |
| 安装开发依赖 | `npm install --save-dev package-name` | `pnpm add -D package-name` | `yarn add --dev package-name` |
| 全局安装包 | `npm install -g package-name` | `pnpm add -g package-name` | `yarn global add package-name` |
| 执行安装的包 | `npx package-name` | `pnpm dlx package-name` | `yarn dlx package-name` |
| 运行脚本 | `npm run script-name` | `pnpm run script-name` 或 `pnpm script-name` | `yarn run script-name` 或 `yarn script-name` |
| 更新所有依赖 | `npm update` | `pnpm update` | `yarn upgrade` |
| 更新特定依赖 | `npm update package-name` | `pnpm update package-name` | `yarn upgrade package-name` |
| 删除依赖 | `npm uninstall package-name` | `pnpm remove package-name` | `yarn remove package-name` |
| 清理缓存 | `npm cache clean --force` | `pnpm store prune` | `yarn cache clean` |
| 查看包信息 | `npm view package-name` | `pnpm view package-name` | `yarn info package-name` |
| 列出已安装的包 | `npm list` | `pnpm list` | `yarn list` |
| 审计依赖安全性 | `npm audit` | `pnpm audit` | `yarn audit` |
| 初始化新项目 | `npm init` | `pnpm init` | `yarn init` |
| 发布包 | `npm publish` | `pnpm publish` | `yarn publish` |


## 选择建议

1. **如果你需要最广泛的兼容性和社区支持**：选择 npm。它是最广泛使用的包管理器，几乎所有 Node.js 项目都支持它。

2. **如果你在寻求更好的性能和工作区功能**：考虑 yarn。特别是如果你在使用 monorepo 结构，yarn 的工作区功能非常强大。

3. **如果你关注磁盘空间使用和严格的依赖管理**：pnpm 可能是最佳选择。它提供了优秀的性能，同时大幅减少了磁盘空间使用。

4. **如果你正在开始一个新项目**：考虑尝试 pnpm 或 yarn 2+。这些新一代的包管理器提供了许多现代化的特性和优化。

## 结论

npm、yarn 和 pnpm 各有其优势和特点。npm 提供了最广泛的兼容性，yarn 带来了性能提升和强大的工作区功能，而 pnpm 则在磁盘空间使用和依赖管理方面表现出色。

选择使用哪个包管理器取决于你的具体需求、项目规模和团队偏好。好消息是，这三个工具的基本命令非常相似，这使得在它们之间切换相对容易。

无论你选择哪个工具，了解这些差异和命令可以帮助你更有效地管理你的 Node.js 项目。持续关注包管理器的发展和新特性，将有助于你在未来做出更明智的选择。
