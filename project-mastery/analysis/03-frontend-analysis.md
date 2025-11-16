# 阶段4：前端架构分析 / Phase 4: Frontend Architecture Analysis

## 🎨 前端架构概览 / Frontend Architecture Overview

### 架构模式 / Architecture Pattern
- **框架**: Next.js 15.5.3 (App Router)
- **UI 库**: React 19.1.0
- **渲染模式**: Server Components + Client Components 混合
- **路由**: 文件系统路由 + 路由组 (Route Groups)
- **样式**: Tailwind CSS v4

### 核心特性 / Core Features
- ✅ Server-Side Rendering (SSR)
- ✅ 客户端流式 AI 响应
- ✅ 类型安全的 API 调用 (Hono RPC)
- ✅ 自动代码分割
- ✅ 响应式设计

---

## 📁 前端代码结构 / Frontend Code Structure

```
Wave-Secondbrain-AI-Agent-Saas/
├── app/
│   ├── (routes)/                      # 路由组
│   │   ├── (web)/                     # 公开页面
│   │   │   ├── page.tsx               # 首页
│   │   │   ├── layout.tsx             # Web 布局
│   │   │   └── _common/               # 共享组件
│   │   │       ├── nav-bar.tsx
│   │   │       ├── hero.tsx
│   │   │       └── app-preview.tsx
│   │   ├── (dashboard)/               # 仪表板 (需认证)
│   │   │   ├── layout.tsx             # Dashboard 布局
│   │   │   ├── home/                  # 主页
│   │   │   ├── chat/                  # 聊天
│   │   │   │   ├── page.tsx           # 新对话
│   │   │   │   └── [chatId]/          # 聊天详情
│   │   │   ├── billing/               # 订阅计费
│   │   │   ├── settings/              # 设置
│   │   │   └── _common/               # 共享组件
│   │   └── auth/                      # 认证
│   │       ├── layout.tsx
│   │       ├── sign-in/
│   │       ├── sign-up/
│   │       └── _common/
│   ├── layout.tsx                     # 根布局
│   └── globals.css                    # 全局样式
│
├── components/                        # React 组件
│   ├── ui/                            # 基础 UI 组件 (Shadcn/ui)
│   ├── chat/                          # 聊天相关
│   ├── sidebar/                       # 侧边栏
│   ├── note-dialog/                   # 笔记对话框
│   ├── ai-elements/                   # AI 渲染元素
│   ├── empty-state/                   # 空状态
│   └── logo/                          # Logo
│
├── context/                           # React Context
│   ├── providers.tsx                  # 根提供商
│   ├── theme-provider.tsx             # 主题
│   └── query-provider.tsx             # React Query
│
├── hooks/                             # 自定义 Hooks
│   ├── use-auth-token.ts              # 认证令牌 (Zustand)
│   ├── use-localchat.ts               # 聊天状态 (Zustand)
│   ├── use-note-id.ts                 # 笔记 ID (URL 状态)
│   └── use-view-state.ts              # 视图状态
│
└── features/                          # 功能模块 (React Query)
    ├── use-chat.ts                    # 聊天数据
    ├── use-note.ts                    # 笔记管理
    └── use-subscription.ts            # 订阅状态
```

---

## 🛣️ 路由结构分析 / Routing Structure

### 路由组 (Route Groups)

Next.js 13+ 的路由组允许在不影响 URL 的情况下组织文件。

#### (web) - 公开页面

**位置**: `app/(routes)/(web)/`

| 路由 | 文件 | 说明 |
|------|------|------|
| `/` | `page.tsx` | 首页 (Hero Section) |

**布局**: `layout.tsx` - 包含导航栏

**共享组件**:
- `nav-bar.tsx` - 顶部导航 (Logo + 登录/注册按钮)
- `hero.tsx` - Hero Section
- `app-preview.tsx` - 应用预览

---

#### (dashboard) - 仪表板页面 (需认证)

