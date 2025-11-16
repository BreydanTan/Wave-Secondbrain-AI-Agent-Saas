# 阶段2：数据库深度分析 / Phase 2: Database Deep Dive

## 📊 数据库概览 / Database Overview

### 数据库类型 / Database Type
- **数据库**: PostgreSQL
- **ORM**: Prisma 6.15.0
- **客户端位置**: `generated/prisma/`
- **Schema 文件**: `prisma/schema.prisma:1`

### 数据库连接配置 / Database Connection
```prisma
datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")    // 连接池 URL
  directUrl = env("DIRECT_URL")      // 直连 URL (迁移用)
}
```

**说明**:
- `DATABASE_URL`: 用于应用查询，通常使用连接池 (如 PgBouncer、Prisma Accelerate)
- `DIRECT_URL`: 用于数据库迁移，需要直连数据库

---

## 🗂️ 数据模型清单 / Data Models Inventory

项目共有 **6 个主要数据模型** 和 **2 个枚举类型**:

### 主要模型 / Main Models
1. **User** - 用户
2. **Session** - 会话
3. **Account** - 账户 (第三方登录)
4. **Note** - 笔记
5. **Chat** - 聊天
6. **Message** - 消息
7. **Subscription** - 订阅

### 枚举类型 / Enums
1. **Role** - 消息角色 (user, assistant, system)
2. **Plan** - 订阅计划 (free, plus, premium)

---

## 📋 详细模型分析 / Detailed Model Analysis

### 1. User Model (用户模型)

**文件位置**: `prisma/schema.prisma:20-38`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | - | 主键，用户唯一标识 |
| `name` | String | ✓ | - | 用户名 |
| `email` | String | ✓ | - | 邮箱 (唯一) |
| `emailVerified` | Boolean | ✓ | false | 邮箱验证状态 |
| `stripeCustomerId` | String? | ✗ | null | Stripe 客户 ID |
| `image` | String? | ✗ | null | 用户头像 URL |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 (自动) |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 说明 |
|---------|---------|---------|------|
| `sessions` | 1:N | Session | 用户的所有会话 |
| `accounts` | 1:N | Account | 用户的所有账户 (OAuth) |
| `notes` | 1:N | Note | 用户创建的所有笔记 |
| `Chat` | 1:N | Chat | 用户的所有聊天 |
| `subscriptions` | 1:N | Subscription | 用户的订阅记录 |

#### 约束 / Constraints
- **唯一约束**: `email` (每个邮箱只能注册一次)
- **表名映射**: `@map("user")`

#### 业务逻辑说明 / Business Logic
- `id` 由 BetterAuth 生成，不是自增 ID
- `stripeCustomerId` 在用户注册时通过 BetterAuth Stripe 插件自动创建
- `emailVerified` 用于邮箱验证流程 (当前默认 false)
- 用户删除时会级联删除所有关联数据 (通过 `onDelete: Cascade`)

---

### 2. Session Model (会话模型)

**文件位置**: `prisma/schema.prisma:40-53`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | - | 主键，会话 ID |
| `expiresAt` | DateTime | ✓ | - | 过期时间 |
| `token` | String | ✓ | - | 会话 Token (唯一) |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 (自动) |
| `ipAddress` | String? | ✗ | null | IP 地址 |
| `userAgent` | String? | ✗ | null | 浏览器 User Agent |
| `userId` | String | ✓ | - | 外键 → User.id |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `user` | N:1 | User | ✓ |

#### 约束 / Constraints
- **唯一约束**: `token`
- **表名映射**: `@map("session")`

#### 业务逻辑说明 / Business Logic
- BetterAuth 使用 Session 管理用户登录状态
- `token` 存储在客户端 Cookie 中
- `expiresAt` 用于自动清理过期 Session
- `ipAddress` 和 `userAgent` 用于安全审计

---

### 3. Account Model (账户模型)

**文件位置**: `prisma/schema.prisma:55-72`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | - | 主键 |
| `accountId` | String | ✓ | - | OAuth 提供商账户 ID |
| `providerId` | String | ✓ | - | OAuth 提供商 (google, github 等) |
| `userId` | String | ✓ | - | 外键 → User.id |
| `accessToken` | String? | ✗ | null | OAuth Access Token |
| `refreshToken` | String? | ✗ | null | OAuth Refresh Token |
| `idToken` | String? | ✗ | null | OAuth ID Token |
| `accessTokenExpiresAt` | DateTime? | ✗ | null | Access Token 过期时间 |
| `refreshTokenExpiresAt` | DateTime? | ✗ | null | Refresh Token 过期时间 |
| `scope` | String? | ✗ | null | OAuth Scope |
| `password` | String? | ✗ | null | 密码哈希 (Email/Password 认证) |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `user` | N:1 | User | ✓ |

