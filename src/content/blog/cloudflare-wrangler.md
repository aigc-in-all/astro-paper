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

登录之后，可以查询个人信息：

```bash
wrangler whoami
```

## Wrangler D1([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#d1))

- create：创建 D1 数据库
  ```bash
  wrangler d1 create <DATABASE_NAME> [OPTIONS]
  ```

- info：查看 D1 数据库信息
  ```bash
  wrangler d1 info <DATABASE_NAME> [OPTIONS]
  ```

- list：列出 D1 数据库
  ```bash
  wrangler d1 list [OPTIONS]
  ```

- delete：删除 D1 数据库
  ```bash
  wrangler d1 delete <DATABASE_NAME> [OPTIONS]
  ```
- execute：执行 SQL 语句
  ```bash
  wrangler d1 execute <DATABASE_NAME> [OPTIONS]
  ```

- export：导出 D1 数据库到 .sql 文件
  ```bash
  wrangler d1 export <DATABASE_NAME> [OPTIONS]
  ```

## Wrangler KV Namespace([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#kv-namespace))

- create：创建 KV 命名空间
  ```bash
  wrangler kv namespace create <NAMESPACE> [OPTIONS]
  ```

- list：列出 KV 命名空间
  ```bash
  wrangler kv namespace list
  ```
  ```bash
  npx wrangler kv namespace list | jq "."
  ```

- delete：删除 KV 命名空间
  ```bash
  wrangler kv namespace delete {--binding=<BINDING>|--namespace-id=<NAMESPACE_ID>} [OPTIONS]
  ```

  ```bash
  npx wrangler kv namespace delete --binding=MY_KV
  ```

## Wrangler KV Key([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#kv-key))

- put: 设置 KV 键值
  ```bash
  wrangler kv key put <KEY> {<VALUE>|--path=<PATH>} {--binding=<BINDING>|--namespace-id=<NAMESPACE_ID>} [OPTIONS]
  ```

  ```bash
  npx wrangler kv key put --binding=MY_KV "my-key" "some-value"
  ```

- get: 获取 KV 键值
  ```bash
  wrangler kv key get <KEY> {--binding=<BINDING>|--namespace-id=<NAMESPACE_ID>} [OPTIONS]
  ```

  ```bash
  npx wrangler kv key get --binding=MY_KV "my-key"
  ```

- list: 列出 KV 键值
  ```bash
  wrangler kv key list {--binding=<BINDING>|--namespace-id=<NAMESPACE_ID>} [OPTIONS]
  ```
  
  ```bash
  npx wrangler kv key list --binding=MY_KV --prefix="public" | jq "."
  ```

- delete: 删除 KV 键值
  ```bash
  wrangler kv key delete <KEY> {--binding=<BINDING>|--namespace-id=<NAMESPACE_ID>} [OPTIONS]
  ```

  ```bash
  npx wrangler kv key delete --binding=MY_KV "my-key"
  ```

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

## Wragler Deploy([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#deploy))

部署 Worker 到 Cloudflare

```bash
wrangler deploy [<SCRIPT>] [OPTIONS]
```
> 该命令的所有选项都不是必需的。 此外，许多选项可以在 `wrangler.toml` 文件中设置。 更多信息请参阅 [wrangler.toml 配置文件](https://developers.cloudflare.com/workers/wrangler/configuration/)。

## Wrangler Pages([官方文档](https://developers.cloudflare.com/workers/wrangler/commands/#pages))

- project list：列出 Pages 项目
  ```bash
  wrangler pages project list
  ```

- deploy：部署 Pages
  ```bash
  wrangler pages deploy <BUILD_OUTPUT_DIRECTORY> [OPTIONS]
  ```
  举例：

  ```json
  {
    ...
    "scripts": {
      "deploy": "next build && wrangler pages deploy out --project-name xxx"
    }
    ...
  }
  ```