**位置**: `app/(routes)/(dashboard)/`

| 路由 | 文件 | 说明 |
|------|------|------|
| `/home` | `home/page.tsx` | 主页 (欢迎 + 最近笔记) |
| `/chat` | `chat/page.tsx` | 新建聊天 |
| `/chat/:chatId` | `chat/[chatId]/page.tsx` | 聊天详情 (动态路由) |
| `/billing` | `billing/page.tsx` | 订阅计费 |
| `/settings` | `settings/page.tsx` | 设置 |

**布局**: `layout.tsx`

**认证保护** (代码位置: `app/(routes)/(dashboard)/layout.tsx`):
```typescript
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function DashboardLayout({ children }) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    redirect("/auth/sign-in");  // 未认证用户重定向
  }

  return (
    <div className="flex h-screen">
      <AppSidebar />
      <main className="flex-1">
        {children}
      </main>
    </div>
  );
}
```

**共享组件**:
- `header.tsx` - Dashboard 顶部栏
- `main-content.tsx` - 主内容容器
- `chat-history.tsx` - 聊天历史侧边栏

---

#### auth - 认证页面

**位置**: `app/(routes)/auth/`

| 路由 | 文件 | 说明 |
|------|------|------|
| `/auth/sign-in` | `sign-in/page.tsx` | 登录页 |
| `/auth/sign-up` | `sign-up/page.tsx` | 注册页 |

**布局**: `layout.tsx`

**重定向逻辑** (已认证用户):
```typescript
const session = await auth.api.getSession();
if (session) {
  redirect("/home");  // 已登录用户跳转到首页
}
```

---

### 动态路由 / Dynamic Routes

#### [chatId] - 聊天详情

**文件位置**: `app/(routes)/(dashboard)/chat/[chatId]/page.tsx`

**实现**:
```typescript
"use client";

export default function ChatPage() {
  const params = useParams<{ chatId: string }>();
  const chatId = params.chatId;

  const { data, isLoading } = useChatById({
    id: chatId,
    enabled: !!chatId,
  });

  if (isLoading) return <LoaderOverlay />;

  const messages = data?.data?.messages || [];

  return (
    <ChatInterface
      id={chatId}
      initialMessages={messages}
    />
  );
}
```

**数据获取**: 使用 React Query 的 `useChatById` hook

---

## 🧩 组件层次分析 / Component Hierarchy

### UI 基础组件 (Shadcn/ui)

**位置**: `components/ui/`

基于 Radix UI 的可访问性优先组件库，使用 Tailwind CSS 样式化。

| 组件 | 说明 | Radix 基础 |
|------|------|-----------|
| `button.tsx` | 按钮 | - |
| `input.tsx` | 输入框 | - |
| `textarea.tsx` | 文本域 | - |
| `label.tsx` | 标签 | @radix-ui/react-label |
| `dialog.tsx` | 对话框 | @radix-ui/react-dialog |
| `dropdown-menu.tsx` | 下拉菜单 | @radix-ui/react-dropdown-menu |
| `select.tsx` | 选择器 | @radix-ui/react-select |
| `popover.tsx` | 弹出框 | @radix-ui/react-popover |
| `separator.tsx` | 分隔符 | @radix-ui/react-separator |
| `avatar.tsx` | 头像 | @radix-ui/react-avatar |
| `scroll-area.tsx` | 滚动区域 | @radix-ui/react-scroll-area |
| `sidebar.tsx` | 侧边栏 | @radix-ui/react-collapsible |
| `tooltip.tsx` | 工具提示 | @radix-ui/react-tooltip |
| `hover-card.tsx` | 悬停卡片 | @radix-ui/react-hover-card |
| `progress.tsx` | 进度条 | @radix-ui/react-progress |

**表单组件**:
```typescript
// components/ui/form.tsx
- FormField      // React Hook Form 集成
- FormItem       // 表单项容器
- FormLabel      // 表单标签
- FormControl    // 表单控件包装器
- FormDescription // 描述文本
- FormMessage    // 错误消息
```

