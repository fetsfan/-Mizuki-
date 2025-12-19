---
title: Netlify 部署完全指南
published: 2025-09-14
description: 一份全面的指南，介绍如何将您的网站项目部署到 Netlify。
author: Fantes
tags: [Netlify]
category: 网站
---

# Netlify 部署完全指南

## 目录
1. [Netlify 概述](#netlify-概述)
2. [准备工作](#准备工作)
3. [部署模式](#部署模式)
4. [通过 Git 集成自动部署](#通过-git-集成自动部署)
5. [通过拖拽部署](#通过拖拽部署)
6. [通过 Netlify CLI 部署](#通过-netlify-cli-部署)
7. [环境变量配置](#环境变量配置)
8. [自定义域名](#自定义域名)
9. [表单处理](#表单处理)
10. [最佳实践](#最佳实践)
11. [常见问题](#常见问题)

## Netlify 概述

Netlify 是一个专为现代 Web 开发而设计的云平台，提供静态网站托管、无服务器函数、表单处理等功能。它以简单易用、功能强大著称，特别适合 JAMstack 架构的项目。

### 核心功能
- **Git 集成**：连接 GitHub, GitLab, Bitbucket 仓库，实现自动部署。
- **全球 CDN**：自动将您的站点分发到全球边缘网络，提供快速访问。
- **无服务器函数**：支持 Netlify Functions，轻松部署后端逻辑。
- **表单处理**：内置表单处理功能，无需后端即可收集用户数据。
- **分支部署**：为每个 Git 分支创建独立的部署预览。
- **免费额度**：提供慷慨的免费套餐，适合个人和小型项目。

## 准备工作

在开始部署之前，请确保您已经：

1. **拥有一个 Netlify 账户**：您可以直接使用 GitHub, GitLab 或 Bitbucket 账户注册。
2. **拥有一个网站项目**：确保您的项目在本地可以正常构建和运行。
3. **将项目推送到 Git 仓库**（可选）：如果使用 Git 集成部署，需要将代码推送到远程仓库。

## 部署模式

Netlify 支持多种部署方式：

| 特性 | Git 集成 (推荐) | 拖拽部署 | Netlify CLI |
|------|----------|----------|----------|
| 触发方式 | `git push` | 手动拖拽文件夹 | 手动运行命令 |
| 自动化程度 | 高 | 低 | 中等 |
| 分支部署 | 自动为分支创建预览 | 不支持 | 支持 |
| 配置复杂度 | 简单，一次性设置 | 最简单 | 中等 |
| 适用场景 | 持续集成和部署 (CI/CD) | 快速原型、静态文件 | 开发测试、自动化脚本 |

## 通过 Git 集成自动部署

这是最推荐的方式，可以实现完全自动化的持续部署。

### 步骤 1：新建站点
1. 登录 Netlify 控制台。
2. 点击 "New site from Git"。
3. 选择您的 Git 提供商（GitHub, GitLab, Bitbucket）并授权 Netlify 访问。
4. 选择要部署的仓库。

### 步骤 2：配置构建设置
Netlify 会尝试自动检测您的项目类型并配置构建设置：

- **Branch to deploy**: 选择要部署的分支（通常是 `main` 或 `master`）。
- **Build command**: 构建命令，如 `npm run build`、`yarn build` 等。
- **Publish directory**: 构建输出目录，如 `dist`、`build`、`public` 等。

### 常见框架的配置示例：

**React (Create React App)**
```
Build command: npm run build
Publish directory: build
```

**Vue.js**
```
Build command: npm run build
Publish directory: dist
```

**Astro**
```
Build command: npm run build
Publish directory: dist
```

**Next.js (静态导出)**
```
Build command: npm run build && npm run export
Publish directory: out
```

### 步骤 3：部署
确认配置无误后，点击 "Deploy site"。Netlify 将开始第一次构建和部署。

### 自动更新
配置完成后，每当您向指定分支推送新的提交时，Netlify 都会自动触发重新构建和部署。

## 通过拖拽部署

这是最简单的部署方式，适合静态文件或已经构建好的项目。

### 步骤：
1. 在本地构建您的项目（如运行 `npm run build`）。
2. 登录 Netlify 控制台。
3. 将构建输出文件夹直接拖拽到 Netlify 的部署区域。
4. Netlify 会自动上传并部署您的站点。

### 注意事项：
- 这种方式不支持自动更新，每次更改都需要手动重新拖拽。
- 适合快速原型或一次性部署。

## 通过 Netlify CLI 部署

Netlify CLI 提供了命令行部署功能，适合自动化脚本和开发测试。

### 步骤 1：安装 Netlify CLI
```bash
npm install -g netlify-cli
```

### 步骤 2：登录
```bash
netlify login
```

### 步骤 3：初始化项目
在项目根目录运行：
```bash
netlify init
```

### 步骤 4：部署
```bash
# 部署到预览环境
netlify deploy

# 部署到生产环境
netlify deploy --prod
```

## 环境变量配置

Netlify 支持多种方式配置环境变量。

### 在 Netlify 控制台中配置
1. 进入您的站点设置。
2. 点击 "Environment variables"。
3. 添加您的环境变量键值对。

### 在 `netlify.toml` 中配置
在项目根目录创建 `netlify.toml` 文件：
```toml
[build.environment]
  NODE_VERSION = "18"
  NPM_VERSION = "8"

[context.production.environment]
  API_URL = "https://api.example.com"

[context.deploy-preview.environment]
  API_URL = "https://staging-api.example.com"
```

## 自定义域名

Netlify 提供了简单的域名配置功能。

### 步骤：
1. 进入站点设置的 "Domain management"。
2. 点击 "Add custom domain"。
3. 输入您的域名。
4. 根据提示配置 DNS 记录：
   - **子域名**：添加 CNAME 记录指向您的 Netlify 站点 URL。
   - **根域名**：添加 A 记录指向 Netlify 的 IP 地址。
5. Netlify 会自动为您的域名配置 SSL 证书。

## 表单处理

Netlify 提供了内置的表单处理功能，无需后端即可收集用户数据。

### 基本用法：
```html
<form name="contact" method="POST" data-netlify="true">
  <p>
    <label>姓名: <input type="text" name="name" /></label>
  </p>
  <p>
    <label>邮箱: <input type="email" name="email" /></label>
  </p>
  <p>
    <label>消息: <textarea name="message"></textarea></label>
  </p>
  <p>
    <button type="submit">发送</button>
  </p>
</form>
```

### 添加验证码：
```html
<form name="contact" method="POST" data-netlify="true" data-netlify-recaptcha="true">
  <!-- 表单字段 -->
  <div data-netlify-recaptcha="true"></div>
  <button type="submit">发送</button>
</form>
```

## 最佳实践

### 1. 使用 `netlify.toml` 配置文件
在项目根目录创建 `netlify.toml` 来管理构建和部署配置：
```toml
[build]
  publish = "dist"
  command = "npm run build"

[build.environment]
  NODE_VERSION = "18"

[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### 2. 配置重定向规则
对于单页应用（SPA），配置重定向规则以支持客户端路由：
```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### 3. 优化构建性能
- 使用缓存：Netlify 会自动缓存 `node_modules`。
- 设置合适的 Node.js 版本。
- 使用 `netlify-plugin-cache` 插件缓存其他依赖。

### 4. 利用分支部署
为不同的分支配置不同的部署环境：
```toml
[context.production]
  command = "npm run build:prod"

[context.deploy-preview]
  command = "npm run build:preview"

[context.branch-deploy]
  command = "npm run build:dev"
```

### 5. 监控和分析
- 启用 Netlify Analytics 监控站点性能。
- 使用 Netlify 的构建日志调试部署问题。
- 设置部署通知，及时了解部署状态。

## 常见问题

### Q1: 构建失败，提示找不到命令？
**解决方案：**
- 检查 `package.json` 中是否定义了构建脚本。
- 确认构建命令在 Netlify 配置中正确设置。
- 检查 Node.js 版本是否兼容，可在 `netlify.toml` 中指定版本。

### Q2: 单页应用刷新页面出现 404 错误？
**解决方案：**
添加重定向规则到 `netlify.toml`：
```toml
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200
```

### Q3: 环境变量在客户端无法访问？
**解决方案：**
- 确保环境变量名有正确的前缀（如 React 需要 `REACT_APP_`）。
- 检查变量是否在正确的部署上下文中配置。
- 重新部署以应用新的环境变量。

### Q4: 表单提交后没有收到数据？
**解决方案：**
- 确保表单包含 `data-netlify="true"` 属性。
- 检查表单的 `name` 属性是否设置。
- 在 Netlify 控制台的 "Forms" 部分查看提交的数据。

### Q5: 自定义域名配置后无法访问？
**解决方案：**
- 检查 DNS 记录是否正确配置。
- 等待 DNS 传播完成（通常需要几分钟到几小时）。
- 确保域名没有被其他服务占用。

## 总结

Netlify 为现代网站项目提供了简单而强大的部署解决方案。通过其 Git 集成、表单处理、无服务器函数等功能，您可以快速构建和部署功能完整的 Web 应用。

关键要点：
- 优先使用 Git 集成实现自动化部署。
- 利用 `netlify.toml` 配置文件管理项目设置。
- 充分利用 Netlify 的内置功能（表单处理、重定向等）。
- 为不同环境配置相应的构建和部署策略。
- 定期监控站点性能和构建状态。

如有其他问题，请参考 Netlify 官方文档或您所使用框架的部署指南。