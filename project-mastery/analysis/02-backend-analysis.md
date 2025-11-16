# 阶段3：后端架构分析 / Phase 3: Backend Architecture Analysis

## 🏗️ 后端架构概览 / Backend Architecture Overview

### 架构模式 / Architecture Pattern
- **类型**: API-first + Server Actions
- **框架**: Hono (轻量级 Web 框架)
- **运行时**: Node.js
- **部署**: Serverless (Vercel Edge Functions)

### 技术栈 / Tech Stack
- **API 框架**: Hono 4.9.4
- **ORM**: Prisma 6.15.0
- **认证**: BetterAuth 1.3.7
- **AI SDK**: Vercel AI SDK 5.0.45
- **验证**: Zod 4.1.8
- **支付**: Stripe 18.5.0

---

## 📁 后端代码结构 / Backend Code Structure

```
app/
├── api/                          # API 路由
│   ├── [[...route]]/             # Hono Catch-all 路由
│   │   ├── route.ts              # 主路由文件 (导出 HTTP 方法)
│   │   ├── chat.ts               # 聊天 API 路由
│   │   ├── note.ts               # 笔记 API 路由
│   │   └── subscription.ts       # 订阅 API 路由
│   └── auth/[...all]/            # BetterAuth API 路由
│       └── route.ts              # 认证端点
│
├── actions/                      # Next.js Server Actions
│   └── action.ts                 # 服务器端操作
│
lib/
├── ai/                           # AI 相关逻辑
│   ├── tools/                    # AI 工具实现
│   │   ├── create-note.ts        # 创建笔记工具
│   │   ├── search-note.ts        # 搜索笔记工具
│   │   ├── web-search.ts         # 网络搜索工具
│   │   ├── extract-url.ts        # URL 提取工具
│   │   └── constant.ts           # 工具常量
│   ├── models.ts                 # AI 模型配置
│   ├── providers.ts              # AI 提供商
│   └── prompt.ts                 # 系统提示词
│
├── hono/                         # Hono 相关
│   ├── hono-middleware.ts        # 认证中间件
│   └── hono-rpc.ts               # RPC 类型定义
│
├── auth.ts                       # BetterAuth 配置
├── auth-client.ts                # Auth 客户端
├── prisma.ts                     # Prisma 客户端
├── stripe.ts                     # Stripe 客户端
├── constant.ts                   # 常量定义
└── utils.ts                      # 工具函数
```

---

## 🌐 API 端点完整列表 / Complete API Endpoints

### 认证 API / Authentication API

**BetterAuth 自动生成的端点** (通过 `/api/auth/*`):

| 端点 | 方法 | 说明 |
|------|------|------|
| `/api/auth/sign-in/email` | POST | 邮箱登录 |
| `/api/auth/sign-up/email` | POST | 邮箱注册 |
| `/api/auth/sign-out` | POST | 登出 |
| `/api/auth/get-session` | GET | 获取当前 Session |
| `/api/auth/webhook/stripe` | POST | Stripe Webhook 处理 |

**代码位置**: `app/api/auth/[...all]/route.ts` (由 BetterAuth 自动处理)

---

### 聊天 API / Chat API

**基础路径**: `/api/chat`

| 端点 | 方法 | 认证 | 请求体 | 响应 |
|------|------|------|--------|------|
| `/api/chat` | POST | ✓ | ChatSchema | Stream (SSE) |
| `/api/chat` | GET | ✓ | - | 聊天列表 |
| `/api/chat/:id` | GET | ✓ | - | 聊天详情 + 消息 |

**代码位置**: `app/api/[[...route]]/chat.ts:38-202`

#### 1. POST /api/chat - 发送消息
**用途**: 向 AI 发送消息，获取流式响应

**请求体 Schema**:
```typescript
{
  id: string;              // Chat ID (如果是新对话，前端生成)
  message: UIMessage;      // 用户消息
  selectedModelId: string; // AI 模型 ID
  selectedToolName: string | null; // 选中的工具名称
}
```