**工具函数**:
```typescript
// lib/utils.ts
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}
```

**用途**: 合并 CSS 类名，避免冲突

---

### 聊天组件 / Chat Components

**位置**: `components/chat/`

#### 主组件: ChatInterface

**文件位置**: `components/chat/index.tsx`

**核心功能**:
```typescript
import { useChat } from "@ai-sdk/react";

export function ChatInterface({ id, initialMessages }) {
  const { localModelId } = useLocalChat();
  const { data: generationsData } = useCheckGenerations();

  const {
    messages,
    setMessages,
    sendMessage,
    status,
    stop,
    error,
  } = useChat<UIMessage>({
    id,
    messages: initialMessages,
    generateId: generateUUID,
    transport: new DefaultChatTransport({
      api: "/api/chat",
      prepareSendMessagesRequest({ messages, id, body }) {
        return {
          body: {
            id,
            message: messages.at(-1),
            selectedModelId: localModelId,
            selectedToolName: null,
            ...body,
          },
        };
      },
    }),
    async onToolCall({ toolCall }) {
      if (toolCall.toolName === ToolNameEnum.CreateNote) {
        queryClient.invalidateQueries({ queryKey: ["notes"] });
      }
    },
  });

  return (
    <div className="flex flex-col h-screen">
      <ChatMessages messages={messages} />
      <ChatInput
        onSend={sendMessage}
        isLoading={status === "streaming"}
        generationsLimit={generationsData}
      />
    </div>
  );
}
```

**AI SDK 集成**:
- `useChat` hook 处理流式响应
- 自动管理消息状态
- 支持工具调用 (Tool Calling)
- 生成唯一消息 ID

---

#### 子组件

| 组件 | 文件 | 说明 |
|------|------|------|
| **ChatMessages** | `chat-messages.tsx` | 消息列表容器 |
| **Message** | `message.tsx` | 单条消息渲染 |
| **ChatInput** | `chat-input.tsx` | 输入框 + 工具选择器 |
| **MessageAction** | `message-action.tsx` | 消息操作 (复制、编辑) |
| **ToolCall** | `tool-call.tsx` | 工具调用显示 |
| **ToolStatus** | `tool-status.tsx` | 工具执行状态 |
| **ToolNotePreview** | `tool-note-preview.tsx` | 笔记预览卡片 |
| **ToolSearchExtractPreview** | `tool-search-extract-preview.tsx` | 搜索/提取结果 |

---

### AI 元素组件 / AI Elements

**位置**: `components/ai-elements/`

这些组件用于渲染 AI 响应的不同部分。

| 组件 | 说明 |
|------|------|
| `response.tsx` | AI 响应容器 |
| `message.tsx` | 消息模板 |
| `code-block.tsx` | 代码块 + 语法高亮 (react-syntax-highlighter) |
| `artifact.tsx` | 工件显示 |
| `tool.tsx` | 工具调用卡片 |
| `task.tsx` | 任务显示 |
| `reasoning.tsx` | 推理过程 |
| `chain-of-thought.tsx` | 思维链 |
| `web-preview.tsx` | Web 预览 |
| `image.tsx` | 图片渲染 |
| `sources.tsx` | 引用来源 |
| `suggestion.tsx` | 建议 |

**代码高亮示例** (`code-block.tsx`):
```typescript
import { Prism as SyntaxHighlighter } from "react-syntax-highlighter";
import { oneDark } from "react-syntax-highlighter/dist/cjs/styles/prism";

export function CodeBlock({ code, language }) {
  return (
    <SyntaxHighlighter
      language={language}
      style={oneDark}
      showLineNumbers
    >
      {code}
    </SyntaxHighlighter>
  );
}
```

---

### 侧边栏组件 / Sidebar Components

**位置**: `components/sidebar/`

