---
title: "给 Next.js 项目添加博客功能（Cloudflare Pages）"
pubDatetime: 2024-08-17T07:02:10.150Z
featured: false
draft: false
tags:
  - nextjs
  - cloudflare
description: 快速给 Next.js 项目添加博客功能，包括隐私条款、利用规约等
---

使用 Next.js 创建完网站后，可以使用如下方法快速添加博客功能，博客使用 Markdown 文件存储。

也可以使用这套方法快速添加其他内容，如隐私条款、利用规约等。

> 这篇文章所讲的方法适用于所有部署（不仅限于 Cloudflare Pages，使用 Vercel 部署也可以）。

与这篇文章（[给 Next.js 项目添加博客功能](/posts/add-blog-to-nextjs-project)）的方法有什么区别？

因为 Cloudflare Pages 部署 Next.js 项目时，使用的是 Edge Runtime，无法使用 Node.js 的 fs 模块读取文件，所以这篇文章主要是讲另一种方法。

核心原理：**在构建时**，将 Markdown 文件解析并生成 JSON 文件，然后在 Edge Runtime 读取 JSON 文件实现。

## Table of contents

## 添加依赖
```bash
pnpm add gray-matter next-mdx-remote
pnpm add -D @tailwindcss/typography
pnpm add lucide-react
```
> - gray-matter： 解析 Markdown 文件元数据
> - next-mdx-remote：渲染 Markdown 文件
> - @tailwindcss/typography：添加 Tailwind Typography 插件，用于渲染 Markdown 文件

## 配置 tailwind.config.ts

```ts
// tailwind.config.ts
import type { Config } from "tailwindcss";
import typography from "@tailwindcss/typography"; <-- Add this line

const config: Config = {
  content: [
    "./pages/**/*.{js,ts,jsx,tsx,mdx}",
    "./components/**/*.{js,ts,jsx,tsx,mdx}",
    "./app/**/*.{js,ts,jsx,tsx,mdx}",
  ],
  theme: {
    extend: {
      backgroundImage: {
        "gradient-radial": "radial-gradient(var(--tw-gradient-stops))",
        "gradient-conic":
          "conic-gradient(from 180deg at 50% 50%, var(--tw-gradient-stops))",
      },
    },
  },
  plugins: [typography], <-- Modify this line
};
export default config;
```

## 创建示例 Markdown 文章

我们将会创建以下示例文章，注意目录结构：
```
posts/
  blog/
    welcome.md
  about-us.md
  privacy-policy.md
  terms-of-service.md
```

下面分别开始创建这些文件：

```md
// posts/about-us.md

---
title: "About Us"
slug: "about"
date: "2024-05-15"
author: "Alex Bennet"
description: "About Us"
---

# About Us

about us
```

```md
// posts/privacy-policy.md

---
title: "Privacy Policy"
slug: "privacy"
date: "2024-05-15"
author: "Alex Bennet"
description: "privacy policy"
---

# Privacy Policy

privacy policy
```

```md
// posts/terms-of-service.md

---
title: "Terms of Service"
slug: "terms"
date: "2024-05-15"
author: "Alex Bennet"
description: "terms of service"
---

# Terms of Service

terms of service
```

```md
# // posts/blog/welcome.md

---
title: "welcome"
slug: "welcome"
date: "2024-05-15"
author: "Alex Bennet"
description: "Welcome to my blog"
---

welcome

```

## 创建 Markdown 读取配置

文章开头介绍了，这里要先把 Markdown 文件解析成 JSON 文件，然后直接读取 JSON 文件实现，避免使用 `fs` 和 `path` 这些在 Edge Runtime 不可用的模块。

### 创建解析脚本

