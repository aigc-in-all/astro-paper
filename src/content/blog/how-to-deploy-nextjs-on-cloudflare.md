---
title: "怎么把 Next.js 项目部署到 Cloudflare ？"
pubDatetime: 2024-08-16T02:07:46.187Z
featured: false
draft: false
tags:
  - nextjs
  - cloudflare
description: 记录怎么把 Next.js 项目部署到 Cloudflare Pages
---

这篇文章主要用于记录怎么在 Cloudflare Pages 上部署 Next.js 项目。包括一些常见的问题和解决方案。

[Cloudflare Framework Guides](https://developers.cloudflare.com/pages/framework-guides/nextjs/ssr/)

## Table of contents

## 部署（使用命令行）

[文档](https://developers.cloudflare.com/pages/framework-guides/nextjs/ssr/get-started/)

### 直接创建新项目

如果是创建新项目，直接使用 cloudflare 创建即可。

```bash
# npm
npm create cloudflare@latest my-next-app -- --framework=next
# pnpm
pnpm create cloudflare@latest my-next-app -- --framework=next
```
[npm vs pnpm vs yarn](/posts/npm-pnpm-yarn-differences-and-commands)

### 已存在的项目

如果是已存在的项目，需要手动配置。

#### 1. 安装 next-on-pages

```bash
# npm
npm install --save-dev @cloudflare/next-on-pages
# pnpm
pnpm add -D @cloudflare/next-on-pages
```

#### 2. 添加 `wrangler.toml` 文件

在项目根目录创建 `wrangler.toml`

```toml
name = "my-app"
compatibility_date = "2024-07-29"
compatibility_flags = ["nodejs_compat"]
pages_build_output_dir = ".vercel/output/static"
```

#### 3. 更新 `next.config.mjs`

```js
import { setupDevPlatform } from "@cloudflare/next-on-pages/next-dev";

/** @type {import('next').NextConfig} */
const nextConfig = {};

if (process.env.NODE_ENV === "development") {
  await setupDevPlatform();
}

export default nextConfig;
```

#### 4. 确保所有服务端渲染的路由都使用 Edge Runtime

对所有的 api 路由文件 route.ts 和所有的页面路由文件 page.tsx 都添加一行代码，指定使用 edge 运行时：

```js
export const runtime = "edge";
```

什么是 Edge Runtime？

![nodejs edge runtime](@assets/images/nodejs-edge-runtime.jpg)

#### 5. 更新 `package.json`

```json
"pages:build": "npx @cloudflare/next-on-pages",
"preview": "npm run pages:build && wrangler pages dev",
"deploy": "npm run pages:build && wrangler pages deploy"
```

* `npm run pages:build`: Runs next build, and then transforms its output to be compatible with Cloudflare Pages.
* `npm run preview`: Builds your app, and runs it locally in workerd, the open-source Workers Runtime. (next dev will only run your app in Node.js)
* `npm run deploy`: Builds your app, and then deploys it to Cloudflare

#### 6. 部署到 Cloudflare Pages

```bash
# npm
npm run deploy
# pnpm
pnpm run deploy
```
## 部署（直接从 Github 导入）
