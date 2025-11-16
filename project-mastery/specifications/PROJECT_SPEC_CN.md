# Wave AI 项目技术规格文档（中文版）

> 第二大脑 AI 智能体 SaaS 平台完整技术规格

**生成日期**: 2025-11-16
**文档版本**: v1.0
**目标读者**: 开发者、产品经理、非技术人员

---

## 📖 目录

1. [项目概述](#1-项目概述)
2. [技术架构](#2-技术架构)
3. [数据库设计](#3-数据库设计)
4. [核心业务流程](#4-核心业务流程)
5. [前后端交互](#5-前后端交互)
6. [API接口文档](#6-api接口文档)
7. [环境配置指南](#7-环境配置指南)
8. [二次开发指南](#8-二次开发指南)
9. [部署指南](#9-部署指南)
10. [附录](#10-附录)

---

## 1. 项目概述

### 1.1 项目简介

Wave AI 是一个**"第二大脑"AI 助手平台**，就像拥有一个永不疲倦的私人助理，可以：

- **智能对话**: 与多个顶级AI模型（Claude、GPT-4、Gemini等）对话
- **笔记管理**: AI帮你创建、搜索、整理笔记
- **网络搜索**: 获取实时信息，不局限于AI的训练数据
- **内容提取**: 从任何网页提取和总结内容

**类比说明**:
想象你有一个超级助手，它能：
- 像Google一样搜索互联网
- 像Notion一样管理笔记
- 像ChatGPT一样对话
- 还能自动帮你保存有价值的信息

### 1.2 主要功能列表

| 功能 | 描述 | 适用场景 |
|------|------|---------|
| **AI聊天** | 多模型对话，流式响应 | 日常咨询、学习辅助 |
| **笔记系统** | 创建、编辑、搜索笔记 | 知识管理、内容整理 |
| **AI工具链** | AI主动调用工具完成任务 | 复杂任务自动化 |
| **网络搜索** | 实时获取最新信息 | 新闻、研究、市场信息 |
| **订阅计费** | Free/Plus/Premium三级计划 | 按需付费 |

### 1.3 技术亮点

1. **多AI模型统一接口**: 通过AI Gateway无缝切换Claude、Grok、GPT-4、Gemini
2. **AI工具调用系统**: AI可以主动调用笔记、搜索等工具
3. **流式响应**: 实时显示AI生成内容，体验流畅
4. **Stripe订阅集成**: 完整的支付流程和Webhook同步
5. **现代化技术栈**: Next.js 15 + React 19 + TypeScript 5

### 1.4 适用场景

- **个人知识管理**: 研究人员、学生、内容创作者
- **企业内部工具**: 团队协作、知识库
- **SaaS产品学习**: 全栈开发、AI集成、支付系统

---

## 2. 技术架构

### 2.1 整体架构图

```
                    ┌─────────────────────────────────┐
                    │      用户浏览器 (Browser)        │
                    │  ┌───────────┐  ┌────────────┐ │
                    │  │ React 19  │  │ Tailwind  │ │
                    │  │ 前端组件   │  │  CSS 样式  │ │
                    │  └───────────┘  └────────────┘ │
                    └───────────────┬─────────────────┘
                                    │ HTTP/WebSocket
                    ┌───────────────▼─────────────────┐
                    │      Next.js 15 App Router      │
                    │ ┌─────────────────────────────┐ │
                    │ │  Server Components (SSR)    │ │
                    │ └─────────────────────────────┘ │
                    │ ┌─────────────────────────────┐ │
                    │ │  API Routes (Hono)          │ │
                    │ │  - /api/chat                │ │
                    │ │  - /api/note                │ │
                    │ │  - /api/subscription        │ │
                    │ └──────────┬──────────────────┘ │
                    └────────────┼────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
        ▼                        ▼                        ▼
┌───────────────┐      ┌──────────────────┐    ┌─────────────────┐
│  BetterAuth   │      │  AI SDK (Vercel) │    │  Prisma ORM     │
│  认证系统      │      │  - AI Gateway    │    │  数据访问层      │
│               │      │  - 流式响应       │    │                 │
└───────┬───────┘      └────────┬─────────┘    └────────┬────────┘
        │                       │                       │
        ▼                       ▼                       ▼
┌───────────────┐      ┌──────────────────┐    ┌─────────────────┐
│  Stripe API   │      │  External APIs   │    │  PostgreSQL     │
│  支付处理      │      │  - Tavily Search │    │  数据库          │
└───────────────┘      │  - Claude API    │    └─────────────────┘
                       │  - OpenAI API    │
                       └──────────────────┘
```

### 2.2 技术栈说明

| 技术 | 版本 | 作用 | 为什么选择它 |
|------|------|------|-------------|
| **Next.js** | 15.5.3 | 全栈框架 | 服务端渲染、API路由、文件系统路由 |
| **React** | 19.1.0 | UI库 | 组件化开发、丰富生态 |
| **TypeScript** | 5+ | 类型系统 | 类型安全、减少bug |
| **Hono** | 4.9.4 | API框架 | 轻量、快速、Edge兼容 |
| **Prisma** | 6.15.0 | ORM | 类型安全、自动迁移、DevEx优秀 |
| **BetterAuth** | 1.3.7 | 认证 | 开箱即用、Stripe集成 |
| **AI SDK** | 5.0.45 | AI集成 | 统一接口、流式响应、工具调用 |
| **Tailwind CSS** | 4 | 样式 | 快速开发、响应式、可定制 |

### 2.3 前后端交互流程

```
用户操作
  │
  ├─ 点击按钮
  │    ↓
  │  React组件
  │    ↓
  │  调用 React Query Hook
  │    ↓
  │  Hono RPC 客户端 (类型安全)
  │    ↓
  │  HTTP请求
  │    ↓
  │  Next.js API Route
  │    ↓
  │  Hono 中间件 (认证)
  │    ↓
  │  业务逻辑层
  │    ↓
  │  Prisma查询数据库
  │    ↓
  │  返回JSON响应
  │    ↓
  │  React Query 缓存
  │    ↓
  │  更新UI
  └─ 用户看到结果
```

### 2.4 数据流图

```
┌─────────┐        ┌──────────┐        ┌──────────┐
│  前端    │ ─────▶ │  API层   │ ─────▶ │  数据库  │
│ (React) │        │  (Hono)  │        │(Postgres)│
└─────────┘        └──────────┘        └──────────┘
     │                   │                    │
     │                   │                    │
     │             ┌─────▼────────┐          │
     │             │  AI SDK层    │          │
     │             │ - AI工具调用 │          │
     │             └──────────────┘          │
     │                                        │
     └────────────── 状态管理 ───────────────┘
         (Zustand + React Query)
```

---

## 3. 数据库设计

### 3.1 ER关系图

```
User (用户)
├── sessions (1:N) → Session (会话)
├── accounts (1:N) → Account (账户)
├── notes (1:N) → Note (笔记)
├── Chat (1:N) → Chat (聊天)
│   └── messages (1:N) → Message (消息)
└── subscriptions (1:N) → Subscription (订阅)
```

### 3.2 核心表说明

#### User (用户表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | String (PK) | 用户唯一标识 |
| email | String (唯一) | 邮箱 |
| name | String | 用户名 |
| stripeCustomerId | String? | Stripe客户ID |
| createdAt | DateTime | 创建时间 |

**业务含义**: 存储用户基本信息，注册时自动创建。

#### Note (笔记表)

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID (PK) | 笔记ID |
| title | String | 标题 |
| content | String | 内容(Markdown) |
| userId | String (FK) | 所属用户 |

**业务含义**: 用户创建的笔记，AI可以通过工具访问。

#### Chat & Message (聊天与消息)

**Chat表**:
- id: 聊天会话ID
- title: AI自动生成的标题
- userId: 所属用户

**Message表**:
- id: 消息ID
- role: "user" | "assistant" | "system"
- parts: JSON (支持文本、工具调用等多模态内容)
- chatId: 所属聊天

**业务含义**: Chat是对话容器，Message存储每条消息。

#### Subscription (订阅表)

| 字段 | 类型 | 说明 |
|------|------|------|
| plan | Enum | "free" / "plus" / "premium" |
| stripeSubscriptionId | String? | Stripe订阅ID |
| periodStart | DateTime? | 计费周期开始 |
| periodEnd | DateTime? | 计费周期结束 |

**业务含义**: 管理用户订阅状态，控制AI生成次数限制。

---

## 4. 核心业务流程

### 4.1 用户注册流程

```
用户填写注册表单
  │
  ▼
[前端] 表单验证 (React Hook Form + Zod)
  │
  ▼
调用 authClient.signUp.email()
  │  文件: app/(routes)/auth/_common/signup-form.tsx
  ▼
[BetterAuth] 创建 User 记录
  │  数据库: user 表
  ▼
[Stripe 插件] 在 Stripe 创建 Customer
  │  设置 user.stripeCustomerId
  ▼
[回调] createDefaultSubscription()
  │  文件: app/actions/action.ts:30-64
  ▼
创建 Subscription 记录 (plan = "free")
  │  数据库: subscriptions 表
  ▼
创建 Account 记录 (密码哈希)
  │  数据库: account 表
  ▼
创建 Session 并返回 Token
  │  数据库: session 表
  ▼
前端自动登录并跳转到 /home
```

**涉及的文件**:
- `lib/auth.ts:12-38` - BetterAuth配置
- `app/actions/action.ts:30-64` - 创建默认订阅
- `app/(routes)/auth/_common/signup-form.tsx` - 注册表单

**涉及的表**: User, Account, Session, Subscription

### 4.2 AI聊天流程

```
用户输入消息
  │
  ▼
[前端] useChat hook 处理
  │  文件: components/chat/index.tsx
  ▼
POST /api/chat
  │  请求体: { id, message, selectedModelId }
  ▼
[后端] 验证用户认证
  │  中间件: lib/hono/hono-middleware.ts:19-33
  ▼
检查生成限制
  │  函数: checkGenerationLimit()
  ├─ 超限 → 返回403错误
  └─ 未超限 → 继续
  ▼
查找或创建 Chat 记录
  │  数据库: chats 表
  ▼
保存用户消息
  │  数据库: messages 表
  ▼
调用 AI streamText API
  │  - 加载消息历史
  │  - 注册AI工具 (createNote, searchNote, webSearch, extractWebUrl)
  │  - 流式生成响应
  │  文件: app/api/[[...route]]/chat.ts:101-140
  ▼
AI 决定是否调用工具
  ├─ 调用工具 → 执行工具 → 返回结果 → 继续生成
  └─ 不调用 → 直接生成文本
  ▼
流式返回响应到前端
  │  SSE (Server-Sent Events)
  ▼
onFinish 回调: 保存所有AI消息
  │  数据库: messages 表
  ▼
前端实时显示AI响应
```

**涉及的文件**:
- `app/api/[[...route]]/chat.ts` - 聊天API
- `components/chat/index.tsx` - 聊天界面
- `lib/ai/tools/*` - AI工具实现

**涉及的表**: Chat, Message, Note (如果AI创建笔记)

### 4.3 笔记创建流程

```
方式1: 用户手动创建
  │
  ▼
点击 "New Note" 按钮
  │  文件: components/sidebar/nav-notes.tsx
  ▼
调用 useCreateNote()
  │  文件: features/use-note.ts
  ▼
POST /api/note/create
  │  请求体: { title: "Untitled", content: "" }
  ▼
[后端] 创建 Note 记录
  │  数据库: notes 表
  ▼
返回新笔记
  │  响应: { success: true, data: note }
  ▼
前端打开笔记对话框
  │  设置 URL 参数: ?noteId=xxx
  ▼
用户编辑笔记

方式2: AI工具创建
  │
  ▼
用户: "帮我创建一个关于TypeScript的笔记"
  │
  ▼
AI决定调用 createNote 工具
  │  文件: lib/ai/tools/create-note.ts
  ▼
工具执行: prisma.note.create()
  │  数据库: notes 表
  ▼
返回结果给AI
  │
  ▼
AI告知用户: "✓ 已创建笔记 'TypeScript'"
  │
  ▼
前端刷新笔记列表
  │  React Query 自动失效缓存
```

**涉及的文件**:
- `app/api/[[...route]]/note.ts` - 笔记API
- `lib/ai/tools/create-note.ts` - AI创建笔记工具
- `components/note-dialog/*` - 笔记编辑界面

**涉及的表**: Note

### 4.4 订阅升级流程

```
用户点击 "Upgrade to Plus"
  │  文件: app/(routes)/(dashboard)/billing/page.tsx
  ▼
调用 useUpgradeSubscription()
  │  文件: features/use-subscription.ts
  ▼
POST /api/subscription/upgrade
  │  请求体: { plan: "plus", callbackUrl: "/billing" }
  ▼
[后端] 调用 BetterAuth Stripe API
  │  auth.api.upgradeSubscription()
  │  文件: app/api/[[...route]]/subscription.ts:18-67
  ▼
[BetterAuth] 创建 Stripe Checkout Session
  │  - customer = user.stripeCustomerId
  │  - price = STRIPE_PLUS_PLAN_ID
  │  - successUrl, cancelUrl
  ▼
返回 Checkout URL
  │  响应: { success: true, checkoutUrl: "https://..." }
  ▼
前端重定向到 Stripe 支付页面
  │
  ▼
用户完成支付
  │
  ▼
Stripe 发送 Webhook 事件
  │  → POST /api/auth/webhook/stripe
  │  事件: checkout.session.completed
  ▼
[BetterAuth] 处理 Webhook
  │  - 验证签名
  │  - 更新 Subscription 记录
  │  数据库: subscriptions 表
  │  {
  │    plan: "plus",
  │    status: "active",
  │    stripeSubscriptionId: "sub_xxx",
  │    periodStart, periodEnd
  │  }
  ▼
用户获得 Plus 计划权限
  │  生成限制: 10 → 300 次/月
```

**涉及的文件**:
- `app/api/[[...route]]/subscription.ts` - 订阅API
- `lib/auth.ts:23-36` - Stripe插件配置

**涉及的表**: Subscription

---

## 5. 前后端交互

### 5.1 API调用方式

项目使用 **Hono RPC** 实现类型安全的API调用：

**后端定义** (app/api/[[...route]]/route.ts):
```typescript
const app = new Hono();
app.get('/note/:id', getAuthUser, async (c) => {
  // ...
  return c.json({ success: true, data: note });
});

export type AppType = typeof app;
```

**前端调用** (features/use-note.ts):
```typescript
import { api } from "@/lib/hono/hono-rpc";

const response = await api.note[":id"].$get({
  param: { id: "note-123" }
});
const data = await response.json();
// ✓ 完全类型推断
```

**优势**:
- 自动类型推断
- 参数类型检查
- 无需手动维护API类型

### 5.2 Server Actions

用于服务器端逻辑，无需创建API端点：

```typescript
"use server";

export async function generateTitleForUserMessage({ message }) {
  const { text } = await generateText({
    model: myProvider.languageModel("title-model"),
    prompt: JSON.stringify(message),
  });
  return text;
}
```

**使用场景**:
- 辅助函数（生成标题、检查限制）
- 不需要前端直接调用的逻辑

### 5.3 数据获取模式

使用 **React Query** 管理服务器状态：

```typescript
// 查询数据
const { data, isLoading } = useNotes(1, 20);

// 修改数据
const { mutate } = useCreateNote();
mutate({ title: "New Note", content: "" });

// 自动失效缓存
onSuccess: () => {
  queryClient.invalidateQueries({ queryKey: ["notes"] });
}
```

**优势**:
- 自动缓存
- 后台重新验证
- 乐观更新
- 重试机制

### 5.4 表单验证

使用 **React Hook Form + Zod**：

```typescript
const schema = z.object({
  email: z.string().email("Invalid email"),
  password: z.string().min(6, "Too short"),
});

const form = useForm({
  resolver: zodResolver(schema),
});
```

**流程**:
1. 用户输入 → 实时验证
2. 提交表单 → Schema验证
3. 验证失败 → 显示错误
4. 验证成功 → 调用API

---

## 6. API接口文档

### 6.1 认证API

#### POST /api/auth/sign-up/email

**描述**: 用户注册

**请求体**:
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "password": "password123"
}
```

**响应** (成功):
```json
{
  "user": {
    "id": "user_xxx",
    "email": "john@example.com",
    "name": "John Doe"
  },
  "session": {
    "token": "...",
    "expiresAt": "2025-12-16T00:00:00Z"
  }
}
```

**错误码**:
- `400`: 邮箱已存在
- `400`: 验证失败

#### POST /api/auth/sign-in/email

**描述**: 用户登录

**请求体**:
```json
{
  "email": "john@example.com",
  "password": "password123"
}
```

**响应**: 同注册

### 6.2 聊天API

#### POST /api/chat

**描述**: 发送消息，获取AI流式响应

**认证**: 必需 (Cookie)

**请求体**:
```json
{
  "id": "chat-uuid-123",
  "message": {
    "id": "msg-uuid-456",
    "role": "user",
    "parts": [
      { "type": "text", "text": "你好" }
    ]
  },
  "selectedModelId": "anthropic/claude-sonnet-4",
  "selectedToolName": null
}
```

**响应**: Server-Sent Events (SSE) 流

**curl示例**:
```bash
curl -X POST http://localhost:3000/api/chat \
  -H "Cookie: better-auth.session_token=..." \
  -H "Content-Type: application/json" \
  -d '{
    "id": "chat-123",
    "message": {...},
    "selectedModelId": "anthropic/claude-sonnet-4"
  }'
```

#### GET /api/chat

**描述**: 获取用户所有聊天

**认证**: 必需

**响应**:
```json
{
  "success": true,
  "data": [
    {
      "id": "chat-123",
      "title": "讨论AI工具",
      "userId": "user_xxx",
      "createdAt": "2025-11-16T00:00:00Z",
      "updatedAt": "2025-11-16T01:00:00Z"
    }
  ]
}
```

#### GET /api/chat/:id

**描述**: 获取聊天详情及消息

**路径参数**: `id` - 聊天ID

**响应**:
```json
{
  "success": true,
  "data": {
    "id": "chat-123",
    "title": "讨论AI工具",
    "messages": [
      {
        "id": "msg-456",
        "role": "user",
        "parts": [
          { "type": "text", "text": "你好" }
        ]
      },
      {
        "id": "msg-789",
        "role": "assistant",
        "parts": [
          { "type": "text", "text": "你好！" }
        ]
      }
    ]
  }
}
```

### 6.3 笔记API

#### POST /api/note/create

**描述**: 创建笔记

**请求体**:
```json
{
  "title": "我的笔记",
  "content": "这是内容..."
}
```

**响应**:
```json
{
  "success": true,
  "data": {
    "id": "note-uuid-123",
    "title": "我的笔记",
    "content": "这是内容...",
    "userId": "user_xxx",
    "createdAt": "2025-11-16T00:00:00Z"
  }
}
```

#### GET /api/note/all

**查询参数**:
- `page`: 页码 (默认 1)
- `limit`: 每页数量 (默认 20)

**响应**:
```json
{
  "success": true,
  "data": [...], // 笔记数组
  "pagination": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "totalPages": 3
  }
}
```

#### PATCH /api/note/update/:id

**描述**: 更新笔记

**请求体**:
```json
{
  "title": "新标题",
  "content": "新内容"
}
```

#### DELETE /api/note/delete/:id

**描述**: 删除笔记

**响应**:
```json
{
  "success": true,
  "message": "Note deleted successfully"
}
```

### 6.4 订阅API

#### POST /api/subscription/upgrade

**描述**: 创建Stripe Checkout会话

**请求体**:
```json
{
  "plan": "plus",
  "callbackUrl": "https://app.example.com/billing"
}
```

**响应**:
```json
{
  "success": true,
  "checkoutUrl": "https://checkout.stripe.com/pay/cs_test_..."
}
```

#### GET /api/subscription/generations

**描述**: 查询生成限制

**响应**:
```json
{
  "success": true,
  "data": {
    "isAllowed": true,
    "plan": "free",
    "generationsUsed": 3,
    "generationsLimit": 10,
    "remainingGenerations": 7
  }
}
```

---

## 7. 环境配置指南

### 7.1 环境变量清单

| 变量名 | 必需 | 说明 | 示例 |
|--------|------|------|------|
| `DATABASE_URL` | ✓ | PostgreSQL连接池URL | `postgresql://...?pgbouncer=true` |
| `DIRECT_URL` | ✓ | PostgreSQL直连URL | `postgresql://...` |
| `BETTER_AUTH_SECRET` | ✓ | 认证密钥 | `openssl rand -base64 32` |
| `NEXT_PUBLIC_APP_URL` | ✓ | 应用URL | `http://localhost:3000` |
| `STRIPE_SECRET_KEY` | ✓ | Stripe密钥 | `sk_test_...` |
| `STRIPE_WEBHOOK_SECRET` | ✓ | Webhook签名密钥 | `whsec_...` |
| `STRIPE_PLUS_PLAN_ID` | ✓ | Plus计划价格ID | `price_...` |
| `STRIPE_PREMIUM_PLAN_ID` | ✓ | Premium计划价格ID | `price_...` |
| `GOOGLE_GENERATIVE_AI_API_KEY` | ✓ | Google AI密钥 | `...` |
| `TAVILY_API_KEY` | ✓ | Tavily搜索密钥 | `tvly-...` |
| `AI_GATEWAY_URL` | ✗ | AI Gateway地址 | `https://...` |
| `ANTHROPIC_API_KEY` | ✗ | Claude密钥 | `sk-ant-...` |

### 7.2 获取服务密钥

#### PostgreSQL数据库 (推荐 Neon/Supabase)

**Supabase**:
1. 访问 https://supabase.com
2. 创建新项目
3. Settings → Database → Connection Pooling
4. 复制 "Transaction Pooler" URL → `DATABASE_URL`
5. 复制 "Direct connection" URL → `DIRECT_URL`

**Neon**:
1. 访问 https://neon.tech
2. 创建新项目
3. 从Dashboard获取连接字符串

#### Stripe

1. 注册 https://stripe.com
2. Dashboard → Developers → API keys
3. 复制 Secret key → `STRIPE_SECRET_KEY`
4. Products → 创建Plus产品 ($12/月)
5. 复制 Price ID → `STRIPE_PLUS_PLAN_ID`
6. 创建Premium产品 ($24/月) → `STRIPE_PREMIUM_PLAN_ID`

**配置Webhook** (本地开发):
```bash
# 安装Stripe CLI
brew install stripe/stripe-cli/stripe

# 登录
stripe login

# 转发Webhook到本地
stripe listen --forward-to localhost:3000/api/auth/webhook/stripe
# 复制显示的 whsec_xxx → STRIPE_WEBHOOK_SECRET
```

#### Google AI

1. 访问 https://aistudio.google.com/app/apikey
2. 点击 "Create API Key"
3. 复制密钥 → `GOOGLE_GENERATIVE_AI_API_KEY`

#### Tavily搜索

1. 注册 https://tavily.com
2. Dashboard → API Keys
3. 复制密钥 → `TAVILY_API_KEY`

### 7.3 本地开发配置

**步骤 1**: 克隆项目
```bash
git clone <repository>
cd Wave-Secondbrain-AI-Agent-Saas
```

**步骤 2**: 安装依赖
```bash
npm install
```

**步骤 3**: 配置环境变量
```bash
cp .env.example .env.local
# 编辑 .env.local，填入真实值
```

**步骤 4**: 初始化数据库
```bash
npx prisma generate --no-engine
npx prisma migrate dev --name init
```

**步骤 5**: 启动开发服务器
```bash
npm run dev
```

访问 http://localhost:3000

### 7.4 生产环境配置

参见 [第9章 部署指南](#9-部署指南)

---

## 8. 二次开发指南

### 8.1 开发环境搭建

见 [7.3 本地开发配置](#73-本地开发配置)

### 8.2 项目结构导航

```
app/
├── (routes)/          # 路由组
│   ├── (web)/         # 公开页面 (首页)
│   ├── (dashboard)/   # 仪表板 (需认证)
│   │   ├── home/      # 主页
│   │   ├── chat/      # 聊天
│   │   └── billing/   # 订阅
│   └── auth/          # 登录/注册
├── api/               # API路由
│   ├── [[...route]]/  # Hono API
│   └── auth/          # BetterAuth
└── actions/           # Server Actions

components/            # UI组件
├── ui/                # Shadcn基础组件
├── chat/              # 聊天组件
├── sidebar/           # 侧边栏
└── note-dialog/       # 笔记对话框

lib/                   # 核心库
├── ai/                # AI集成
│   └── tools/         # AI工具
├── hono/              # Hono配置
├── auth.ts            # 认证
└── prisma.ts          # 数据库

features/              # React Query hooks
└── use-note.ts        # 笔记操作

hooks/                 # 自定义Hooks
└── use-localchat.ts   # 本地状态

prisma/
└── schema.prisma      # 数据库Schema
```

### 8.3 常见开发任务

#### 任务1: 添加新数据模型

**步骤**:
1. 编辑 `prisma/schema.prisma`
```prisma
model Task {
  id        String   @id @default(uuid())
  title     String
  completed Boolean  @default(false)
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  createdAt DateTime @default(now())

  @@map("tasks")
}
```

2. 更新User模型
```prisma
model User {
  // ...
  tasks     Task[]
}
```

3. 创建迁移
```bash
npx prisma migrate dev --name add_tasks
```

4. 生成客户端
```bash
npx prisma generate --no-engine
```

#### 任务2: 创建新API端点

**文件**: `app/api/[[...route]]/task.ts`

```typescript
import { Hono } from "hono";
import { zValidator } from "@hono/zod-validator";
import { z } from "zod";
import prisma from "@/lib/prisma";
import { getAuthUser } from "@/lib/hono/hono-middleware";

const app = new Hono();

const taskSchema = z.object({
  title: z.string().min(1),
});

// 创建任务
app.post(
  "/create",
  zValidator("json", taskSchema),
  getAuthUser,
  async (c) => {
    const user = c.get("user");
    const { title } = c.req.valid("json");

    const task = await prisma.task.create({
      data: {
        userId: user.id,
        title,
      },
    });

    return c.json({ success: true, data: task });
  }
);

// 获取所有任务
app.get("/all", getAuthUser, async (c) => {
  const user = c.get("user");

  const tasks = await prisma.task.findMany({
    where: { userId: user.id },
  });

  return c.json({ success: true, data: tasks });
});

export default app;
```

**注册路由** (`app/api/[[...route]]/route.ts`):
```typescript
import taskApp from "./task";

app.route("/task", taskApp);
```

#### 任务3: 添加新页面

**文件**: `app/(routes)/(dashboard)/tasks/page.tsx`

```typescript
"use client";

import { useTasks } from "@/features/use-task";
import { Button } from "@/components/ui/button";

export default function TasksPage() {
  const { data, isLoading } = useTasks();

  if (isLoading) return <div>Loading...</div>;

  const tasks = data?.data || [];

  return (
    <div className="p-8">
      <h1 className="text-2xl font-bold mb-4">Tasks</h1>
      <ul>
        {tasks.map((task) => (
          <li key={task.id}>{task.title}</li>
        ))}
      </ul>
    </div>
  );
}
```

**创建React Query Hook** (`features/use-task.ts`):
```typescript
import { useQuery, useMutation } from "@tanstack/react-query";
import { api } from "@/lib/hono/hono-rpc";

export const useTasks = () => {
  return useQuery({
    queryKey: ["tasks"],
    queryFn: async () => {
      const response = await api.task.all.$get();
      return response.json();
    },
  });
};

export const useCreateTask = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async (json: { title: string }) => {
      const response = await api.task.create.$post({ json });
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["tasks"] });
    },
  });
};
```

#### 任务4: 修改组件样式

使用Tailwind CSS:

```typescript
// Before
<div className="bg-white p-4">
  <h1 className="text-xl">Title</h1>
</div>

// After - 响应式 + 深色模式
<div className="bg-white dark:bg-gray-800 p-4 md:p-6">
  <h1 className="text-xl md:text-2xl text-gray-900 dark:text-white">
    Title
  </h1>
</div>
```

**常用类名**:
- 间距: `p-4`, `mx-auto`, `gap-2`
- 布局: `flex`, `grid`, `items-center`
- 响应式: `md:`, `lg:`, `xl:`
- 深色模式: `dark:`

### 8.4 调试技巧

**1. 查看数据库**:
```bash
npx prisma studio
```
访问 http://localhost:5555

**2. 查看React Query状态**:
安装 React Query Devtools (已内置)

**3. 查看API请求**:
- 浏览器DevTools → Network
- Console → 查看错误

**4. 后端日志**:
```typescript
console.log("调试信息:", data);
```

**5. TypeScript错误**:
```bash
# 类型检查
npx tsc --noEmit

# 查看推断的类型
// 鼠标悬停在变量上
```

### 8.5 性能优化建议

1. **使用动态导入**:
```typescript
const HeavyComponent = dynamic(() => import('./Heavy'), {
  loading: () => <Spinner />,
});
```

2. **优化图片**:
```typescript
import Image from 'next/image';

<Image src="/logo.png" width={200} height={200} alt="Logo" />
```

3. **React Query缓存**:
```typescript
staleTime: 1000 * 60 * 5, // 5分钟内不重新获取
```

4. **数据库索引**:
```prisma
@@index([userId])
```

5. **Prisma查询优化**:
```typescript
// 只查询需要的字段
select: { id: true, title: true }

// 并行查询
const [notes, chats] = await Promise.all([
  prisma.note.findMany(),
  prisma.chat.findMany(),
]);
```

---

## 9. 部署指南

### 9.1 部署平台选择

推荐 **Vercel**，原因：
- Next.js官方平台
- 零配置部署
- 自动HTTPS
- 全球CDN
- 免费额度充足

**其他选择**:
- Railway (简单易用)
- Fly.io (Docker部署)
- AWS/GCP (企业级)

### 9.2 Vercel部署步骤

**前置准备**:
1. GitHub仓库已创建
2. 代码已推送

**步骤**:

1. **连接GitHub**:
   - 登录 https://vercel.com
   - New Project
   - Import Git Repository
   - 选择项目

2. **配置环境变量**:

在Vercel项目设置中添加：
```
DATABASE_URL=postgresql://...?pgbouncer=true
DIRECT_URL=postgresql://...
BETTER_AUTH_SECRET=...
STRIPE_SECRET_KEY=sk_live_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PLUS_PLAN_ID=price_...
STRIPE_PREMIUM_PLAN_ID=price_...
GOOGLE_GENERATIVE_AI_API_KEY=...
TAVILY_API_KEY=...
NODE_ENV=production
NEXT_PUBLIC_APP_URL=https://your-domain.vercel.app
```

3. **部署**:
   - 点击 "Deploy"
   - 等待构建完成 (约2-3分钟)

4. **配置Stripe Webhook**:
   - Stripe Dashboard → Webhooks
   - Add endpoint: `https://your-domain.vercel.app/api/auth/webhook/stripe`
   - 选择事件:
     * checkout.session.completed
     * customer.subscription.created
     * customer.subscription.updated
   - 复制Signing secret → 更新 `STRIPE_WEBHOOK_SECRET`

5. **重新部署**:
   - Vercel会自动检测到新的环境变量
   - 或手动触发重新部署

### 9.3 自定义域名

1. Vercel项目 → Settings → Domains
2. 添加域名
3. 配置DNS记录 (Vercel会提供指引)
4. 等待SSL证书生成 (自动)
5. 更新 `NEXT_PUBLIC_APP_URL`

### 9.4 常见问题排查

**问题1**: 构建失败 - "Module not found"
```bash
# 解决：检查依赖是否在 package.json 中
npm install <missing-package> --save
```

**问题2**: 数据库连接超时
```bash
# 检查：
# 1. DATABASE_URL 是否使用连接池
# 2. DIRECT_URL 是否正确
# 3. 数据库防火墙是否允许Vercel IP
```

**问题3**: Stripe Webhook失败
```bash
# 检查：
# 1. Webhook URL是否正确
# 2. STRIPE_WEBHOOK_SECRET 是否匹配
# 3. 在Stripe Dashboard查看Webhook日志
```

**问题4**: 环境变量未生效
```bash
# 解决：
# 1. 确认已添加到Vercel
# 2. 重新部署 (环境变量更新需重新部署)
# 3. 检查变量名是否拼写正确
```

---

## 10. 附录

### 10.1 技术术语表

| 术语 | 简单解释 | 类比 |
|------|---------|------|
| **SSR** | Server-Side Rendering，服务器端渲染 | 就像餐厅厨房提前做好菜，端上桌就能吃 |
| **API** | Application Programming Interface，接口 | 就像餐厅菜单，你点菜，厨房做菜 |
| **ORM** | Object-Relational Mapping，对象关系映射 | 把数据库表翻译成JavaScript对象 |
| **Webhook** | 网络钩子 | 就像快递到了给你发短信通知 |
| **JWT** | JSON Web Token，令牌 | 就像演唱会门票，证明你有权进入 |
| **流式响应** | Streaming | 就像视频边下载边播放 |
| **中间件** | Middleware | 就像机场安检，所有人必须经过 |

### 10.2 常用命令速查

| 命令 | 用途 |
|------|------|
| `npm run dev` | 启动开发服务器 |
| `npm run build` | 构建生产版本 |
| `npm run start` | 启动生产服务器 |
| `npx prisma studio` | 打开数据库GUI |
| `npx prisma migrate dev` | 创建数据库迁移 |
| `npx prisma generate` | 生成Prisma客户端 |
| `npm run lint` | 代码检查 |

### 10.3 学习资源推荐

**官方文档**:
- Next.js: https://nextjs.org/docs
- Prisma: https://prisma.io/docs
- Hono: https://hono.dev
- AI SDK: https://sdk.vercel.ai/docs

**视频教程**:
- Next.js 15 教程: YouTube搜索
- Prisma入门: Prisma官网
- TypeScript基础: TypeScript官网

**中文资源**:
- 掘金: https://juejin.cn
- SegmentFault: https://segmentfault.com

### 10.4 常见问题FAQ

**Q: 如何修改AI模型？**
A: 前端聊天界面有模型选择器，或修改 `lib/ai/models.ts` 中的 `DEFAULT_MODEL_ID`

**Q: 如何添加新的AI工具？**
A:
1. 在 `lib/ai/tools/` 创建新工具文件
2. 在 `app/api/[[...route]]/chat.ts` 注册工具
3. 更新系统提示词

**Q: 如何修改订阅计划？**
A: 编辑 `lib/constant.ts` 中的 `PLANS` 数组

**Q: 如何导出用户数据？**
A: 在Prisma Studio中查询并导出，或创建新的API端点

**Q: 如何集成其他支付方式？**
A: BetterAuth支持多种支付，参考官方文档集成

---

## 📝 结语

本文档提供了Wave AI项目的完整技术规格。如有疑问，可参考：

- **分析文档**: `project-mastery/analysis/`
- **重建提示词**: `project-mastery/analysis/prompts-generated/`
- **教学指南**: `project-mastery/teaching-guide/` (如需生成)

**文档更新**: 随项目演进持续更新

**反馈**: 如发现文档错误或需要补充，请提交Issue

---

**© 2025 Wave AI Project Documentation**
