# 阶段1：项目概览与技术栈分析 / Phase 1: Project Overview & Tech Stack Analysis

## 📋 项目基本信息 / Project Information

### 项目名称 / Project Name
**Wave AI - 第二大脑 AI 智能体 SaaS 平台**
**Wave AI - Second-Brain AI Agent SaaS Platform**

### 项目简介 / Project Description
这是一个基于 Next.js 15 和 React 19 的全栈 AI SaaS 应用，旨在构建一个"第二大脑"系统。用户可以：
- 与 AI 智能体对话
- 创建和管理笔记
- 从 URL 提取和总结内容
- 使用内置的网络搜索工具
- 通过 AI 工具链实现复杂功能

This is a full-stack AI SaaS application built with Next.js 15 and React 19, designed to create a "second-brain" system. Users can:
- Chat with AI agents
- Create and manage notes
- Extract and summarize content from URLs
- Use built-in web search tools
- Achieve complex functions through AI tool chaining

### 项目类型 / Project Type
- **架构模式**: 全栈 Web 应用 (Full-stack Web Application)
- **部署方式**: Serverless (Vercel)
- **数据库**: PostgreSQL
- **认证方式**: BetterAuth (Email/Password)
- **支付集成**: Stripe Subscription

---

## 🛠️ 核心技术栈 / Core Tech Stack

### 前端框架 / Frontend Framework

| 技术 | 版本 | 用途 |
|------|------|------|
| **Next.js** | 15.5.3 | 全栈 React 框架，支持服务端渲染和 App Router |
| **React** | 19.1.0 | UI 组件库 |
| **TypeScript** | ^5 | 类型安全的 JavaScript |
| **Tailwind CSS** | ^4 | CSS 工具类框架 |

**代码位置**:
- Next.js 配置: `next.config.ts:1`
- TypeScript 配置: `tsconfig.json:1`
- 全局样式: `app/globals.css:1`

### 后端框架 / Backend Framework

| 技术 | 版本 | 用途 |
|------|------|------|
| **Hono** | ^4.9.4 | 轻量级 API 框架，用于 API 路由 |
| **Prisma** | ^6.15.0 | ORM 数据库工具 |
| **BetterAuth** | ^1.3.7 | 认证解决方案 |

**代码位置**:
- API 路由: `app/api/[[...route]]/route.ts:1`
- Prisma Schema: `prisma/schema.prisma:1`
- Auth 配置: `lib/auth.ts:1`

### AI 集成 / AI Integration

| 技术 | 版本 | 用途 |
|------|------|------|
| **AI SDK** | ^5.0.45 | Vercel AI SDK，统一的 AI 接口 |
| **@ai-sdk/gateway** | ^1.0.23 | AI Gateway，统一多个 AI 提供商 |
| **@ai-sdk/google** | ^2.0.11 | Google AI (Gemini) 集成 |
| **@ai-sdk/react** | ^2.0.23 | React AI Hooks |
| **@tavily/core** | ^0.5.11 | Web 搜索 API |

**代码位置**:
- AI 模型配置: `lib/ai/models.ts:1`
- AI 提供商设置: `lib/ai/providers.ts:1`
- AI 系统提示词: `lib/ai/prompt.ts:1`
- AI 工具:
  - 创建笔记: `lib/ai/tools/create-note.ts:1`
  - 搜索笔记: `lib/ai/tools/search-note.ts:1`
  - 提取 URL 内容: `lib/ai/tools/extract-url.ts:1`
  - 网络搜索: `lib/ai/tools/web-search.ts:1`
  - 工具常量: `lib/ai/tools/constant.ts:1`

**支持的 AI 模型**:
1. Claude Sonnet 4 (Anthropic) - 默认模型
2. Grok 4 (xAI)
3. GPT-4.1 (OpenAI)
4. Gemini 2.5 Flash (Google)

### 支付系统 / Payment System

