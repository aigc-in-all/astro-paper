---
title: "快速创建 Next.js 项目"
pubDatetime: 2024-08-14T07:56:02.002Z
modDatetime: 2024-10-26T16:05:05.019Z
featured: false
draft: false
tags:
  - nextjs
  - shadcn-ui
description: 快速创建 Next.js 项目，包括一些基本配置
---

## Table of contents


## 创建 Next.js 项目

官网：https://nextjs.org/docs/getting-started/installation

```bash
pnpm create next-app@latest my-app --typescript --tailwind --eslint
```

更新依赖（可选）

```bash
pnpm update
```

## 安装 shadcn-ui 组件库

官网：https://ui.shadcn.com/docs/installation/next

### 初始化项目

```bash
pnpm dlx shadcn@latest init
```

在配置 `components.json` 时会提示风格选择（一般一路默认就行了）

```
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Do you want to use CSS variables for colors? › no / yes
```
根据情况选择就行，一般选默认，具体参考 [文档](https://ui.shadcn.com/themes)

### 安装组件

```bash
pnpm dlx shadcn@latest add button

# 常用
pnpm dlx shadcn@latest add button card input select sheet sonner
```

官方组件库：https://ui.shadcn.com/docs/components/accordion