#### AppSidebar (主组件)

**文件位置**: `components/sidebar/index.tsx`

**结构**:
```typescript
import { Sidebar, SidebarContent, SidebarHeader, SidebarFooter } from "@/components/ui/sidebar";

export function AppSidebar() {
  return (
    <Sidebar>
      <SidebarHeader>
        <Logo />
      </SidebarHeader>

      <SidebarContent>
        <NavMenu />      {/* 主导航 */}
        <NavNotes />     {/* 笔记列表 */}
      </SidebarContent>

      <SidebarFooter>
        <NavUser />      {/* 用户菜单 */}
      </SidebarFooter>
    </Sidebar>
  );
}
```

#### NavMenu - 导航菜单

**文件位置**: `components/sidebar/nav-menu.tsx`

**菜单项**:
```typescript
const menuItems = [
  { icon: Home, label: "Home", href: "/home" },
  { icon: MessageSquare, label: "Chat", href: "/chat" },
  { icon: CreditCard, label: "Billing", href: "/billing" },
  { icon: Settings, label: "Settings", href: "/settings" },
];
```

#### NavNotes - 笔记列表

**文件位置**: `components/sidebar/nav-notes.tsx`

**功能**:
```typescript
export function NavNotes() {
  const { data, isLoading } = useNotes(1, 20);
  const { mutate: createNote, isPending } = useCreateNote();
  const { setNoteId } = useNoteId();

  const handleCreate = () => {
    createNote({
      title: "Untitled Note",
      content: "",
    });
  };

  const handleNoteClick = (noteId: string) => {
    setNoteId(noteId);  // 打开 Sheet 对话框
  };

  return (
    <div>
      <Button onClick={handleCreate}>
        <Plus /> New Note
      </Button>

      {notes.map((note) => (
        <div key={note.id} onClick={() => handleNoteClick(note.id)}>
          {note.title}
        </div>
      ))}
    </div>
  );
}
```

**React Query 集成**: 自动缓存和刷新笔记列表

---

### 笔记对话框 / Note Dialog

**位置**: `components/note-dialog/`

#### NoteDialog (主组件)

**文件位置**: `components/note-dialog/note-dialog.tsx`

**实现**:
```typescript
import { Sheet, SheetContent } from "@/components/ui/sheet";
import { useNoteId } from "@/hooks/use-note-id";

export function NoteDialog() {
  const { noteId, setNoteId } = useNoteId();
  const isOpen = !!noteId;

  const handleClose = () => {
    setNoteId(null);  // 清除 URL 参数
  };

  return (
    <Sheet open={isOpen} onOpenChange={handleClose}>
      <SheetContent side="right" className="w-full md:w-1/2">
        {noteId && <NoteView noteId={noteId} />}
      </SheetContent>
    </Sheet>
  );
}
```

**URL 状态管理**: 使用 `nuqs` 库将 noteId 存储在 URL 查询参数中，支持深链接

#### NoteView - 笔记编辑视图

**文件位置**: `components/note-dialog/note-view.tsx`

**功能**:
- 查询笔记详情 (`useNote`)
- 实时编辑 (防抖更新)
- 删除笔记 (`useDeleteNote`)

---

## 🔄 状态管理 / State Management

### 1. Zustand Stores

#### 认证令牌存储

**文件位置**: `hooks/use-auth-token.ts`

```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";

interface AuthTokenState {
  bearerToken: string | null;
  setBearerToken: (token: string | null) => void;
  clearBearerToken: () => void;
}

export const useAuthToken = create<AuthTokenState>()(
  persist(
    (set) => ({
      bearerToken: null,
      setBearerToken: (token) => set({ bearerToken: token }),
      clearBearerToken: () => set({ bearerToken: null }),
    }),
    {
      name: "auth-storage",  // localStorage key
    }
  )
);
```

**用途**:
- 存储 BetterAuth 的 Bearer Token
- 在 API 调用中使用 (`authClient` 配置)
- 登录时设置，登出时清除