**请求示例**:
```json
{
  "id": "chat-uuid-123",
  "message": {
    "id": "msg-uuid-456",
    "role": "user",
    "parts": [
      { "type": "text", "text": "帮我创建一个关于 AI 的笔记" }
    ]
  },
  "selectedModelId": "anthropic/claude-sonnet-4",
  "selectedToolName": null
}
```

**处理流程**:
```
1. 验证用户认证 (getAuthUser 中间件)
   ↓
2. 检查生成限制 (checkGenerationLimit)
   - Free: 10/月
   - Plus: 300/月
   - Premium: 无限
   ↓
3. 查找或创建 Chat
   - 如果是新聊天: 使用 AI 生成标题
   ↓
4. 从数据库加载历史消息
   ↓
5. 保存用户消息到数据库
   ↓
6. 调用 AI streamText API
   - 传递系统提示词
   - 传递消息历史
   - 注册 AI 工具 (createNote, searchNote, webSearch, extractWebUrl)
   ↓
7. 流式返回 AI 响应
   ↓
8. onFinish 回调: 保存所有 AI 消息到数据库
```

**响应格式**: Server-Sent Events (SSE) 流

**代码位置**: `app/api/[[...route]]/chat.ts:39-147`

---

#### 2. GET /api/chat - 获取聊天列表
**用途**: 获取当前用户的所有聊天

**请求参数**: 无

**响应示例**:
```json
{
  "success": true,
  "data": [
    {
      "id": "chat-uuid-123",
      "title": "讨论 AI 工具",
      "userId": "user-uuid",
      "createdAt": "2025-11-16T00:00:00Z",
      "updatedAt": "2025-11-16T01:00:00Z"
    }
  ]
}
```

**代码位置**: `app/api/[[...route]]/chat.ts:148-163`

---

#### 3. GET /api/chat/:id - 获取聊天详情
**用途**: 获取特定聊天及其所有消息

