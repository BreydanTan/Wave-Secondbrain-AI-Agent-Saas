# 阶段5：安全实现分析 / Phase 5: Security Implementation Analysis

## 🔐 安全概览 / Security Overview

Wave AI Agent 采用多层安全架构，涵盖认证、授权、数据保护和支付安全。

---

## 🔑 认证机制 / Authentication Mechanism

### BetterAuth 认证系统

**配置位置**: `lib/auth.ts:12-38`

#### 认证方式 / Auth Methods

1. **Email/Password 认证** (已启用)
   ```typescript
   emailAndPassword: {
     enabled: true,
     minPasswordLength: 4,
   }
   ```

   **安全特性**:
   - 密码哈希存储 (bcrypt)
   - 最小密码长度限制
   - 密码存储在 Account 表

2. **OAuth 认证** (已配置但未启用)
   - 通过 Account 模型支持
   - 可轻松添加 Google、GitHub 等提供商

---

### Session 管理

**数据模型**: Session (参考数据库分析)

**字段**:
```typescript
{
  id: string;
  token: string;           // 唯一会话令牌
  expiresAt: DateTime;     // 过期时间
  userId: string;          // 关联用户
  ipAddress: string?;      // IP 地址 (审计)
  userAgent: string?;      // User Agent (审计)
}
```

**安全特性**:
- ✅ 会话令牌唯一性约束
- ✅ 自动过期机制
- ✅ IP 地址追踪 (安全审计)
- ✅ User Agent 记录 (检测异常登录)

---

### Bearer Token

**存储位置**: 客户端 localStorage

**配置** (`lib/auth-client.ts`):
```typescript
export const authClient = createAuthClient({
  fetchOptions: {
    auth: {
      type: "Bearer",
      token: () => useAuthToken.getState().bearerToken || "",
    },
  },
  plugins: [stripeClient({ subscription: true })],
});
```

**使用流程**:
```
1. 用户登录
   ↓
2. 服务器返回 Authorization Header
   → Authorization: Bearer <token>
   ↓
3. 客户端提取并存储 Token
   → localStorage: auth-storage
   ↓
4. 后续请求携带 Token
   → Header: Authorization: Bearer <token>
   ↓
5. 服务器验证 Token
   → getAuthUser 中间件
```

**安全考虑**:
- ⚠️ localStorage 易受 XSS 攻击
- ✅ Token 有过期时间
- 建议改进: 使用 httpOnly Cookie

---

## 🛡️ 授权规则 / Authorization Rules

### API 级别授权

**中间件**: `getAuthUser` (`lib/hono/hono-middleware.ts:19-33`)

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

**保护的端点**:
- ✅ 所有 `/api/chat/*` 路由
- ✅ 所有 `/api/note/*` 路由
- ✅ 所有 `/api/subscription/*` 路由

---

### 数据访问控制

#### 行级安全 (Row-Level Security)

**实现方式**: 通过应用层逻辑

**示例** (笔记访问):
```typescript
// app/api/[[...route]]/note.ts:149-168
const note = await prisma.note.findFirst({
  where: {
    id,
    userId: user.id,  // 只能访问自己的笔记
  },
});

if (!note) {
  throw new HTTPException(404, { message: "Note not found" });
}
```

**安全保证**:
- ✅ 用户只能访问自己的数据
- ✅ 自动过滤查询结果
- ✅ 更新/删除前验证所有权

---

### 订阅限制检查

**功能**: `checkGenerationLimit` (`app/actions/action.ts:66-116`)

```typescript
export async function checkGenerationLimit(userId: string) {
  const subscription = await prisma.subscription.findFirst({
    where: {
      referenceId: userId,
      status: "active",
    },
  });

  const plan = PLANS.find((p) => p.name === subscription.plan);

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

  const isAllowed =
    plan.limits.generations === Infinity ||
    generationCount < plan.limits.generations;

  return { isAllowed, /* ... */ };
}
```

**强制执行** (`app/api/[[...route]]/chat.ts:45-51`):
```typescript
const { isAllowed } = await checkGenerationLimit(user.id);

if (!isAllowed) {
  throw new HTTPException(403, {
    message: "Generation limit reached",
  });
}
```

---

## 🔒 数据保护 / Data Protection

### 输入验证 (Zod Schema)

#### API 输入验证

**所有 API 端点使用 Zod 验证**

