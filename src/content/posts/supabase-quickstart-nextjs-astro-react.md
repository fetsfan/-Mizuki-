---
title: Supabase 快速入门：集成 Next.js、Astro 与 React
published: 2025-09-15
description: 一站式快速上手教程，提供可复制命令与最小可运行示例，涵盖 Supabase 项目创建、客户端初始化、Auth/DB 基础操作与三种前端框架集成要点。
image: ''
tags: [Supabase, 数据库]
category: 数据库
draft: false
lang: zh-CN
---

> 本文目标：在 30-60 分钟内，通过最小可运行示例，帮助你快速掌握 Supabase 与三种主流前端框架（Next.js, Astro, React）的集成方法，覆盖基础的数据库读写操作。

:::tip[核心优势]
- **零配置启动**: 提供完整的代码片段与命令行指令，实现真正的“开箱即用”。
- **框架全覆盖**: 一篇文章同时满足 Next.js, Astro, React 开发者的入门需求。
- **生产级实践**: 虽然是快速入门，但代码结构与提示均考虑了后续扩展性。
:::

## 1. 准备工作

在开始之前，请确保你已准备好以下环境与账号：

- **Node.js**: 版本 18 或更高。
- **包管理器**: pnpm, npm, 或 yarn。
- **Supabase 账号**: 注册一个免费账号即可满足本教程所有需求。

## 2. Supabase 项目设置

### 创建项目

