---
title: "快速创建 Next.js 项目"
pubDatetime: 2024-08-14T07:56:02.002Z
modDatetime: 2024-08-14T08:13:58.871Z
featured: false
draft: false
tags:
  - nextjs
description: 快速创建 Next.js 项目，包括一些基本配置
---

## Table of contents


## 创建 Next.js 项目

```bash
pnpm create next-app@latest my-app --typescript --tailwind --eslint
```

更新依赖

```bash
pnpm update
```

## 安装 shadcn-ui 组件库

### 初始化项目

```bash
pnpm dlx shadcn-ui@latest init
```

在配置 `components.json` 时会提示风格选择

```
Which style would you like to use? › Default
Which color would you like to use as base color? › Slate
Do you want to use CSS variables for colors? › no / yes
```
根据情况选择就行，一般选默认，具体参考 [文档](https://ui.shadcn.com/themes)

### 安装组件

```bash
pnpm dlx shadcn-ui@latest add button
```

[官方组件库](https://ui.shadcn.com/docs/components/accordion)

### 安装图标库

```bash
pnpm install lucide-react
```
更多其它图标库选择，可以 [参考](/nextjs-fullstack-tech-stack-guide)

## 配置 Markdown（隐私条款、利用规约、博客）

[给 Next.js 项目添加博客](/add-blog-to-nextjs-project)