**路径参数**: `id` (Chat ID)

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": "chat-uuid-123",
    "title": "讨论 AI 工具",
    "userId": "user-uuid",
    "createdAt": "2025-11-16T00:00:00Z",
    "updatedAt": "2025-11-16T01:00:00Z",
    "messages": [
      {
        "id": "msg-uuid-456",
        "role": "user",
        "parts": [
          { "type": "text", "text": "你好" }
        ],
        "metadata": { "createdAt": "2025-11-16T00:00:00Z" }
      },
      {
        "id": "msg-uuid-789",
        "role": "assistant",
        "parts": [
          { "type": "text", "text": "你好！我是 AI 助手。" }
        ],
        "metadata": { "createdAt": "2025-11-16T00:00:01Z" }
      }
    ]
  }
}
```

**代码位置**: `app/api/[[...route]]/chat.ts:164-202`

---

### 笔记 API / Notes API

**基础路径**: `/api/note`

| 端点 | 方法 | 认证 | 请求体 | 响应 |
|------|------|------|--------|------|
| `/api/note/create` | POST | ✓ | NoteSchema | 笔记对象 |
| `/api/note/update/:id` | PATCH | ✓ | NoteUpdateSchema | 更新后的笔记 |
| `/api/note/delete/:id` | DELETE | ✓ | - | 成功消息 |
| `/api/note/all` | GET | ✓ | Query Params | 分页笔记列表 |
| `/api/note/:id` | GET | ✓ | - | 笔记详情 |

**代码位置**: `app/api/[[...route]]/note.ts:19-168`

#### 1. POST /api/note/create - 创建笔记
**请求体 Schema**:
```typescript
{
  title: string;    // 最少 1 个字符
  content: string;  // 笔记内容
}
```

**响应示例**:
```json
{
  "success": true,
  "data": {
    "id": "note-uuid-123",
    "title": "AI 学习笔记",
    "content": "关于 AI 的内容...",
    "userId": "user-uuid",
    "createdAt": "2025-11-16T00:00:00Z",
    "updatedAt": "2025-11-16T00:00:00Z"
  }
}
```

**代码位置**: `app/api/[[...route]]/note.ts:20-40`

---

#### 2. PATCH /api/note/update/:id - 更新笔记
**路径参数**: `id` (Note ID)

**请求体 Schema** (部分更新):
```typescript
{
  title?: string;
  content?: string;
}
```

**安全检查**: 验证笔记所有权 (`userId === user.id`)

**代码位置**: `app/api/[[...route]]/note.ts:41-78`

---

#### 3. DELETE /api/note/delete/:id - 删除笔记
**路径参数**: `id` (Note ID)

**安全检查**: 验证笔记所有权

**响应示例**:
```json
{
  "success": true,
  "message": "Note deleted successfully"
}
```

**代码位置**: `app/api/[[...route]]/note.ts:79-110`

---

#### 4. GET /api/note/all - 获取笔记列表 (分页)
**查询参数**:
- `page`: 页码 (默认 1)
- `limit`: 每页数量 (默认 20)

**响应示例**:
```json
{
  "success": true,
  "data": [ /* 笔记数组 */ ],
  "pagination": {
    "total": 45,
    "page": 1,
    "limit": 20,
    "totalPages": 3,
    "skip": 0
  }
}
```

**代码位置**: `app/api/[[...route]]/note.ts:111-148`

---

#### 5. GET /api/note/:id - 获取笔记详情
**路径参数**: `id` (Note ID)

**安全检查**: 验证笔记所有权

**代码位置**: `app/api/[[...route]]/note.ts:149-168`

---

### 订阅 API / Subscription API

**基础路径**: `/api/subscription`

| 端点 | 方法 | 认证 | 请求体 | 响应 |
|------|------|------|--------|------|
| `/api/subscription/upgrade` | POST | ✓ | UpgradeSchema | Stripe Checkout URL |
| `/api/subscription/generations` | GET | ✓ | - | 生成限制信息 |

**代码位置**: `app/api/[[...route]]/subscription.ts:17-84`

#### 1. POST /api/subscription/upgrade - 升级订阅
**用途**: 创建 Stripe Checkout Session

**请求体 Schema**:
```typescript
{
  plan: "plus" | "premium";
  callbackUrl: string;
}
```

**请求示例**:
```json
{
  "plan": "plus",
  "callbackUrl": "https://app.example.com/billing"
}
```

**处理流程**:
```
1. 验证用户认证
   ↓
2. 查询现有订阅
   - 如果已是相同计划: 返回错误
   ↓
3. 调用 BetterAuth Stripe API
   - auth.api.upgradeSubscription()
   - 传递: plan, successUrl, cancelUrl
   - 如果有现有订阅: 传递 subscriptionId (用于升级/降级)
   ↓