#### 约束 / Constraints
- **表名映射**: `@map("account")`

#### 业务逻辑说明 / Business Logic
- 支持多种认证方式：
  - Email/Password: `password` 字段存储哈希密码
  - OAuth: `accessToken`, `refreshToken` 等字段存储 OAuth 信息
- 一个用户可以有多个 Account (例如同时绑定 Google 和 GitHub)
- 当前项目只启用了 Email/Password 认证

---

### 4. Note Model (笔记模型)

**文件位置**: `prisma/schema.prisma:74-87`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | uuid() | 主键，UUID 格式 |
| `title` | String | ✓ | - | 笔记标题 |
| `content` | String | ✓ | - | 笔记内容 (Markdown) |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 (自动) |
| `userId` | String | ✓ | - | 外键 → User.id |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `user` | N:1 | User | ✓ |

#### 索引 / Indexes
- **索引**: `userId` (加速查询用户的所有笔记)

#### 约束 / Constraints
- **表名映射**: `@map("notes")`

#### 业务逻辑说明 / Business Logic
- `id` 使用 UUID 自动生成
- `content` 存储 Markdown 格式内容
- AI 工具可以创建和搜索笔记
- Free 计划对笔记数量有限制

---

### 5. Chat Model (聊天模型)

**文件位置**: `prisma/schema.prisma:89-101`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | uuid() | 主键，UUID 格式 |
| `title` | String | ✓ | - | 聊天标题 (AI 自动生成) |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 (自动) |
| `userId` | String | ✓ | - | 外键 → User.id |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `user` | N:1 | User | ✓ |
| `messages` | 1:N | Message | ✓ |

#### 约束 / Constraints
- **表名映射**: `@map("chats")`

#### 业务逻辑说明 / Business Logic
- 每次新对话创建一个 Chat
- `title` 由 AI 根据第一轮对话自动生成
- 聊天删除时会级联删除所有消息

---

### 6. Message Model (消息模型)

**文件位置**: `prisma/schema.prisma:110-122`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | uuid() | 主键，UUID 格式 |
| `role` | Role | ✓ | - | 消息角色 (枚举) |
| `parts` | Json | ✓ | - | 消息内容 (JSON 格式) |
| `createdAt` | DateTime | ✓ | now() | 创建时间 |
| `updatedAt` | DateTime | ✓ | now() | 更新时间 (自动) |
| `chatId` | String | ✓ | - | 外键 → Chat.id |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `chat` | N:1 | Chat | ✓ |

#### 索引 / Indexes
- **索引**: `chatId` (加速查询聊天的所有消息)

#### 约束 / Constraints
- **表名映射**: `@map("messages")`

#### 业务逻辑说明 / Business Logic
- `role` 可能值: `user`, `assistant`, `system`
- `parts` 存储消息的多模态内容:
  - 文本: `{ type: "text", text: "..." }`
  - 工具调用: `{ type: "tool-call", toolName: "...", args: {...} }`
  - 工具结果: `{ type: "tool-result", result: {...} }`
- 使用 JSON 类型允许灵活存储不同类型的消息内容

---

### 7. Subscription Model (订阅模型)

**文件位置**: `prisma/schema.prisma:130-146`

#### 字段定义 / Field Definitions

| 字段名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| `id` | String | ✓ | uuid() | 主键，UUID 格式 |
| `plan` | Plan | ✓ | free | 订阅计划 (枚举) |
| `referenceId` | String | ✓ | - | 引用 ID (User.id) |
| `stripeCustomerId` | String? | ✗ | null | Stripe 客户 ID |
| `stripeSubscriptionId` | String? | ✗ | null | Stripe 订阅 ID |
| `status` | String | ✓ | - | 订阅状态 |
| `periodStart` | DateTime? | ✗ | null | 计费周期开始 |
| `periodEnd` | DateTime? | ✗ | null | 计费周期结束 |
| `cancelAtPeriodEnd` | Boolean? | ✗ | false | 周期结束时取消 |
| `seats` | Int? | ✗ | null | 席位数 (团队版) |
| `trialStart` | DateTime? | ✗ | null | 试用开始时间 |
| `trialEnd` | DateTime? | ✗ | null | 试用结束时间 |

#### 关系 / Relations