1.  登录 [Supabase Dashboard](https://app.supabase.com/)。
2.  点击 "New project" 创建一个新项目。
3.  在项目创建后，导航到 **Settings -> API** 页面。
4.  记录下你的 **Project URL** 和 **anon (public) API key**，后续步骤将频繁使用。

### 准备数据库表与 RLS 策略

为了演示数据读写，我们需要一张 `messages` 表，并为其配置行级安全 (Row Level Security, RLS) 策略。

:::caution[安全提示]
以下 SQL 策略仅为演示目的，允许任何匿名用户读写 `messages` 表。在生产环境中，你**必须**根据业务需求编写更严格的安全策略。
:::

在 Supabase SQL Editor 中执行以下脚本：

```sql
-- 1) 创建用于演示的 "messages" 表
create table if not exists public.messages (
  id bigserial primary key,
  content text not null,
  created_at timestamp with time zone default now()
);

-- 2) 为 "messages" 表启用行级安全 (RLS)
alter table public.messages enable row level security;

-- 3) 创建宽松的演示策略（生产环境请勿直接使用！）
create policy "Allow anonymous read" on public.messages for select using (true);
create policy "Allow anonymous insert" on public.messages for insert with check (true);
```

### 配置环境变量

在你的本地前端项目根目录下，为不同框架创建对应的 `.env.local` 或 `.env` 文件，并填入之前记录的 Supabase 项目信息。

:::info[命名约定]
不同前端框架对环境变量有不同的命名要求，请务必遵循：
- **Next.js**: `NEXT_PUBLIC_`
- **Astro**: `PUBLIC_`
- **React (Vite)**: `VITE_`
:::

**Next.js (.env.local):**
```bash
NEXT_PUBLIC_SUPABASE_URL=你的项目URL
NEXT_PUBLIC_SUPABASE_ANON_KEY=你的anonKey
```

**Astro (.env 或 .env.local):**
```bash
PUBLIC_SUPABASE_URL=你的项目URL
PUBLIC_SUPABASE_ANON_KEY=你的anonKey
```

**React (Vite) (.env.local):**
```bash
VITE_SUPABASE_URL=你的项目URL
VITE_SUPABASE_ANON_KEY=你的anonKey
```

---

## 3. 前端框架集成示例

现在，我们将分别演示如何在 Next.js, Astro, 和 React (Vite) 中集成 Supabase。

### Next.js 最小可运行示例

#### 1) 初始化项目与依赖

```bash
pnpm dlx create-next-app@latest supa-next --ts --app --use-pnpm
cd supa-next
pnpm add @supabase/supabase-js
```

#### 2) 创建 Supabase 客户端

新建 `src/lib/supabaseClient.ts` 文件：

```ts
// src/lib/supabaseClient.ts
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
)
```

#### 3) 在页面中读写数据

修改 `app/page.tsx` 文件，实现消息的加载与新增：

```tsx
// app/page.tsx
'use client'

import { useEffect, useState, useCallback } from 'react'
import { supabase } from '@/lib/supabaseClient'

type Message = {
  id: number;
  content: string;
};

export default function Page() {
  const [text, setText] = useState('')
  const [rows, setRows] = useState<Message[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const loadMessages = useCallback(async () => {
    setLoading(true)
    setError(null)
    try {
      const { data, error: dbError } = await supabase
        .from('messages')
        .select('id, content')
        .order('id', { ascending: false })
        .limit(20)
      
      if (dbError) throw dbError
      setRows(data || [])
    } catch (err: any) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }, [])

  const insertMessage = async () => {
    if (!text.trim()) return
    setLoading(true)
    setError(null)
    try {
      const { error: dbError } = await supabase.from('messages').insert({ content: text.trim() })
      if (dbError) throw dbError
      setText('')
      await loadMessages()
    } catch (err: any) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => {
    void loadMessages()
  }, [loadMessages])

  return (
    <main style={{ padding: 24, maxWidth: '600px', margin: 'auto' }}>
      <h1>Supabase × Next.js</h1>
      <div style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
        <input
          value={text}
          onChange={(e) => setText(e.target.value)}
          placeholder="输入消息"
          style={{ flex: 1 }}
        />
        <button onClick={insertMessage} disabled={loading}>
          {loading ? '提交中…' : '新增'}
        </button>
      </div>
      {error && <p style={{ color: 'red' }}>Error: {error}</p>}
      {loading && <p>Loading messages...</p>}
      <ul>
        {rows.map((r) => (
          <li key={r.id}>#{r.id}: {r.content}</li>
        ))}
      </ul>
    </main>
  )
}
```

#### 4) 运行

```bash
pnpm dev
```

### Astro 最小可运行示例

Astro 示例将利用其组件化的特性，将客户端逻辑封装起来，使页面更整洁。

#### 1) 初始化项目

```bash
pnpm create astro@latest supa-astro -- --template minimal
cd supa-astro
pnpm add @supabase/supabase-js
```

#### 2) 创建 Supabase 客户端组件

新建 `src/components/SupaMessages.astro` 文件。这个组件将处理所有与 Supabase 的交互。

```html
---
// src/components/SupaMessages.astro
---
<div id="messages-container">
  <h1>Supabase × Astro</h1>
  <div style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
    <input id="msg-input" placeholder="输入消息" style={{ flex: 1 }} />
    <button id="add-btn">新增</button>
  </div>
  <div id="error-display" style={{ color: 'red' }}></div>
  <p id="loading-display">Loading messages...</p>
  <ul id="messages-list"></ul>
</div>

<script>
  import { createClient } from "@supabase/supabase-js";

  const supabase = createClient(
    import.meta.env.PUBLIC_SUPABASE_URL,
    import.meta.env.PUBLIC_SUPABASE_ANON_KEY
  );

  const container = document.getElementById('messages-container');
  const listEl = container.querySelector('#messages-list');
  const inputEl = container.querySelector('#msg-input') as HTMLInputElement;
  const addBtn = container.querySelector('#add-btn');
  const errorDisplay = container.querySelector('#error-display');
  const loadingDisplay = container.querySelector('#loading-display');

  const loadMessages = async () => {
    loadingDisplay.style.display = 'block';
    errorDisplay.textContent = '';
    try {
      const { data, error } = await supabase
        .from('messages')
        .select('id, content')
        .order('id', { ascending: false })
        .limit(20);

      if (error) throw error;
      
      listEl.innerHTML = (data || [])
        .map((r) => `<li>#${r.id}: ${r.content}</li>`)
        .join('');
    } catch (err: any) {
      errorDisplay.textContent = `Error: ${err.message}`;
    } finally {
      loadingDisplay.style.display = 'none';
    }
  };

  addBtn.addEventListener('click', async () => {
    const content = inputEl.value.trim();
    if (!content) return;

    addBtn.setAttribute('disabled', 'true');
    errorDisplay.textContent = '';
    
    try {
      const { error } = await supabase.from('messages').insert({ content });
      if (error) throw error;
      inputEl.value = '';
      await loadMessages();
    } catch (err: any) {
      errorDisplay.textContent = `Error: ${err.message}`;
    } finally {
      addBtn.removeAttribute('disabled');
    }
  });

  // Initial load
  loadMessages();
