# 后端API重建提示词 / Backend API Reconstruction Prompts

## 📋 使用说明 / Instructions

这些提示词按顺序执行，将帮助你重建完整的后端API系统。每个提示词都是独立的，可以直接复制给AI使用。

---

## 提示词 2.1: 配置认证系统 / Setup Authentication System

```
我需要使用 BetterAuth 配置认证系统。请帮我创建以下内容：

1. 在 lib/auth.ts 文件中配置 BetterAuth：
   - 使用 Prisma Adapter (PostgreSQL)
   - 启用 Email/Password 认证（最小密码长度 4）
   - 集成 Stripe 插件：
     * 注册时自动创建 Stripe 客户
     * 配置 Webhook 密钥
     * 启用订阅功能
   - 添加 OpenAPI 和 Bearer Token 插件

2. 在 lib/auth-client.ts 创建客户端：
   - 使用 better-auth/react 的 createAuthClient
   - 配置 baseURL 为 NEXT_PUBLIC_APP_URL

3. 创建 app/api/auth/[...all]/route.ts：
   - 导出 auth.handler() 的 GET 和 POST 方法

参考数据库模型：
- User: id, name, email, emailVerified, stripeCustomerId
- Session: id, token, expiresAt, userId
- Account: id, accountId, providerId, userId, password

环境变量需要：
- BETTER_AUTH_SECRET
- STRIPE_SECRET_KEY
- STRIPE_WEBHOOK_SECRET
- NEXT_PUBLIC_APP_URL
```

**预期结果**:
- `lib/auth.ts` 完整配置
- `lib/auth-client.ts` 客户端配置
- `app/api/auth/[...all]/route.ts` API 路由

**验证方式**:
```bash
# 启动开发服务器后，访问：
curl http://localhost:3000/api/auth/get-session
# 应返回 401 或 session 数据
```

---

## 提示词 2.2: 创建 Hono API 基础框架 / Create Hono API Foundation

```
我需要使用 Hono 创建 API 框架。请帮我创建：

1. 在 app/api/[[...route]]/route.ts：
   - 创建 Hono app 实例
   - 配置 CORS 中间件（允许所有来源）
   - 添加错误处理中间件
   - 导出 GET、POST、PATCH、DELETE 方法
   - 导出 AppType 类型供 RPC 使用

2. 在 lib/hono/hono-middleware.ts：
   - 创建 getAuthUser 中间件
   - 使用 auth.api.getSession() 验证用户
   - 未认证时抛出 401 HTTPException
   - 认证成功时设置 c.set("user", session.user)

3. 在 lib/hono/hono-rpc.ts：
   - 使用 hono/client 的 hc 创建类型安全的 RPC 客户端
   - 导出 api 对象

错误处理示例：
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse();
  }
  return c.json({ error: "internal error" });
});
```

**预期结果**:
- Hono 应用配置完成
- 认证中间件可用
- RPC 客户端配置完成

**验证方式**:
```typescript
// 在前端测试：
import { api } from "@/lib/hono/hono-rpc";
const response = await api.note.all.$get();
```

---

## 提示词 2.3: 实现笔记 API / Implement Notes API

```
基于以下需求实现完整的笔记 API (app/api/[[...route]]/note.ts):

**数据模型**:
- Note: id (uuid), title, content, userId, createdAt, updatedAt

**API 端点**:

1. POST /api/note/create
   - 认证: 必需 (getAuthUser)
   - 请求体验证 (zValidator):
     * title: string (最少1个字符)
     * content: string
   - 创建笔记，自动关联当前用户
   - 返回: { success: true, data: note }

2. PATCH /api/note/update/:id
   - 认证: 必需
   - 请求体验证:
     * title?: string
     * content?: string
   - 安全检查: 验证笔记所有权 (note.userId === user.id)
   - 返回: { success: true, data: updatedNote }

3. DELETE /api/note/delete/:id
   - 认证: 必需
   - 安全检查: 验证笔记所有权
   - 返回: { success: true, message: "Note deleted successfully" }

4. GET /api/note/all
   - 认证: 必需
   - 查询参数: page (默认1), limit (默认20)
   - 实现分页查询
   - 返回: {
       success: true,
       data: notes[],
       pagination: { total, page, limit, totalPages, skip }
     }

5. GET /api/note/:id
   - 认证: 必需
   - 安全检查: 验证笔记所有权
   - 返回: { success: true, data: note }

**注册路由**:
将所有端点注册到主 Hono app (route.ts)

使用 Prisma 客户端: import prisma from "@/lib/prisma";
```

**预期结果**: 完整的笔记 CRUD API

**测试示例**:
```bash
# 创建笔记
curl -X POST http://localhost:3000/api/note/create \
  -H "Cookie: session=..." \
  -d '{"title":"Test","content":"Content"}'

# 获取所有笔记
curl http://localhost:3000/api/note/all?page=1&limit=10 \
  -H "Cookie: session=..."
```