| 关系字段 | 关系类型 | 目标模型 | 级联删除 |
|---------|---------|---------|---------|
| `user` | N:1 | User | ✓ |

#### 约束 / Constraints
- **表名映射**: `@map("subscriptions")`

#### 业务逻辑说明 / Business Logic
- 每个用户注册时自动创建一个 Free 计划订阅
- `status` 可能值: `active`, `canceled`, `past_due`, `trialing` 等
- Stripe Webhook 同步订阅状态到数据库
- `referenceId` 指向 User.id，支持未来扩展为组织订阅

---

## 🔗 实体关系图 / Entity Relationship Diagram (ER Diagram)

```
┌─────────────────────────────────────────────────────────────────┐
│                           User                                  │
│─────────────────────────────────────────────────────────────────│
│ 🔑 id: String (PK)                                              │
│    name: String                                                 │
│    email: String (UNIQUE)                                       │
│    emailVerified: Boolean = false                               │
│    stripeCustomerId: String?                                    │
│    image: String?                                               │
│    createdAt: DateTime = now()                                  │
│    updatedAt: DateTime = now()                                  │
└──────────────┬──────────────┬──────────────┬──────────────┬─────┘
               │              │              │              │
               │ 1            │ 1            │ 1            │ 1
               │              │              │              │
               │ N            │ N            │ N            │ N
               │              │              │              │
     ┌─────────▼─────┐ ┌──────▼──────┐ ┌────▼────┐ ┌──────▼──────┐
     │   Session     │ │   Account   │ │  Note   │ │    Chat     │
     │───────────────│ │─────────────│ │─────────│ │─────────────│
     │🔑 id: String  │ │🔑 id: String│ │🔑 id    │ │🔑 id: String│
     │   token: Str  │ │  accountId  │ │  title  │ │   title     │
     │   expiresAt   │ │  providerId │ │  content│ │   userId ──┐│
     │   ipAddress   │ │  password?  │ │  userId │ │   createdAt││
     │   userAgent   │ │  accessToken│ │createdAt│ │   updatedAt││
     │   userId ────┐│ │  userId ───┐│ │updatedAt│ └─────┬───────┘│
     │   createdAt  ││ │  createdAt ││ └─────────┘       │        │
     │   updatedAt  ││ │  updatedAt ││                   │ 1      │
     └──────────────┘│ └────────────┘│                   │        │
                     │                │                   │ N      │
                     └────────────────┴───────────────────┘        │
                                                           ┌────────▼─────┐
                                                           │   Message    │
                                                           │──────────────│
                                                           │🔑 id: String │
                                                           │   role: Role │
                                                           │   parts: JSON│
                                                           │   chatId ────┘
                                                           │   createdAt  │
                                                           │   updatedAt  │
                                                           └──────────────┘

     ┌───────────────────────────┐
     │      Subscription         │
     │───────────────────────────│
     │🔑 id: String              │
     │   plan: Plan = free       │
     │   referenceId → User.id   │
     │   stripeCustomerId        │
     │   stripeSubscriptionId    │
     │   status: String          │
     │   periodStart             │
     │   periodEnd               │
     │   cancelAtPeriodEnd       │
     │   seats                   │
     │   trialStart              │
     │   trialEnd                │
     └───────────────────────────┘

┌──────────────┐         ┌──────────────────┐
│  Enum: Role  │         │   Enum: Plan     │
│──────────────│         │──────────────────│
│ • user       │         │ • free           │
│ • assistant  │         │ • plus           │
│ • system     │         │ • premium        │
└──────────────┘         └──────────────────┘

图例 / Legend:
🔑 = 主键 (Primary Key)
→ = 外键 (Foreign Key)
1:N = 一对多关系 (One-to-Many)
UNIQUE = 唯一约束
```

---

## 📐 数据库关系详解 / Database Relationships Explained

### 1. User → Sessions (1:N)
- **关系**: 一个用户可以有多个会话 (多设备登录)
- **级联删除**: ✓ (删除用户时删除所有会话)
- **外键**: Session.userId → User.id

### 2. User → Accounts (1:N)
- **关系**: 一个用户可以有多个认证账户 (Email + OAuth)
- **级联删除**: ✓
- **外键**: Account.userId → User.id
- **说明**: 当前只使用 Email/Password，但支持未来添加 OAuth

### 3. User → Notes (1:N)
- **关系**: 一个用户可以创建多个笔记
- **级联删除**: ✓
- **外键**: Note.userId → User.id
- **索引**: userId (优化查询性能)

