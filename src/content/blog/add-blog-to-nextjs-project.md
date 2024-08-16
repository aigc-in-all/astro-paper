---
title: "给 Next.js 项目添加博客"
pubDatetime: 2024-08-11T14:33:05.162Z
featured: false
draft: false
tags:
  - nextjs
description: 快速给 Next.js 项目添加博客功能，包括隐私条款、利用规约等
---

使用 Next.js 创建完网站后，可以使用如下方法快速添加博客功能，博客使用 Markdown 文件存储。

也可以使用这套方法快速添加其他内容，如隐私条款、利用规约等。

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

posts/about-us.md
```md
# About Us

welcome
```
posts/privacy-policy.md
```md
# Privacy Policy

Last updated: 2024-03-15

```
posts/terms-of-service.md
```md
# Terms of Service

Last updated: 2024-03-15

```
posts/blog/welcome.md
```md
---
title: "welcome"
slug: "welcome"
date: 2024-05-15
author: "Alex Bennet"
---

welcome
```

## 创建 Markdown 读取配置

```ts
// lib/posts.ts

import fs from 'fs';
import path from 'path';
import matter from 'gray-matter';

interface PostData {
    metadata: PostMetadata;
    content: string;
}

interface PostMetadata {
    title: string;
    slug: string;
    date: Date;
    dateStr: string;
    author?: string;
    tags?: string[];
}

const blogDirName = "posts/blog";

export function getBlogList(): PostData[] {
    const blogsDirectory = path.join(process.cwd(), blogDirName);
    const fileNames = fs.readdirSync(blogsDirectory);
    const posts = fileNames.map((fileName) => {
        const fullPath = path.join(process.cwd(), blogDirName, fileName);
        const fileContents = fs.readFileSync(fullPath, 'utf8');
        const { data, content } = matter(fileContents);
        return {
            metadata: {
                title: data.title,
                slug: data.slug,
                date: data.date,
                dateStr: data.date.toISOString().split('T')[0],
                author: data.author,
                tags: data.tags,
            },
            content: content,
        }
    });
    posts.sort((a, b) => (a.metadata.date > b.metadata.date ? -1 : 1));
    return posts;
}

export function getBlogData(slug: string): PostData {
    const fullPath = path.join(process.cwd(), blogDirName, `${slug}.md`);
    const fileContents = fs.readFileSync(fullPath, 'utf8');
    const { data, content } = matter(fileContents);

    return {
        metadata: {
            title: data.title,
            slug: data.slug,
            date: data.date,
            dateStr: data.date.toISOString().split('T')[0],
            author: data.author,
            tags: data.tags,
        },
        content: content,
    };
}

export function getAllBlogSlugs(): string[] {
    const blogsDirectory = path.join(process.cwd(), blogDirName);
    const fileNames = fs.readdirSync(blogsDirectory);
    return fileNames.map(fileName => {
        const fullPath = path.join(blogsDirectory, fileName);
        const fileContents = fs.readFileSync(fullPath, 'utf8');
        const { data } = matter(fileContents);
        return data.slug;
    });
}

export function getPrivacyPolicyContent(): string {
    const filePath = path.join(process.cwd(), 'posts/privacy-policy.md');
    return fs.readFileSync(filePath, 'utf8');
}

export function getTermsOfServiceContent(): string {
    const filePath = path.join(process.cwd(), 'posts/terms-of-service.md');
    return fs.readFileSync(filePath, 'utf8');
}

export function getAboutContent(): string {
    const filePath = path.join(process.cwd(), 'posts/about-us.md');
    return fs.readFileSync(filePath, 'utf8');
}
```

## 创建博客展示页面

### 创建私隐私条款页面

```tsx
// app/privacy/page.tsx

import { metaConfig } from "@/config/site";
import { getPrivacyPolicyContent } from "@/lib/posts";
import { Metadata } from "next";
import { MDXRemote } from "next-mdx-remote/rsc";

export const metadata: Metadata = {
    title: `Brown Noise Privacy Policy | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'privacy',
    }
};