| 技术 | 版本 | 用途 |
|------|------|------|
| **Stripe** | ^18.5.0 | 订阅支付处理 |
| **@better-auth/stripe** | ^1.3.9 | BetterAuth Stripe 插件 |

**代码位置**:
- Stripe 客户端: `lib/stripe.ts:1`
- 订阅 API: `app/api/[[...route]]/subscription.ts:1`

### UI 组件库 / UI Component Library

| 技术 | 版本 | 用途 |
|------|------|------|
| **Shadcn/ui (Radix UI)** | Multiple | 可访问性优先的组件库 |
| **Lucide React** | ^0.544.0 | 图标库 |
| **Motion** | ^12.23.16 | 动画库 (Framer Motion) |
| **Sonner** | ^2.0.7 | Toast 通知 |

**代码位置**:
- UI 组件: `components/ui/*`
- Logo 组件: `components/logo/*`
- 聊天组件: `components/chat/*`
- 笔记对话框: `components/note-dialog/*`
- 侧边栏: `components/sidebar/*`
- 空状态: `components/empty-state/*`
- AI 元素: `components/ai-elements/*`

### 状态管理 / State Management

| 技术 | 版本 | 用途 |
|------|------|------|
| **Zustand** | ^5.0.8 | 轻量级状态管理 |
| **@tanstack/react-query** | ^5.85.5 | 服务器状态管理 |
| **React Hook Form** | ^7.62.0 | 表单状态管理 |
| **Zod** | ^4.1.8 | Schema 验证 |

**代码位置**:
- Context: `context/*`
- Hooks: `hooks/*`

### 其他重要依赖 / Other Important Dependencies

| 技术 | 版本 | 用途 |
|------|------|------|
| **nanoid** | ^5.1.5 | 唯一 ID 生成 |
| **uuid** | ^11.1.0 | UUID 生成 |
| **date-fns** | ^4.1.0 | 日期处理 |
| **clsx** & **tailwind-merge** | Multiple | CSS 类名处理 |
| **next-themes** | ^0.4.6 | 主题切换 (深色/浅色模式) |
| **nuqs** | ^2.5.2 | URL 状态管理 |
| **streamdown** | ^1.3.0 | Markdown 流式处理 |
| **react-syntax-highlighter** | ^15.6.6 | 代码高亮 |
| **tokenlens** | ^1.2.1 | Token 计数 |

---

## 📁 项目目录结构 / Project Structure

