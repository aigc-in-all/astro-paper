---
title: "构建 Next.js、Prisma 和 Vercel Postgres 全栈应用"
pubDatetime: 2024-08-24T14:33:05.162Z
featured: false
draft: false
tags:
  - nextjs
  - prisma
  - postgres
  - vercel
description: 本文详细探讨如何使用 Next.js、Prisma ORM 和 Vercel Postgres 构建全栈应用。从环境设置到部署，每个步骤都有深入解释，包括命令详解和最佳实践。
---

在当今快速发展的 Web 开发领域，选择正确的技术栈和工具对于构建高效、可维护的应用至关重要。本指南将深入探讨如何使用 pnpm 包管理器来构建一个集成了 Next.js、Prisma ORM 和 Vercel Postgres 的全栈应用。我们将详细解释每个步骤，包括命令的作用和最佳实践。

## Table of contents

## 技术栈详解

- **Next.js**: 基于 React 的服务端渲染框架，提供开箱即用的性能优化。
- **Prisma**: 现代 ORM 工具，简化数据库操作并提供类型安全。
- **Vercel Postgres**: Vercel 提供的全托管 PostgreSQL 数据库服务，与 Vercel 部署无缝集成。

## 项目初始化

使用 pnpm 创建新的 Next.js 项目：

```bash
pnpm create next-app my-fullstack-app
```

这个命令会启动一个交互式的设置过程。您将被问及一系列问题，如是否使用 TypeScript、是否使用 ESLint 等。根据您的需求进行选择。

完成后，进入项目目录：

```bash
cd my-fullstack-app
```

## Prisma 设置与配置

安装 Prisma 相关依赖：

```bash
pnpm add -D prisma
pnpm add @prisma/client
```

这里，我们使用 `-D` 标志将 `prisma` 添加为开发依赖，因为它主要用于开发过程中的数据库迁移和客户端生成。而 `@prisma/client` 被添加为运行时依赖，因为它在应用运行时用于数据库操作。

初始化 Prisma：

```bash
pnpm prisma init
```

这个命令会创建一个 `prisma` 目录，其中包含一个 `schema.prisma` 文件，以及一个 `.env` 文件用于环境变量。

编辑 `.env` 文件，添加 Vercel Postgres 连接 URL：

```
DATABASE_URL="postgres://username:password@host:port/database?sslmode=require"
```

确保将 `username`、`password`、`host`、`port` 和 `database` 替换为您的 Vercel Postgres 数据库的实际值。

## 数据模型设计与数据库迁移

在 `prisma/schema.prisma` 中定义数据模型：

```prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  name      String?
  posts     Post[]
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}

model Post {
  id        Int      @id @default(autoincrement())
  title     String
  content   String?
  published Boolean  @default(false)
  author    User     @relation(fields: [authorId], references: [id])
  authorId  Int
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

创建迁移并应用到数据库：

```bash
pnpm prisma migrate dev --name init
```

这个命令做了几件事：
1. 创建一个新的迁移文件，记录数据库结构的变化。
2. 将这些变化应用到您的数据库。
3. 生成 Prisma 客户端，以匹配新的数据库结构。

`--name init` 参数给这个迁移一个描述性的名称。在后续的迁移中，您应该使用能描述此次变更的名称。

## Prisma 客户端使用

创建 `lib/db.ts` 文件来初始化 Prisma 客户端：

```typescript
import { PrismaClient } from '@prisma/client'

// 声明全局接口
declare global {
    var prisma: PrismaClient | undefined
}

// 创建 Prisma 客户端实例
const createPrismaClient = (): PrismaClient => {
    return new PrismaClient({
        log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
    })
}

// 初始化 Prisma 客户端
const prisma: PrismaClient = global.prisma ?? createPrismaClient()

// 在非生产环境下，将 prisma 实例保存到全局对象中以便重用
if (process.env.NODE_ENV !== 'production') {
    global.prisma = prisma
}

