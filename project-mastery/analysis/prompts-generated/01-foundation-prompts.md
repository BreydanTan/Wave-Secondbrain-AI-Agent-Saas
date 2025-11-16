# 重建提示词 - 第1部分：基础设施 / Rebuild Prompts - Part 1: Foundation

## 📘 使用说明 / Usage Instructions

这些提示词设计用于与 AI 编程助手（如 Claude、GPT-4、Cursor 等）配合使用，逐步重建 Wave AI Agent 项目。

**使用步骤**:
1. 按顺序执行每个提示词
2. 验证每个步骤的输出
3. 出现错误时参考"预期结果"部分排查
4. 完成一个提示词后再进入下一个

---

## 🚀 提示词 1.1：初始化 Next.js 项目

### 提示词内容
```
请帮我创建一个新的 Next.js 15 项目，技术栈配置如下：

框架和语言：
- Next.js 15.5.3 (使用 App Router)
- React 19.1.0
- TypeScript 5

包管理器：npm

初始化命令：
npx create-next-app@latest wave-ai-agent --typescript --tailwind --app --no-src-dir --import-alias "@/*"

项目配置要求：
- 使用 TypeScript
- 使用 Tailwind CSS v4
- 使用 App Router
- 不使用 src 目录
- 路径别名：@/*

完成后，请：
1. 显示生成的目录结构
2. 显示 package.json 内容
3. 确认项目可以启动 (npm run dev)
```

### 预期结果
```
wave-ai-agent/
├── app/
│   ├── layout.tsx
│   ├── page.tsx
│   └── globals.css
├── public/
├── package.json
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
└── postcss.config.mjs
```

### 验证步骤
```bash
cd wave-ai-agent
npm run dev
# 打开 http://localhost:3000 应该看到 Next.js 欢迎页面
```

---

## 📦 提示词 1.2：安装核心依赖

### 提示词内容
```
请在项目中安装以下依赖包，确保使用精确的版本号：

核心依赖（生产环境）：
npm install \
  @ai-sdk/gateway@1.0.23 \
  @ai-sdk/google@2.0.11 \
  @ai-sdk/react@2.0.23 \
  @better-auth/stripe@1.3.9 \
  @hono/zod-validator@0.7.2 \
  @hookform/resolvers@5.2.2 \
  @prisma/client@6.14.0 \
  @prisma/extension-accelerate@2.0.2 \
  @tanstack/react-query@5.85.5 \
  @tavily/core@0.5.11 \
  ai@5.0.45 \
  better-auth@1.3.7 \
  clsx@2.1.1 \
  date-fns@4.1.0 \
  hono@4.9.4 \
  lucide-react@0.544.0 \
  nanoid@5.1.5 \
  next-themes@0.4.6 \
  nuqs@2.5.2 \
  react-hook-form@7.62.0 \
  sonner@2.0.7 \
  stripe@18.5.0 \
  tailwind-merge@3.3.1 \
  zod@4.1.8 \
  zustand@5.0.8

开发依赖：
npm install -D \
  @types/node@20 \
  @types/react@19 \
  @types/react-dom@19 \
  prisma@6.15.0 \
  typescript@5

Radix UI 组件（UI 基础库）：
npm install \
  @radix-ui/react-avatar@1.1.10 \
  @radix-ui/react-dialog@1.1.15 \
  @radix-ui/react-dropdown-menu@2.1.16 \
  @radix-ui/react-label@2.1.7 \
  @radix-ui/react-popover@1.1.15 \
  @radix-ui/react-scroll-area@1.2.10 \
  @radix-ui/react-select@2.2.6 \
  @radix-ui/react-separator@1.1.7 \
  @radix-ui/react-slot@1.2.3 \
  @radix-ui/react-tooltip@1.2.8

完成后：
1. 确认 package.json 中所有依赖版本正确
2. 运行 npm install 确保无错误
3. 列出已安装的关键包及其版本
```

### 预期结果
所有包成功安装，无警告或错误

### 验证步骤
```bash
npm list @ai-sdk/react
npm list hono
npm list prisma
# 所有版本应与上述列表匹配
```

---

## 🗄️ 提示词 1.3：配置 Prisma 和数据库

