---
title: "Next.js + next-intl 项目部署到 Cloudflare Pages"
pubDatetime: 2024-12-01T08:06:58.299Z
modDatetime: 2024-12-01T08:06:58.299Z
featured: false
draft: false
tags:
  - nextjs
  - next-intl
  - cloudflare
  - 多语言
description: Next.js 多语言项目部署到 Cloudflare Pages，基于 next-intl 实现多语言切换
---

[next-intl](https://next-intl-docs.vercel.app/) 是一个 nextjs 流行的多语言库，接入简单，支持 app router 和 server components.

对于我们做出海网站来说，一般是先做英语（url不带语言标识），后续如果有量了再做多语言，所以一般的模式是：
- https://example.com/ 英文
- https://example.com/zh/ 中文
- https://example.com/jp/ 日文

正常情况下，如果你的项目部署在 Vercel 上，那么跟着 next-intl 的文档一步步走就行了，但是，走完之后发现默认语言的url也会有en标识，意味着网站的首页将会变成（https://example.com/en），这可不是我们想要的，因为 GSC 已经收录的首页是 https://example.com/。

当然 next-intl 也为我们想到了，所以提供了配置 `localePrefix: 'as-needed'`，这样默认语言就不会有 en 标识了。

至此，就可以愉快的把网站部署到 Vercel 上了。

但是，如果你的项目部署在 Cloudflare Pages 上呢？

Cloudflare Pages 是一个静态网站托管服务，意味着我们需要把 Next.js 项目构建成静态输出才能部署，当然 Next.js 确实是支持的，但是会有以下几个问题：

1. 静态输出的项目，不支持 middleware
2. `localePrefix: 'as-needed'` 失效，会使用 `localePrefix: 'always'`, 也就是说默认语言url会有 en 标识

那怎么办呢？

针对这个问题，我提供三种方案：

1. 在根目录下`app/`也创建一份专门用于英语的页面，比如 `app/layout.tsx`、`app/page.tsx`、`app/about/page.tsx`、...
2. 结合 Cloudflare 的 Rewrite 功能，把 `/` 重写成 `/en` （参考我的回答：https://stackoverflow.com/a/79112995/5221795）
3. 静态构建到 `out` 目录后，手动把 `out/en` 目录下的文件移动到 `out` 目录下

三种方式我都试过，只有第 3 种方式是改动最小的，并且不需要改变项目代码，所以这篇文章就是讲第 3 种方案。

## Table of contents

## 创建 Next.js 项目

跟着 Next.js 的官网操作就行了：https://nextjs.org/docs/app/getting-started/installation

## 安装 next-intl

跟着 Next.js 的官网操作就行了：https://next-intl-docs.vercel.app/docs/getting-started/app-router/with-i18n-routing

```bash
pnpm add next-intl
```

创建 `messages/en.json`

```json
{
  "HomePage": {
    "title": "Hello world!",
    "about": "Go to the about page"
  }
}
```

修改 `next.config.mjs`

```mjs
import createNextIntlPlugin from 'next-intl/plugin';
 
const withNextIntl = createNextIntlPlugin();
 
/** @type {import('next').NextConfig} */
const nextConfig = {};
 
export default withNextIntl(nextConfig);
```

创建 `i18n/routing.ts`

```ts
import { defineRouting } from 'next-intl/routing';
import { createNavigation } from 'next-intl/navigation';
import { isDevelopment } from '@/lib/utils';

export const routing = defineRouting({
    // A list of all locales that are supported
    locales: ['en', 'zh-TW', 'de'],

    // Used when no locale matches
    defaultLocale: 'en',

    // Prefix the locale to the pathname based on environment
    localePrefix: isDevelopment() ? 'always' : 'as-needed'
});

export type Locale = (typeof routing.locales)[number];

// Lightweight wrappers around Next.js' navigation APIs
// that will consider the routing configuration
export const { Link, redirect, usePathname, useRouter, getPathname } =
    createNavigation(routing);
```

创建 `i18n/request.ts`
```ts
import { getRequestConfig } from 'next-intl/server';
import { routing } from './routing';

export default getRequestConfig(async ({ requestLocale }) => {
    // This typically corresponds to the `[locale]` segment
    let locale = await requestLocale;

    // Ensure that a valid locale is used
    if (!locale || !routing.locales.includes(locale as any)) {
        locale = routing.defaultLocale;
    }

    return {
        locale,
        messages: (await import(`../messages/${locale}.json`)).default
    };
});
```

修改 `app/[locale]/layout.tsx`
```tsx
import {NextIntlClientProvider} from 'next-intl';
import {getMessages} from 'next-intl/server';
import {notFound} from 'next/navigation';
import {routing} from '@/i18n/routing';
 
export default async function LocaleLayout({
  children,
  params: {locale}
}: {
  children: React.ReactNode;
  params: {locale: string};
}) {
  // Ensure that the incoming `locale` is valid
  if (!routing.locales.includes(locale as any)) {
    notFound();
  }
 
  // Providing all messages to the client
  // side is the easiest way to get started
  const messages = await getMessages();
 
  return (
    <html lang={locale}>
      <body>
        <NextIntlClientProvider messages={messages}>
          {children}
        </NextIntlClientProvider>
      </body>
    </html>
  );
}
```

修改 `app/[locale]/page.tsx`

```tsx
import {useTranslations} from 'next-intl';
import {Link} from '@/i18n/routing';
 
export default function HomePage() {
  const t = useTranslations('HomePage');
  return (
    <div>
      <h1>{t('title')}</h1>
      <Link href="/about">{t('about')}</Link>
    </div>
  );
}
```

## 配置 Static Export

### 1. 更改 next.config.js

```js

import createNextIntlPlugin from 'next-intl/plugin';

const withNextIntl = createNextIntlPlugin();

/** @type {import('next').NextConfig} */
const nextConfig = {
    output: 'export'
};

export default withNextIntl(nextConfig);

```

### 2. 创建脚本: `script/moveEnPages.js`

```js
const fs = require('fs-extra');
const path = require('path');

async function moveEnPages() {
    const outDir = 'out';
    const enDir = path.join(outDir, 'en');

    try {
        // 重命名 en.html 为 index.html
        const enHtmlPath = path.join(outDir, 'en.html');
        const indexHtmlPath = path.join(outDir, 'index.html');
        if (fs.existsSync(enHtmlPath)) {
            await fs.rename(enHtmlPath, indexHtmlPath);
            console.log('重命名: en.html -> index.html');
        }

        // 确保英文页面目录存在
        if (!fs.existsSync(enDir)) {
            console.error('英文页面目录不存在:', enDir);
            process.exit(1);
        }

        // 将 en 目录下的所有内容复制到 out 目录
        await fs.copy(enDir, outDir);
        console.log('英文页面已复制到根目录');

        // 删除 en 目录
        await fs.remove(enDir);
        console.log('已删除 en 目录');

    } catch (error) {
        console.error('处理文件时出错:', error);
        process.exit(1);
    }
}

moveEnPages(); 
```

### 3. 修改 package.json

```json
{
    "scripts": {
        "moveEnPages": "node scripts/moveEnPages.js",
        "deploy": "next build && npm run moveEnPages && wrangler pages deploy out"
    }
}
```

### 4. 验证
```
pnpm run build
pnpm run moveEnPages

# 以静态方式访问 out 目录
cd out
npx http-server
```

类似访问：http://127.0.0.1:8080/，查看是否正常


### 4. 部署

```bash
pnpm run deploy
```

最后的效果：https://nextjs-template-7n9.pages.dev/