```
Wave-Secondbrain-AI-Agent-Saas/
├── app/                           # Next.js App Router
│   ├── (routes)/                  # 路由组
│   │   ├── (web)/                 # 网站公开页面
│   │   │   ├── _common/           # 共享组件 (Hero, Nav, Preview)
│   │   │   ├── layout.tsx         # Web 布局
│   │   │   └── page.tsx           # 首页
│   │   ├── (dashboard)/           # 仪表板页面 (需认证)
│   │   │   ├── _common/           # 共享组件 (Header, Chat History)
│   │   │   ├── home/              # 主页
│   │   │   ├── chat/              # 聊天页面
│   │   │   │   └── [chatId]/      # 动态聊天路由
│   │   │   ├── billing/           # 订阅计费页面
│   │   │   ├── settings/          # 设置页面
│   │   │   └── layout.tsx         # Dashboard 布局
│   │   └── auth/                  # 认证页面
│   │       ├── _common/           # 登录/注册表单
│   │       ├── sign-in/           # 登录页
│   │       ├── sign-up/           # 注册页
│   │       └── layout.tsx         # Auth 布局
│   ├── api/                       # API 路由
│   │   ├── [[...route]]/          # Hono API 路由
│   │   │   ├── route.ts           # 主路由文件
│   │   │   ├── chat.ts            # 聊天 API
│   │   │   ├── note.ts            # 笔记 API
│   │   │   └── subscription.ts    # 订阅 API
│   │   └── auth/[...all]/         # BetterAuth API
│   ├── actions/                   # Server Actions
│   │   └── action.ts              # 服务器端操作
│   ├── layout.tsx                 # 根布局
│   ├── globals.css                # 全局样式
│   └── favicon.ico                # 网站图标
│
├── components/                    # React 组件
│   ├── ui/                        # Shadcn/ui 基础组件
│   ├── chat/                      # 聊天相关组件
│   ├── sidebar/                   # 侧边栏组件
│   ├── note-dialog/               # 笔记对话框
│   ├── ai-elements/               # AI 相关元素
│   ├── empty-state/               # 空状态组件
│   ├── logo/                      # Logo 组件
│   └── loader-overlay/            # 加载覆盖层
│
├── lib/                           # 工具库和配置
│   ├── ai/                        # AI 相关
│   │   ├── tools/                 # AI 工具实现
│   │   │   ├── create-note.ts     # 创建笔记工具
│   │   │   ├── search-note.ts     # 搜索笔记工具
│   │   │   ├── extract-url.ts     # 提取 URL 工具
│   │   │   ├── web-search.ts      # 网络搜索工具
│   │   │   └── constant.ts        # 工具常量
│   │   ├── models.ts              # AI 模型配置
│   │   ├── providers.ts           # AI 提供商
│   │   └── prompt.ts              # 系统提示词
│   ├── hono/                      # Hono 相关
│   │   ├── hono-middleware.ts     # Hono 中间件
│   │   └── hono-rpc.ts            # RPC 类型定义
│   ├── auth.ts                    # BetterAuth 配置
│   ├── auth-client.ts             # Auth 客户端
│   ├── prisma.ts                  # Prisma 客户端
│   ├── stripe.ts                  # Stripe 客户端
│   ├── constant.ts                # 常量定义 (订阅计划)
│   └── utils.ts                   # 工具函数
│
├── prisma/                        # Prisma ORM
│   ├── schema.prisma              # 数据库 Schema
│   └── migrations/                # 数据库迁移文件
│
├── generated/                     # 自动生成的文件
│   └── prisma/                    # Prisma 客户端生成文件
│
├── context/                       # React Context
├── hooks/                         # 自定义 React Hooks
├── features/                      # 功能模块
├── public/                        # 静态资源
│   └── images/                    # 图片资源
│
├── testsprite_tests/              # TestSprite 测试
├── _ai-sdk-example/               # AI SDK 示例代码
│
├── next.config.ts                 # Next.js 配置
├── tsconfig.json                  # TypeScript 配置
├── tailwind.config.js             # Tailwind CSS 配置 (隐式)
├── postcss.config.mjs             # PostCSS 配置
├── eslint.config.mjs              # ESLint 配置
├── components.json                # Shadcn/ui 配置
├── package.json                   # 依赖配置
├── .gitignore                     # Git 忽略文件
├── README.md                      # 项目说明
└── TECHWITHEMMA-LICENSE.md        # 许可证
```

---

## 🔑 核心功能模块 / Core Features

### 1. 用户认证 (Authentication)
- **技术**: BetterAuth + Email/Password
- **功能**:
  - 用户注册 (`app/(routes)/auth/sign-up/`)
  - 用户登录 (`app/(routes)/auth/sign-in/`)
  - Session 管理
  - Stripe 集成 (自动创建 Stripe 客户)
- **数据模型**: User, Session, Account

### 2. AI 聊天 (AI Chat)
- **技术**: AI SDK + Hono API
- **功能**:
  - 多模型支持 (Claude, Grok, GPT-4, Gemini)
  - 流式响应
  - 工具调用 (Tool Chaining)
  - 聊天历史保存
- **数据模型**: Chat, Message
- **API**: `app/api/[[...route]]/chat.ts`

### 3. 笔记管理 (Notes Management)
- **技术**: Prisma + Hono API
- **功能**:
  - 创建笔记
  - 搜索笔记
  - 更新笔记
  - 删除笔记