---

## 提示词 2.4: 实现聊天 API / Implement Chat API

```
基于以下需求实现聊天 API (app/api/[[...route]]/chat.ts):

**数据模型**:
- Chat: id (uuid), title, userId, createdAt, updatedAt
- Message: id (uuid), role (enum), parts (JSON), chatId, createdAt

**API 端点**:

1. POST /api/chat - 发送消息并获取 AI 响应
   - 认证: 必需
   - 请求体验证 (zValidator):
     * id: string (chat ID)
     * message: UIMessage
     * selectedModelId: string
     * selectedToolName: string | null

   - 流程:
     a. 调用 checkGenerationLimit() 检查限制
     b. 查找或创建 Chat
     c. 如果是新对话，调用 generateTitleForUserMessage()
     d. 加载历史消息 (include messages)
     e. 保存用户消息到数据库
     f. 调用 AI streamText API:
        - model: myProvider.languageModel(selectedModelId)
        - system: getSystemPrompt(selectedToolName)
        - messages: 转换后的历史消息
        - tools: { createNote, searchNote, webSearch, extractWebUrl }
        - stopWhen: stepCountIs(5)
     g. 返回流式响应: result.toUIMessageStreamResponse()
     h. onFinish 回调: 保存所有 AI 消息

2. GET /api/chat - 获取所有聊天
   - 认证: 必需
   - 查询当前用户的所有聊天
   - 按 updatedAt 倒序
   - 返回: { success: true, data: chats[] }

3. GET /api/chat/:id - 获取聊天详情
   - 认证: 必需
   - include messages (按 createdAt 升序)
   - 安全检查: chat.userId === user.id
   - 返回: { success: true, data: { ...chat, messages } }

**辅助函数导入**:
- import { generateTitleForUserMessage, checkGenerationLimit } from "@/app/actions/action";
- import { myProvider } from "@/lib/ai/providers";
- import { getSystemPrompt } from "@/lib/ai/prompt";
- import { createNote, searchNote, webSearch, extractWebUrl } from "@/lib/ai/tools/*";
```

**预期结果**: 完整的聊天 API，支持流式 AI 响应

**验证方式**:
```bash
# 前端使用 useChat hook 测试
# 或使用 curl 测试非流式端点
curl http://localhost:3000/api/chat \
  -H "Cookie: session=..."
```

---

## 提示词 2.5: 实现订阅 API / Implement Subscription API

```
基于以下需求实现订阅 API (app/api/[[...route]]/subscription.ts):

**数据模型**:
- Subscription: id, plan (enum: free|plus|premium), referenceId (userId),
  stripeCustomerId, stripeSubscriptionId, status, periodStart, periodEnd

**订阅计划** (lib/constant.ts):
```typescript
export const PLANS = [
  {
    id: "free",
    name: "Free",
    stripePriceId: null,
    monthlyGenerations: 10,
  },
  {
    id: "plus",
    name: "Plus",
    stripePriceId: process.env.STRIPE_PLUS_PLAN_ID!,
    monthlyGenerations: 300,
  },
  {
    id: "premium",
    name: "Premium",
    stripePriceId: process.env.STRIPE_PREMIUM_PLAN_ID!,
    monthlyGenerations: null, // 无限
  },
];
```

**API 端点**:

1. POST /api/subscription/upgrade
   - 认证: 必需
   - 请求体验证:
     * plan: "plus" | "premium"
     * callbackUrl: string

   - 流程:
     a. 查询用户当前订阅
     b. 如果已是相同计划，返回 400 错误
     c. 调用 auth.api.upgradeSubscription({
          plan: json.plan,
          successUrl: json.callbackUrl,
          cancelUrl: json.callbackUrl,
          subscriptionId: 现有订阅ID (如果有)
        })
     d. 返回: { success: true, checkoutUrl: response.url }

2. GET /api/subscription/generations
   - 认证: 必需
   - 调用 checkGenerationLimit() 获取使用情况
   - 返回: {
       success: true,
       data: {
         isAllowed,
         hasPaidSubscription,
         plan,
         generationsUsed,
         generationsLimit,
         remainingGenerations
       }
     }

**BetterAuth Stripe 集成**:
已在 lib/auth.ts 配置，Webhook 自动处理订阅状态同步。
```

**预期结果**: 完整的订阅管理 API

**测试示例**:
```bash
# 检查生成限制
curl http://localhost:3000/api/subscription/generations \
  -H "Cookie: session=..."

# 创建升级 Checkout Session
curl -X POST http://localhost:3000/api/subscription/upgrade \
  -H "Cookie: session=..." \
  -d '{"plan":"plus","callbackUrl":"http://localhost:3000/billing"}'
```

