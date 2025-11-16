# 阶段6：部署配置分析 / Phase 6: Deployment Configuration Analysis

## 🚀 部署概览 / Deployment Overview

Wave AI Agent 设计为部署在 **Vercel** 平台，采用 Serverless 架构。

**推荐部署平台**: Vercel
**运行时**: Node.js
**数据库**: PostgreSQL (外部托管)

---

## 📦 构建配置 / Build Configuration

### Package.json 脚本 / Scripts

**文件位置**: `package.json:5-12`

```json
{
  "scripts": {
    "dev": "next dev --turbopack",
    "build": "next build",
    "start": "next start",
    "lint": "eslint",
    "db:migrate": "npx prisma migrate dev --name init",
    "db:studio": "npx prisma studio",
    "postinstall": "prisma generate --no-engine"
  }
}
```

**脚本说明**:

| 脚本 | 说明 | 何时使用 |
|------|------|---------|
| `dev` | 启动开发服务器 (Turbopack) | 本地开发 |
| `build` | 构建生产版本 | 部署前 / CI/CD |
| `start` | 启动生产服务器 | 自托管部署 |
| `lint` | 代码检查 | CI/CD |
| `db:migrate` | 运行数据库迁移 | 数据库架构变更 |
| `db:studio` | 打开 Prisma Studio (数据库 GUI) | 本地开发 |
| `postinstall` | 生成 Prisma 客户端 | npm install 后自动执行 |

---

### Next.js 配置

**文件位置**: `next.config.ts:1-7`

```typescript
import type { NextConfig } from "next";

const nextConfig: NextConfig = {
  /* config options here */
};

export default nextConfig;
```

**当前配置**: 使用 Next.js 默认配置

**可能需要的配置** (生产环境):
```typescript
const nextConfig: NextConfig = {
  // 图片优化
  images: {
    domains: ["your-cdn.com"],
    formats: ["image/avif", "image/webp"],
  },

  // 环境变量验证
  env: {
    NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
  },

  // 严格模式
  reactStrictMode: true,

  // 实验性特性
  experimental: {
    serverActions: {
      bodySizeLimit: "2mb",
    },
  },
};
```

---

### TypeScript 配置

**文件位置**: `tsconfig.json:1-27`

```json
{
  "compilerOptions": {
    "target": "ES2017",
    "lib": ["dom", "dom.iterable", "esnext"],
    "allowJs": true,
    "skipLibCheck": true,
    "strict": true,
    "noEmit": true,
    "esModuleInterop": true,
    "module": "esnext",
    "moduleResolution": "bundler",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "jsx": "preserve",
    "incremental": true,
    "plugins": [
      {
        "name": "next"
      }
    ],
    "paths": {
      "@/*": ["./*"]
    }
  },
  "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
  "exclude": ["node_modules"]
}
```

**关键配置**:
- `strict: true` - 严格类型检查
- `paths: { "@/*": ["./*"] }` - 路径别名
- `target: "ES2017"` - 编译目标

---

## 🗄️ 数据库配置 / Database Configuration

### Prisma 配置

**Schema 位置**: `prisma/schema.prisma:1-16`

```prisma
generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")    // 连接池 URL
  directUrl = env("DIRECT_URL")      // 直连 URL (迁移用)
}
```

**重要说明**:
- **DATABASE_URL**: 用于应用查询 (推荐使用连接池，如 PgBouncer 或 Prisma Accelerate)
- **DIRECT_URL**: 用于迁移 (需要直接连接数据库)

---

### 推荐数据库托管服务

| 服务 | 优势 | 价格 |
|------|------|------|
| **Vercel Postgres** | 与 Vercel 集成，自动连接池 | 免费套餐 |
| **Supabase** | 开源，免费慷慨额度 | 免费 → $25/月 |
| **Neon** | Serverless PostgreSQL，自动伸缩 | 免费 → $19/月 |
| **Railway** | 简单配置，开发者友好 | $5/月起 |

**环境变量示例** (Vercel Postgres):
```bash
DATABASE_URL="postgres://default:xxx@xxx-pooler.aws.neon.tech:5432/verceldb?pgbouncer=true&sslmode=require"
DIRECT_URL="postgres://default:xxx@xxx.aws.neon.tech:5432/verceldb?sslmode=require"
```

---

### 数据库迁移流程

#### 开发环境
```bash
# 1. 修改 schema.prisma
# 2. 生成并应用迁移
npm run db:migrate

# 等价于
npx prisma migrate dev --name init
```

#### 生产环境
```bash
# 在 CI/CD 或手动执行
npx prisma migrate deploy
```