4. 返回 Stripe Checkout URL
```

**响应示例**:
```json
{
  "success": true,
  "checkoutUrl": "https://checkout.stripe.com/pay/cs_test_..."
}
```

**代码位置**: `app/api/[[...route]]/subscription.ts:18-67`

---

#### 2. GET /api/subscription/generations - 获取生成限制
**用途**: 查询当前用户的 AI 生成使用情况

**响应示例**:
```json
{
  "success": true,
  "data": {
    "isAllowed": true,
    "hasPaidSubscription": false,
    "plan": "free",
    "generationsUsed": 3,
    "generationsLimit": 10,
    "remainingGenerations": 7
  }
}
```

**代码位置**: `app/api/[[...route]]/subscription.ts:68-84`

---

## 🔧 中间件系统 / Middleware System

### 1. 认证中间件 / Authentication Middleware

**文件位置**: `lib/hono/hono-middleware.ts:19-33`

**中间件名称**: `getAuthUser`

**功能**:
- 验证用户 Session
- 将用户信息注入到 Context
- 未认证时返回 401

**实现**:
```typescript
export const getAuthUser = createMiddleware<Env>(async (c, next) => {
  try {
    const session = await auth.api.getSession({
      headers: c.req.raw.headers,
    });
    if (!session) {
      throw new HTTPException(401, { message: "unauthorized" });
    }
    c.set("user", session.user);
    await next();
  } catch (error) {
    throw new HTTPException(401, { message: "unauthorized" });
  }
});
```

**使用示例**:
```typescript
app.post("/api/note/create", getAuthUser, async (c) => {
  const user = c.get("user"); // 获取认证用户
  // ...
});
```

---

### 2. 验证中间件 / Validation Middleware

**库**: `@hono/zod-validator`

**功能**: 使用 Zod Schema 验证请求数据

**使用示例**:
```typescript
const noteSchema = z.object({
  title: z.string().min(1),
  content: z.string(),
});

app.post(
  "/api/note/create",
  zValidator("json", noteSchema),
  getAuthUser,
  async (c) => {
    const { title, content } = c.req.valid("json");
    // 数据已验证，类型安全
  }
);
```

**支持的验证类型**:
- `json`: 请求体 JSON
- `param`: 路径参数
- `query`: 查询参数

---

### 3. 错误处理中间件 / Error Handling Middleware

**文件位置**: `app/api/[[...route]]/route.ts:13-20`

**功能**: 统一处理错误响应

**实现**:
```typescript
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse();
  }
  return c.json({
    error: "internal error",
  });
});
```

**错误类型**:
- `HTTPException`: Hono 自定义异常 (带状态码)
- 其他错误: 返回通用 500 错误

---

## ⚙️ 业务逻辑层 / Business Logic Layer

### 1. Server Actions (服务器操作)

**文件位置**: `app/actions/action.ts`

#### generateTitleForUserMessage
**用途**: 为新聊天生成标题

**代码位置**: `app/actions/action.ts:8-28`

**实现**:
```typescript
export async function generateTitleForUserMessage({
  message,
}: {
  message: UIMessage;
}) {
  try {
    const { text } = await generateText({
      model: myProvider.languageModel("title-model"),
      system: `
        - you will generate a short title based on the first message
        - ensure it is not more than 80 characters long
        - the title should be a summary of the user's message
        - do not use quotes or colons`,
      prompt: JSON.stringify(message),
    });
    return text;
  } catch (error) {
    return "Untitled";
  }
}
```

**调用位置**: `app/api/[[...route]]/chat.ts:57-59`

---

#### createDefaultSubscription
**用途**: 用户注册时创建默认订阅

**代码位置**: `app/actions/action.ts:30-64`

**实现逻辑**:
```
1. 检查是否已有订阅
2. 如果有: 返回现有订阅
3. 如果没有: 创建 Free 计划订阅
   - plan = "free"
   - status = "active"
   - stripeCustomerId = 传入的 Stripe 客户 ID
```

**调用位置**: `lib/auth.ts:27-30` (BetterAuth Stripe 插件的 `onCustomerCreate` 回调)

---

#### checkGenerationLimit
**用途**: 检查用户的 AI 生成限制

**代码位置**: `app/actions/action.ts:66-116`

**实现逻辑**:
```
1. 查询用户的活跃订阅
2. 根据订阅计划获取限制
   - Free: 10 次/月
   - Plus: 300 次/月
   - Premium: 无限
3. 统计当前计费周期内的 AI 消息数量
   - 只统计 role = "assistant" 的消息
   - 时间范围: periodStart ~ periodEnd
