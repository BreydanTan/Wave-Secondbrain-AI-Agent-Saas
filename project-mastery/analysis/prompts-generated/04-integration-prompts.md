# 集成与部署提示词 / Integration & Deployment Prompts

## 📋 使用说明 / Instructions

这些提示词帮助你完成最终的集成、测试和部署工作。

---

## 提示词 4.1: 完整环境配置 / Complete Environment Setup

```
请帮我创建完整的环境配置文件：

**创建 .env.example**:
```bash
# 数据库 / Database
DATABASE_URL=postgresql://user:password@host:5432/database?pgbouncer=true
DIRECT_URL=postgresql://user:password@host:5432/database

# 认证 / Authentication
BETTER_AUTH_SECRET=your-secret-key-here
NEXT_PUBLIC_APP_URL=http://localhost:3000

# Stripe 支付 / Stripe Payment
STRIPE_SECRET_KEY=sk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PLUS_PLAN_ID=price_...
STRIPE_PREMIUM_PLAN_ID=price_...

# AI 服务 / AI Services
AI_GATEWAY_URL=https://your-gateway.com/v1
GOOGLE_GENERATIVE_AI_API_KEY=...
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
XAI_API_KEY=xai-...

# Web 搜索 / Web Search
TAVILY_API_KEY=tvly-...

# 环境 / Environment
NODE_ENV=development
```

**创建 .env.local**:
复制 .env.example 并填入真实值。

**环境变量获取指南**:

1. **PostgreSQL 数据库** (推荐 Supabase 或 Neon):
   - 注册 https://supabase.com
   - 创建新项目
   - 从 Settings → Database 获取连接字符串
   - Transaction Pooler URL → DATABASE_URL
   - Direct URL → DIRECT_URL

2. **BETTER_AUTH_SECRET**:
   ```bash
   openssl rand -base64 32
   ```

3. **Stripe**:
   - 注册 https://stripe.com
   - Dashboard → Developers → API Keys
   - 获取 Secret Key
   - 创建产品和价格，获取 Price IDs
   - Webhooks → 添加端点 (本地用 Stripe CLI):
     ```bash
     stripe listen --forward-to localhost:3000/api/auth/webhook/stripe
     ```

4. **Google AI**:
   - 访问 https://aistudio.google.com/app/apikey
   - 创建 API Key

5. **Tavily 搜索**:
   - 注册 https://tavily.com
   - 获取 API Key

6. **其他 AI 提供商** (可选):
   - Anthropic: https://console.anthropic.com
   - OpenAI: https://platform.openai.com
   - xAI: https://console.x.ai
```

**预期结果**: 完整的环境配置

---

## 提示词 4.2: 数据库迁移与初始化 / Database Migration & Initialization

```
执行数据库设置：

**步骤 1: 安装依赖**
```bash
npm install
```

**步骤 2: 生成 Prisma 客户端**
```bash
npx prisma generate --no-engine
```

**步骤 3: 创建并应用迁移**
```bash
npx prisma migrate dev --name init
```

**步骤 4: (可选) 查看数据库**
```bash
npx prisma studio
```
访问 http://localhost:5555 查看数据库 GUI

**验证数据库**:
检查以下表是否创建成功：
- user
- session
- account
- notes
- chats
- messages
- subscriptions

**初始数据** (可选):
如果需要测试数据，可以创建 prisma/seed.ts:
```typescript
import prisma from "../lib/prisma";

async function main() {
  // 创建测试用户
  const user = await prisma.user.create({
    data: {
      email: "test@example.com",
      name: "Test User",
      emailVerified: true,
    },
  });

  console.log("Seed completed:", user);
}

main();
```

运行:
```bash
npx tsx prisma/seed.ts
```
```

**预期结果**: 数据库完全配置并可用

---

## 提示词 4.3: 测试完整流程 / Test Complete Flow

```
按以下步骤测试完整应用功能：

**启动开发服务器**:
```bash
npm run dev
```

**测试清单**:

1. **认证流程**:
   - [ ] 访问 http://localhost:3000/auth/sign-up
   - [ ] 注册新用户
   - [ ] 检查是否自动登录并跳转到 /home
   - [ ] 检查数据库中是否创建了:
     * User 记录
     * Account 记录
     * Session 记录
     * Subscription 记录 (plan = "free")

2. **聊天功能**:
   - [ ] 点击侧边栏 "Chat"
   - [ ] 发送消息: "你好"
   - [ ] 验证 AI 响应
   - [ ] 检查数据库:
     * Chat 记录已创建
     * Chat.title 由 AI 生成
     * Message 记录(user + assistant)

3. **AI 工具调用**:
   - [ ] 发送: "帮我创建一个关于 TypeScript 的笔记"
   - [ ] 验证 AI 调用 createNote 工具
   - [ ] 检查侧边栏笔记列表更新
   - [ ] 检查数据库 Note 记录

4. **笔记管理**:
   - [ ] 点击侧边栏中的笔记
   - [ ] 验证笔记对话框打开
   - [ ] 编辑标题和内容
   - [ ] 等待自动保存(1秒后)
   - [ ] 刷新页面验证保存成功

5. **搜索笔记**:
   - [ ] 在聊天中: "搜索我的笔记"
   - [ ] 验证 AI 调用 searchNote 工具
   - [ ] 检查返回结果

6. **网络搜索**:
   - [ ] 发送: "搜索最新的 AI 新闻"
   - [ ] 验证 AI 调用 webSearch 工具
   - [ ] 检查返回实时信息

7. **订阅系统**:
   - [ ] 访问 /billing
   - [ ] 点击 "Upgrade to Plus"
   - [ ] 验证跳转到 Stripe Checkout
   - [ ] (测试模式) 使用测试卡 4242 4242 4242 4242
   - [ ] 完成支付
   - [ ] 验证 Webhook 更新订阅状态

8. **生成限制**:
   - [ ] 发送 11 条消息(Free 计划限制 10 次)
   - [ ] 验证显示限制提示
   - [ ] 升级到 Plus 后限制解除

**调试工具**:
- React Query Devtools: 在浏览器中查看请求状态
- 浏览器控制台: 查看错误和网络请求
- Prisma Studio: 查看数据库实时更新
```