### 提示词内容
```
请配置 Prisma ORM 和数据库，步骤如下：

1. 初始化 Prisma（如果未自动初始化）：
npx prisma init

2. 修改 prisma/schema.prisma 文件，配置如下：

generator client {
  provider = "prisma-client-js"
  output   = "../generated/prisma"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// 数据模型
model User {
  id               String         @id
  name             String
  email            String         @unique
  emailVerified    Boolean        @default(false)
  stripeCustomerId String?
  image            String?
  createdAt        DateTime       @default(now())
  updatedAt        DateTime       @default(now()) @updatedAt
  sessions         Session[]
  accounts         Account[]
  notes            Note[]
  Chat             Chat[]
  subscriptions    Subscription[]

  @@map("user")
}

model Session {
  id        String   @id
  expiresAt DateTime
  token     String   @unique
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  ipAddress String?
  userAgent String?
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@map("session")
}

model Account {
  id                    String    @id
  accountId             String
  providerId            String
  userId                String
  user                  User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  accessToken           String?
  refreshToken          String?
  idToken               String?
  accessTokenExpiresAt  DateTime?
  refreshTokenExpiresAt DateTime?
  scope                 String?
  password              String?
  createdAt             DateTime  @default(now())
  updatedAt             DateTime  @updatedAt

  @@map("account")
}

model Note {
  id        String   @id @default(uuid())
  title     String
  content   String
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  userId    String
  user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
  @@map("notes")
}

model Chat {
  id        String    @id @default(uuid())
  title     String
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
  userId    String
  user      User      @relation(fields: [userId], references: [id], onDelete: Cascade)
  messages  Message[]

  @@map("chats")
}

enum Role {
  user
  assistant
  system
}

model Message {
  id        String   @id @default(uuid())
  role      Role
  parts     Json
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  chatId    String
  chat      Chat     @relation(fields: [chatId], references: [id], onDelete: Cascade)

  @@index([chatId])
  @@map("messages")
}

enum Plan {
  free
  plus
  premium
}

model Subscription {
  id                   String    @id @default(uuid())
  plan                 Plan      @default(free)
  referenceId          String
  stripeCustomerId     String?
  stripeSubscriptionId String?
  status               String
  periodStart          DateTime?
  periodEnd            DateTime?
  cancelAtPeriodEnd    Boolean?  @default(false)
  seats                Int?
  trialStart           DateTime?
  trialEnd             DateTime?
  user                 User      @relation(fields: [referenceId], references: [id], onDelete: Cascade)

  @@map("subscriptions")
}

3. 创建 lib/prisma.ts 文件：
import { PrismaClient } from "@/generated/prisma";

const prisma = new PrismaClient();

export default prisma;

4. 更新 package.json scripts：
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

5. 配置环境变量 .env：
DATABASE_URL="postgresql://user:password@localhost:5432/waveai?pgbouncer=true"
DIRECT_URL="postgresql://user:password@localhost:5432/waveai"
BETTER_AUTH_SECRET="your-secret-key-here"
NEXT_PUBLIC_APP_URL="http://localhost:3000"

（注意：实际部署时需要使用真实的数据库 URL）

6. 生成 Prisma 客户端：
npm run postinstall

完成后确认：
1. generated/prisma 目录已创建
2. Prisma 客户端可以导入
3. .env 文件已创建（不要提交到 Git）
```

### 预期结果
```
✓ Prisma schema 已配置
✓ 数据库连接已配置
✓ Prisma 客户端已生成到 generated/prisma/
✓ lib/prisma.ts 可以导入
```

### 验证步骤
```bash
# 检查生成的客户端
ls generated/prisma

# 测试导入（在任何 .ts 文件中）
import prisma from "@/lib/prisma";
```

---

## 🔐 提示词 1.4：配置 BetterAuth 认证