- **数据模型**: Note
- **API**: `app/api/[[...route]]/note.ts`

### 4. AI 工具链 (AI Tool Chaining)
- **工具列表**:
  1. **Create Note** - 创建新笔记
  2. **Search Note** - 搜索现有笔记
  3. **Extract URL** - 从 URL 提取内容
  4. **Web Search** - 使用 Tavily API 搜索网络

### 5. 订阅计费 (Subscription Billing)
- **技术**: Stripe + BetterAuth Stripe Plugin
- **订阅计划**:
  - **Free Plan**: 10 次 AI 生成/月，基础功能
  - **Plus Plan**: $12/月，300 次 AI 生成/月
  - **Premium Plan**: $24/月，无限 AI 生成
- **功能**:
  - Stripe Checkout
  - Webhook 处理
  - 订阅状态同步
- **数据模型**: Subscription
- **API**: `app/api/[[...route]]/subscription.ts`

---

## ⚙️ 关键配置文件 / Key Configuration Files

### 1. package.json
**位置**: `/package.json:1`

**关键脚本**:
```json
{
  "dev": "next dev --turbopack",
  "build": "next build",
  "start": "next start",
  "lint": "eslint",
  "db:migrate": "npx prisma migrate dev --name init",
  "db:studio": "npx prisma studio",
  "postinstall": "prisma generate --no-engine"
}
```

### 2. Prisma Schema
**位置**: `/prisma/schema.prisma:1`

**数据源配置**:
```prisma
datasource db {
  provider = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}
```

### 3. TypeScript 配置
**位置**: `/tsconfig.json:1`

**关键配置**:
- **Target**: ES2017
- **Module**: ESNext
- **Path Alias**: `@/*` → `./\*`
- **JSX**: preserve (Next.js 处理)

### 4. Next.js 配置
**位置**: `/next.config.ts:1`

当前配置为空，使用 Next.js 默认设置。

---

## 🌍 环境变量清单 / Environment Variables

**注意**: 项目中未找到 `.env.example` 文件，以下是基于代码推断的环境变量清单。

### 必需环境变量 / Required Variables

```bash
# 数据库 / Database
DATABASE_URL=         # PostgreSQL 连接池 URL
DIRECT_URL=           # PostgreSQL 直连 URL (用于迁移)

# 认证 / Authentication
BETTER_AUTH_SECRET=   # BetterAuth 密钥

# Stripe 支付 / Stripe Payment
STRIPE_SECRET_KEY=          # Stripe 密钥
STRIPE_WEBHOOK_SECRET=      # Stripe Webhook 签名密钥
STRIPE_PLUS_PLAN_ID=        # Plus 计划价格 ID
STRIPE_PREMIUM_PLAN_ID=     # Premium 计划价格 ID

# AI 服务 / AI Services
AI_GATEWAY_URL=             # AI Gateway URL (可选)
GOOGLE_GENERATIVE_AI_API_KEY=  # Google AI API Key
ANTHROPIC_API_KEY=          # Anthropic API Key (可选)
OPENAI_API_KEY=             # OpenAI API Key (可选)
XAI_API_KEY=                # xAI API Key (可选)

# Web 搜索 / Web Search
TAVILY_API_KEY=       # Tavily 搜索 API Key

# 应用 / Application
NODE_ENV=             # production | development
NEXT_PUBLIC_APP_URL=  # 应用公开 URL
```

**环境变量使用位置**:
- `lib/auth.ts:26` - STRIPE_WEBHOOK_SECRET
- `lib/constant.ts:13-14` - STRIPE_PLUS_PLAN_ID, STRIPE_PREMIUM_PLAN_ID
- `prisma/schema.prisma:14-15` - DATABASE_URL, DIRECT_URL
- `lib/ai/providers.ts:7` - NODE_ENV