**预期结果**: 所有功能正常工作

---

## 提示词 4.4: 部署到 Vercel / Deploy to Vercel

```
部署应用到 Vercel：

**前置准备**:
1. GitHub 仓库已创建并推送代码
2. 已有 Vercel 账号

**部署步骤**:

1. **连接 GitHub**:
   - 登录 https://vercel.com
   - New Project → Import Git Repository
   - 选择你的项目

2. **配置环境变量**:
   在 Vercel 项目设置中添加所有环境变量：
   ```
   DATABASE_URL
   DIRECT_URL
   BETTER_AUTH_SECRET
   STRIPE_SECRET_KEY
   STRIPE_WEBHOOK_SECRET
   STRIPE_PLUS_PLAN_ID
   STRIPE_PREMIUM_PLAN_ID
   GOOGLE_GENERATIVE_AI_API_KEY
   TAVILY_API_KEY
   NODE_ENV=production
   NEXT_PUBLIC_APP_URL=https://your-domain.vercel.app
   ```

3. **配置构建设置**:
   - Framework Preset: Next.js
   - Build Command: `npm run build`
   - Output Directory: `.next`
   - Install Command: `npm install`

4. **添加构建命令** (package.json):
   ```json
   {
     "scripts": {
       "vercel-build": "prisma generate --no-engine && prisma migrate deploy && next build"
     }
   }
   ```

5. **配置 Stripe Webhook**:
   - Stripe Dashboard → Webhooks
   - 添加端点: `https://your-domain.vercel.app/api/auth/webhook/stripe`
   - 选择事件:
     * checkout.session.completed
     * customer.subscription.created
     * customer.subscription.updated
     * customer.subscription.deleted
   - 复制 Webhook Secret 到 STRIPE_WEBHOOK_SECRET

6. **部署**:
   - 点击 "Deploy"
   - 等待构建完成
   - 访问分配的 URL

**部署后检查**:
- [ ] 网站可以访问
- [ ] 注册功能正常
- [ ] 数据库连接成功
- [ ] AI 响应正常
- [ ] Stripe 支付可用

**自定义域名** (可选):
- Vercel 项目 → Settings → Domains
- 添加自定义域名
- 配置 DNS 记录
- 更新 NEXT_PUBLIC_APP_URL
```

**预期结果**: 应用成功部署并运行

---

## 提示词 4.5: 性能优化 / Performance Optimization

```
优化应用性能：

**1. Next.js 优化**:

在 next.config.ts 添加:
```typescript
import type { NextConfig } from "next";

const config: NextConfig = {
  reactStrictMode: true,

  // 图片优化
  images: {
    domains: ["your-cdn.com"],
  },

  // 压缩
  compress: true,

  // 实验性功能
  experimental: {
    optimizeCss: true,
  },
};

export default config;
```

**2. 数据库优化**:

添加索引 (已在 schema.prisma 中):
```prisma
model Message {
  // ...
  @@index([chatId])
}

model Note {
  // ...
  @@index([userId])
}
```

启用 Prisma Accelerate (可选):
```bash
npm install @prisma/extension-accelerate
```

**3. 缓存策略**:

在 API 路由添加缓存头:
```typescript
// app/api/[[...route]]/route.ts
export const dynamic = 'force-dynamic'; // SSR
// 或
export const revalidate = 60; // ISR (60秒)
```

**4. 代码分割**:

使用动态导入:
```typescript
import dynamic from 'next/dynamic';

const ChatInterface = dynamic(() => import('@/components/chat'), {
  loading: () => <LoaderOverlay />,
});
```

**5. 图片优化**:

使用 Next.js Image:
```typescript
import Image from 'next/image';

<Image
  src="/logo.png"
  width={200}
  height={200}
  alt="Logo"
  priority
/>
```

**6. 字体优化**:

在 app/layout.tsx:
```typescript
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }) {
  return (
    <html lang="en" className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```
```

**预期结果**: 应用性能提升

---

## ✅ 集成完成检查清单 / Integration Completion Checklist

- [ ] 环境变量配置完整
- [ ] 数据库迁移成功
- [ ] 本地开发环境运行正常
- [ ] 所有功能测试通过
- [ ] 部署到 Vercel 成功
- [ ] 生产环境测试通过
- [ ] Stripe Webhook 配置正确
- [ ] 性能优化完成

---

## 🔗 相关文档 / Related Documentation

- **部署分析**: `analysis/05-deployment-analysis.md`
- **安全分析**: `analysis/04-security-analysis.md`
- **项目规格**: `specifications/PROJECT_SPEC_CN.md`

---

**生成时间**: 2025-11-16
**版本**: v1.0
**状态**: 集成完成，项目可运行