```javascript
// scripts/generate-posts-data.js

const fs = require('fs');
const path = require('path');
const matter = require('gray-matter');

const postsDir = path.join(process.cwd(), 'posts');
const outputDir = path.join(process.cwd(), 'lib', 'posts');
const blogOutputDir = path.join(outputDir, 'blog');
const blogDir = path.join(postsDir, 'blog');

function ensureDirectoryExistence(filePath) {
    const dirname = path.dirname(filePath);
    if (fs.existsSync(dirname)) {
        return true;
    }
    ensureDirectoryExistence(dirname);
    fs.mkdirSync(dirname);
}

function processMarkdownFile(filePath) {
    const content = fs.readFileSync(filePath, 'utf8');
    const { data, content: markdownContent } = matter(content);
    return {
        metadata: {
            ...data,
        },
        content: markdownContent
    };
}

function generateBlogData() {
    const blogFiles = fs.readdirSync(blogDir).filter(file => file.endsWith('.md'));
    const blogPosts = blogFiles.map(file => {
        const filePath = path.join(blogDir, file);
        const { metadata, content } = processMarkdownFile(filePath);
        return {
            slug: path.parse(file).name,
            metadata: { ...metadata, slug: path.parse(file).name },
            content
        };
    });

    blogPosts.sort((a, b) => new Date(b.metadata.date) - new Date(a.metadata.date));

    const blogMetadata = blogPosts.map(post => post.metadata);
    const blogContent = Object.fromEntries(blogPosts.map(post => [post.slug, { metadata: post.metadata, content: post.content }]));

    const metadataPath = path.join(blogOutputDir, 'blog-metadata.json');
    const contentPath = path.join(blogOutputDir, 'blog-content.json');

    ensureDirectoryExistence(metadataPath);
    ensureDirectoryExistence(contentPath);

    fs.writeFileSync(metadataPath, JSON.stringify(blogMetadata, null, 2));
    fs.writeFileSync(contentPath, JSON.stringify(blogContent, null, 2));

    console.log(`Blog metadata generated at: ${metadataPath}`);
    console.log(`Blog content generated at: ${contentPath}`);
}

function generateOtherPostData(dir = postsDir) {
    const items = fs.readdirSync(dir, { withFileTypes: true });

    for (const item of items) {
        const itemPath = path.join(dir, item.name);

        if (item.isDirectory()) {
            if (item.name !== 'blog') {
                generateOtherContentData(itemPath);
            }
        } else if (item.isFile() && item.name.endsWith('.md')) {
            const { metadata, content } = processMarkdownFile(itemPath);
            const relativePath = path.relative(postsDir, itemPath);
            const outputPath = path.join(outputDir, `${path.parse(relativePath).name}.json`);

            ensureDirectoryExistence(outputPath);
            fs.writeFileSync(outputPath, JSON.stringify({ metadata, content }, null, 2));
            console.log(`Generated: ${outputPath}`);
        }
    }
}

// Delete the output directory if it exists
console.log(`🚀 Cleaning up ${outputDir}...`);
if (fs.existsSync(outputDir)) {
    fs.rmSync(outputDir, { recursive: true, force: true });
}

// Ensure output directories exist
console.log(`🚀 Creating ${outputDir} and ${blogOutputDir}...`);
ensureDirectoryExistence(outputDir);
ensureDirectoryExistence(blogOutputDir);

// Generate blog data
console.log('🚀 Generating blog data...');
generateBlogData();

// Generate other post data
console.log('🚀 Generating other post data...');
generateOtherPostData();

console.log('✅ Post data generation completed.');
```

这个脚本其实现的目的就是将 Markdown 文件解析成 JSON 文件，然后生成如下目录结构：

```
lib/
  posts/
    blog/
      blog-metadata.json <-- blog 元数据列表，包含 /posts/blog/ 目录下所有md文章的元数据，用于展示博客列表
      blog-content.json <-- blog 内容列表，包含 /posts/blog/ 目录下所有md文章的内容
    about-us.json
    privacy-policy.json
    terms-of-service.json
```

> 为什么不直接生成一个 JSON 文件？
>
> 这是为了性能优化！博客列表页面是服务端渲染，而博客文章页面是编译时静态生成，如果全都在一个 JSON 文件里，那全部加载所有博客内容到内存会影响性能，首次打开页面会很慢。