---

## 📊 项目架构模式 / Architecture Pattern

### 整体架构 / Overall Architecture
- **模式**: Monolithic Full-stack (单体全栈)
- **部署**: Serverless (Vercel)
- **数据流**: Client → API Route → Business Logic → Database

### 前端架构 / Frontend Architecture
- **模式**: Component-based (组件化)
- **路由**: Next.js App Router (文件系统路由)
- **布局**: 嵌套布局 (Nested Layouts)
  - Root Layout → Route Group Layout → Page

### 后端架构 / Backend Architecture
- **模式**: API-first
- **框架**: Hono (类 Express 的轻量级框架)
- **路由模式**: Catch-all routes (`[[...route]]`)
- **数据访问**: Prisma ORM

### 数据库架构 / Database Architecture
- **模式**: Relational (关系型)
- **规范化**: 第三范式 (3NF)
- **关系**:
  - User → Sessions (1:N)
  - User → Accounts (1:N)
  - User → Notes (1:N)
  - User → Chats (1:N)
  - User → Subscriptions (1:N)
  - Chat → Messages (1:N)

---

## 📈 项目统计 / Project Statistics

- **TypeScript 文件总数**: 122 个
- **主要代码行数**: 约 30,000+ 行 (估算)
- **组件数量**: 30+ 个
- **API 端点数量**: 10+ 个
- **AI 工具数量**: 4 个
- **数据库表数量**: 6 个 (User, Session, Account, Note, Chat, Message, Subscription)

---

## 🎯 项目特色 / Project Highlights

### 1. 多 AI 模型支持
通过 AI Gateway 统一接口，支持 Claude、Grok、GPT-4、Gemini 等多个模型，用户可在 UI 中切换。

### 2. AI 工具链系统
实现了完整的 AI 工具调用机制：
- AI 可以自主调用工具
- 工具可以链式调用
- 支持流式响应

### 3. 订阅计费集成
完整的 Stripe 订阅流程：
- 自动创建 Stripe 客户
- Webhook 同步订阅状态
- 多层级订阅计划

### 4. 现代化技术栈
- Next.js 15 (最新版)
- React 19 (最新版)
- Tailwind CSS v4 (最新版)
- TypeScript 5

### 5. 开发者体验优化
- Turbopack 开发服务器
- Prisma Studio 数据库管理
- 类型安全的 API (Hono RPC)
- 代码自动生成 (Prisma)

---

## 🚀 快速启动命令 / Quick Start Commands

```bash
# 安装依赖
npm install

# 数据库迁移
npm run db:migrate

# 启动开发服务器 (使用 Turbopack)
npm run dev

# 打开 Prisma Studio (数据库 GUI)
npm run db:studio

# 构建生产版本
npm run build

# 启动生产服务器
npm start

# 代码检查
npm run lint
```

---

## 📝 开发注意事项 / Development Notes

### 1. Prisma 生成位置
Prisma 客户端生成到 `generated/prisma/` 而非默认的 `node_modules/@prisma/client`。
所有导入使用 `@/generated/prisma` 路径。

### 2. API 路由结构
使用 Hono 的 Catch-all 路由模式，所有 API 都在 `app/api/[[...route]]/` 下。

### 3. 认证流程
BetterAuth 提供完整的认证解决方案，包括 Session 管理、OAuth 支持（虽然当前只启用了 Email/Password）。

### 4. AI 模型切换
生产环境使用 AI Gateway，开发环境直接使用 Google Gemini。

### 5. 环境区分
通过 `NODE_ENV` 区分生产和开发环境，影响 AI 提供商选择。

---

## ✅ 阶段1完成标记 / Phase 1 Completion

✓ 技术栈清单（带版本号）
✓ 目录结构树状图
✓ 项目架构模式识别
✓ 关键配置文件列表
✓ 环境变量清单

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 数据库深度分析 (`01-database-analysis.md`)
