---
title: "使用 Uppy 快速给网站添加文件上传功能"
pubDatetime: 2024-08-15T10:28:39.882Z
featured: false
draft: false
tags:
  - nextjs
  - uppy
description: 详细讲解如何使用 Uppy 快速给网站添加文件上传功能
---

在现代 Web 应用中,文件上传功能已经成为一个常见且必要的特性。本文将介绍如何使用 TypeScript 和 Uppy - 一个强大而灵活的文件上传库,快速为你的网站添加类型安全的文件上传功能。

## Table of contents

## 什么是 Uppy?

Uppy 是一个模块化的 JavaScript 文件上传器,专为现代浏览器设计。它提供了丰富的功能,包括拖放上传、进度条、图像预览等,同时保持了极高的可定制性。Uppy 支持从本地磁盘、远程 URLs、Google Drive、Dropbox 等多种来源上传文件。

官网：https://uppy.io/

## 快速开始

让我们通过以下步骤,快速在网站中集成 Uppy:

### 1.安装Uppy

首先，使用 npm 或 pnpm 或 yarn 安装 Uppy:

```bash
# npm
npm install @uppy/core @uppy/dashboard @uppy/xhr-upload @uppy/tus @uppy/compressor

# pnpm
pnpm add @uppy/core @uppy/dashboard @uppy/xhr-upload @uppy/tus @uppy/compressor

# yarn
yarn add @uppy/core @uppy/dashboard @uppy/xhr-upload @uppy/tus @uppy/compressor
```

### 2.引入 Uppy

在你的 JavaScript 文件中引入 Uppy:

```typescript
import Uppy, { UploadResult } from '@uppy/core'
import Dashboard from '@uppy/dashboard'
import XHRUpload from '@uppy/xhr-upload'

import '@uppy/core/dist/style.css'
import '@uppy/dashboard/dist/style.css'
```

### 3.初始化 Uppy

创建 Uppy 实例并配置上传目标:

```typescript
const uppy = new Uppy()
  .use(Dashboard, {
    inline: true,
    target: '#drag-drop-area'
  })
  .use(XHRUpload, {
    endpoint: 'https://your-server.com/upload'
  })
```
也可以使用 Tus 插件来支持断点续传:

```typescript
const uppy = new Uppy()
  .use(Dashboard, {
    inline: true,
    target: '#drag-drop-area'
  })
  .use(Tus, {
      endpoint: "https://tusd.tusdemo.net/files/",
      retryDelays: [0, 1000, 3000, 5000],
      chunkSize: 5 * 1024 * 1024,
    })
```

### 4.添加HTML容器

在 HTML 中添加一个容器来放置 Uppy 的界面:

```html
<div id="drag-drop-area"></div>
```

### 5.处理上传事件

可以监听上传事件来执行自定义操作:

```typescript
uppy.on('upload-success', (file, response: UploadResult) => {
  console.log(file.name, response.body)
})
```

## 3.Demo 代码

```typescript
// components/ImageUploader.tsx

'use client'

import React, { useCallback, useEffect, useState } from 'react';
import Uppy from '@uppy/core';
import Dashboard from '@uppy/dashboard';
import Compressor from '@uppy/compressor';
import Tus from "@uppy/tus";
import '@uppy/core/dist/style.css';
import '@uppy/dashboard/dist/style.css';

interface FileUploaderProps {
    onUploadSuccess?: (uploadedUrl: string, response: any) => void;
    onUploadError?: (error: Error, response: any) => void;
}

const ImageUploader: React.FC<FileUploaderProps> = ({ onUploadSuccess, onUploadError }) => {

    const [uploadedFileUrl, setUploadedFileUrl] = useState<string | null>(null);

    const handleUploadSuccess = useCallback((file: any, response: any) => {
        console.log('上传成功', file, response);
        const uploadedUrl = response.uploadURL;
        if (typeof uploadedUrl === 'string') {
            setUploadedFileUrl(uploadedUrl);
            onUploadSuccess?.(uploadedUrl, response);
        } else {
            console.error('上传URL无效');
            onUploadError?.(new Error('上传URL无效'), response);
        }
    }, [onUploadSuccess, onUploadError]);

    const handleUploadError = useCallback((file: any, error: Error, response: any) => {
        console.error('上传失败', file, error, response);
        onUploadError?.(error, response);
    }, [onUploadError]);

    useEffect(() => {
        const uppy = new Uppy({
            meta: { type: 'avatar' },
            restrictions: {
                maxFileSize: 10 * 1024 * 1024,
                maxNumberOfFiles: 5,
                minNumberOfFiles: 1,
                allowedFileTypes: ['image/*'],
            },
            autoProceed: false,
            allowMultipleUploadBatches: true,
        });

        uppy.use(Dashboard, {
            inline: true,
            target: '#uploader-container',
            showProgressDetails: true,
            hideUploadButton: false,
            showSelectedFiles: true,
            singleFileFullScreen: false,
            note: "Supports JPEG and PNG images only, up to 1 MB.",
            proudlyDisplayPoweredByUppy: false,
        });

        uppy.use(Tus, {
            endpoint: "https://tusd.tusdemo.net/files/",
            retryDelays: [0, 1000, 3000, 5000],
            chunkSize: 5 * 1024 * 1024,
        })

        uppy.use(Compressor, {
            quality: 0.6,
            limit: 10,
        })

        uppy.on('upload-success', handleUploadSuccess)
        uppy.on('upload-error', handleUploadError);

        return () => uppy.destroy();
    }, [handleUploadSuccess, handleUploadError]);

    return (
        <div>
            <div id="uploader-container" />
            {uploadedFileUrl && (
                <div>
                    <h3>上传成功！</h3>
                    <p>文件URL: {uploadedFileUrl}</p>
                </div>
            )}
        </div>
    )
}

export default ImageUploader;
```
效果

![uppy demo](@assets/images/uppy-demo.jpg)
