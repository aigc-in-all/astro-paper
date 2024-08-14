---
title: "Next.js 全栈技术指南"
pubDatetime: 2024-08-13T12:21:47.769Z
featured: false
draft: false
tags:
  - nodejs
description: 介绍一些 Next.js 开发全栈应用时常用的技术栈
---

## Table of contents

## 部署

- [Vercel](https://vercel.com/)
- [Cloudflare Pages](https://pages.cloudflare.com/)

## 数据库

- [Vercel KV](https://vercel.com/docs/storage/vercel-kv)
- [Vercel Blob](https://vercel.com/docs/storage/vercel-blob)
- [Vercel Postgres](https://vercel.com/docs/storage/vercel-postgres)
- [PlanetScale](https://planetscale.com/)

## ORM

- [Prisma](https://www.prisma.io/)
- [Drizzle ORM](https://drizzle.dev/)

## 登录认证 & 用户管理

- [clerk](https://clerk.com/)

## UI框架
| 框架名称                                                                                          | GitHub Stars | 简介                                                        |
| --------------------------------------------------------------------------------------------- | ------------ | --------------------------------------------------------- |
| [shadcn/ui](https://ui.shadcn.com/) ([GitHub](https://github.com/shadcn/ui))                  | 66.3k        | 可定制的 React 组件集合,可与 Tailwind CSS 完美配合。不是传统的组件库,而是可复制粘贴的组件。 |
| [Chakra UI](https://chakra-ui.com/) ([GitHub](https://github.com/chakra-ui/chakra-ui))        | 37.3k        | 简单、模块化且可访问的 React 组件库,虽不是专为 Next.js 设计,但可与其完美集成。          |
| [daisyUI](https://daisyui.com/) ([GitHub](https://github.com/saadeghi/daisyui))               | 32.7k        | 纯 CSS 组件库,基于 Tailwind CSS。可以很好地与 Next.js 项目集成。            |
| [Mantine](https://mantine.dev/) ([GitHub](https://github.com/mantinedev/mantine))             | 25.7k        | 功能丰富的 React 组件库和钩子集合,支持 Next.js 服务器端渲染。                   |
| [Headless UI](https://headlessui.com/) ([GitHub](https://github.com/tailwindlabs/headlessui)) | 25.4k        | 完全无样式、完全可访问的 UI 组件,可与 Tailwind CSS 完美配合。                  |
| [NextUI](https://nextui.org/) ([GitHub](https://github.com/nextui-org/nextui))                | 21.1k        | 基于 Tailwind CSS 构建的现代和美观的 React UI 库,专为 Next.js 应用程序优化。   |
| [Tremor](https://www.tremor.so/) ([GitHub](https://github.com/tremorlabs/tremor))             | 16k          | 专为 React 和 Tailwind CSS 构建的仪表板组件库,适用于数据密集型应用。             |
| [Radix UI](https://www.radix-ui.com/) ([GitHub](https://github.com/radix-ui/primitives))      | 15.2k        | 可访问且不附带样式的组件库,用于构建高质量的设计系统和 web 应用程序。                     |
| [Magic UI](https://magicui.design/) ([GitHub](https://github.com/magicuidesign/magicui))      | 6.9k         | 基于 Tailwind CSS 的现代 UI 组件库,适用于 Next.js 和 React。           |
| [Preline UI](https://preline.co/) ([GitHub](https://github.com/htmlstreamofficial/preline))   | 4.4k         | 基于 Tailwind CSS 的开源 UI 组件库和模板。                            |
| [Float UI](https://www.floatui.com/) ([GitHub](https://github.com/MarsX-dev/floatui))         | 3.3k         | 提供现成的 Tailwind CSS 组件和模板。                                 |
| [Sailboat UI](https://sailboatui.com/) ([GitHub](https://github.com/sailboatui/sailboatui))   | 1.2k         | 基于 Tailwind CSS 的组件库,专注于美观和易用性。                           |
> 以上数据统计自 2024 年 8 月 14 日

## 图标库

| 图标库名称                                                                                                              | GitHub Stars | 简介                                                 |
| ------------------------------------------------------------------------------------------------------------------ | ------------ | -------------------------------------------------- |
| [Font Awesome](https://fontawesome.com/) ([GitHub](https://github.com/FortAwesome/Font-Awesome))                   | 73.4k        | 网络上最受欢迎的图标集和工具包,提供可缩放的矢量图标,可以通过CSS进行自定义。           |
| [Material Design Icons](https://materialdesignicons.com/) ([GitHub](https://github.com/Templarian/MaterialDesign)) | 11k          | 基于Google Material Design的大型开源图标集合,包括社区贡献的图标。       |
| [Heroicons](https://heroicons.com/) ([GitHub](https://github.com/tailwindlabs/heroicons))                          | 21.2k        | 由Tailwind CSS的创建者手工制作的漂亮的SVG图标,可用于UI开发。            |
| [Feather](https://feathericons.com/) ([GitHub](https://github.com/feathericons/feather))                           | 24.7k        | 简单优雅的开源图标集,每个图标都设计在24x24网格上,强调简约和一致性。              |
| [Tabler Icons](https://tabler-icons.io/) ([GitHub](https://github.com/tabler/tabler-icons))                        | 17.8k        | 超过4200个免费和开源的SVG图标,设计简洁一致,适用于网页项目。                 |
| [Ionicons](https://ionic.io/ionicons) ([GitHub](https://github.com/ionic-team/ionicons))                           | 17.5k        | Ionic Framework的高级图标包,为web、iOS、Android和桌面应用程序精心制作。 |
| [Lucide](https://lucide.dev/) ([GitHub](https://github.com/lucide-icons/lucide))                                   | 10k          | Lucide是Feather Icons的社区分支,提供了更多图标和定期更新。            |
| [Remix Icon](https://remixicon.com/) ([GitHub](https://github.com/Remix-Design/RemixIcon))                         | 6.6k         | 中性风格的系统符号图标库,设计简洁现代,适用于各种项目。                       |
| [Boxicons](https://boxicons.com/) ([GitHub](https://github.com/atisawd/boxicons))                                  | 2.9k         | 高质量的网络友好图标,可用于网页设计和应用程序。                           |
| [Octicons](https://primer.style/octicons/) ([GitHub](https://github.com/primer/octicons))                          | 8.2k         | GitHub的官方图标库,用于GitHub的网站和应用程序界面。                   |
| [Eva Icons](https://akveo.github.io/eva-icons/) ([GitHub](https://github.com/akveo/eva-icons))                     | 8.6k         | 一包超过480个精美设计的开源图标,用于常见操作和项目。                       |
| [SVG Repo](https://www.svgrepo.com/)                                                                               | N/A          | 大型的SVG图标和矢量图形集合库,提供免费下载和使用。不是开源项目,没有GitHub仓库。      |
| [Flaticon](https://www.flaticon.com/)                                                                              | N/A          | 提供大量免费和付费的矢量图标和贴纸。不是开源项目,没有GitHub仓库。               |
> 以上数据统计自 2024 年 8 月 14 日

## Logo 设计

- [brandmark](https://app.brandmark.io/v3/)