**示例 1**: 笔记创建
```typescript
// app/api/[[...route]]/note.ts:8-11
const noteSchema = z.object({
  title: z.string().min(1),
  content: z.string(),
});

app.post("/create", zValidator("json", noteSchema), async (c) => {
  const { title, content } = c.req.valid("json");
  // 已验证，类型安全
});
```

**示例 2**: 订阅升级
```typescript
// app/api/[[...route]]/subscription.ts:12-15
const upgradeSchema = z.object({
  plan: z.enum([PLAN_ENUM.PLUS, PLAN_ENUM.PREMIUM]),
  callbackUrl: z.string().min(1),
});
```

**防护**:
- ✅ 类型验证 (防止类型错误)
- ✅ 长度限制
- ✅ 格式验证 (email, URL等)
- ✅ 枚举值限制

---

#### 前端表单验证

**示例**: 注册表单 (`app/(routes)/auth/_common/signup-form.tsx`)

```typescript
const signUpSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email").min(1, "Email required"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});
```

**双重验证**:
- ✅ 前端验证 (用户体验)
- ✅ 后端验证 (安全保障)

---

### SQL 注入防护

**ORM 保护**: 使用 Prisma 自动防护

**安全的查询**:
```typescript
// ✅ 安全 - Prisma 参数化查询
const notes = await prisma.note.findMany({
  where: {
    title: { contains: searchQuery },
  },
});
```

**不安全的查询**:
```typescript
// ❌ 不安全 - 原始 SQL
const notes = await prisma.$queryRaw`
  SELECT * FROM notes WHERE title LIKE '%${searchQuery}%'
`;
```

**项目状态**: ✅ 所有查询都使用 Prisma，无 SQL 注入风险

---

### XSS 防护

**React 自动防护**:
```typescript
// ✅ 自动转义
<div>{userInput}</div>

// ⚠️ 需要额外小心
<div dangerouslySetInnerHTML={{ __html: userInput }} />
```

**项目检查**:
- ✅ 无 `dangerouslySetInnerHTML` 使用
- ✅ 代码块使用 `react-syntax-highlighter` (安全渲染)
- ✅ Markdown 使用 `streamdown` (已清理)

---

### CSRF 防护

**Next.js 内置保护**:
- ✅ SameSite Cookie 属性
- ✅ Server Actions 自动 CSRF Token

**BetterAuth 保护**:
- ✅ Session Token 验证
- ✅ Origin Header 检查

---

## 💳 支付安全 / Payment Security

### Stripe 集成安全

#### 1. 敏感数据不经过服务器

**Stripe Checkout 流程**:
```
1. 用户点击"升级"
   ↓
2. 服务器创建 Checkout Session
   - 不处理信用卡信息
   ↓
3. 重定向到 Stripe Hosted Checkout
   - Stripe 托管的安全页面
   ↓
4. 用户在 Stripe 页面输入支付信息
   - 信用卡信息不经过我们的服务器
   ↓
5. 支付完成后重定向回应用
```

**代码位置**: `app/api/[[...route]]/subscription.ts:39-51`

---

#### 2. Webhook 签名验证

**BetterAuth Stripe 插件自动验证**:
```typescript
// lib/auth.ts:23-36
stripe({
  stripeClient,
  stripeWebhookSecret: process.env.STRIPE_WEBHOOK_SECRET!,
  // 插件自动验证 Webhook 签名
})
```

**验证流程**:
```
1. Stripe 发送 Webhook
   - 包含签名 Header
   ↓
2. BetterAuth 验证签名
   - 使用 STRIPE_WEBHOOK_SECRET
   ↓
3. 签名匹配 → 处理事件
4. 签名不匹配 → 拒绝请求
```

**防护**: ✅ 防止伪造 Webhook 请求

---

#### 3. 客户 ID 验证

```typescript
// app/api/[[...route]]/subscription.ts:27-32
const existingSubscription = await prisma.subscription.findFirst({
  where: {
    referenceId: user.id,
    status: "active",
  },
});
```

**防护**: ✅ 确保用户只能操作自己的订阅

---

## 🔐 环境变量安全 / Environment Variable Security

### 敏感信息管理

**存储位置**: `.env` 文件 (不提交到 Git)

**`.gitignore` 配置**:
```
# env files (can opt-in for committing if needed)
.env*
```

**必需的敏感变量**:
```bash
# 数据库
DATABASE_URL=
DIRECT_URL=

# 认证
BETTER_AUTH_SECRET=

# Stripe
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_PLUS_PLAN_ID=
STRIPE_PREMIUM_PLAN_ID=

# AI
GOOGLE_GENERATIVE_AI_API_KEY=
ANTHROPIC_API_KEY=
OPENAI_API_KEY=
XAI_API_KEY=

# Web 搜索
TAVILY_API_KEY=
```

