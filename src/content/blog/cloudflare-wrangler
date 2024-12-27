---
title: "Cloudflare Wrangler 使用指南"
pubDatetime: 2024-12-27T08:13:39.554Z
featured: false
draft: false
tags:
  - cloudflare
  - wrangler
description: 详细记录 Cloudflare Wrangler 使用指南
---

本文主要记录 Cloudflare Wrangler 使用指南，官方文档：https://developers.cloudflare.com/workers/wrangler/commands/#docs

## Table of contents

## 安装 Wrangler

Cloudflare 建议在项目中本地安装 Wrangler（而不是全局安装），因此运行 Wrangler 的方式将取决于您的具体设置和包管理器。

```bash
pnpm wrangler <COMMAND> <SUBCOMMAND> [PARAMETERS] [OPTIONS]
```

您可以将常用的 Wrangler 命令作为脚本添加到项目的 package.json 文件中：

```json
{
  ...
  "scripts": {
    "deploy": "wrangler deploy",
    "dev": "wrangler dev"
  }
  ...
}
```

但是，我们仍然可以全局安装 Wrangler，这样操作 Cloudflare 的资源会更加方便。

```bash
npm install -g wrangler@latest
```

> 如果下面有些命令不支持，可以先卸载 `npm uninstall -g wrangler` 再安装最新版本

安装完后需要登录：

```bash
wrangler login [OPTIONS]
```

对应也可以登出：

```bash
wrangler logout
```

## Wrangler D1

todo

## Wrangler KV

todo

## Wrangler R2 Bucket([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#r2-bucket))

- list：列出存储桶
  ```bash
  wrangler r2 bucket list
  ```

- create：创建 R2 存储桶
  ```bash
  wrangler r2 bucket create <BUCKET_NAME>
  ```

- info：查看存储桶信息
  ```bash
  wrangler r2 bucket info <BUCKET_NAME>
  ```

- delete：删除存储桶
  ```bash
  wrangler r2 bucket delete <BUCKET_NAME>
  ```

## Wrangler R2 Object([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#r2-object))

- get: 获取对象
  ```bash
  wrangler r2 object get <OBJECT_PATH> [OPTIONS]
  ```

- put: 上传对象
  ```bash
  wrangler r2 object put <OBJECT_PATH> [OPTIONS]
  ```

- delete: 删除对象
  ```bash
  wrangler r2 object delete <OBJECT_PATH> [OPTIONS]
  ```

- list: 列出对象
  ```bash
  wrangler r2 object delete <OBJECT_PATH> [OPTIONS]
  ```