export default prisma
```

这个文件创建了一个 Prisma 客户端的单例实例，我们可以在整个应用中重用它。

## Next.js API 路由集成

创建 `pages/api/users.ts` 文件：

```typescript
import type { NextApiRequest, NextApiResponse } from 'next'
import prisma from '../../lib/db'

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    try {
      const users = await prisma.user.findMany()
      res.status(200).json(users)
    } catch (error) {
      res.status(500).json({ error: 'Failed to fetch users' })
    }
  } else if (req.method === 'POST') {
    try {
      const { email, name } = req.body
      const user = await prisma.user.create({
        data: { email, name },
      })
      res.status(201).json(user)
    } catch (error) {
      res.status(500).json({ error: 'Failed to create user' })
    }
  } else {
    res.setHeader('Allow', ['GET', 'POST'])
    res.status(405).end(`Method ${req.method} Not Allowed`)
  }
}
```

这个 API 路由处理 GET 和 POST 请求，分别用于获取所有用户和创建新用户。

## Prisma Studio 与数据管理

Prisma Studio 是一个可视化的数据库管理工具。使用以下命令启动：

```bash
pnpm prisma studio
```

这会在默认浏览器中打开一个界面，允许您浏览和编辑数据库中的数据。这对于开发和调试非常有用。

为了方便使用，直接配置到脚本里：

```
// package.json
{
  "scripts": {
    ...
    "studio": "prisma studio"
  },
  ...
}

```

## 部署流程详解

1. 确保在 Vercel 项目设置中添加 `DATABASE_URL` 环境变量。

2. 在 `package.json` 中添加构建脚本：

```json
"scripts": {
  "vercel-build": "prisma generate && prisma migrate deploy && next build"
}
```

这个脚本做了三件事：
- `prisma generate`: 生成 Prisma 客户端。
- `prisma migrate deploy`: 应用所有待处理的数据库迁移。
- `next build`: 构建 Next.js 应用。

3. 部署时，Vercel 会自动运行 `vercel-build` 脚本。

4. 使用 Vercel CLI 或通过 GitHub 集成部署您的应用。

## 性能优化与最佳实践

1. **连接池优化**:
   在生产环境中，使用连接池可以显著提高性能。修改 `lib/db.ts`:

   ```typescript
   import { PrismaClient } from '@prisma/client'

    declare global {
        var prisma: PrismaClient | undefined
    }

    const createPrismaClient = (): PrismaClient => {
        return new PrismaClient({
            log: process.env.NODE_ENV === 'development' ? ['query', 'error', 'warn'] : ['error'],
        })
    }

    const prisma: PrismaClient = global.prisma ?? createPrismaClient()

    if (process.env.NODE_ENV !== 'production') {
        global.prisma = prisma
    }

    export default prisma
   ```

2. **查询优化**:
   使用 Prisma 的 `select` 和 `include` 只获取必要的数据：

   ```typescript
   const user = await prisma.user.findUnique({
     where: { id: userId },
     select: {
       id: true,
       name: true,
       posts: {
         select: {
           id: true,
           title: true
         }
       }
     }
   })
   ```

3. **批量操作**:
   使用 `createMany`、`updateMany` 等方法进行批量操作：

   ```typescript
   const createdUsers = await prisma.user.createMany({
     data: [
       { name: 'Alice', email: 'alice@example.com' },
       { name: 'Bob', email: 'bob@example.com' },
     ],
   })
   ```

4. **缓存策略**:
   考虑使用 Redis 等缓存解决方案减少数据库查询。可以使用 `ioredis` 库与 Next.js 集成。

5. **索引优化**:
   为经常查询的字段添加适当的索引。在 Prisma schema 中：

   ```prisma
   model User {
     id    Int    @id @default(autoincrement())
     email String @unique
     name  String

     @@index([name])
   }
   ```

6. **类型安全**:
   充分利用 TypeScript 和 Prisma 的类型推断能力，确保代码的类型安全。

## 总结与进阶建议

通过使用 Next.js、Prisma 和 Vercel Postgres，我们构建了一个现代、高效的全栈应用。Prisma 简化了数据库操作，而 Next.js 和 Vercel Postgres 则提供了强大的前后端解决方案。

进阶建议：
1. 考虑实现数据验证，可以使用 Zod 或 Yup 等库。
2. 实施适当的错误处理和日志记录策略。
3. 为您的 API 编写单元测试和集成测试。
4. 考虑使用 Docker 来标准化您的开发和生产环境。
5. 持续监控应用性能，使用 Vercel Analytics 或其他监控工具。

随着您的应用不断发展，记得定期更新依赖，关注性能指标，并持续优化您的代码库。全栈开发是一个不断学习和改进的过程，希望本指南能为您的开发之旅提供有价值的指导。祝您在构建现代 Web 应用的过程中取得成功！