### 提示词内容
```
请配置 BetterAuth 认证系统，包括 Stripe 集成：

1. 创建 lib/stripe.ts 文件：
import Stripe from "stripe";

export const stripeClient = new Stripe(
  process.env.STRIPE_SECRET_KEY!,
  {
    apiVersion: "2024-11-20.acacia", // 使用最新的 API 版本
  }
);

2. 创建 lib/constant.ts 文件：
export const PLAN_ENUM = {
  FREE: "free",
  PLUS: "plus",
  PREMIUM: "premium",
} as const;

export type PlanEnumType = (typeof PLAN_ENUM)[keyof typeof PLAN_ENUM];
export type PaidPlanEnumType = Exclude<PlanEnumType, "free">;

export const UPGRADEABLE_PLANS = [PLAN_ENUM.PLUS, PLAN_ENUM.PREMIUM];

const PLUS_PRICE_ID = process.env.STRIPE_PLUS_PLAN_ID!;
const PREMIUM_PRICE_ID = process.env.STRIPE_PREMIUM_PLAN_ID!;

export const PLANS = [
  {
    id: 1,
    name: PLAN_ENUM.FREE,
    price: 0,
    priceId: undefined,
    features: [
      "20 AI generations per month",
      "Basic support",
      "Limited notes creation",
      "Access to core features",
      "Community access",
      "Single user only",
    ],
    limits: {
      generations: 10,
    },
  },
  {
    id: 2,
    name: PLAN_ENUM.PLUS,
    price: 12,
    priceId: PLUS_PRICE_ID,
    features: [
      "300 AI generations per month",
      "Unlimited notes creation",
      "Priority support",
      "Access to all features",
      "AI Advanced search",
    ],
    limits: {
      generations: 300,
    },
  },
  {
    id: 3,
    name: PLAN_ENUM.PREMIUM,
    price: 24,
    priceId: PREMIUM_PRICE_ID,
    features: [
      "Unlimited AI generations",
      "Unlimited notes creation",
      "Priority support",
      "Early access to new features",
      "AI Advanced search",
      "Advanced admin & analytics",
      "Custom integrations & API access",
    ],
    limits: {
      generations: Infinity,
    },
  },
];

3. 创建 app/actions/action.ts 文件：
"use server";
import { myProvider } from "@/lib/ai/providers";
import { PLAN_ENUM, PLANS } from "@/lib/constant";
import prisma from "@/lib/prisma";
import { generateText, type UIMessage } from "ai";
import { HTTPException } from "hono/http-exception";

export async function generateTitleForUserMessage({
  message,
}: {
  message: UIMessage;
}) {
  try {
    const { text } = await generateText({
      model: myProvider.languageModel("title-model"),
      system: `\n
    - you will generate a short title based on the first message a user begins a conversation with
    - ensure it is not more than 80 characters long
    - the title should be a summary of the user's message
    - do not use quotes or colons`,
      prompt: JSON.stringify(message),
    });
    return text;
  } catch (error) {
    console.log("Title ai error", error);
    return "Untitled";
  }
}

export async function createDefaultSubscription(
  userId: string,
  stripeCustomerId: string
) {
  try {
    const existingSubscription = await prisma.subscription.findFirst({
      where: {
        referenceId: userId,
      },
    });
    if (existingSubscription) {
      return {
        success: true,
        subscription: existingSubscription,
      };
    }

    const subscription = await prisma.subscription.create({
      data: {
        referenceId: userId,
        plan: PLAN_ENUM.FREE,
        stripeCustomerId: stripeCustomerId,
        status: "active",
      },
    });

    return { success: true, subscription };
  } catch (error) {
    console.log(error);
    return {
      success: false,
      error: "Failed to create subscription",
    };
  }
}

export async function checkGenerationLimit(userId: string) {
  const subscription = await prisma.subscription.findFirst({
    where: {
      referenceId: userId,
      status: "active",
    },
  });

  if (!subscription) {
    throw new HTTPException(404, { message: "No active subscription" });
  }

  const plan = PLANS.find((p) => p.name === subscription.plan);
  if (!plan)
    throw new HTTPException(400, {
      message: "No Subscription or invalid plan",
    });

  const periodStart = subscription.periodStart ?? new Date(0);
  const periodEnd = subscription.periodEnd ?? new Date();

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

  const maxLimit = Math.max(0, plan.limits.generations - generationCount);

  const hasPaidSubscription = !!subscription.stripeSubscriptionId;

  return {
    isAllowed,
    hasPaidSubscription,
    plan: subscription.plan,
    generationsUsed: generationCount,
    generationsLimit:
      plan.limits.generations === Infinity ? null : plan.limits.generations,
    remainingGenerations:
      plan.limits.generations === Infinity ? "Unlimited" : maxLimit,
  };
}

4. 创建 lib/auth.ts 文件：
import { betterAuth } from "better-auth";
import { openAPI, bearer } from "better-auth/plugins";
import { prismaAdapter } from "better-auth/adapters/prisma";
import { stripe } from "@better-auth/stripe";
import { PrismaClient } from "@/generated/prisma";
import { stripeClient } from "./stripe";
import { PLANS } from "./constant";
import { createDefaultSubscription } from "@/app/actions/action";

const prisma = new PrismaClient();

export const auth = betterAuth({
  database: prismaAdapter(prisma, {
    provider: "postgresql",
  }),
  emailAndPassword: {
    enabled: true,
    minPasswordLength: 4,
  },
  plugins: [
    openAPI(),
    bearer(),
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

5. 创建 lib/auth-client.ts 文件：
import { createAuthClient } from "better-auth/react";
import { stripeClient } from "@better-auth/stripe/client";

export const authClient = createAuthClient({
  baseURL: process.env.NEXT_PUBLIC_APP_URL,
  plugins: [stripeClient({ subscription: true })],
});

6. 创建认证 API 路由 app/api/auth/[...all]/route.ts：
import { auth } from "@/lib/auth";
import { toNextJsHandler } from "better-auth/next-js";

export const { GET, POST } = toNextJsHandler(auth);

7. 更新 .env 文件，添加：
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."
STRIPE_PLUS_PLAN_ID="price_..."
STRIPE_PREMIUM_PLAN_ID="price_..."

完成后确认：
1. BetterAuth 配置文件已创建
2. Stripe 集成已配置
3. 认证 API 路由已创建
4. 环境变量已添加
```

### 预期结果
```
✓ lib/auth.ts 已创建
✓ lib/auth-client.ts 已创建
✓ app/api/auth/[...all]/route.ts 已创建
✓ Stripe 集成已配置
```

### 验证步骤
访问 `http://localhost:3000/api/auth/session` 应该返回 JSON 响应（未认证状态）

---

## ✅ 第1部分完成检查 / Part 1 Completion Checklist

完成以上所有提示词后，你应该有：

- [x] Next.js 15 项目已初始化
- [x] 所有依赖已安装
- [x] Prisma 数据库已配置
- [x] BetterAuth 认证已配置
- [x] Stripe 集成已配置
- [x] 项目可以启动 (npm run dev)

**下一步**: 继续 `02-backend-prompts.md` 构建后端 API