---

## 提示词 2.6: 实现 AI 工具系统 / Implement AI Tools

```
创建 4 个 AI 工具，让 AI 可以调用来完成任务：

**1. 创建笔记工具** (lib/ai/tools/create-note.ts):
```typescript
import { tool } from "ai";
import { z } from "zod";
import prisma from "@/lib/prisma";

export const createNote = (userId: string) => tool({
  description: "Create a note or Save to Note with title and content. Use this when the user asks to create, save, or make a note.",
  parameters: z.object({
    title: z.string().describe("Note title"),
    content: z.string().describe("Note content"),
  }),
  execute: async ({ title, content }) => {
    const note = await prisma.note.create({
      data: { userId, title, content },
    });
    return {
      success: true,
      message: `Note "${title}" created successfully`,
      noteId: note.id,
      title: note.title,
      content: note.content,
    };
  },
});
```

**2. 搜索笔记工具** (lib/ai/tools/search-note.ts):
```typescript
export const searchNote = (userId: string) => tool({
  description: "Search through the user's notes by keywords in title or content. Use this when the user asks to find or search or lookup notes.",
  parameters: z.object({
    query: z.string().describe("Search keywords"),
    limit: z.number().default(10).describe("Maximum number of results"),
  }),
  execute: async ({ query, limit }) => {
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
      notes,
    };
  },
});
```

**3. 网络搜索工具** (lib/ai/tools/web-search.ts):
```typescript
import { tvly } from "tavily";

export const webSearch = () => tool({
  description: "Search the web for current information. Use when you need up-to-date info or when user asks to search the internet.",
  parameters: z.object({
    query: z.string().describe("Search query"),
  }),
  execute: async ({ query }) => {
    const response = await tvly.search(query, {
      includeAnswer: true,
      includeFavicon: true,
      includeImages: false,
      maxResults: 3,
    });

    const results = response.results.map((r) => ({
      title: r.title,
      url: r.url,
      content: r.content,
      favicon: r.favicon,
    }));

    return {
      success: true,
      answer: response.answer,
      results,
      response_time: response.responseTime,
    };
  },
});
```

**4. 提取 URL 内容工具** (lib/ai/tools/extract-url.ts):
```typescript
export const extractWebUrl = () => tool({
  description: "Extract content from one or more URLs. Use this to retrieve, summarize, or analyze page content.",
  parameters: z.object({
    urls: z.array(z.string()).describe("Array of URLs to extract"),
  }),
  execute: async ({ urls }) => {
    const response = await tvly.extract(urls, {
      includeFavicon: true,
      includeImages: false,
      topic: "general",
      format: "markdown",
      extractDepth: "basic",
    });

    const results = response.results.map((r) => ({
      url: r.url,
      content: r.rawContent,
      favicon: r.favicon,
    }));

    return {
      success: true,
      urls,
      results,
      response_time: response.responseTime,
    };
  },
});
```

**工具常量** (lib/ai/tools/constant.ts):
```typescript
export enum ToolNameEnum {
  CreateNote = "createNote",
  SearchNote = "searchNote",
  WebSearch = "webSearch",
  ExtractWebUrl = "extractWebUrl",
}
```

环境变量需要:
- TAVILY_API_KEY
```

**预期结果**: 4 个可用的 AI 工具

**验证方式**: 在聊天中测试工具调用
- "帮我创建一个笔记" → 触发 createNote
- "搜索我的笔记" → 触发 searchNote
- "搜索最新新闻" → 触发 webSearch
- "总结这个网址" → 触发 extractWebUrl

---

## 提示词 2.7: 配置 AI 提供商 / Configure AI Providers

```
配置 AI 模型提供商系统，支持多个 AI 模型：

**1. AI 模型配置** (lib/ai/models.ts):
```typescript
export interface ChatModel {
  id: string;
  name: string;
  provider: string;
}

export const chatModels: ChatModel[] = [
  {
    id: "anthropic/claude-sonnet-4",
    name: "Claude Sonnet 4",
    provider: "Anthropic",
  },
  {
    id: "x-ai/grok-4",
    name: "Grok 4",
    provider: "xAI",
  },
  {
    id: "openai/gpt-4.1",
    name: "GPT-4.1",
    provider: "OpenAI",
  },
  {
    id: "google/gemini-2.5-flash",
    name: "Gemini 2.5 Flash",
    provider: "Google",
  },
];

export const DEFAULT_MODEL_ID = chatModels[0].id;
export const DEVELOPMENT_CHAT_MODEL = "google/gemini-2.0-flash-thinking-exp";
```

**2. AI 提供商配置** (lib/ai/providers.ts):
```typescript
import { customProvider } from "ai";
import { aiGateway } from "@ai-sdk/gateway";
import { google } from "@ai-sdk/google";

