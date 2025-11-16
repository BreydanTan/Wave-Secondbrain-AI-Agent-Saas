# 📖 概念词典 / Concept Dictionary

> **技术术语的简单解释 - 用日常生活类比理解编程概念**

这个词典包含Wave AI项目中所有重要的技术术语。每个术语都有：
- 简单解释（用日常类比）
- 在本项目中的作用
- 代码位置
- 相关概念
- 学习资源

---

## 🎯 如何使用这个词典

1. **遇到不懂的术语**：按 Ctrl+F 搜索关键词
2. **系统学习**：从上到下阅读（按难度排序）
3. **快速查找**：使用目录跳转
4. **深入学习**：点击"学习资源"链接

---

## 📚 目录

### 基础概念
- [API](#api)
- [前端 Frontend](#前端-frontend)
- [后端 Backend](#后端-backend)
- [数据库 Database](#数据库-database)
- [HTTP](#http)
- [JSON](#json)
- [环境变量 Environment Variables](#环境变量-environment-variables)

### 框架与工具
- [Next.js](#nextjs)
- [React](#react)
- [TypeScript](#typescript)
- [Tailwind CSS](#tailwind-css)
- [Prisma](#prisma)
- [Hono](#hono)

### 认证与安全
- [BetterAuth](#betterauth)
- [Session](#session)
- [JWT](#jwt)
- [Webhook](#webhook)

### AI相关
- [AI SDK](#ai-sdk)
- [流式响应 Streaming](#流式响应-streaming)
- [工具调用 Tool Calling](#工具调用-tool-calling)
- [提示词 Prompt](#提示词-prompt)

### 数据库概念
- [ORM](#orm)
- [Schema](#schema)
- [Migration](#migration)
- [关系 Relation](#关系-relation)

### 前端概念
- [组件 Component](#组件-component)
- [Hook](#hook)
- [State 状态](#state-状态)
- [Props](#props)
- [路由 Router](#路由-router)

### 开发工具
- [npm](#npm)
- [Git](#git)
- [VS Code](#vs-code)
- [环境 Development/Production](#环境-developmentproduction)

---

## 📘 词条详解

### API

**简单解释**

API就像一个**专门的电话号码**。你拨打不同的号码（端点），就能得到不同的服务。

- 110 → 报警
- 114 → 查号
- `/api/users` → 获取用户信息
- `/api/login` → 登录验证

**在本项目中的作用**

后端提供了多个API端点，前端通过这些端点获取和提交数据。

主要API端点：
- `/api/chat` - AI聊天
- `/api/note` - 笔记管理
- `/api/subscription` - 订阅管理
- `/api/auth` - 用户认证

**代码位置**
- 定义：`app/api/[[...route]]/route.ts:1`
- 调用：组件中的 `fetch()` 或 Hono RPC 客户端

**相关概念**
- [HTTP](#http) - API使用的通信协议
- [JSON](#json) - API数据格式
- [端点 Endpoint](#端点-endpoint)

**学习资源**
- [什么是API？（知乎）](https://www.zhihu.com/question/38594466)
- [RESTful API教程](https://restfulapi.cn/)

---

### 前端 Frontend

**简单解释**

前端就是**餐厅的前台**，是用户能看到和互动的部分。

就像餐厅前台：
- 有菜单（界面）
- 有服务员（交互逻辑）
- 顾客在这里点餐（发送请求）

**在本项目中的作用**

所有用户能看到的页面和交互都是前端：
- 登录页面
- 聊天界面
- 笔记列表
- 设置页面

**代码位置**
- 页面：`app/` 文件夹
- 组件：`components/` 文件夹
- 样式：`app/globals.css`, Tailwind类名

**技术栈**
- React 19 - UI组件
- Next.js 15 - 框架
- Tailwind CSS - 样式

**相关概念**
- [后端 Backend](#后端-backend)
- [React](#react)
- [组件 Component](#组件-component)

---

### 后端 Backend

**简单解释**

后端就是**餐厅的厨房**，顾客看不到，但负责处理所有核心工作。

厨房的工作：
- 收到订单（接收请求）
- 做菜（处理业务逻辑）
- 从仓库拿食材（查询数据库）
- 把菜品传给服务员（返回响应）

**在本项目中的作用**

处理所有业务逻辑：
- 验证用户身份
- 调用AI模型
- 操作数据库
- 处理支付

**代码位置**
- API路由：`app/api/` 文件夹
- 业务逻辑：`lib/` 文件夹
- 数据库操作：使用 Prisma

**技术栈**
- Hono - API框架
- Prisma - 数据库ORM
- BetterAuth - 认证系统

**相关概念**
- [前端 Frontend](#前端-frontend)
- [API](#api)
- [数据库 Database](#数据库-database)

---

### 数据库 Database

**简单解释**

数据库就是**餐厅的仓库**，用来存储所有数据。

仓库特点：
- 有序存放（表结构）
- 分类管理（不同的表）
- 快速查找（索引）
- 长期保存（持久化）

**在本项目中的作用**

存储所有应用数据：
- 用户信息（User表）
- 笔记内容（Note表）
- 聊天记录（Conversation、Message表）
- 订阅状态（Subscription表）

**代码位置**
- 数据库模型：`prisma/schema.prisma:1`
- 查询代码：`lib/` 文件夹中使用 `prisma.xxx.xxx()`

**使用的技术**
- PostgreSQL - 数据库系统
- Prisma - 操作数据库的工具

**相关概念**
- [Prisma](#prisma)
- [Schema](#schema)
- [Migration](#migration)

**学习资源**
- [数据库入门（B站）](https://www.bilibili.com/video/BV1UE41147KC/)

---

### HTTP

**简单解释**

HTTP是**互联网的快递协议**，规定了数据如何传输。

快递流程：
1. 寄件（发送请求）
2. 运输（网络传输）
3. 收件（接收响应）

**HTTP方法（快递类型）**

| 方法 | 类比 | 用途 |
|------|------|------|
| GET | 查询快递 | 获取数据 |
| POST | 寄新快递 | 创建数据 |
| PUT | 更换快递 | 更新数据 |
| DELETE | 退回快递 | 删除数据 |

**在本项目中的作用**

所有前后端通信都使用HTTP：
```
前端发送: POST /api/chat
请求体: { message: "你好" }
↓
后端接收并处理
↓
后端返回: { reply: "你好！有什么可以帮你？" }
```

**代码位置**
- 发送HTTP请求：组件中的 `fetch()` 或 `useMutation()`
- 接收HTTP请求：`app/api/` 中的路由处理器

**相关概念**
- [API](#api)
- [JSON](#json)
- [状态码 Status Code](#状态码-status-code)

---

### JSON

**简单解释**

JSON是**数据的标准格式**，像一张结构化的表格。

类比：
- Word文档 = 给人看的
- JSON = 给程序看的

**示例**

```json
{
  "name": "张三",
  "age": 25,
  "hobbies": ["编程", "阅读"],
  "address": {
    "city": "北京",
    "street": "中关村"
  }
}
```

**在本项目中的作用**

所有API数据都用JSON格式传输：

创建笔记请求：
```json
{
  "title": "学习Next.js",
  "content": "今天学习了路由系统"
}
```

响应：
```json
{
  "id": "note_123",
  "title": "学习Next.js",
  "createdAt": "2025-11-16T10:30:00Z"
}
```

**代码位置**
- 发送JSON：`JSON.stringify(data)`
- 解析JSON：`response.json()`

**相关概念**
- [HTTP](#http)
- [API](#api)

**学习资源**
- [JSON教程（菜鸟教程）](https://www.runoob.com/json/json-tutorial.html)

---

### 环境变量 Environment Variables

**简单解释**

环境变量就是**项目的秘密配置文件**，像银行卡密码。

为什么需要：
- 数据库密码（不能暴露给别人）
- API密钥（每个人不一样）
- 配置信息（开发/生产环境不同）

**在本项目中的作用**

存储敏感信息和配置：

文件：`.env.local`
```bash
DATABASE_URL="postgresql://user:password@localhost:5432/waveai"
ANTHROPIC_API_KEY="sk-ant-xxx"
STRIPE_SECRET_KEY="sk_test_xxx"
```

**如何使用**

```typescript
// 读取环境变量
const apiKey = process.env.ANTHROPIC_API_KEY
const dbUrl = process.env.DATABASE_URL
```

**代码位置**
- 配置文件：`.env.local`（不会上传到Git）
- 使用位置：`lib/ai/providers.ts`, `lib/auth.ts` 等

**重要规则**
- ❌ 不要提交 `.env.local` 到Git
- ✅ 使用 `.env.example` 作为模板
- ✅ 服务器端变量：`process.env.XXX`
- ✅ 客户端变量：必须以 `NEXT_PUBLIC_` 开头

**相关概念**
- [配置管理](#配置管理)
- [安全性](#安全性)

---

### Next.js

**简单解释**

Next.js是**React的超级版**，像给普通汽车加上了自动驾驶功能。

React：普通汽车（需要你手动配置很多东西）
Next.js：特斯拉（自动配置，开箱即用）

**Next.js额外提供的功能**

1. **文件系统路由**
   ```
   app/about/page.tsx → 自动变成 /about 路由
   app/blog/[id]/page.tsx → 自动变成 /blog/123 动态路由
   ```

2. **服务端渲染（SSR）**
   页面在服务器生成，加载更快

3. **API路由**
   在同一个项目里写后端API

4. **图片优化**
   自动压缩和优化图片

**在本项目中的作用**

- 作为整个应用的框架
- 提供页面路由系统
- 提供API路由（`app/api/`）
- 服务端组件和客户端组件

**代码位置**
- 配置文件：`next.config.ts:1`
- 页面文件：`app/` 文件夹中的 `page.tsx`
- 布局文件：`app/layout.tsx`

**关键文件**

| 文件名 | 作用 |
|--------|------|
| `page.tsx` | 页面内容 |
| `layout.tsx` | 页面布局（多个页面共享） |
| `loading.tsx` | 加载状态 |
| `error.tsx` | 错误处理 |
| `route.ts` | API端点 |

**相关概念**
- [React](#react)
- [路由 Router](#路由-router)
- [服务端渲染 SSR](#服务端渲染-ssr)

**学习资源**
- [Next.js官方文档（中文）](https://nextjs.org/docs)
- [Next.js教程（B站）](https://www.bilibili.com/video/BV1Sg411t7Bj/)

---

### React

**简单解释**

React是**搭建界面的乐高积木**，用小积木组装成大作品。

传统方式：画一整幅画（写一整个HTML页面）
React方式：用积木拼装（组合小组件）

**React的核心概念**

1. **组件（Component）**
   ```tsx
   function Button() {
     return <button>点击我</button>
   }
   ```

2. **状态（State）**
   ```tsx
   const [count, setCount] = useState(0)
   ```

3. **Props（属性）**
   ```tsx
   <Button text="提交" color="blue" />
   ```

**在本项目中的作用**

所有UI都是React组件：
- `<ChatMessage />` - 聊天消息气泡
- `<NoteCard />` - 笔记卡片
- `<Button />` - 按钮
- `<Input />` - 输入框

**代码位置**
- UI组件：`components/ui/` 文件夹
- 功能组件：`components/chat/`, `components/note/`
- 页面组件：`app/` 文件夹中的 `.tsx` 文件

**相关概念**
- [组件 Component](#组件-component)
- [Hook](#hook)
- [State 状态](#state-状态)

**学习资源**
- [React官方文档（中文）](https://react.dev/)
- [React入门教程（掘金）](https://juejin.cn/post/7085145274200875022)

---

### TypeScript

**简单解释**

TypeScript是**带类型检查的JavaScript**，像是给代码加上了质检员。

JavaScript：自由发挥（容易出错）
```javascript
let age = "25"  // 字符串
age = 25        // 数字，没问题！
```

TypeScript：严格检查（提前发现错误）
```typescript
let age: number = 25
age = "25"  // ❌ 错误！类型不匹配
```

**为什么使用TypeScript**

1. **提前发现错误**
   ```typescript
   function add(a: number, b: number) {
     return a + b
   }
   add(1, "2")  // ❌ TypeScript会在编写时就报错
   ```

2. **代码提示更好**
   IDE会自动提示可用的属性和方法

3. **代码更易维护**
   看到类型就知道应该传什么

**在本项目中的作用**

所有代码都使用TypeScript：
```typescript
// 定义类型
interface Note {
  id: string
  title: string
  content: string
  createdAt: Date
}

// 使用类型
function createNote(note: Note) {
  // ...
}
```

**代码位置**
- 配置：`tsconfig.json:1`
- 所有 `.ts` 和 `.tsx` 文件都是TypeScript

**常见类型**

| 类型 | 示例 | 说明 |
|------|------|------|
| `string` | `"hello"` | 字符串 |
| `number` | `42` | 数字 |
| `boolean` | `true/false` | 布尔值 |
| `string[]` | `["a", "b"]` | 字符串数组 |
| `object` | `{name: "张三"}` | 对象 |
| `any` | 任何值 | 不推荐使用 |

**相关概念**
- [JavaScript](#javascript)
- [接口 Interface](#接口-interface)
- [类型 Type](#类型-type)

**学习资源**
- [TypeScript官方文档（中文）](https://www.typescriptlang.org/zh/)
- [TypeScript入门教程](https://ts.xcatliu.com/)

---

### Tailwind CSS

**简单解释**

Tailwind CSS是**预制的样式积木**，像宜家家具（组装即用）。

传统CSS：自己设计家具（写所有CSS）
```css
.button {
  background-color: blue;
  padding: 12px 24px;
  border-radius: 8px;
}
```

Tailwind：买宜家家具组装（用预定义类名）
```tsx
<button className="bg-blue-500 px-6 py-3 rounded-lg">
  点击我
</button>
```

**常用类名**

| 类名 | 作用 | CSS等价 |
|------|------|---------|
| `flex` | 弹性布局 | `display: flex` |
| `text-center` | 文字居中 | `text-align: center` |
| `bg-blue-500` | 蓝色背景 | `background-color: #3b82f6` |
| `p-4` | 内边距 | `padding: 1rem` |
| `rounded` | 圆角 | `border-radius: 0.25rem` |

**在本项目中的作用**

所有样式都用Tailwind类名：
```tsx
<div className="flex flex-col gap-4 p-6 bg-white rounded-xl shadow-lg">
  <h1 className="text-2xl font-bold text-gray-900">标题</h1>
  <p className="text-gray-600">内容</p>
</div>
```

**代码位置**
- 全局样式：`app/globals.css:1`
- 组件中：所有 `className` 属性

**相关概念**
- [CSS](#css)
- [响应式设计](#响应式设计)

**学习资源**
- [Tailwind官方文档](https://tailwindcss.com/docs)
- [Tailwind速查表](https://tailwindcomponents.com/cheatsheet/)

---

### Prisma

**简单解释**

Prisma是**操作数据库的翻译官**，把人类语言翻译成数据库语言。

你说："我要所有笔记"
Prisma翻译成：`SELECT * FROM Note`

**使用Prisma vs 直接写SQL**

直接写SQL（复杂）：
```sql
SELECT * FROM "Note" WHERE "userId" = $1 ORDER BY "createdAt" DESC LIMIT 10
```

使用Prisma（简单）：
```typescript
await prisma.note.findMany({
  where: { userId: userId },
  orderBy: { createdAt: 'desc' },
  take: 10
})
```

**在本项目中的作用**

所有数据库操作都通过Prisma：

1. **查询数据**
   ```typescript
   const notes = await prisma.note.findMany()
   ```

2. **创建数据**
   ```typescript
   await prisma.note.create({
     data: { title: "新笔记", content: "内容" }
   })
   ```

3. **更新数据**
   ```typescript
   await prisma.note.update({
     where: { id: "note_123" },
     data: { title: "修改后的标题" }
   })
   ```

4. **删除数据**
   ```typescript
   await prisma.note.delete({
     where: { id: "note_123" }
   })
   ```

**代码位置**
- Schema定义：`prisma/schema.prisma:1`
- Prisma客户端：`lib/db.ts:1`
- 使用位置：`lib/`, `app/api/` 中调用 `prisma.xxx`

**常用命令**

| 命令 | 作用 |
|------|------|
| `npx prisma generate` | 生成Prisma客户端 |
| `npx prisma db push` | 同步数据库 |
| `npx prisma studio` | 打开数据库可视化工具 |
| `npx prisma migrate dev` | 创建迁移 |

**相关概念**
- [ORM](#orm)
- [Schema](#schema)
- [Migration](#migration)

**学习资源**
- [Prisma官方文档](https://www.prisma.io/docs)
- [Prisma教程（掘金）](https://juejin.cn/post/7012526456886878215)

---

### Hono

**简单解释**

Hono是**轻量级的API框架**，专门用来处理HTTP请求。

类比：Hono是快递分拣中心，根据地址（路由）把包裹（请求）送到正确的处理点。

**为什么用Hono而不是Next.js Route Handler**

- 更快（性能更好）
- 类型安全的RPC客户端
- 中间件系统更强大
- 支持Edge Runtime

**在本项目中的作用**

所有API路由都用Hono构建：

```typescript
// app/api/[[...route]]/route.ts
import { Hono } from 'hono'

const app = new Hono()
  .post('/chat', chatHandler)
  .get('/note', getNoteHandler)
  .post('/note', createNoteHandler)

export const GET = handle(app)
export const POST = handle(app)
```

**代码位置**
- 主路由：`app/api/[[...route]]/route.ts:1`
- Chat API：`app/api/[[...route]]/chat.ts:1`
- Note API：`app/api/[[...route]]/note.ts:1`
- RPC客户端：`lib/client.ts:1`

**Hono RPC（远程过程调用）**

前端调用API时有类型提示：
```typescript
// 前端代码
import { client } from '@/lib/client'

// 完全类型安全！
const notes = await client.note.$get()
```

**相关概念**
- [API](#api)
- [中间件 Middleware](#中间件-middleware)
- [RPC](#rpc)

**学习资源**
- [Hono官方文档](https://hono.dev/)

---

### BetterAuth

**简单解释**

BetterAuth是**认证系统**，像门禁卡系统，控制谁能进入。

门禁功能：
- 注册：办理门禁卡
- 登录：刷卡进门
- 登出：注销门禁卡
- 权限：VIP卡能进更多区域

**在本项目中的作用**

负责所有用户认证：

1. **邮箱密码注册/登录**
   ```typescript
   await authClient.signUp.email({
     email: "user@example.com",
     password: "password123"
   })
   ```

2. **Session管理**
   保持登录状态

3. **Stripe集成**
   注册时自动创建Stripe客户

**代码位置**
- 服务端配置：`lib/auth.ts:1`
- 客户端：`lib/auth-client.ts:1`
- API路由：`app/api/auth/[...all]/route.ts:1`

**使用示例**

检查用户是否登录：
```typescript
const session = await auth.api.getSession({
  headers: request.headers
})

if (!session) {
  return Response.json({ error: "未登录" }, { status: 401 })
}
```

**相关概念**
- [Session](#session)
- [JWT](#jwt)
- [中间件 Middleware](#中间件-middleware)

**学习资源**
- [BetterAuth官方文档](https://www.better-auth.com/)

---

### Session

**简单解释**

Session是**登录凭证**，像电影院的手环。

流程：
1. 买票（登录）→ 戴上手环（获得Session）
2. 进出场（访问页面）→ 出示手环（携带Session）
3. 电影结束（登出）→ 摘下手环（删除Session）

**Session vs Cookie vs JWT**

| 方式 | 存储位置 | 安全性 | 本项目使用 |
|------|---------|-------|-----------|
| Session | 服务器 | 高 | ✅ 主要方式 |
| Cookie | 浏览器 | 中 | ✅ 存储Token |
| JWT | 客户端 | 中 | ✅ API认证 |

**在本项目中的作用**

用户登录后创建Session：
```typescript
// 登录后
session = {
  id: "session_123",
  userId: "user_456",
  token: "random_token",
  expiresAt: "2025-12-16"
}
```

每次请求都检查Session：
```typescript
const session = await auth.api.getSession({ headers })
const userId = session.user.id  // 获取当前用户ID
```

**代码位置**
- Session表：`prisma/schema.prisma:45`
- 检查Session：`lib/auth.ts` 中间件
- 使用：所有API路由中

**相关概念**
- [BetterAuth](#betterauth)
- [Cookie](#cookie)
- [JWT](#jwt)

---

### AI SDK

**简单解释**

AI SDK是**统一的AI遥控器**，一个遥控器控制所有品牌的电视。

没有AI SDK：
- Claude用一种方式调用
- GPT用另一种方式
- Gemini又是另一种

有了AI SDK：
```typescript
// 统一接口，切换模型只需改一个参数
const result = await streamText({
  model: claude,  // 可以轻松换成 gpt4, gemini
  messages: [...]
})
```

**在本项目中的作用**

1. **统一多个AI模型**
   ```typescript
   // lib/ai/models.ts
   export const models = {
     'claude-sonnet-4': claude('claude-sonnet-4'),
     'gpt-4.1': openai('gpt-4.1'),
     'gemini-2.5-flash': google('gemini-2.5-flash-latest')
   }
   ```

2. **流式响应**
   AI回答一边生成一边显示

3. **工具调用**
   AI可以主动调用笔记、搜索等工具

**代码位置**
- AI配置：`lib/ai/` 文件夹
- 模型定义：`lib/ai/models.ts:1`
- 工具定义：`lib/ai/tools/` 文件夹
- 聊天接口：`app/api/[[...route]]/chat.ts:1`

**前端使用**

```typescript
import { useChat } from '@ai-sdk/react'

const { messages, input, handleSubmit } = useChat({
  api: '/api/chat'
})
```

**相关概念**
- [流式响应 Streaming](#流式响应-streaming)
- [工具调用 Tool Calling](#工具调用-tool-calling)
- [提示词 Prompt](#提示词-prompt)

**学习资源**
- [AI SDK官方文档](https://sdk.vercel.ai/docs)

---

### 流式响应 Streaming

**简单解释**

流式响应就像**看直播**，内容一边生成一边显示。

普通响应（等全部生成完）：
```
用户: 写一篇文章
[等待10秒...]
AI: [整篇文章一次性出现]
```

流式响应（实时显示）：
```
用户: 写一篇文章
AI: 今天...
AI: 今天我们...
AI: 今天我们来讨论...
AI: 今天我们来讨论编程...
```

**在本项目中的作用**

AI聊天使用流式响应，用户体验更好：

```typescript
// 后端
const result = streamText({
  model: claude,
  messages: messages,
  onChunk: ({ chunk }) => {
    // 每生成一小段就发送
  }
})

// 前端自动处理流式数据
const { messages } = useChat()
```

**代码位置**
- 后端：`app/api/[[...route]]/chat.ts:67` - `streamText()`
- 前端：`app/(main)/chat/[id]/page.tsx:45` - `useChat()`

**技术细节**

使用Server-Sent Events (SSE)：
```
前端 ← ← ← 后端
   接收   发送
   流式   流式
   数据   数据
```

**相关概念**
- [AI SDK](#ai-sdk)
- [SSE](#sse)
- [WebSocket](#websocket)

---

### 工具调用 Tool Calling

**简单解释**

工具调用就是**给AI一个工具箱**，让它主动选择工具来完成任务。

类比修理工：
- 你：帮我修桌子
- 修理工（AI）：
  1. 打开工具箱
  2. 选择锤子和钉子（工具调用）
  3. 修好桌子

**在本项目中的作用**

AI有4个工具可以使用：

1. **create-note** - 创建笔记
   ```typescript
   用户: 帮我记录：学习React
   AI思考: 需要使用create-note工具
   AI调用: createNoteTool({ title: "学习React", content: "..." })
   AI回复: ✅ 已为您创建笔记
   ```

2. **search-note** - 搜索笔记
3. **web-search** - 网络搜索
4. **extract-url** - 提取URL内容

**代码位置**
- 工具定义：`lib/ai/tools/` 文件夹
- 工具注册：`lib/ai/index.ts:78` - `streamText({ tools: {...} })`

**工具定义示例**

```typescript
// lib/ai/tools/create-note.ts
export const createNoteTool = tool({
  description: "创建一条新笔记",
  parameters: z.object({
    title: z.string(),
    content: z.string()
  }),
  execute: async ({ title, content }, { user }) => {
    const note = await prisma.note.create({
      data: { userId: user.id, title, content }
    })
    return { noteId: note.id, success: true }
  }
})
```

**相关概念**
- [AI SDK](#ai-sdk)
- [Function Calling](#function-calling)

---

### 组件 Component

**简单解释**

组件就是**可复用的UI积木**，像乐高块。

一个按钮是组件：
```tsx
function Button({ text }) {
  return <button>{text}</button>
}
```

使用这个组件：
```tsx
<Button text="提交" />
<Button text="取消" />
<Button text="确认" />
```

**组件的层次**

```
页面 (Page)
├── 布局 (Layout)
│   ├── 导航栏 (Navbar)
│   │   ├── Logo组件
│   │   └── 菜单组件
│   └── 侧边栏 (Sidebar)
│       └── 菜单项组件
└── 内容区 (Content)
    ├── 标题组件
    └── 卡片列表
        └── 卡片组件 ← 可复用！
```

**在本项目中的作用**

所有UI都是组件组成：

**UI基础组件** (`components/ui/`)
- `<Button />` - 按钮
- `<Input />` - 输入框
- `<Card />` - 卡片

**功能组件** (`components/`)
- `<ChatMessage />` - 聊天消息
- `<NoteCard />` - 笔记卡片
- `<SubscriptionCard />` - 订阅卡片

**页面组件** (`app/`)
- Chat页面
- Note页面
- Settings页面

**代码位置**
- 所有 `.tsx` 文件都定义组件
- UI组件：`components/ui/`
- 业务组件：`components/chat/`, `components/note/`

**组件示例**

```tsx
// components/chat/message.tsx
export function ChatMessage({ content, role }) {
  return (
    <div className={role === 'user' ? 'text-right' : 'text-left'}>
      <p>{content}</p>
    </div>
  )
}
```

**相关概念**
- [React](#react)
- [Props](#props)
- [State 状态](#state-状态)

---

### Hook

**简单解释**

Hook是**React的魔法函数**，给组件添加超能力。

类比：
- 组件 = 普通人
- Hook = 超能力药水

常用Hook：
- `useState` - 记忆能力（记住数据）
- `useEffect` - 感知能力（感知变化）
- `useRef` - 引用能力（指向DOM）

**在本项目中的作用**

**1. useState - 状态管理**
```tsx
const [count, setCount] = useState(0)

<button onClick={() => setCount(count + 1)}>
  点击了 {count} 次
</button>
```

**2. useEffect - 副作用**
```tsx
useEffect(() => {
  // 组件加载时执行
  fetchData()
}, [])  // 空数组表示只执行一次
```

**3. 自定义Hook**
```typescript
// hooks/use-auth-token.ts
export function useAuthToken() {
  const [token, setToken] = useState(null)
  return { token, setToken }
}
```

**项目中的常用Hook**

| Hook | 用途 | 代码位置 |
|------|------|---------|
| `useChat` | AI聊天 | `@ai-sdk/react` |
| `useMutation` | 数据修改 | `@tanstack/react-query` |
| `useAuthToken` | 认证令牌 | `hooks/use-auth-token.ts` |
| `useLocalChat` | 聊天设置 | `hooks/use-localchat.ts` |

**代码位置**
- 自定义Hook：`hooks/` 文件夹
- 使用位置：所有 `.tsx` 组件文件

**相关概念**
- [React](#react)
- [State 状态](#state-状态)
- [Effect 副作用](#effect-副作用)

**学习资源**
- [React Hooks文档](https://react.dev/reference/react)

---

### State 状态

**简单解释**

State就是**组件的记忆**，记住数据的变化。

类比：
- 没有State = 失忆患者（每次都忘记）
- 有State = 正常人（能记住东西）

**示例**

```tsx
function Counter() {
  const [count, setCount] = useState(0)  // State

  return (
    <div>
      <p>当前计数: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        +1
      </button>
    </div>
  )
}
```

**State类型**

1. **本地状态（Local State）**
   只在一个组件内使用
   ```tsx
   const [isOpen, setIsOpen] = useState(false)
   ```

2. **全局状态（Global State）**
   多个组件共享
   ```tsx
   // Zustand
   const bearerToken = useAuthToken(state => state.bearerToken)
   ```

3. **服务器状态（Server State）**
   来自API的数据
   ```tsx
   // React Query
   const { data: notes } = useQuery({
     queryKey: ['notes'],
     queryFn: fetchNotes
   })
   ```

**在本项目中的作用**

**本地状态示例**：
- 模态框开关
- 表单输入值
- 加载状态

**全局状态示例**：
- 用户认证token（`useAuthToken`）
- AI模型选择（`useLocalChat`）

**服务器状态示例**：
- 笔记列表
- 聊天历史
- 用户订阅信息

**代码位置**
- Zustand状态：`hooks/use-*.ts`
- 服务器状态：使用React Query的组件

**相关概念**
- [Hook](#hook)
- [Zustand](#zustand)
- [React Query](#react-query)

---

### Props

**简单解释**

Props就是**组件的参数**，像函数的参数一样。

普通函数：
```javascript
function greet(name) {  // name是参数
  return `你好，${name}`
}
greet("张三")  // 传入参数
```

React组件：
```tsx
function Greeting({ name }) {  // name是Props
  return <h1>你好，{name}</h1>
}
<Greeting name="张三" />  {/* 传入Props */}
```

**Props的特点**

1. **只读**（不能修改）
   ```tsx
   function Button({ text }) {
     text = "新文字"  // ❌ 错误！不能修改Props
   }
   ```

2. **从父组件传入**
   ```tsx
   // 父组件
   <ChatMessage content="你好" role="user" />

   // 子组件
   function ChatMessage({ content, role }) {
     return <div>{content}</div>
   }
   ```

**在本项目中的作用**

几乎所有组件都接收Props：

```tsx
// components/note/note-card.tsx
interface NoteCardProps {
  id: string
  title: string
  content: string
  createdAt: Date
  onDelete?: () => void
}

export function NoteCard({ id, title, content, createdAt, onDelete }: NoteCardProps) {
  return (
    <div>
      <h3>{title}</h3>
      <p>{content}</p>
      <span>{createdAt.toLocaleDateString()}</span>
      {onDelete && <button onClick={onDelete}>删除</button>}
    </div>
  )
}
```

使用：
```tsx
<NoteCard
  id="note_123"
  title="学习React"
  content="Props很重要"
  createdAt={new Date()}
  onDelete={() => handleDelete("note_123")}
/>
```

**代码位置**
- 所有组件都使用Props
- 查看组件定义了解接收哪些Props

**相关概念**
- [组件 Component](#组件-component)
- [TypeScript接口](#typescript接口)

---

## 🔍 快速查找表

### 按场景查找

**我想理解...**
- 项目整体架构 → [架构可视化](./01-architecture-visuals.md)
- 数据如何存储 → [数据库](#数据库-database), [Prisma](#prisma)
- 前后端如何通信 → [API](#api), [HTTP](#http)
- AI如何工作 → [AI SDK](#ai-sdk), [流式响应](#流式响应-streaming)
- 用户如何登录 → [BetterAuth](#betterauth), [Session](#session)

**我遇到错误...**
- "Cannot find module" → 查看[错误百科](./resources/error-encyclopedia.md)
- 类型错误 → [TypeScript](#typescript)
- 网络错误 → [HTTP](#http), [API](#api)
- 数据库错误 → [Prisma](#prisma)

**我想添加功能...**
- 新页面 → [Next.js](#nextjs), [路由](#路由-router)
- 新API → [Hono](#hono), [API](#api)
- 新组件 → [React](#react), [组件](#组件-component)
- 新数据模型 → [Prisma](#prisma), [Schema](#schema)

---

## 📚 下一步

1. **理解架构**: 阅读 [架构可视化](./01-architecture-visuals.md)
2. **动手实践**: 前往 [阶段1教学](./03-stage1-foundation.md)
3. **深入学习**: 查看 [项目规格文档](../specifications/PROJECT_SPEC_CN.md)
4. **遇到问题**: 使用 [错误百科](./resources/error-encyclopedia.md)

---

**生成日期**: 2025-11-16
**文档版本**: v1.0
**文档数量**: 45+ 概念词条