4. 计算是否允许生成
5. 返回详细信息
```

**返回数据**:
```typescript
{
  isAllowed: boolean;
  hasPaidSubscription: boolean;
  plan: string;
  generationsUsed: number;
  generationsLimit: number | null;
  remainingGenerations: number | "Unlimited";
}
```

**调用位置**:
- `app/api/[[...route]]/chat.ts:45`
- `app/api/[[...route]]/subscription.ts:71`

---

### 2. AI 工具系统 / AI Tools System

项目实现了 4 个 AI 工具，允许 AI 主动调用来完成任务。

#### 工具注册位置
**代码位置**: `app/api/[[...route]]/chat.ts:106-111`

```typescript
const result = streamText({
  model: modelProvider,
  system: getSystemPrompt(selectedToolName),
  messages: modelMessages,
  tools: {
    createNote: createNote(user.id),
    searchNote: searchNote(user.id),
    webSearch: webSearch(),
    extractWebUrl: extractWebUrl(),
  },
  toolChoice: "auto",
});
```

---

#### 工具 1: createNote - 创建笔记

**文件位置**: `lib/ai/tools/create-note.ts:5-39`

**描述**: "Create a note or Save to Note with title and content. Use this when the user asks to create, save, or make a note."

**输入 Schema**:
```typescript
{
  title: string;    // 笔记标题
  content: string;  // 笔记内容
}
```

**执行逻辑**:
```typescript
execute: async ({ title, content }) => {
  const note = await prisma.note.create({
    data: {
      userId,
      title: title,
      content: content,
    },
  });
  return {
    success: true,
    message: `Note "${title}" created successfully`,
    noteId: note.id,
    title: note.title,
    content: note.content,
  };
}
```

**AI 调用示例**:
```
用户: "帮我保存一个关于 React Hooks 的笔记"
AI 思考: 用户想创建笔记 → 调用 createNote 工具
工具执行: { title: "React Hooks", content: "..." }
AI 响应: "✓ 已为您创建笔记 'React Hooks'"
```

---

#### 工具 2: searchNote - 搜索笔记

**文件位置**: `lib/ai/tools/search-note.ts:5-56`

**描述**: "Search through the user's notes by keywords in title or content. Use this when the user asks to find or search or lookup notes."

**输入 Schema**:
```typescript
{
  query: string;       // 搜索关键词
  limit?: number;      // 最大结果数 (默认 10)
}
```

**执行逻辑**:
```typescript
execute: async ({ query, limit = 10 }) => {
  const notes = await prisma.note.findMany({
    where: {
      userId,
      OR: [
        { title: { contains: query, mode: "insensitive" } },
        { content: { contains: query, mode: "insensitive" } }
      ],
    },
    orderBy: { createdAt: "desc" },
    take: limit,
  });
  return {
    success: true,
    message: `Found ${notes.length} notes matching "${query}"`,
    notes: notes,
  };
}
```

**搜索特性**:
- 不区分大小写 (`mode: "insensitive"`)
- 同时搜索标题和内容
- 按创建时间倒序

---

#### 工具 3: webSearch - 网络搜索

**文件位置**: `lib/ai/tools/web-search.ts:8-45`

**描述**: "Search the web for current information. Use when you need up-to-date info or when user asks to search the internet."

**外部服务**: Tavily API

**输入 Schema**:
```typescript
{
  query: string;  // 搜索查询
}
```

**执行逻辑**:
```typescript
execute: async ({ query }) => {
  const response = await tvly.search(query, {
    includeAnswer: true,      // 包含 AI 总结
    includeFavicon: true,     // 包含网站图标
    includeImages: false,     // 不包含图片
    maxResults: 3,            // 最多 3 个结果
  });

  const results = response.results.map((r) => ({
    title: r.title,
    url: r.url,
    content: r.content,
    favicon: r.favicon,
  }));

  return {
    success: true,
    answer: response.answer,  // Tavily AI 生成的答案
    results: results,
    response_time: response.responseTime,
  };
}
```

**返回示例**:
```json
{
  "success": true,
  "answer": "Claude Sonnet 4 是 Anthropic 发布的最新模型...",
  "results": [
    {
      "title": "Anthropic 官网",
      "url": "https://anthropic.com",
      "content": "摘要内容...",
      "favicon": "https://..."
    }
  ],
  "response_time": 1.2
}
```

---

#### 工具 4: extractWebUrl - 提取 URL 内容

**文件位置**: `lib/ai/tools/extract-url.ts:8-45`

**描述**: "Extract content from one or more URLs. Use this to retrieve, summarize, or analyze page content."

**外部服务**: Tavily Extract API

**输入 Schema**:
```typescript
{
  urls: string[];  // URL 数组
}
```

**执行逻辑**:
```typescript
execute: async ({ urls }) => {
  const response = await tvly.extract(urls, {
    includeFavicon: true,
    includeImages: false,
    topic: "general",
    format: "markdown",       // Markdown 格式
    extractDepth: "basic",
  });

  const results = response.results.map((r) => ({
    url: r.url,
    content: r.rawContent,    // Markdown 内容
    favicon: r.favicon,
  }));

  return {
    success: true,
    urls: urls,
    results: results,
    response_time: response.responseTime,
  };
}
```

**使用场景**:
```
用户: "总结这篇文章 https://example.com/article"
AI: 调用 extractWebUrl → 获取文章内容 → 总结
AI: "文章主要讲述了..."
```

---

## 🔐 认证与授权 / Authentication & Authorization

### BetterAuth 配置

**文件位置**: `lib/auth.ts:12-38`

**配置详解**:
```typescript
export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 4,
  },
  plugins: [
    openAPI(),        // 提供 OpenAPI 文档
    bearer(),         // Bearer Token 支持
    stripe({
      stripeClient,
      stripeWebhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
      createCustomerOnSignUp: true,
      onCustomerCreate: async ({ stripeCustomer, user }) => {
        const userId = user.id;
        const stripeCustomerId = stripeCustomer.id;
        await createDefaultSubscription(userId, stripeCustomerId);
      },
      subscription: {
        enabled: true,
        plans: PLANS,
      },
    }),
  ],
});
```

### 认证流程 / Authentication Flow

#### 注册流程
```
1. 前端提交注册表单
   ↓