export default function PrivacyPage() {
    const content = getPrivacyPolicyContent();

    return (
        <div className="flex flex-col">
            <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
                <article>
                    <div className="max-w-none prose dark:prose-invert">
                        <MDXRemote source={content} />
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
import { getTermsOfServiceContent } from "@/lib/posts";
import { Metadata } from "next";
import { MDXRemote } from "next-mdx-remote/rsc";

export const metadata: Metadata = {
    title: `Brown Noise Terms of Service | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'terms',
    }
};

export default function TermsPage() {
    const content = getTermsOfServiceContent();

    return (
        <div className="flex flex-col">
            <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
                <article>
                    <div className="max-w-none prose dark:prose-invert">
                        <MDXRemote source={content} />
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
import { getBlogList } from '@/lib/posts';
import { Calendar, User } from 'lucide-react';
import { Metadata } from 'next';
import { metaConfig } from '@/config/site';

export const metadata: Metadata = {
    title: `Brown Noise Blog | ${metaConfig.name}`,
    description: metaConfig.description,
    alternates: {
        canonical: metaConfig.base + 'blog',
    }
};

export default function BlogList() {
    const posts = getBlogList();

    return (
        <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
            <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                {posts.map((post) => (
                    <Link href={`/blog/${post.metadata.slug}`} key={post.metadata.slug}>
                        <div className="bg-white dark:bg-gray-800 rounded-lg shadow-md hover:shadow-xl transition-shadow duration-300 overflow-hidden">
                            <div className="p-6">
                                <h2 className="text-xl font-semibold mb-2 hover:text-blue-600 transition-colors duration-300">
                                    {post.metadata.title}
                                </h2>
                                {/* <p className="text-gray-600 dark:text-gray-300 mb-4 line-clamp-3">
                                    {post.metadata.description || "No description available"}
                                </p> */}
                                <div className="flex items-center text-sm text-gray-500 dark:text-gray-400">
                                    <span className="flex items-center mr-4">
                                        <Calendar className="mr-1 h-4 w-4" />
                                        {post.metadata.dateStr}
                                    </span>
                                    {post.metadata.author && (
                                        <span className="flex items-center">
                                            <User className="mr-1 h-4 w-4" />
                                            {post.metadata.author}
                                        </span>
                                    )}
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
import { Calendar, User, Tag, ArrowLeft } from 'lucide-react';
import { Metadata } from 'next';
import { metaConfig } from '@/config/site';
import Link from 'next/link';

export async function generateMetadata({ params }: Props): Promise<Metadata> {
    const slug = params.slug
    const postData = getBlogData(slug);
    return {
        title: `${postData.metadata.title} | ${metaConfig.name}`,
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

export function generateStaticParams() {
    const slugs = getAllBlogSlugs();
    return slugs.map(slug => ({ slug }));
}

export default function BlogPost({ params }: Props) {
    const postData = getBlogData(params.slug);

    return (
        <main className="w-full max-w-5xl mx-auto p-4 md:p-6 flex-grow">
            <article>
                <header className="mb-8 text-center">
                    <h1 className="text-4xl font-bold mb-4">{postData.metadata.title}</h1>
                    <div className="flex flex-wrap justify-center items-center space-x-4 text-sm text-muted-foreground mb-4">
                        <span className="flex items-center">
                            <Calendar className="mr-1 h-4 w-4" />
                            {postData.metadata.dateStr}
                        </span>
                        {postData.metadata.author && (
                            <span className="flex items-center">
                                <User className="mr-1 h-4 w-4" />
                                {postData.metadata.author}
                            </span>
                        )}
                    </div>
                    {postData.metadata.tags && postData.metadata.tags.length > 0 && (
                        <div className="flex justify-center items-center space-x-2">
                            <Tag className="h-4 w-4 text-muted-foreground" />
                            {postData.metadata.tags.map((tag) => (
                                <span key={tag}>{tag}</span>
                            ))}
                        </div>
                    )}
                </header>
                <div className="max-w-none prose dark:prose-invert">
                    <MDXRemote source={postData.content} />
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
