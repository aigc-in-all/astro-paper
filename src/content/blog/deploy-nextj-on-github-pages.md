---
title: "把 Next.js 项目部署到 GitHub Pages 的两种方式"
pubDatetime: 2024-12-15T15:24:13.810Z
featured: false
draft: false
tags:
  - nextjs
  - github
description: 详细记录把 Next.js 项目部署到 GitHub Pages 的两种方式，一种是单仓库，另一种是多个仓库
---

本文主要记录把 Next.js 项目部署到 GitHub Pages 的两种方式，一种是单仓库，另一种是多仓库。它们有什么区别？

- 单仓库：即直接在源码仓库（public）中配置 Github Action 构建项目，然后部署到 Github Pages
- 多仓库：即源码仓库（private 或 public）和部署仓库（public）分开，源码仓库中配置 Github Action 构建产物，然后把产物提交到部署仓库，部署仓库配置 Github Pages

> 最终用于部署 Github Pages 的仓库必须是 public

## Table of contents

## 方式一（单仓库）

单个仓库的方式，关键在于github action的配置。

在项目根目录创建 `.github/workflows/nextjs.yml` 文件，内容如下：

```yaml
name: Deploy Next.js site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      
      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          
      - name: Install pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8
          run_install: false

      - name: Get pnpm store directory
        id: pnpm-cache
        shell: bash
        run: |
          echo "STORE_PATH=$(pnpm store path)" >> $GITHUB_OUTPUT

      - name: Setup pnpm cache
        uses: actions/cache@v4
        with:
          path: ${{ steps.pnpm-cache.outputs.STORE_PATH }}
          key: ${{ runner.os }}-pnpm-store-${{ hashFiles('**/pnpm-lock.yaml') }}
          restore-keys: |
            ${{ runner.os }}-pnpm-store-

      - name: Setup Pages
        uses: actions/configure-pages@v5
        with:
          static_site_generator: next
          
      - name: Install dependencies
        run: pnpm install

      - name: Build with Next.js
        run: pnpm next build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./out

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4

```

然后在项目设置里，找到 Pages 选项，`Build and deployment` 里 `Source` 选择 `GitHub Actions` 即可。

## 方式二（多仓库）

假设我们用 A 仓库存储源码（private 或 public都可以），B 仓库存产物（必须 public），用 B 仓库部署到 Github Pages。

### 1. 源码仓库（A）

#### 配置 Github Action

在源码仓库中创建 `.github/workflows/deploy.yml` 文件，内容如下：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source Repository
        uses: actions/checkout@v3

      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          
      - name: Setup pnpm
        uses: pnpm/action-setup@v2
        with:
          version: 8

      - name: Install Dependencies
        run: pnpm install

      - name: Build Project
        run: pnpm run build
      
      - name: Copy README
        run: |
          if [ -f README.md ]; then
            cp README.md out/
          fi

      - name: Deploy to GitHub Pages Repository
        uses: peaceiris/actions-gh-pages@v3
        with:
          personal_token: ${{ secrets.DEPLOY_TOKEN }}
          external_repository: xxx/xxx.github.io  # B 仓库的地址
          publish_dir: ./out
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'

```

#### 配置 Github Token

注意，以上配置中使用了 `DEPLOY_TOKEN`，这个 token 是用于通过脚本向 B 仓库提交代码。下面是获取这个 token 的步骤：

1. 首先，需要创建 Personal Access Token (PAT)：
   - 登录 GitHub，点击右上角头像
   - 选择 Settings
   - 滚动到底部，点击左侧菜单中的 Developer settings
   - 选择 Personal access tokens → Tokens (classic)
   - 点击 Generate new token → Generate new token (classic)
   - 给 token 起个名字，比如 `DEPLOY_TOKEN`
   - 设置过期时间（建议选择合适的期限，比如90天）
   - 勾选权限范围：
     - 必须勾选 `repo` （这会自动包含所有 repo 相关权限）
   - 点击底部的 Generate token
   - **立即复制生成的 token！这个 token 只显示一次**

2. 然后，在源码仓库（仓库A）中设置这个 token：
   - 进入源码仓库
   - 点击 Settings 标签
   - 在左侧菜单找到 Secrets and variables → Actions
   - 点击 New repository secret
   - Name 填写 `DEPLOY_TOKEN`（必须和 workflow 文件中的名字一致）
   - Value 填写你刚才复制的 PAT
   - 点击 Add secret

这样，当 GitHub Actions 运行时，`${{ secrets.DEPLOY_TOKEN }}` 就会被自动替换为你设置的 PAT，使 workflow 有权限推送代码到仓库 B。

### 2. 部署仓库（B）

从源码仓库（A）提交到部署仓库（B）的代码会在 `gh-pages` 分支上，所以只需要需要配置 Github Pages 的源码为 `gh-pages` 分支即可访问。

### 举例

源码仓库名：example

部署仓库名：example.github.io

部署完成后，可以通过 [https://example.github.io](https://example.github.io/) 直接访问