2. POST /api/auth/sign-up/email
   - email
   - password
   - name
   ↓
3. BetterAuth 创建 User 记录
   ↓
4. BetterAuth Stripe 插件:
   - 在 Stripe 创建 Customer
   - user.stripeCustomerId = 设置
   ↓
5. onCustomerCreate 回调:
   - 调用 createDefaultSubscription()
   - 创建 Free 计划订阅
   ↓
6. 创建 Account 记录 (password hash)
   ↓
7. 创建 Session
   ↓
8. 返回 Session Token (存储在 Cookie)
```

#### 登录流程
```
1. 前端提交登录表单
   ↓
2. POST /api/auth/sign-in/email
   - email
   - password
   ↓
3. BetterAuth 验证密码
   ↓
4. 创建 Session
   ↓
5. 返回 Session Token (存储在 Cookie)
```

#### Session 验证
```
1. API 请求携带 Cookie
   ↓
2. getAuthUser 中间件执行
   ↓
3. auth.api.getSession({ headers })
   - 从 Cookie 提取 token
   - 查询 Session 表
   - 检查是否过期
   ↓
4. 如果有效: 返回 session.user
5. 如果无效: 抛出 401 错误
```

---

## 💳 支付集成 / Payment Integration

### Stripe 配置

**文件位置**: `lib/stripe.ts`

```typescript
import Stripe from "stripe";

export const stripeClient = new Stripe(
  process.env.STRIPE_SECRET_KEY!,
  {
    apiVersion: "...",
  }
);
```

### 订阅升级流程 / Subscription Upgrade Flow

```
1. 用户点击"升级到 Plus"
   ↓
2. POST /api/subscription/upgrade
   { plan: "plus", callbackUrl: "..." }
   ↓