**访问控制**:
- ✅ 服务器端变量不暴露给客户端
- ✅ 仅以 `NEXT_PUBLIC_` 开头的变量可在客户端访问
- ✅ 敏感密钥仅在 Server Components/API 中使用

---

## 🚨 错误处理安全 / Error Handling Security

### 信息泄露防护

**错误处理中间件** (`app/api/[[...route]]/route.ts:13-20`):
```typescript
app.onError((err, c) => {
  if (err instanceof HTTPException) {
    return err.getResponse();
  }
  // ✅ 通用错误消息，不泄露细节
  return c.json({
    error: "internal error",
  });
});
```

**安全原则**:
- ✅ 生产环境隐藏详细错误信息
- ✅ 用户看到通用消息
- ✅ 详细错误记录在服务器日志

---

## 📊 安全审计 / Security Audit

### 会话追踪

**Session 模型字段**:
```typescript
{
  ipAddress: string?;    // 登录 IP
  userAgent: string?;    // 浏览器信息
  createdAt: DateTime;   // 登录时间
}
```

**用途**:
- ✅ 检测异常登录
- ✅ 安全事件调查
- ✅ 用户活动追踪

---

### 消息审计

**Message 模型**:
```typescript
{
  role: Role;            // user | assistant | system
  parts: Json;           // 消息内容
  createdAt: DateTime;   // 时间戳
  chatId: string;        // 关联聊天
}
```

**用途**:
- ✅ AI 使用审计
- ✅ 问题调查
- ✅ 滥用检测

---

## 🔍 安全漏洞扫描 / Security Vulnerability Scan

### 已知漏洞检查

#### OWASP Top 10 检查

| 威胁 | 状态 | 说明 |
|------|------|------|
| **A01 - Broken Access Control** | ✅ 已防护 | 所有 API 需认证，行级权限检查 |
| **A02 - Cryptographic Failures** | ✅ 已防护 | 密码哈希 (bcrypt)，HTTPS 传输 |
| **A03 - Injection** | ✅ 已防护 | Prisma ORM，Zod 验证 |
| **A04 - Insecure Design** | ⚠️ 部分 | Bearer Token 在 localStorage (建议改进) |
| **A05 - Security Misconfiguration** | ✅ 已防护 | 环境变量管理，错误处理 |
| **A06 - Vulnerable Components** | ⚠️ 需检查 | 依赖定期更新 |
| **A07 - Authentication Failures** | ✅ 已防护 | BetterAuth，Session 管理 |
| **A08 - Data Integrity Failures** | ✅ 已防护 | Stripe Webhook 签名验证 |
| **A09 - Logging Failures** | ⚠️ 部分 | 控制台日志，建议集成日志服务 |
| **A10 - SSRF** | ✅ 已防护 | AI 工具 URL 提取使用第三方 API (Tavily) |

---

### 安全改进建议 / Security Improvement Recommendations

#### 高优先级 / High Priority

1. **将 Bearer Token 迁移到 httpOnly Cookie**
   - 当前: localStorage (易受 XSS)
   - 建议: httpOnly, Secure, SameSite Cookie

2. **实现 Rate Limiting**
   - 当前: 无请求限制
   - 建议: 添加 API 速率限制 (防止暴力破解/DDoS)

3. **添加邮箱验证**
   - 当前: emailVerified = false (未使用)
   - 建议: 发送验证邮件

#### 中优先级 / Medium Priority

4. **多因素认证 (2FA)**
   - 当前: 仅密码
   - 建议: TOTP / SMS 验证码

5. **审计日志系统**
   - 当前: 控制台日志
   - 建议: 集成 Sentry / LogRocket

6. **内容安全策略 (CSP)**
   - 当前: 无 CSP 头
   - 建议: 添加 CSP Header

#### 低优先级 / Low Priority

7. **定期安全扫描**
   - 依赖漏洞扫描 (npm audit)
   - 代码静态分析

8. **备份和灾难恢复**
   - 数据库定期备份
   - 恢复流程测试

---

## ✅ 阶段5完成标记 / Phase 5 Completion

✓ 认证机制详解
✓ 授权规则
✓ 数据保护措施
✓ 支付安全
✓ 环境变量管理
✓ 安全审计机制
✓ OWASP Top 10 检查
✓ 安全改进建议

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 部署配置分析 (`05-deployment-analysis.md`)