</script>
```

#### 3) 在页面中使用组件

修改 `src/pages/index.astro`：

```html
---
// src/pages/index.astro
import SupaMessages from '../components/SupaMessages.astro';
---

<html lang="zh-CN">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Supabase × Astro</title>
    <style>
      body {
        padding: 24px;
        max-width: 600px;
        margin: auto;
      }
    </style>
  </head>
  <body>
    <SupaMessages />
  </body>
</html>
```

#### 4) 运行

```bash
pnpm dev
```

### React（Vite）最小可运行示例

#### 1) 初始化项目与依赖

```bash
pnpm create vite@latest supa-react -- --template react-ts
cd supa-react
pnpm add @supabase/supabase-js
```

#### 2) 创建 Supabase 客户端

新建 `src/lib/supabase.ts`：

```ts
// src/lib/supabase.ts
import { createClient } from '@supabase/supabase-js'

export const supabase = createClient(
  import.meta.env.VITE_SUPABASE_URL!,
  import.meta.env.VITE_SUPABASE_ANON_KEY!
)
```

#### 3) 在 `App.tsx` 中读写数据

修改 `src/App.tsx`：

```tsx
// src/App.tsx
import { useEffect, useState, useCallback } from 'react'
import { supabase } from './lib/supabase'

type Message = {
  id: number;
  content: string;
};