### 配置解析脚本

```json
{
  ...
  "scripts": {
    "posts": "node scripts/generate-posts-data.js", <-- Add this line
    "dev": "pnpm posts && next dev", <-- Modify this line
    "build": "pnpm posts && next build", <-- Modify this line
    "start": "pnpm posts && next start", <-- Modify this line
    "lint": "pnpm posts && next lint", <-- Modify this line
    "pages:build": "pnpm posts && pnpm next-on-pages", <-- Modify this line
    "preview": "pnpm pages:build && wrangler pages dev",
    "deploy": "pnpm pages:build && wrangler pages deploy",
    "cf-typegen": "wrangler types --env-interface CloudflareEnv env.d.ts"
  },
  ...
}
```

其实就是给 `dev`、`build`、`start`、`lint`、`pages:build` 命令添加了 `posts` 命令，这样每次执行这些命令时都会先生成 JSON 文件。

### 运行解析脚本生成博客 JSON 文件

```bash
pnpm posts
```
运行成功后，将会在 `lib/posts` 目录下生成一批 JSON 文件。


### 从 JSON 文件读取博客数据

```ts
// lib/posts.ts

interface PostMetadata {
    title: string;
    slug: string;
    date: string;
    author: string;
    description: string;
}

interface PostData {
    metadata: PostMetadata;
    content: string;
}

type BlogContent = Record<string, PostData>;

export async function getBlogMetadataList(): Promise<PostMetadata[]> {
    const { default: blogMetadata } = await import('./posts/blog/blog-metadata.json');
    return blogMetadata as PostMetadata[];
}

export async function getBlogData(slug: string): Promise<PostData | undefined> {
    const { default: blogContent } = await import('./posts/blog/blog-content.json');
    return (blogContent as BlogContent)[slug];
}

export async function getAllBlogSlugs(): Promise<string[]> {
    const blogMetadata = await getBlogMetadataList();
    return blogMetadata.map(post => post.slug);
}

export async function getAboutPost(): Promise<PostData> {
    const { default: contentData } = await import(`./posts/about.json`);
    return contentData;
}

export async function getPrivacyPolicyPost(): Promise<PostData> {
    const { default: contentData } = await import(`./posts/privacy-policy.json`);
    return contentData;
}

export async function getTermsOfServicePost(): Promise<PostData> {
    const { default: contentData } = await import(`./posts/terms-of-service.json`);
    return contentData;
}
```

## 创建博客展示页面

### 创建私隐私条款页面

```tsx
// app/privacy/page.tsx

import { metaConfig } from "@/config/site";
import { getPrivacyPolicyPost } from "@/lib/posts";
import { Metadata } from "next";
import { MDXRemote } from "next-mdx-remote/rsc";

export const metadata: Metadata = {
    title: `XXX Policy | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'privacy',
    }
};

export default async function PrivacyPage() {
    const post = await getPrivacyPolicyPost();

    return (
        <div className="flex flex-col">
            <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
                <article>
                    <div className="max-w-none prose dark:prose-invert">
                        <MDXRemote source={post.content} />
                    </div>
                </article>
            </main>
        </div>
    );
}

```

### 创建利用规约页面

```tsx
// app/terms/page.tsx

import { metaConfig } from "@/config/site";
import { getTermsOfServicePost } from "@/lib/posts";
import { Metadata } from "next";
import { MDXRemote } from "next-mdx-remote/rsc";

export const metadata: Metadata = {
    title: `XXX of Service | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'terms',
    }
};

export default async function TermsPage() {
    const post = await getTermsOfServicePost();

    return (
        <div className="flex flex-col">
            <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
                <article>
                    <div className="max-w-none prose dark:prose-invert">
                        <MDXRemote source={post.content} />
                    </div>
                </article>
            </main>
        </div>
    );
}

```

### 创建博客列表页面

```tsx
// app/blog/page.tsx

import Link from 'next/link';
import { getBlogMetadataList } from '@/lib/posts';
import { Calendar, User } from 'lucide-react';
import { Metadata } from 'next';
import { metaConfig } from '@/config/site';

export const metadata: Metadata = {
    title: `XXX Blog | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'blog',
    }
};