---

#### 本地聊天状态

**文件位置**: `hooks/use-localchat.ts`

```typescript
interface LocalChatState {
  localModelId: string;            // 当前选中的 AI 模型
  isHistoryOpen: boolean;          // 聊天历史侧边栏状态
  onToggleHistory: () => void;
  setLocalModelId: (id: string) => void;
}

export const useLocalChat = create<LocalChatState>()(
  persist(
    (set, get) => ({
      localModelId: DEFAULT_MODEL_ID,
      isHistoryOpen: true,
      onToggleHistory: () => set({ isHistoryOpen: !get().isHistoryOpen }),
      setLocalModelId: (id) => set({ localModelId: id }),
    }),
    {
      name: "local-chat",
    }
  )
);
```

**用途**:
- 用户选择的 AI 模型 (Claude / Grok / GPT-4 / Gemini)
- UI 状态 (侧边栏展开/收起)

---

### 2. URL 状态 (Nuqs)

#### 笔记 ID 状态

**文件位置**: `hooks/use-note-id.ts`

```typescript
import { parseAsString, useQueryState } from "nuqs";

export function useNoteId() {
  const [noteId, setNoteId] = useQueryState(
    "noteId",
    parseAsString.withDefault("")
  );

  return {
    noteId: noteId || null,
    setNoteId,
  };
}
```

**URL 示例**:
- 关闭笔记: `/home`
- 打开笔记: `/home?noteId=note-uuid-123`

**优势**:
- 支持浏览器前进/后退
- 可分享的深链接
- 刷新页面后状态保持

---

#### 视图状态

**文件位置**: `hooks/use-view-state.ts`

```typescript
export function useViewState() {
  const [isChatView, setIsChatView] = useQueryState(
    "isChatView",
    parseAsBoolean.withDefault(false)
  );

  return { isChatView, setIsChatView };
}
```

---

### 3. React Query (TanStack Query)

**配置位置**: `context/query-provider.tsx`

```typescript
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      refetchOnWindowFocus: false,
      staleTime: 1000 * 60 * 5,  // 5 分钟
    },
  },
});

export function QueryProvider({ children }) {
  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

**使用的 Query Keys**:
```typescript
["chat", chatId]           // 单个聊天
["chats"]                  // 聊天列表
["note", noteId]           // 单个笔记
["notes", page, limit]     // 笔记列表 (分页)
["generations"]            // 生成限制
```

---

## 📡 数据获取模式 / Data Fetching Patterns

### Hono RPC 客户端

**配置位置**: `lib/hono/hono-rpc.ts`

```typescript
import { hc } from "hono/client";
import type { AppType } from "@/app/api/[[...route]]/route";

const NEXT_PUBLIC_APP_URL = process.env.NEXT_PUBLIC_APP_URL || "http://localhost:3000";

export const api = hc<AppType>(NEXT_PUBLIC_APP_URL).api;
```

**类型安全调用**:
```typescript
// 完全类型推断
const response = await api.note.create.$post({
  json: {
    title: "My Note",
    content: "Content...",
  },
});

const data = await response.json();
// data.success: boolean
// data.data: Note
```

---

### React Query Hooks

#### 聊天数据

**文件位置**: `features/use-chat.ts`

```typescript
// 获取单个聊天
export const useChatById = ({ id, enabled }) => {
  return useQuery({
    queryKey: ["chat", id],
    queryFn: async () => {
      const response = await api.chat[":id"].$get({
        param: { id },
      });
      if (!response.ok) throw new Error("Failed to fetch chat");
      return response.json();
    },
    enabled,
  });
};