### 4. User → Chats (1:N)
- **关系**: 一个用户可以有多个聊天会话
- **级联删除**: ✓
- **外键**: Chat.userId → User.id

### 5. Chat → Messages (1:N)
- **关系**: 一个聊天包含多条消息
- **级联删除**: ✓ (删除聊天时删除所有消息)
- **外键**: Message.chatId → Chat.id
- **索引**: chatId (优化查询性能)

### 6. User → Subscriptions (1:N)
- **关系**: 一个用户可以有多个订阅记录 (历史记录)
- **级联删除**: ✓
- **外键**: Subscription.referenceId → User.id
- **说明**: 实际业务中通常只有一个活跃订阅

---

## 💾 数据流分析 / Data Flow Analysis

### 用户注册流程 / User Registration Flow

```
1. 用户提交注册表单
   ↓
2. BetterAuth 创建 User 记录
   - 生成 user.id
   - 设置 emailVerified = false
   ↓
3. BetterAuth Stripe 插件触发
   - 在 Stripe 创建 Customer
   - 设置 user.stripeCustomerId
   ↓
4. onCustomerCreate 回调执行
   - 调用 createDefaultSubscription()
   - 创建 Subscription 记录 (plan = free)
   ↓
5. 创建 Account 记录
   - providerId = "credential"
   - password = bcrypt hash
   ↓
6. 创建 Session 记录
   - 生成 session.token
   - 设置 expiresAt
   ↓
7. 返回 session.token 到客户端
```

**代码位置**:
- User 创建: `lib/auth.ts:12-38`
- Subscription 创建: `app/actions/action.ts` (createDefaultSubscription)

---

### 创建笔记流程 / Create Note Flow

```
1. 用户发送消息到 AI
   ↓
2. AI 决定调用 create_note 工具
   ↓
3. 后端执行工具函数
   - 验证用户认证
   - 检查订阅计划限制
   ↓
4. 创建 Note 记录
   - id = uuid()
   - userId = 当前用户 ID
   - title, content = AI 提供的参数
   ↓
5. 返回笔记信息给 AI
   ↓
6. AI 告知用户笔记已创建
```

**代码位置**:
- 工具定义: `lib/ai/tools/create-note.ts:1`
- API 端点: `app/api/[[...route]]/note.ts:1`

---

### 聊天消息流程 / Chat Message Flow

```
1. 用户在前端发送消息
   ↓
2. 检查是否有现有 Chat
   - 如果没有: 创建新 Chat (title = "New Chat")
   - 如果有: 使用现有 Chat
   ↓
3. 创建 Message 记录 (role = user)
   - chatId = 当前 Chat ID
   - parts = [{ type: "text", text: "用户消息" }]
   ↓
4. 调用 AI API (流式响应)
   - 传递消息历史
   - 传递可用工具
   ↓
5. AI 生成响应
   - 如果调用工具: 创建 tool-call Message
   - 如果返回文本: 创建 assistant Message
   ↓
6. 如果是第一轮对话:
   - 使用 title-model 生成 Chat.title
   - 更新 Chat 记录
   ↓
7. 流式返回响应到前端
```

**代码位置**:
- 聊天 API: `app/api/[[...route]]/chat.ts:1`
- 前端聊天组件: `components/chat/*`

---

### 订阅升级流程 / Subscription Upgrade Flow

```
1. 用户选择 Plus/Premium 计划
   ↓
2. 创建 Stripe Checkout Session
   - customer = user.stripeCustomerId
   - price = plan.priceId
   ↓
3. 用户在 Stripe 完成支付
   ↓
4. Stripe 发送 Webhook 事件
   - checkout.session.completed
   - customer.subscription.created
   ↓
5. BetterAuth Stripe 插件处理 Webhook
   ↓
6. 更新 Subscription 记录
   - plan = plus/premium
   - status = active
   - stripeSubscriptionId = sub_xxx
   - periodStart, periodEnd = 计费周期
   ↓
7. 用户获得新计划权限
```

**代码位置**:
- Stripe 配置: `lib/auth.ts:23-36`
- Subscription API: `app/api/[[...route]]/subscription.ts:1`

---

## 🔍 常用查询示例 / Common Query Examples

### 1. 获取用户的所有笔记 / Get User's Notes

```typescript
// lib/prisma.ts 或任何服务层
const notes = await prisma.note.findMany({
  where: {
    userId: "user-id-here"
  },
  orderBy: {
    updatedAt: 'desc'
  }
});
```

### 2. 获取聊天及其所有消息 / Get Chat with Messages