export default async function BlogList() {
    const postMetadataList = await getBlogMetadataList();

    return (
        <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                {postMetadataList.map((metadata) => (
                    <Link href={`/blog/${metadata.slug}`} key={metadata.slug}>
                        <div className="bg-white dark:bg-gray-800 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 overflow-hidden">
                            <div className="p-6">
                                <h2 className="text-xl font-semibold mb-2 hover:text-blue-600 transition-colors duration-300">
                                    {metadata.title}
                                </h2>
                                {/* <p className="text-gray-600 dark:text-gray-300 mb-4 line-clamp-3">
                                    {post.metadata.description || "No description available"}
                                </p> */}
                                <div className="flex items-center text-sm text-gray-500 dark:text-gray-400">
                                    <span className="flex items-center mr-4">
                                        <Calendar className="mr-1 h-4 w-4" />
                                        {metadata.date}
                                    </span>
                                    <span className="flex items-center">
                                        <User className="mr-1 h-4 w-4" />
                                        {metadata.author}
                                    </span>
                                </div>
                            </div>
                        </div>
                    </Link>
                ))}
            </div>
        </main>
    );
}

```

### 创建博客文章页面

```tsx
// app/blog/[slug]/page.tsx

import { MDXRemote } from 'next-mdx-remote/rsc';
import { getAllBlogSlugs, getBlogData } from '@/lib/posts';
import { Calendar, User, ArrowLeft } from 'lucide-react';
import { Metadata } from 'next';
import { metaConfig } from '@/config/site';
import Link from 'next/link';

export const dynamicParams = false;

export async function generateMetadata({ params }: Props): Promise<Metadata> {
    const slug = params.slug
    const postData = await getBlogData(slug);
    return {
        title: `${postData?.metadata.title} | ${metaConfig.name}`,
        alternates: {
            canonical: `${metaConfig.base}blog/${slug}`,
        }
    }
}

interface Props {
    params: {
        slug: string;
    };
}

export async function generateStaticParams() {
    const slugs = await getAllBlogSlugs();
    return slugs.map(slug => ({ slug }));
}

export default async function BlogPost({ params }: Props) {
    const post = await getBlogData(params.slug);

    if (!post) {
        return <div>Post not found</div>;
    }

    return (
        <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
            <article>
                <header className="mb-8 text-center">
                    <h1 className="text-4xl font-bold mb-4">{post.metadata.title}</h1>
                    <div className="flex flex-wrap justify-center items-center space-x-4 text-sm text-muted-foreground mb-4">
                        <span className="flex items-center">
                            <Calendar className="mr-1 h-4 w-4" />
                            {post.metadata.date}
                        </span>
                        {post.metadata.author && (
                            <span className="flex items-center">
                                <User className="mr-1 h-4 w-4" />
                                {post.metadata.author}
                            </span>
                        )}
                    </div>
                </header>
                <div className="max-w-none prose dark:prose-invert">
                    <MDXRemote source={post.content} />
                </div>
            </article>
            <Link
                href="/blog"
                className="flex items-center my-8 text-gray-700 dark:text-gray-300 hover:text-gray-900 dark:hover:text-white transition-colors duration-300">
                <ArrowLeft className="mr-2" size={20} />
                <span className="underline">Back</span>
            </Link>
        </main>
    );
}

```

## 验证

访问以下 url 验证这些页面是否正常工作：

- [http://localhost:3000/privacy](http://localhost:3000/privacy)
- [http://localhost:3000/terms](http://localhost:3000/terms)
- [http://localhost:3000/blog/](http://localhost:3000/blog/)
- [http://localhost:3000/blog/xxx](http://localhost:3000/blog/xxx)