function App() {
  const [text, setText] = useState('')
  const [rows, setRows] = useState<Message[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  const loadMessages = useCallback(async () => {
    setLoading(true)
    setError(null)
    try {
      const { data, error: dbError } = await supabase
        .from('messages')
        .select('id, content')
        .order('id', { ascending: false })
        .limit(20)
      
      if (dbError) throw dbError
      setRows(data || [])
    } catch (err: any) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }, [])

  const insertMessage = async () => {
    if (!text.trim()) return
    setLoading(true)
    setError(null)
    try {
      const { error: dbError } = await supabase.from('messages').insert({ content: text.trim() })
      if (dbError) throw dbError
      setText('')
      await loadMessages()
    } catch (err: any) {
      setError(err.message)
    } finally {
      setLoading(false)
    }
  }

  useEffect(() => {
    void loadMessages()
  }, [loadMessages])

  return (
    <div style={{ padding: 24, maxWidth: '600px', margin: 'auto' }}>
      <h1>Supabase × React (Vite)</h1>
      <div style={{ display: 'flex', gap: 8, marginBottom: 16 }}>
        <input
          value={text}
          onChange={(e) => setText(e.target.value)}
          placeholder="输入消息"
          style={{ flex: 1 }}
        />
        <button onClick={insertMessage} disabled={loading}>
          {loading ? '提交中…' : '新增'}
        </button>
      </div>
      {error && <p style={{ color: 'red' }}>Error: {error}</p>}
      {loading && <p>Loading messages...</p>}
      <ul>
        {rows.map((r) => <li key={r.id}>#{r.id}: {r.content}</li>)}
      </ul>
    </div>
  )
}

export default App
```

#### 4) 运行

```bash
pnpm dev
```

## 4. 常见问题与排错

- **403/401 权限错误**:
  - **检查 RLS 策略**: 确保你的 RLS 策略允许当前操作（`select`, `insert` 等）。对于入门测试，可以使用教程中提供的宽松策略。
  - **检查 API Key**: 确认客户端使用的是 `anon` (public) key，而不是 `service_role` (secret) key。

- **连接失败**:
  - **核对 URL 和 Key**: 仔细检查 `.env` 文件中的 `URL` 和 `anon key` 是否与 Supabase 项目设置中的完全一致，注意不要有空格或特殊字符。

- **环境变量未加载**:
  - **遵循命名约定**: 确保变量前缀符合框架要求 (`NEXT_PUBLIC_`, `VITE_`, `PUBLIC_`)。
  - **重启开发服务器**: 修改 `.env` 文件后，需要重启开发服务器才能生效。

- **CORS 跨域问题**:
  - Supabase 默认已为你的 API 启用 CORS。如果你通过自定义域名或代理访问，请检查浏览器控制台的错误信息，并在 Supabase Dashboard 的 **API Settings -> CORS Configuration** 中添加你的访问源。

## 5. 后续扩展方向

- **用户认证 (Auth)**: 探索 Supabase 强大的认证功能，如邮箱/密码登录、社交登录 (GitHub, Google)、魔法链接等。
- **Edge Functions**: 将复杂的业务逻辑或需要安全环境的操作（如调用第三方服务、处理支付）封装在 Edge Functions 中。
- **实时 (Realtime)**: 利用 Supabase 的实时订阅功能，监听数据库变更，构建聊天、协作等动态应用。
- **存储 (Storage)**: 学习如何上传、下载和管理文件，如用户头像、文档等。

## 6. 附录：核心功能扩展

### 用户认证 (Auth)

Supabase 提供了强大且易于使用的认证解决方案。以下是如何在你的应用中集成基础的邮箱密码登录功能。

#### 1) 调整 RLS 策略

为了让登录用户能够管理自己的消息，我们需要更新 `messages` 表的 RLS 策略。将之前的宽松策略替换为以下更安全的版本：

```sql
-- 删除旧的匿名策略
DROP POLICY IF EXISTS "Allow anonymous read" ON public.messages;
DROP POLICY IF EXISTS "Allow anonymous insert" ON public.messages;

-- 创建新的基于用户的策略
-- 允许用户读取自己的消息
CREATE POLICY "Allow individual read access" ON public.messages FOR SELECT USING (auth.uid() = user_id);
-- 允许用户插入自己的消息
CREATE POLICY "Allow individual insert access" ON public.messages FOR INSERT WITH CHECK (auth.uid() = user_id);

-- 为表添加 user_id 字段
ALTER TABLE public.messages ADD COLUMN IF NOT EXISTS user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE;
```

:::caution[重要提示]
执行此操作前，请确保你的 `messages` 表中已有一个 `user_id` 字段，用于关联 `auth.users` 表。如果还没有，请先添加该字段。
:::

#### 2) 构建认证组件

你可以创建一个简单的认证组件，处理注册、登录和登出逻辑。以下是一个 React 示例：

```tsx
// components/Auth.tsx
import { useState } from 'react';
import { supabase } from '../lib/supabase'; // 假设你的 Supabase 客户端在这里

export default function Auth() {
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [loading, setLoading] = useState(false);

  const handleLogin = async (e) => {
    e.preventDefault();
    setLoading(true);
    const { error } = await supabase.auth.signInWithPassword({ email, password });
    if (error) alert(error.message);
    setLoading(false);
  };

  const handleSignup = async (e) => {
    e.preventDefault();
    setLoading(true);
    const { error } = await supabase.auth.signUp({ email, password });
    if (error) alert(error.message);
    else alert('注册成功，请检查你的邮箱以激活账号！');
    setLoading(false);
  };

  return (
    <div>
      <form onSubmit={handleLogin}>
        <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="邮箱" />
        <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="密码" />
        <button disabled={loading}>{loading ? '处理中...' : '登录'}</button>
        <button onClick={handleSignup} disabled={loading}>注册</button>
      </form>
    </div>
  );
}
```

### 实时 (Realtime)

通过 Supabase 的实时订阅，你的应用可以即时响应数据库的变化。

```javascript
// 监听 messages 表的插入事件
const channel = supabase.channel('messages-channel');

channel
  .on('postgres_changes', { event: 'INSERT', schema: 'public', table: 'messages' }, (payload) => {
    console.log('New message:', payload.new);
    // 在这里更新你的 UI
  })
  .subscribe();

// 不再需要时，记得取消订阅
// supabase.removeChannel(channel);
```

### 部署指南 (Deployment Guide)

将集成了 Supabase 的前端应用部署到 Vercel 或 Netlify 等平台非常简单，核心是正确配置环境变量。

1.  **登录你的托管平台** (Vercel, Netlify, etc.)。
2.  **导入你的 Git 仓库**。
3.  **配置环境变量**:
    - 在项目的设置页面中，找到环境变量 (Environment Variables) 配置区域。
    - 添加你在 `.env.local` 文件中使用的 `PUBLIC_SUPABASE_URL` 和 `PUBLIC_SUPABASE_ANON_KEY` (或对应框架的前缀)。
    - **重要**: 确保变量名与你的代码中的 `import.meta.env` 或 `process.env` 调用完全匹配。
4.  **触发部署**。

平台会自动检测你的框架 (Next.js, Astro, Vite) 并执行正确的构建命令。部署成功后，你的应用就可以在线访问了。