const isProduction = process.env.NODE_ENV === "production";

const gateway = aiGateway({
  url: process.env.AI_GATEWAY_URL!,
});

const createLanguageModels = (isProduction: boolean) => {
  const models: Record<string, any> = {};

  // 生产环境: 使用 AI Gateway
  if (isProduction) {
    chatModels.forEach((model) => {
      models[model.id] = gateway.languageModel(model.id);
    });
  }

  // 开发环境: 使用 Google Gemini
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

**3. 系统提示词** (lib/ai/prompt.ts):
创建 getSystemPrompt(toolName: string | null) 函数，根据选择的工具返回不同的系统提示词。

环境变量需要:
- AI_GATEWAY_URL (生产环境)
- GOOGLE_GENERATIVE_AI_API_KEY
- ANTHROPIC_API_KEY (可选)
- OPENAI_API_KEY (可选)
- XAI_API_KEY (可选)
```

**预期结果**:
- 多模型支持配置完成
- 开发/生产环境自动切换

---

## 提示词 2.8: 实现 Server Actions / Implement Server Actions

```
创建 Server Actions 提供服务器端逻辑 (app/actions/action.ts):

**1. 生成聊天标题**:
```typescript
"use server";

import { generateText } from "ai";
import { myProvider } from "@/lib/ai/providers";
import type { UIMessage } from "@/types";

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

**2. 创建默认订阅**:
```typescript
export async function createDefaultSubscription(
  userId: string,
  stripeCustomerId: string
) {
  const existingSubscription = await prisma.subscription.findFirst({
    where: { referenceId: userId },
  });

  if (existingSubscription) {
    return existingSubscription;
  }

  return await prisma.subscription.create({
    data: {
      referenceId: userId,
      stripeCustomerId,
      plan: "free",
      status: "active",
    },
  });
}
```

**3. 检查生成限制**:
```typescript
import { PLANS } from "@/lib/constant";

export async function checkGenerationLimit(userId: string) {
  // 查询活跃订阅
  const subscription = await prisma.subscription.findFirst({
    where: {
      referenceId: userId,
      status: "active",
    },
  });

  const plan = PLANS.find((p) => p.id === (subscription?.plan || "free"))!;
  const limit = plan.monthlyGenerations;

  // 统计当前计费周期的生成次数
  const periodStart = subscription?.periodStart || new Date();
  const periodEnd = subscription?.periodEnd || new Date();

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

  const isAllowed = limit === null || generationCount < limit;

  return {
    isAllowed,
    hasPaidSubscription: subscription?.plan !== "free",
    plan: subscription?.plan || "free",
    generationsUsed: generationCount,
    generationsLimit: limit,
    remainingGenerations: limit === null
      ? "Unlimited"
      : Math.max(0, limit - generationCount),
  };
}
```

**在 BetterAuth 配置中使用**:
在 lib/auth.ts 的 Stripe 插件配置中添加：
```typescript
onCustomerCreate: async ({ stripeCustomer, user }) => {
  await createDefaultSubscription(user.id, stripeCustomer.id);
}
```
```

**预期结果**: 3 个 Server Actions 可用

**测试方式**:
- 注册新用户 → 自动创建订阅
- 新建聊天 → 自动生成标题
- 发送消息 → 自动检查限制

---

## ✅ 后端完成检查清单 / Backend Completion Checklist

完成所有提示词后，验证以下功能：

### 认证系统
- [ ] 用户可以注册 (Email + Password)
- [ ] 用户可以登录
- [ ] Session 正确存储和验证
- [ ] 注册时自动创建 Stripe 客户

### API 端点
- [ ] 笔记 CRUD (创建、读取、更新、删除) 正常工作
- [ ] 聊天列表和详情查询正常
- [ ] 流式 AI 响应正常
- [ ] 订阅升级跳转到 Stripe Checkout

### AI 工具
- [ ] 创建笔记工具可以被 AI 调用
- [ ] 搜索笔记工具返回正确结果
- [ ] 网络搜索工具返回实时信息
- [ ] URL 提取工具返回网页内容

### 订阅系统
- [ ] 新用户默认为 Free 计划
- [ ] 生成限制正确计算
- [ ] Stripe Webhook 同步订阅状态

### 数据库
- [ ] 所有关系正确建立
- [ ] 级联删除正常工作
- [ ] 数据隔离 (用户只能访问自己的数据)

---

## 🔗 相关文档 / Related Documentation

- **后端架构分析**: `analysis/02-backend-analysis.md`
- **数据库分析**: `analysis/01-database-analysis.md`
- **基础架构提示词**: `prompts-generated/01-foundation-prompts.md`

---

**生成时间**: 2025-11-16
**版本**: v1.0
**下一步**: 前端组件重建提示词 (`03-frontend-prompts.md`)