**Vercel 部署**:
- 在 Vercel 项目设置中配置环境变量
- 迁移可以在本地运行，然后推送代码
- 或使用 Vercel Cron Jobs / GitHub Actions

---

## 🌍 环境变量配置 / Environment Variables

### 必需环境变量清单

#### 数据库 / Database
```bash
DATABASE_URL=postgres://...        # PostgreSQL 连接池 URL
DIRECT_URL=postgres://...          # PostgreSQL 直连 URL
```

#### 认证 / Authentication
```bash
BETTER_AUTH_SECRET=xxx             # BetterAuth 密钥 (生成: openssl rand -base64 32)
BETTER_AUTH_URL=https://your-domain.com
```

#### Stripe 支付 / Stripe Payment
```bash
STRIPE_SECRET_KEY=sk_live_...             # Stripe 密钥
STRIPE_WEBHOOK_SECRET=whsec_...           # Webhook 签名密钥
STRIPE_PLUS_PLAN_ID=price_...             # Plus 计划价格 ID
STRIPE_PREMIUM_PLAN_ID=price_...          # Premium 计划价格 ID
```

#### AI 服务 / AI Services
```bash
# Google AI (开发环境)
GOOGLE_GENERATIVE_AI_API_KEY=xxx

# AI Gateway (生产环境推荐)
AI_GATEWAY_URL=https://gateway.ai.cloudflare.com/v1/xxx

# 可选: 直接使用特定模型 API
ANTHROPIC_API_KEY=sk-ant-xxx
OPENAI_API_KEY=sk-xxx
XAI_API_KEY=xxx
```

#### Web 搜索 / Web Search
```bash
TAVILY_API_KEY=tvly-xxx            # Tavily 搜索 API
```

#### 应用配置 / Application Config
```bash
NODE_ENV=production                 # 环境 (development | production)
NEXT_PUBLIC_APP_URL=https://your-domain.com  # 公开应用 URL
```

---

### Vercel 环境变量配置

#### 设置步骤
```
1. 进入 Vercel Dashboard
   ↓
2. 选择项目
   ↓
3. Settings → Environment Variables
   ↓
4. 添加每个变量
   - Name: DATABASE_URL
   - Value: postgres://...
   - Environment: Production, Preview, Development
   ↓
5. 保存并重新部署
```

#### 环境分离
```
Production:    真实数据库、真实 Stripe、真实 AI API
Preview:       测试数据库、Stripe 测试模式、开发 AI API
Development:   本地数据库、Stripe 测试模式、开发 AI API
```

---

## 🔧 Vercel 部署配置 / Vercel Deployment Config

### 自动部署流程

```
1. 推送代码到 GitHub
   ↓
2. Vercel 自动触发部署
   ↓
3. 安装依赖 (npm install)
   - 执行 postinstall: prisma generate --no-engine
   ↓
4. 构建 (npm run build)
   - Next.js 构建优化
   - 代码分割
   - 静态页面生成
   ↓
5. 部署到 Edge Network
   ↓
6. 健康检查
   ↓
7. 自动切换流量到新版本
```

---

### 部署设置

#### Build & Development Settings
```
Framework Preset: Next.js
Build Command: npm run build
Output Directory: .next
Install Command: npm install
Development Command: npm run dev
```

#### Root Directory
```
Root Directory: ./
```

#### Node.js Version
```
Node.js Version: 20.x (推荐)
```

---

### Serverless Function 配置

**Next.js API Routes 自动部署为 Serverless Functions**

**限制** (Vercel Free Tier):
- 执行时间: 10 秒
- 内存: 1024 MB
- 请求体大小: 4.5 MB

**优化建议**:
- AI 流式响应 (已实现) - 绕过时间限制
- 大文件上传使用客户端直传 (如 S3)

---

## 🔄 CI/CD 配置 / CI/CD Configuration

### GitHub Actions 示例

虽然项目未包含 `.github/workflows`，但以下是推荐的 CI/CD 配置：

**文件**: `.github/workflows/ci.yml`

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: "20"
      - run: npm install
      - run: npm run lint

  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npm run build

  migrate:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
      - run: npm install
      - run: npx prisma migrate deploy
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
```

---

## 📊 性能优化 / Performance Optimization

### Next.js 优化

#### 1. 图片优化
```typescript
import Image from "next/image";

<Image
  src="/logo.png"
  alt="Logo"
  width={100}
  height={100}
  priority  // 首屏图片
/>
```

#### 2. 字体优化
```typescript
// app/layout.tsx
import { Inter } from "next/font/google";

