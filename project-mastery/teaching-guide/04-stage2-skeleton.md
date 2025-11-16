# 📂 阶段2：理解结构 / Stage 2: Understanding Structure

> **目标**: 理解项目文件组织，快速定位代码
>
> **时长**: 2-3天  |  **难度**: ⭐️⭐️ 中等

---

## 🎯 学习目标

- ✅ 理解 `app/`, `components/`, `lib/` 等文件夹的作用
- ✅ 能在30秒内找到任意功能的代码位置
- ✅ 理解"组件"、"路由"、"API"的概念
- ✅ 掌握文件命名规则

---

## 📚 核心概念

参考：[概念词典 - 组件](./02-concept-dictionary.md#组件-component)

---

## 🗂️ 项目结构解析

```
Wave-Secondbrain-AI-Agent-Saas/
│
├── 📁 app/                    # 页面和API路由（Next.js App Router）
│   ├── (auth)/                # 认证相关页面（登录、注册）
│   ├── (main)/                # 主应用页面（需要登录）
│   ├── api/                   # 后端API端点
│   ├── layout.tsx             # 全局布局
│   └── page.tsx               # 首页
│
├── 📁 components/             # 可复用UI组件
│   ├── ui/                    # 基础UI（按钮、输入框）
│   ├── chat/                  # 聊天相关组件
│   └── note/                  # 笔记相关组件
│
├── 📁 lib/                    # 工具函数和配置
│   ├── ai/                    # AI相关（模型、工具）
│   ├── auth.ts                # 认证配置
│   ├── db.ts                  # 数据库客户端
│   └── utils.ts               # 通用工具
│
├── 📁 prisma/                 # 数据库
│   └── schema.prisma          # 数据库模型定义
│
├── 📁 public/                 # 静态资源（图片、字体）
│
└── 📁 hooks/                  # 自定义React Hooks
```

---

## 🎯 实践任务

### 任务1：寻宝游戏

找到以下功能的代码位置：

1. **登录页面**: `app/(auth)/sign-in/page.tsx`
2. **聊天界面**: `app/(main)/chat/[id]/page.tsx`
3. **笔记列表**: `app/(main)/note/page.tsx`
4. **AI聊天API**: `app/api/[[...route]]/chat.ts`
5. **创建笔记工具**: `lib/ai/tools/create-note.ts`

**验证清单**:
- [ ] 每个文件都能在VS Code中打开
- [ ] 能简单描述每个文件的作用

---

### 任务2：理解路由系统

**Next.js文件系统路由规则**:

| 文件路径 | URL路由 | 说明 |
|---------|---------|------|
| `app/page.tsx` | `/` | 首页 |
| `app/about/page.tsx` | `/about` | 关于页 |
| `app/blog/[id]/page.tsx` | `/blog/123` | 动态路由 |
| `app/(main)/chat/page.tsx` | `/chat` | 路由组（括号不影响URL） |

**AI协作提示词**:
```
基于项目的app文件夹结构，
请列出所有可访问的页面路由及其对应的文件位置。
```

---

## 📊 阶段评估

- [ ] 我能在30秒内找到"登录按钮"的代码
- [ ] 我理解什么是"组件"
- [ ] 我知道`app/`和`components/`的区别
- [ ] 我理解动态路由`[id]`的含义

**分数**: _____ / 4  （3分以上进入下一阶段）

---

## ➡️ 下一步

[阶段3：数据库连接](./05-stage3-core-features.md)

---

**相关文档**:
- [架构可视化](./01-architecture-visuals.md)
- [概念词典](./02-concept-dictionary.md)