```typescript
const chatWithMessages = await prisma.chat.findUnique({
  where: {
    id: "chat-id-here"
  },
  include: {
    messages: {
      orderBy: {
        createdAt: 'asc'
      }
    }
  }
});
```

### 3. 检查用户订阅状态 / Check User Subscription

```typescript
const subscription = await prisma.subscription.findFirst({
  where: {
    referenceId: userId,
    status: 'active'
  }
});

const plan = subscription?.plan || 'free';
const canGenerate = subscription?.limits?.generations > usedGenerations;
```

### 4. 创建聊天和首条消息 / Create Chat with First Message

```typescript
const chat = await prisma.chat.create({
  data: {
    title: "New Chat",
    userId: userId,
    messages: {
      create: {
        role: 'user',
        parts: [{ type: "text", text: "Hello!" }]
      }
    }
  },
  include: {
    messages: true
  }
});
```

### 5. 搜索笔记 / Search Notes

```typescript
const notes = await prisma.note.findMany({
  where: {
    userId: userId,
    OR: [
      { title: { contains: searchQuery, mode: 'insensitive' } },
      { content: { contains: searchQuery, mode: 'insensitive' } }
    ]
  }
});
```

**代码位置**: `lib/ai/tools/search-note.ts:1`

---

## 📊 数据完整性约束 / Data Integrity Constraints

### 主键 / Primary Keys
- 所有模型都有 `id` 主键
- User, Session, Account: String (BetterAuth 生成)
- Note, Chat, Message, Subscription: String (UUID)

### 外键 / Foreign Keys
- 所有关系都通过外键强制引用完整性
- 使用 `onDelete: Cascade` 级联删除

### 唯一约束 / Unique Constraints
- User.email - 防止重复注册
- Session.token - 确保 Token 唯一性

### 索引 / Indexes
- Note.userId - 优化用户笔记查询
- Message.chatId - 优化聊天消息查询

### 默认值 / Default Values
- 时间戳: `createdAt`, `updatedAt`
- 布尔值: `emailVerified = false`, `cancelAtPeriodEnd = false`
- 枚举: `plan = free`

---

## 🔄 数据库迁移 / Database Migrations

### 迁移文件位置 / Migration Files
**目录**: `prisma/migrations/`

### 迁移命令 / Migration Commands

```bash
# 创建新迁移
npm run db:migrate

# 等价于
npx prisma migrate dev --name init

# 应用迁移到生产环境
npx prisma migrate deploy

# 重置数据库 (开发环境)
npx prisma migrate reset
```

### 迁移流程 / Migration Process
1. 修改 `schema.prisma`
2. 运行 `npm run db:migrate`
3. Prisma 生成 SQL 迁移文件
4. Prisma 应用迁移到数据库
5. Prisma 重新生成客户端

---

## 💡 数据库设计亮点 / Database Design Highlights

### 1. 灵活的认证系统
- 通过 Account 模型支持多种认证方式
- 可以轻松添加 OAuth 提供商

### 2. 多模态消息存储
- 使用 JSON 类型存储 Message.parts
- 支持文本、工具调用、工具结果等多种消息类型

### 3. 订阅系统设计
- 分离用户和订阅 (referenceId)
- 支持未来的组织/团队订阅
- 完整的 Stripe 集成字段

### 4. 性能优化
- 在高频查询字段添加索引 (userId, chatId)
- 使用 Prisma Accelerate 连接池

### 5. 数据隔离
- 所有用户数据通过 userId 隔离
- 级联删除确保数据一致性

---

## ⚠️ 潜在改进建议 / Potential Improvements

### 1. 全文搜索
当前笔记搜索使用 `contains`，对于大量数据性能不佳。
建议使用 PostgreSQL 全文搜索或集成 Elasticsearch。

### 2. 软删除
当前使用硬删除 (Cascade)，无法恢复。
建议添加 `deletedAt` 字段实现软删除。

### 3. 审计日志
缺少数据变更审计。
建议添加 AuditLog 模型记录重要操作。

### 4. 数据归档
聊天消息会无限增长。
建议定期归档旧消息到冷存储。

### 5. 并发控制
缺少乐观锁机制。
建议添加 `version` 字段防止并发更新冲突。

---

## ✅ 阶段2完成标记 / Phase 2 Completion

✓ 完整数据模型清单
✓ 每个表的详细说明
✓ ER 关系图
✓ 字段类型和约束
✓ 关系说明
✓ 数据流图
✓ 常用查询示例
✓ 迁移说明

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 后端架构分析 (`02-backend-analysis.md`)
