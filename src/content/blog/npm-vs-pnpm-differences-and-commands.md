---
title: "NPM vs PNPM: 区别和命令对比"
pubDatetime: 2024-08-13T11:12:44.065Z
featured: false
draft: false
tags:
  - npm
  - pnpm
  - nodejs
description: 详细对比 npm 和 pnpm 的特性与命令
---

在 Node.js 生态系统中,包管理器扮演着至关重要的角色。npm (Node Package Manager) 长期以来一直是最流行的选择,但近年来,pnpm (performant npm) 因其独特的优势正在获得越来越多的关注。本文将深入探讨 npm 和 pnpm 之间的区别,并对比它们的常用命令。

## Table of contents

## npm 和 pnpm 的主要区别

### 1. 依赖存储方式

- **npm**: 使用扁平的依赖结构,所有包都安装在项目根目录的 `node_modules` 文件夹中。
- **pnpm**: 使用内容寻址存储,所有包都存储在一个集中的位置,通过符号链接连接到项目。

### 2. 性能和磁盘空间使用

- **npm**: 可能导致重复安装和较大的磁盘空间占用。
- **pnpm**: 通常安装速度更快,并且大幅减少磁盘空间使用。

### 3. "幽灵依赖"问题

- **npm**: 扁平结构可能导致访问未在 `package.json` 中声明的依赖。
- **pnpm**: 严格遵守依赖声明,只允许访问明确声明的依赖。

### 4. Monorepo 支持

- **npm**: 从版本 7 开始支持 workspaces。
- **pnpm**: 对 monorepo 的支持更加成熟和高效。

### 5. 锁文件

- **npm**: 使用 `package-lock.json`
- **pnpm**: 使用 `pnpm-lock.yaml`

## 命令对比

以下是 npm 和 pnpm 在常见操作上的命令对比:

| 操作 | npm | pnpm |
|------|-----|------|
| 安装所有依赖 | `npm install` | `pnpm install` |
| 安装特定依赖 | `npm install package-name` | `pnpm add package-name` |
| 安装开发依赖 | `npm install --save-dev package-name` | `pnpm add -D package-name` |
| 全局安装包 | `npm install -g package-name` | `pnpm add -g package-name` |
| 执行安装的包 | `npx package-name` | `pnpm dlx package-name` |
| 运行脚本 | `npm run script-name` | `pnpm run script-name` 或 `pnpm script-name` |
| 更新所有依赖 | `npm update` | `pnpm update` |
| 更新特定依赖 | `npm update package-name` | `pnpm update package-name` |
| 删除依赖 | `npm uninstall package-name` | `pnpm remove package-name` |
| 清理缓存 | `npm cache clean --force` | `pnpm store prune` |
| 查看包信息 | `npm view package-name` | `pnpm view package-name` |
| 列出已安装的包 | `npm list` | `pnpm list` |
| 审计依赖安全性 | `npm audit` | `pnpm audit` |
| 初始化新项目 | `npm init` | `pnpm init` |
| 发布包 | `npm publish` | `pnpm publish` |

## 命令使用注意事项

1. **安装依赖**: pnpm 使用 `add` 而不是 `install` 来添加新包。
2. **执行安装的包**: npm 使用 `npx`,而 pnpm 使用 `pnpm dlx`。
3. **运行脚本**: pnpm 允许直接使用 `pnpm script-name`,省略 `run`。
4. **更新依赖**: 两者都支持 `--latest` 标志来更新到最新版本。
5. **删除依赖**: npm 使用 `uninstall`,而 pnpm 使用 `remove`。
6. **清理缓存**: pnpm 的 `store prune` 命令会删除不再引用的包。