3. 调用 BetterAuth Stripe API
   auth.api.upgradeSubscription({
     plan: "plus",
     successUrl: "...",
     cancelUrl: "...",
   })
   ↓
4. BetterAuth 内部:
   - 创建 Stripe Checkout Session
   - 关联 Customer (user.stripeCustomerId)
   - 设置 Price (STRIPE_PLUS_PLAN_ID)
   ↓
5. 返回 Checkout URL
   ↓
6. 前端重定向到 Stripe Checkout 页面
   ↓
7. 用户完成支付
   ↓
8. Stripe 发送 Webhook 到 /api/auth/webhook/stripe
   - Event: checkout.session.completed
   - Event: customer.subscription.created
   ↓
9. BetterAuth 处理 Webhook:
   - 更新 Subscription 记录
   - plan = "plus"
   - status = "active"
   - stripeSubscriptionId = sub_xxx
   - periodStart, periodEnd = 设置
```

---

## 🤖 AI 集成架构 / AI Integration Architecture

### AI 提供商系统

**文件位置**: `lib/ai/providers.ts:11-28`

**架构设计**:
```typescript
const createLanguageModels = (isProduction: boolean) => {
  const models: Record<string, any> = {};

  // 生产环境: 使用 AI Gateway
  chatModels.forEach((model) =>
    models[model.id] = gateway.languageModel(model.id)
  );

  // 开发环境: 直接使用 Google Gemini
  models[DEVELOPMENT_CHAT_MODEL] = google.languageModel(DEVELOPMENT_CHAT_MODEL);

  // Title 生成模型
  models["title-model"] = isProduction
    ? gateway.languageModel("google/gemini-2.0-flash")
    : google.languageModel(DEVELOPMENT_CHAT_MODEL);

  return models;
};

export const myProvider = customProvider({
  languageModels: createLanguageModels(isProduction),
});
```

**优势**:
1. **统一接口**: 通过 `myProvider.languageModel(id)` 调用任意模型
2. **环境隔离**: 开发环境使用免费的 Gemini，生产环境使用 Gateway
3. **灵活切换**: 支持 Claude、Grok、GPT-4、Gemini 等多个模型

---

### 系统提示词系统

**文件位置**: `lib/ai/prompt.ts`

**功能**: 根据选择的工具生成不同的系统提示词

**使用位置**: `app/api/[[...route]]/chat.ts:103`

---

### AI 流式响应 / Streaming Response

**技术**: Server-Sent Events (SSE)

**实现位置**: `app/api/[[...route]]/chat.ts:101-140`

```typescript
const result = streamText({
  model: modelProvider,
  system: getSystemPrompt(selectedToolName),
  messages: modelMessages,
  stopWhen: stepCountIs(5),  // 最多 5 步工具调用
  tools: { /* ... */ },
  toolChoice: "auto",
  onError: (error) => {
    console.log("Streaming error", error);
  },
});

return result.toUIMessageStreamResponse({
  sendSources: true,
  generateMessageId: () => generateUUID(),
  onFinish: async ({ messages, responseMessage }) => {
    // 保存所有消息到数据库
    await prisma.message.createMany({
      data: messages.map((m) => ({ /* ... */ })),
      skipDuplicates: true,
    });
  },
});
```

**流程**:
```
1. AI 开始生成响应
   ↓
2. 每生成一段文本 → 通过 SSE 发送到前端
   ↓
3. 如果需要调用工具:
   - 发送 tool-call 消息
   - 执行工具
   - 发送 tool-result 消息
   - AI 继续基于工具结果生成
   ↓
4. 生成完成 → onFinish 回调
   - 保存所有消息到数据库
```

---

## 🔄 数据访问模式 / Data Access Pattern

### Prisma 客户端

**文件位置**: `lib/prisma.ts`

```typescript
import { PrismaClient } from "@/generated/prisma";

const prisma = new PrismaClient();

