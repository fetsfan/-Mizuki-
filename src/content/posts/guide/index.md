---
title: Vercel 部署完全指南
published: 2025-09-14
description: 一份全面的指南，介绍如何将您的网站项目部署到 Vercel。
image: ./cover.png
author: Fantes
tags: [Vercel]
category: 网站
---

# Vercel 部署完全指南

## 目录
1. [Vercel 概述](#vercel-概述)
2. [准备工作](#准备工作)
3. [部署模式](#部署模式)
4. [通过 Git 集成自动部署](#通过-git-集成自动部署)
5. [通过 Vercel CLI 手动部署](#通过-vercel-cli-手动部署)
6. [环境变量配置](#环境变量配置)
7. [自定义域名](#自定义域名)
8. [最佳实践](#最佳实践)
9. [常见问题](#常见问题)

## Vercel 概述

Vercel 是一个为前端开发者设计的云平台，提供零配置的部署体验。它与 Next.js, React, Vue, Astro 等多种现代框架深度集成，能够实现快速、可靠的全球部署。

### 核心功能
- **Git 集成**：连接 GitHub, GitLab, Bitbucket 仓库，实现自动部署。
- **全球 CDN**：自动将您的站点分发到全球边缘网络，加速访问。
- **Serverless Functions**：轻松部署后端 API。
- **预览部署**：为每个 Git 分支和 Pull Request 创建独立的预览环境。
- **免费额度**：提供慷慨的个人和爱好者免费套餐。

## 准备工作

在开始部署之前，请确保您已经：

1.  **拥有一个 Vercel 账户**：您可以直接使用 GitHub, GitLab 或 Bitbucket 账户注册。
2.  **拥有一个网站项目**：确保您的项目在本地可以正常构建和运行。
3.  **将项目推送到 Git 仓库**：将您的项目代码推送到 GitHub, GitLab 或 Bitbucket。

## 部署模式

Vercel 主要支持两种部署方式：

| 特性 | Git 集成 (推荐) | Vercel CLI |
|------|----------|----------|
| 触发方式 | `git push` | 手动运行命令 |
| 自动化程度 | 高 | 低 |
| 预览部署 | 自动为 PR 创建 | 不支持 |
| 配置复杂度 | 简单，一次性设置 | 每次部署都需要手动操作 |
| 适用场景 | 持续集成和部署 (CI/CD) | 快速测试、一次性部署 |

## 通过 Git 集成自动部署

这是最推荐的方式，可以实现无缝的持续部署。

### 步骤 1：新建项目
1.  登录 Vercel 仪表盘。
2.  点击 "Add New..." -> "Project"。
3.  选择 "Import Git Repository"，然后选择您存放项目的仓库。如果 Vercel 没有权限，需要授权访问。

### 步骤 2：配置项目
Vercel 通常会自动识别出您的项目所使用的框架，并配置好大部分设置。

- **Framework Preset**: Vercel 会尝试自动选择框架预设 (如 Next.js, Astro, Vue.js 等)。如果识别错误，您可以手动选择。
- **Build and Output Settings**:
    - **Build Command**: 根据您的框架和包管理器填写构建命令，如 `npm run build`。
    - **Output Directory**: 填写框架构建后静态文件的输出目录，常见的有 `dist`, `build`, `.output` 等。
    - **Install Command**: `npm install` 或 `pnpm install` 等，根据您的包管理器而定。

确认配置无误后，点击 "Deploy"。Vercel 将开始第一次构建和部署。

### 自动更新
配置完成后，每当您向主分支 `git push` 新的提交时，Vercel 都会自动拉取代码、构建并部署到生产环境。对于 Pull Request，Vercel 会自动创建一个可供预览的部署。

## 通过 Vercel CLI 手动部署

如果您不想关联 Git 仓库，或者只是想进行一次快速部署，可以使用 Vercel CLI。

### 步骤 1：安装 Vercel CLI
在您的终端中运行以下命令：
```bash
npm install -g vercel
```

### 步骤 2：登录
在终端中运行 `vercel login` 并按照提示完成登录。

### 步骤 3：部署
1.  在您的项目根目录下，运行 `vercel` 命令。
2.  CLI 会询问您一系列问题来配置项目：
    - `Set up and deploy “path/to/your/project”? [Y/n]` -> 回答 `Y`
    - `Which scope do you want to deploy to?` -> 选择您的 Vercel 账户
    - `Link to existing project? [y/N]` -> 如果是第一次部署，回答 `N`
    - `What’s your project’s name?` -> 输入项目名称
    - `In which directory is your code located?` -> 保持默认 (`./`)
3.  Vercel CLI 会自动检测项目类型并进行部署。

### 生产部署
要直接部署到生产环境，请使用 `vercel --prod` 命令。

## 环境变量配置

在项目开发中，我们经常需要使用环境变量来存储敏感信息（如 API 密钥）或配置信息。

### 在 Vercel 仪表盘中配置
1.  进入您的项目仪表盘。
2.  点击 "Settings" -> "Environment Variables"。
3.  添加您的环境变量，可以选择将其暴露给生产、预览或开发环境。

### 在 `vercel.json` 中配置
您也可以在项目根目录创建一个 `vercel.json` 文件来管理配置，但不建议在此文件中存储敏感信息。
```json
{
  "env": {
    "MY_VARIABLE": "some_value"
  }
}
```

## 自定义域名

Vercel 允许您轻松地为项目绑定自定义域名。

1.  进入您的项目仪表盘。
2.  点击 "Settings" -> "Domains"。
3.  输入您想要添加的域名，并按照 Vercel 提供的指引在您的域名注册商处修改 DNS 记录（通常是添加 CNAME 或 A 记录）。
4.  Vercel 会自动为您的域名配置 SSL 证书。

## 最佳实践

### 1. 使用框架特定的适配器
为了更好地利用 Vercel 的 Serverless Functions 等功能（如果您的项目使用了 SSR），建议安装并配置框架官方提供的 Vercel 适配器，例如 `@astrojs/vercel` 或 Next.js 内置的集成。

### 2. 利用预览部署
在合并代码到主分支之前，充分利用 Pull Request 生成的预览部署链接进行测试，确保所有功能正常。

### 3. 缓存配置
对于不需要频繁更新的静态资源，可以在 `vercel.json` 中配置缓存策略，以提高性能和减少带宽消耗。
```json
{
  "headers": [
    {
      "source": "/(.*).(jpg|jpeg|png|gif|webp|svg)$",
      "headers": [
        {
          "key": "Cache-Control",
          "value": "public, max-age=31536000, immutable"
        }
      ]
    }
  ]
}
```

### 4. 监控和日志
定期检查 Vercel 仪表盘中的 "Logs" 和 "Analytics" 选项，监控应用健康状况和用户访问数据。

## 常见问题

### Q1: 部署失败，提示找不到输出目录？
**检查清单：**
- [ ] 确认您的框架构建配置中指定的输出目录 (`outDir`, `buildDir` 等)是什么。
- [ ] 在 Vercel 的项目配置中，"Output Directory" 是否正确设置为该目录。
- [ ] 检查 `build` 命令是否成功执行。查看部署日志中的构建步骤是否有错误。

### Q2: 环境变量在客户端代码中无法访问？
**可能原因：**
- 许多前端框架（如 React, Vue, Astro）为了安全，默认只将特定前缀（如 `VITE_`，`PUBLIC_`）的环境变量暴露给客户端。请查阅您所使用框架的文档。
- 检查您在 Vercel 仪表盘中配置的环境变量是否应用到了正确的环境（生产/预览/开发）。

### Q3: 如何回滚到之前的部署？
**步骤：**
1.  进入项目仪表盘的 "Deployments" 标签页。
2.  找到您想要回滚到的那个部署版本。
3.  点击该部署右侧的 "..." 菜单，选择 "Promote to Production"。

### Q4: 自定义域名配置后无法访问？
**可能原因：**
- DNS 记录未生效，通常需要几分钟到几小时不等。
- DNS 配置错误，请仔细核对 Vercel 提供的记录值。
- SSL 证书生成失败，可以尝试在 Vercel 仪表盘中重新生成。

## 总结

Vercel 为现代网站项目提供了极其流畅和强大的部署体验。通过利用其 Git 集成、全球 CDN 和预览部署等功能，您可以轻松构建一个现代、高效的 Web 应用工作流。

记住关键要点：
- 优先使用 Git 集成进行自动化部署。
- 根据项目需求（SSR/SSG）选择是否使用框架特定的 Vercel 适配器。
- 善用环境变量管理敏感数据。
- 充分利用预览部署进行测试。

如有其他问题，请参考 Vercel 和您所使用框架的官方文档。