// 获取所有聊天
export const useChats = () => {
  return useQuery({
    queryKey: ["chats"],
    queryFn: async () => {
      const response = await api.chat.$get();
      return response.json();
    },
  });
};
```

---

#### 笔记管理

**文件位置**: `features/use-note.ts`

```typescript
// 创建笔记 (Mutation)
export const useCreateNote = () => {
  const queryClient = useQueryClient();
  const { setNoteId } = useNoteId();

  return useMutation({
    mutationFn: async (json: { title: string; content: string }) => {
      const response = await api.note.create.$post({ json });
      if (!response.ok) throw new Error("Failed to create note");
      return response.json();
    },
    onSuccess: (response) => {
      toast.success("Note created");
      setNoteId(response.data.id);  // 打开新笔记
      queryClient.invalidateQueries({ queryKey: ["notes"] });
    },
    onError: (error) => {
      toast.error(error.message);
    },
  });
};

// 更新笔记
export const useUpdateNote = () => {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: async ({ id, json }) => {
      const response = await api.note.update[":id"].$patch({
        param: { id },
        json,
      });
      return response.json();
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["notes"] });
      toast.success("Note updated");
    },
  });
};

// 删除笔记
export const useDeleteNote = () => {
  const queryClient = useQueryClient();
  const { setNoteId } = useNoteId();

  return useMutation({
    mutationFn: async (id: string) => {
      const response = await api.note.delete[":id"].$delete({
        param: { id },
      });
      return response.json();
    },
    onSuccess: () => {
      setNoteId(null);  // 关闭笔记
      queryClient.invalidateQueries({ queryKey: ["notes"] });
      toast.success("Note deleted");
    },
  });
};

// 获取笔记列表 (分页)
export const useNotes = (page = 1, limit = 20) => {
  return useQuery({
    queryKey: ["notes", page, limit],
    queryFn: async () => {
      const response = await api.note.all.$get({
        query: { page: page.toString(), limit: limit.toString() },
      });
      return response.json();
    },
  });
};

// 获取单个笔记
export const useNote = (id: string) => {
  return useQuery({
    queryKey: ["note", id],
    queryFn: async () => {
      const response = await api.note[":id"].$get({
        param: { id },
      });
      return response.json();
    },
    enabled: !!id,
  });
};
```

---

#### 订阅状态

**文件位置**: `features/use-subscription.ts`

```typescript
// 检查生成限制
export const useCheckGenerations = () => {
  return useQuery({
    queryKey: ["generations"],
    queryFn: async () => {
      const response = await api.subscription.generations.$get();
      return response.json();
    },
    refetchOnMount: true,
    refetchOnReconnect: true,
  });
};