export default prisma;
```

**使用位置**:
- 所有 API 路由
- AI 工具
- Server Actions

---

### 常见查询模式 / Common Query Patterns

#### 1. 带关系查询
```typescript
const chat = await prisma.chat.findFirst({
  where: { id, userId: user.id },
  include: {
    messages: {
      orderBy: { createdAt: "asc" },
    },
  },
});
```

#### 2. 分页查询
```typescript
const [notes, total] = await Promise.all([
  prisma.note.findMany({
    where: { userId: user.id },
    orderBy: { createdAt: "desc" },
    skip: (page - 1) * limit,
    take: limit,
  }),
  prisma.note.count({ where: { userId: user.id } }),
]);
```

#### 3. 条件查询
```typescript
const notes = await prisma.note.findMany({
  where: {
    userId,
    OR: [
      { title: { contains: query, mode: "insensitive" } },
      { content: { contains: query, mode: "insensitive" } }
    ],
  },
});
```

#### 4. 聚合查询
```typescript
const generationCount = await prisma.message.count({
  where: {
    chat: { userId },
    role: "assistant",
    createdAt: {
      gte: periodStart,
      lte: periodEnd,
    },
  },
});
```

---

## 🚨 错误处理模式 / Error Handling Pattern

### 统一错误处理

```typescript
try {
  // 业务逻辑
} catch (error) {
  if (error instanceof HTTPException) {
    throw error;  // 重新抛出 HTTP 异常
  }
  throw new HTTPException(500, { message: "Internal server error" });
}
```

### 常见错误类型

| 状态码 | 场景 | 消息 |
|--------|------|------|
| 401 | 未认证 | "unauthorized" |
| 403 | 超过限制 | "Generation limit reached" |
| 404 | 资源不存在 | "Note not found" / "Chat not found" |
| 400 | 业务错误 | "Already on this plan" |
| 500 | 服务器错误 | "Internal server error" |

---

## 📊 API 响应格式 / API Response Format

### 成功响应
```json
{
  "success": true,
  "data": { /* 数据对象 */ }
}
```

### 分页响应
```json
{
  "success": true,
  "data": [ /* 数据数组 */ ],
  "pagination": {
    "total": 100,
    "page": 1,
    "limit": 20,
    "totalPages": 5,
    "skip": 0
  }
}
```

### 错误响应
```json
{
  "message": "Error message"
}
```

---

## 🔍 后端性能优化 / Backend Performance Optimization

### 1. 数据库查询优化
- 使用索引 (userId, chatId)
- 并行查询 (`Promise.all`)
- 分页加载

### 2. AI 响应优化
- 流式响应 (减少感知延迟)
- 限制工具调用步数 (`stepCountIs(5)`)
- 批量保存消息 (`createMany`)

### 3. Stripe 集成优化
- 异步处理 Webhook
- 幂等性检查 (`skipDuplicates`)

---

## 🎯 后端架构亮点 / Backend Architecture Highlights

### 1. Hono 框架选择
- 轻量级 (< 14KB)
- 性能优异
- 类型安全的 RPC
- 完美适配 Vercel Edge

### 2. AI SDK 集成
- 统一的多模型接口
- 流式响应支持
- 工具调用机制
- 自动消息管理

### 3. BetterAuth + Stripe
- 开箱即用的认证
- 内置 Stripe 集成
- Webhook 自动处理
- Session 管理

### 4. 类型安全
- Zod Schema 验证
- Prisma 类型生成
- TypeScript 全栈类型共享

---

## ✅ 阶段3完成标记 / Phase 3 Completion

✓ API 端点完整列表（表格形式）
✓ 请求/响应示例
✓ 认证流程图
✓ 中间件执行流程
✓ 外部服务集成说明
✓ 错误处理模式
✓ AI 工具系统详解
✓ 数据访问模式

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 前端架构分析 (`03-frontend-analysis.md`)