const inter = Inter({ subsets: ["latin"] });

export default function RootLayout({ children }) {
  return (
    <html className={inter.className}>
      {children}
    </html>
  );
}
```

#### 3. 代码分割 (自动)
- Next.js 自动按页面分割
- 动态导入: `const Component = dynamic(() => import('./Component'))`

#### 4. 缓存策略
```typescript
// 静态页面
export const revalidate = 3600;  // 1 小时

// 动态数据
export const dynamic = "force-dynamic";
```

---

### Prisma 优化

#### 1. 连接池
使用 DATABASE_URL 连接池 (PgBouncer / Prisma Accelerate)

#### 2. 查询优化
```typescript
// ✅ 只选择需要的字段
const notes = await prisma.note.findMany({
  select: {
    id: true,
    title: true,
    // 不包含 content (减少传输)
  },
});

// ✅ 批量查询
const [notes, total] = await Promise.all([
  prisma.note.findMany(),
  prisma.note.count(),
]);
```

---

## 🚨 部署前检查清单 / Pre-Deployment Checklist

### 代码检查
- [ ] 所有 TypeScript 错误已修复
- [ ] ESLint 无错误
- [ ] 所有测试通过 (如果有)
- [ ] 无 `console.log` (生产环境)
- [ ] 无硬编码的密钥

### 配置检查
- [ ] `.env` 文件已配置完整
- [ ] `.env` 已添加到 `.gitignore`
- [ ] 环境变量已在 Vercel 设置
- [ ] DATABASE_URL 和 DIRECT_URL 都已配置

### 数据库检查
- [ ] 生产数据库已创建
- [ ] 数据库迁移已应用 (`prisma migrate deploy`)
- [ ] Prisma 客户端已生成

### 第三方服务检查
- [ ] Stripe 账户已创建 (生产模式)
- [ ] Stripe Webhook 已配置
- [ ] AI API 密钥已获取
- [ ] Tavily API 密钥已获取

### 安全检查
- [ ] BETTER_AUTH_SECRET 使用强随机密钥
- [ ] Stripe Webhook Secret 已配置
- [ ] CORS 设置正确
- [ ] 环境变量不在代码中暴露

### 性能检查
- [ ] 图片已优化
- [ ] 代码分割正常
- [ ] Lighthouse 分数 > 80

---

## 🌐 域名配置 / Domain Configuration

### Vercel 域名设置

```
1. 购买域名 (如 Namecheap, GoDaddy)
   ↓
2. 在 Vercel Dashboard 添加域名
   - Settings → Domains
   - 输入: your-domain.com
   ↓
3. 配置 DNS 记录
   A Record:
     Name: @
     Value: 76.76.21.21 (Vercel IP)

   CNAME Record:
     Name: www
     Value: cname.vercel-dns.com
   ↓
4. SSL 自动配置 (Let's Encrypt)
   ↓
5. 更新环境变量
   NEXT_PUBLIC_APP_URL=https://your-domain.com
   BETTER_AUTH_URL=https://your-domain.com
```

---

## 📈 监控和日志 / Monitoring & Logging

### Vercel Analytics

**启用方法**:
```typescript
// app/layout.tsx
import { Analytics } from "@vercel/analytics/react";

export default function RootLayout({ children }) {
  return (
    <html>
      <body>
        {children}
        <Analytics />
      </body>
    </html>
  );
}
```

**功能**:
- 页面访问量
- 用户地理位置
- 设备类型
- 性能指标

---

### 推荐监控服务

| 服务 | 用途 | 价格 |
|------|------|------|
| **Sentry** | 错误追踪 | 免费 → $26/月 |
| **LogRocket** | 会话回放 | $99/月起 |
| **Better Stack** | 日志聚合 | $19/月起 |
| **Vercel Speed Insights** | 性能监控 | 免费 |

---

## 🔄 回滚策略 / Rollback Strategy

### Vercel 自动回滚

```
1. 部署失败
   → Vercel 保留上一个版本
   → 流量不受影响
   ↓
2. 手动回滚
   → Deployments 页面
   → 选择之前的部署
   → 点击 "Promote to Production"
```

---

## ✅ 阶段6完成标记 / Phase 6 Completion

✓ 构建配置
✓ 数据库配置
✓ 环境变量清单
✓ Vercel 部署流程
✓ CI/CD 配置示例
✓ 性能优化建议
✓ 部署前检查清单
✓ 域名配置
✓ 监控和日志

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 生成重建提示词 (`analysis/prompts-generated/`)