// 升级订阅
export const useUpgradeSubscription = () => {
  return useMutation({
    mutationFn: async (json: { plan: string; callbackUrl: string }) => {
      const response = await api.subscription.upgrade.$post({ json });
      if (!response.ok) throw new Error("Failed to create checkout");
      return response.json();
    },
    onSuccess: (data) => {
      // 重定向到 Stripe Checkout
      window.location.href = data.checkoutUrl;
    },
    onError: (error) => {
      toast.error(error.message);
    },
  });
};
```

---

## 📝 表单处理 / Form Handling

### React Hook Form + Zod

#### 注册表单

**文件位置**: `app/(routes)/auth/_common/signup-form.tsx`

```typescript
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const signUpSchema = z.object({
  name: z.string().min(2, "Name must be at least 2 characters"),
  email: z.string().email("Invalid email").min(1, "Email required"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

type SignUpFormValues = z.infer<typeof signUpSchema>;

export function SignUpForm() {
  const [loading, setLoading] = useState(false);
  const router = useRouter();

  const form = useForm<SignUpFormValues>({
    resolver: zodResolver(signUpSchema),
    defaultValues: {
      name: "",
      email: "",
      password: "",
    },
  });

  const onSubmit = async (values: SignUpFormValues) => {
    setLoading(true);
    try {
      const { data, error } = await authClient.signUp.email({
        name: values.name,
        email: values.email,
        password: values.password,
      });

      if (error) {
        toast.error(error.message);
        return;
      }

      toast.success("Account created successfully");
      router.replace("/home");
    } catch (error) {
      toast.error("Something went wrong");
    } finally {
      setLoading(false);
    }
  };

  return (
    <Form {...form}>
      <form onSubmit={form.handleSubmit(onSubmit)}>
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />
        {/* email 和 password 字段类似 */}

        <Button type="submit" disabled={loading}>
          {loading ? "Creating..." : "Sign Up"}
        </Button>
      </form>
    </Form>
  );
}
```

---

#### 登录表单

**文件位置**: `app/(routes)/auth/_common/signin-form.tsx`

```typescript
const signInSchema = z.object({
  email: z.string().email("Invalid email").min(1, "Email required"),
  password: z.string().min(6, "Password must be at least 6 characters"),
});

export function SignInForm() {
  const { setBearerToken } = useAuthToken();

  const onSubmit = async (values: SignInFormValues) => {
    const { data, error, response } = await authClient.signIn.email({
      email: values.email,
      password: values.password,
    });

    if (error) {
      toast.error(error.message);
      return;
    }

    // 提取 Bearer Token
    const token = response?.headers.get("Authorization")?.replace("Bearer ", "");
    if (token) {
      setBearerToken(token);
    }

    router.replace("/home");
  };

  // ... 表单 JSX
}
```

**BetterAuth 集成**: 使用 `authClient` 提供的方法进行认证

---

## 🎨 样式系统 / Styling System

### Tailwind CSS v4

**配置位置**: `postcss.config.mjs`

```javascript
export default {
  plugins: {
    "@tailwindcss/postcss": {},
  },
};
```

**全局样式**: `app/globals.css`

```css
@import "tailwindcss";

/* 自定义 CSS 变量 */
@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 222.2 84% 4.9%;
    --primary: 221.2 83.2% 53.3%;
    /* ... 更多颜色变量 */
  }

  .dark {
    --background: 222.2 84% 4.9%;
    --foreground: 210 40% 98%;
    /* ... 深色模式颜色 */
  }
}
```

---

### 主题系统 / Theme System

**库**: `next-themes`

**配置位置**: `context/theme-provider.tsx`

```typescript
import { ThemeProvider as NextThemesProvider } from "next-themes";

export function ThemeProvider({ children }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="system"
      enableSystem
      disableTransitionOnChange
    >
      {children}
    </NextThemesProvider>
  );
}
```

**使用示例** (设置页面):
```typescript
import { useTheme } from "next-themes";

export function SettingsPage() {
  const { theme, setTheme } = useTheme();

  return (
    <div>
      <Button onClick={() => setTheme("light")}>Light</Button>
      <Button onClick={() => setTheme("dark")}>Dark</Button>
      <Button onClick={() => setTheme("system")}>System</Button>
    </div>
  );
}
```

**支持的主题**:
- `light` - 浅色模式
- `dark` - 深色模式
- `system` - 跟随系统

---

## 🤖 AI SDK 集成 / AI SDK Integration

### useChat Hook

**来源**: `@ai-sdk/react`

**核心功能**:
```typescript
const {
  messages,        // 消息数组
  setMessages,     // 设置消息
  sendMessage,     // 发送新消息
  status,          // "streaming" | "idle" | "loading"
  stop,            // 停止流式响应
  error,           // 错误对象
} = useChat<UIMessage>({
  id: chatId,
  messages: initialMessages,
  generateId: generateUUID,

  // 自定义传输
  transport: new DefaultChatTransport({
    api: "/api/chat",
    prepareSendMessagesRequest({ messages, id, body }) {
      return {
        body: {
          id,
          message: messages.at(-1),
          selectedModelId: localModelId,
          selectedToolName: null,
          ...body,
        },
      };
    },
  }),

  // 工具调用回调
  async onToolCall({ toolCall }) {
    if (toolCall.toolName === ToolNameEnum.CreateNote) {
      // 刷新笔记列表
      queryClient.invalidateQueries({ queryKey: ["notes"] });
    }
  },
});
```

**消息类型** (`UIMessage`):
```typescript
interface UIMessage {
  id: string;
  role: "user" | "assistant" | "system";
  parts: UIMessagePart[];
  metadata?: {
    createdAt: Date;
  };
}

type UIMessagePart =
  | { type: "text"; text: string }
  | { type: "tool-call"; toolName: string; args: any; result?: any }
  | { type: "tool-result"; toolName: string; result: any };
```

---

### 流式响应处理

**客户端**:
```typescript
// AI SDK 自动处理 SSE 流
sendMessage({ content: "Hello AI" });

// 每接收到新 token 就更新 messages
// status 自动从 "loading" → "streaming" → "idle"
```

**服务器端** (参考后端分析):
```typescript
// app/api/[[...route]]/chat.ts
const result = streamText({
  model: modelProvider,
  messages: modelMessages,
  tools: { /* ... */ },
});

return result.toUIMessageStreamResponse({
  sendSources: true,
  generateMessageId: () => generateUUID(),
  onFinish: async ({ messages }) => {
    // 保存消息到数据库
  },
});
```

---

## 📱 响应式设计 / Responsive Design

### 布局断点 / Breakpoints

Tailwind CSS 默认断点:
```css
sm:  640px   /* 小型设备 */
md:  768px   /* 中型设备 */
lg:  1024px  /* 大型设备 */
xl:  1280px  /* 超大型设备 */
2xl: 1536px  /* 超超大型设备 */
```

### 响应式组件示例

**侧边栏**:
```typescript
<Sidebar className="hidden md:block w-64">
  {/* 桌面端显示 */}
</Sidebar>

<Sheet>
  <SheetTrigger className="md:hidden">
    <Menu />  {/* 移动端显示汉堡菜单 */}
  </SheetTrigger>
  <SheetContent side="left">
    {/* 移动端侧边栏 */}
  </SheetContent>
</Sheet>
```

**笔记对话框**:
```typescript
<SheetContent className="w-full md:w-1/2">
  {/* 移动端全屏，桌面端半屏 */}
</SheetContent>
```

---

## 🔄 客户端 vs 服务器组件 / Client vs Server Components

### Server Components (默认)

**位置**: 所有 `layout.tsx` 和 `page.tsx` (除非标记 `"use client"`)

**优势**:
- 直接访问数据库 (Prisma)
- 调用 BetterAuth API (`await auth.api.getSession()`)
- 减少客户端 JavaScript
- SEO 友好

**示例**:
```typescript
// app/(routes)/(dashboard)/layout.tsx
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";

export default async function DashboardLayout({ children }) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    redirect("/auth/sign-in");
  }

  return <div>{children}</div>;
}
```

---

### Client Components ("use client")

**标记**: 文件顶部添加 `"use client"`

**何时使用**:
- 使用 React Hooks (useState, useEffect, etc.)
- 事件处理 (onClick, onChange, etc.)
- 浏览器 API (localStorage, window, etc.)
- Context 消费
- Zustand / React Query

**示例**:
```typescript
// components/chat/index.tsx
"use client";

import { useChat } from "@ai-sdk/react";
import { useLocalChat } from "@/hooks/use-localchat";

export function ChatInterface() {
  const { localModelId } = useLocalChat();
  const { messages, sendMessage } = useChat({ /* ... */ });

  return <div>{/* ... */}</div>;
}
```

---

## ✅ 阶段4完成标记 / Phase 4 Completion

✓ 路由结构图
✓ 组件层次树
✓ 状态管理架构
✓ 数据流图 (前端到后端)
✓ 关键交互流程
✓ AI SDK 集成详解
✓ 表单处理模式
✓ 响应式设计

---

**生成时间**: 2025-11-16
**分析版本**: v1.0
**下一阶段**: 安全实现分析 (`04-security-analysis.md`)
