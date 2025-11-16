# 前端组件重建提示词 / Frontend Component Reconstruction Prompts

## 📋 使用说明 / Instructions

这些提示词将帮助你重建完整的前端界面系统。按顺序执行，每个提示词都包含完整的组件代码和配置。

---

## 提示词 3.1: 配置状态管理系统 / Setup State Management

```
我需要配置前端状态管理系统。请帮我创建：

**1. Zustand 状态 - 认证令牌** (hooks/use-auth-token.ts):
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
      name: "auth-storage", // localStorage key
    }
  )
);
```

**2. Zustand 状态 - 本地聊天状态** (hooks/use-localchat.ts):
```typescript
import { create } from "zustand";
import { persist } from "zustand/middleware";
import { DEFAULT_MODEL_ID } from "@/lib/ai/models";

interface LocalChatState {
  localModelId: string;
  isHistoryOpen: boolean;
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

**3. URL 状态 - 笔记 ID** (hooks/use-note-id.ts):
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

**4. React Query 配置** (context/query-provider.tsx):
```typescript
"use client";

import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useState } from "react";

export function QueryProvider({ children }: { children: React.ReactNode }) {
  const [queryClient] = useState(
    () =>
      new QueryClient({
        defaultOptions: {
          queries: {
            refetchOnWindowFocus: false,
            staleTime: 1000 * 60 * 5, // 5 分钟
          },
        },
      })
  );

  return (
    <QueryClientProvider client={queryClient}>
      {children}
    </QueryClientProvider>
  );
}
```

**5. 主题提供商** (context/theme-provider.tsx):
```typescript
"use client";

import { ThemeProvider as NextThemesProvider } from "next-themes";

export function ThemeProvider({ children }: { children: React.ReactNode }) {
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

**6. 根提供商** (context/providers.tsx):
```typescript
"use client";

import { NuqsAdapter } from "nuqs/adapters/next/app";
import { QueryProvider } from "./query-provider";
import { ThemeProvider } from "./theme-provider";
import { Toaster } from "sonner";

export function Providers({ children }: { children: React.ReactNode }) {
  return (
    <NuqsAdapter>
      <QueryProvider>
        <ThemeProvider>
          {children}
          <Toaster position="top-center" />
        </ThemeProvider>
      </QueryProvider>
    </NuqsAdapter>
  );
}
```

依赖包:
- zustand
- nuqs
- @tanstack/react-query
- next-themes
- sonner
```

**预期结果**:
- Zustand stores 配置完成
- React Query 配置完成
- URL 状态管理可用
- 主题切换可用

---

## 提示词 3.2: 创建 React Query Hooks / Create React Query Hooks

```
创建数据获取 hooks (features/):

**1. 聊天数据 hooks** (features/use-chat.ts):
```typescript
import { useQuery } from "@tanstack/react-query";
import { api } from "@/lib/hono/hono-rpc";

// 获取单个聊天
export const useChatById = ({ id, enabled }: { id: string; enabled: boolean }) => {
  return useQuery({
    queryKey: ["chat", id],
    queryFn: async () => {
      const response = await api.chat[":id"].$get({ param: { id } });
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

**2. 笔记管理 hooks** (features/use-note.ts):
```typescript
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { api } from "@/lib/hono/hono-rpc";
import { toast } from "sonner";
import { useNoteId } from "@/hooks/use-note-id";

// 创建笔记
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
      setNoteId(response.data.id);
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
    mutationFn: async ({ id, json }: { id: string; json: any }) => {
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
      setNoteId(null);
      queryClient.invalidateQueries({ queryKey: ["notes"] });
      toast.success("Note deleted");
    },
  });
};

// 获取笔记列表
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
      const response = await api.note[":id"].$get({ param: { id } });
      return response.json();
    },
    enabled: !!id,
  });
};
```

**3. 订阅状态 hooks** (features/use-subscription.ts):
```typescript
import { useMutation, useQuery } from "@tanstack/react-query";
import { api } from "@/lib/hono/hono-rpc";
import { toast } from "sonner";

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
      window.location.href = data.checkoutUrl;
    },
    onError: (error) => {
      toast.error(error.message);
    },
  });
};
```
```

**预期结果**: 所有数据操作的 React Query hooks 可用

---

## 提示词 3.3: 创建认证页面 / Create Authentication Pages

```
创建认证相关页面和组件：

**1. 注册表单** (app/(routes)/auth/_common/signup-form.tsx):
```typescript
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";
import { toast } from "sonner";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  Form,
  FormControl,
  FormField,
  FormItem,
  FormLabel,
  FormMessage,
} from "@/components/ui/form";

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
      <form onSubmit={form.handleSubmit(onSubmit)} className="space-y-4">
        <FormField
          control={form.control}
          name="name"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Name</FormLabel>
              <FormControl>
                <Input {...field} placeholder="John Doe" />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="email"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Email</FormLabel>
              <FormControl>
                <Input {...field} type="email" placeholder="you@example.com" />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <FormField
          control={form.control}
          name="password"
          render={({ field }) => (
            <FormItem>
              <FormLabel>Password</FormLabel>
              <FormControl>
                <Input {...field} type="password" placeholder="••••••" />
              </FormControl>
              <FormMessage />
            </FormItem>
          )}
        />

        <Button type="submit" disabled={loading} className="w-full">
          {loading ? "Creating..." : "Sign Up"}
        </Button>
      </form>
    </Form>
  );
}
```

**2. 登录表单** (app/(routes)/auth/_common/signin-form.tsx):
类似结构，但使用 authClient.signIn.email() 并保存 Bearer Token:

```typescript
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
```

**3. 注册页面** (app/(routes)/auth/sign-up/page.tsx):
```typescript
import Link from "next/link";
import { SignUpForm } from "../_common/signup-form";

export default function SignUpPage() {
  return (
    <div className="flex min-h-screen items-center justify-center">
      <div className="w-full max-w-md space-y-8 p-8">
        <div className="text-center">
          <h1 className="text-2xl font-bold">Create an account</h1>
          <p className="text-muted-foreground">Get started with your free account</p>
        </div>

        <SignUpForm />

        <p className="text-center text-sm">
          Already have an account?{" "}
          <Link href="/auth/sign-in" className="text-primary hover:underline">
            Sign in
          </Link>
        </p>
      </div>
    </div>
  );
}
```

**4. 认证布局** (app/(routes)/auth/layout.tsx):
```typescript
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";
import { headers } from "next/headers";

export default async function AuthLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (session) {
    redirect("/home"); // 已登录用户跳转
  }

  return <>{children}</>;
}
```
```

**预期结果**:
- 注册页面: `/auth/sign-up`
- 登录页面: `/auth/sign-in`
- 已登录用户自动重定向

---

## 提示词 3.4: 创建聊天界面 / Create Chat Interface

```
创建完整的聊天界面系统：

**1. 主聊天组件** (components/chat/index.tsx):
```typescript
"use client";

import { useChat } from "@ai-sdk/react";
import { DefaultChatTransport } from "@ai-sdk/react";
import { generateUUID } from "@/lib/utils";
import { useLocalChat } from "@/hooks/use-localchat";
import { useCheckGenerations } from "@/features/use-subscription";
import { useQueryClient } from "@tanstack/react-query";
import { ToolNameEnum } from "@/lib/ai/tools/constant";
import { ChatMessages } from "./chat-messages";
import { ChatInput } from "./chat-input";

interface ChatInterfaceProps {
  id: string;
  initialMessages: any[];
}

export function ChatInterface({ id, initialMessages }: ChatInterfaceProps) {
  const { localModelId } = useLocalChat();
  const { data: generationsData } = useCheckGenerations();
  const queryClient = useQueryClient();

  const {
    messages,
    setMessages,
    sendMessage,
    status,
    stop,
    error,
  } = useChat({
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

**2. 消息列表** (components/chat/chat-messages.tsx):
```typescript
"use client";

import { ScrollArea } from "@/components/ui/scroll-area";
import { Message } from "./message";

export function ChatMessages({ messages }: { messages: any[] }) {
  return (
    <ScrollArea className="flex-1 p-4">
      <div className="space-y-4">
        {messages.map((message) => (
          <Message key={message.id} message={message} />
        ))}
      </div>
    </ScrollArea>
  );
}
```

**3. 单条消息** (components/chat/message.tsx):
```typescript
"use client";

import { cn } from "@/lib/utils";
import { Avatar, AvatarFallback } from "@/components/ui/avatar";

export function Message({ message }: { message: any }) {
  const isUser = message.role === "user";

  return (
    <div className={cn("flex gap-3", isUser && "justify-end")}>
      {!isUser && (
        <Avatar>
          <AvatarFallback>AI</AvatarFallback>
        </Avatar>
      )}

      <div
        className={cn(
          "rounded-lg px-4 py-2 max-w-[80%]",
          isUser ? "bg-primary text-primary-foreground" : "bg-muted"
        )}
      >
        {message.parts.map((part: any, index: number) => {
          if (part.type === "text") {
            return <p key={index}>{part.text}</p>;
          }
          // 处理其他类型 (tool-call, tool-result 等)
          return null;
        })}
      </div>

      {isUser && (
        <Avatar>
          <AvatarFallback>U</AvatarFallback>
        </Avatar>
      )}
    </div>
  );
}
```

**4. 输入框** (components/chat/chat-input.tsx):
```typescript
"use client";

import { useState } from "react";
import { Button } from "@/components/ui/button";
import { Textarea } from "@/components/ui/textarea";
import { Send } from "lucide-react";

export function ChatInput({ onSend, isLoading, generationsLimit }: any) {
  const [input, setInput] = useState("");

  const handleSubmit = (e: React.FormEvent) => {
    e.preventDefault();
    if (!input.trim() || isLoading) return;

    onSend({ content: input });
    setInput("");
  };

  const canSend = generationsLimit?.data?.isAllowed !== false;

  return (
    <form onSubmit={handleSubmit} className="border-t p-4">
      {!canSend && (
        <p className="text-sm text-destructive mb-2">
          Generation limit reached. Please upgrade your plan.
        </p>
      )}

      <div className="flex gap-2">
        <Textarea
          value={input}
          onChange={(e) => setInput(e.target.value)}
          placeholder="Type your message..."
          disabled={isLoading || !canSend}
          className="min-h-[60px]"
        />
        <Button type="submit" disabled={isLoading || !canSend}>
          <Send className="h-4 w-4" />
        </Button>
      </div>
    </form>
  );
}
```

**5. 聊天页面** (app/(routes)/(dashboard)/chat/[chatId]/page.tsx):
```typescript
"use client";

import { useParams } from "next/navigation";
import { useChatById } from "@/features/use-chat";
import { ChatInterface } from "@/components/chat";
import { LoaderOverlay } from "@/components/loader-overlay";

export default function ChatPage() {
  const params = useParams<{ chatId: string }>();
  const chatId = params.chatId;

  const { data, isLoading } = useChatById({
    id: chatId,
    enabled: !!chatId,
  });

  if (isLoading) return <LoaderOverlay />;

  const messages = data?.data?.messages || [];

  return <ChatInterface id={chatId} initialMessages={messages} />;
}
```
```

**预期结果**:
- 完整的聊天界面
- 流式 AI 响应
- 消息历史显示
- 生成限制提示

---

## 提示词 3.5: 创建侧边栏系统 / Create Sidebar System

```
创建应用侧边栏及相关组件：

**1. 主侧边栏** (components/sidebar/index.tsx):
```typescript
"use client";

import { Sidebar, SidebarContent, SidebarHeader, SidebarFooter } from "@/components/ui/sidebar";
import { Logo } from "@/components/logo";
import { NavMenu } from "./nav-menu";
import { NavNotes } from "./nav-notes";
import { NavUser } from "./nav-user";

export function AppSidebar() {
  return (
    <Sidebar className="w-64">
      <SidebarHeader className="p-4">
        <Logo />
      </SidebarHeader>

      <SidebarContent className="p-2">
        <NavMenu />
        <NavNotes />
      </SidebarContent>

      <SidebarFooter className="p-4">
        <NavUser />
      </SidebarFooter>
    </Sidebar>
  );
}
```

**2. 导航菜单** (components/sidebar/nav-menu.tsx):
```typescript
"use client";

import Link from "next/link";
import { usePathname } from "next/navigation";
import { cn } from "@/lib/utils";
import { Home, MessageSquare, CreditCard, Settings } from "lucide-react";

const menuItems = [
  { icon: Home, label: "Home", href: "/home" },
  { icon: MessageSquare, label: "Chat", href: "/chat" },
  { icon: CreditCard, label: "Billing", href: "/billing" },
  { icon: Settings, label: "Settings", href: "/settings" },
];

export function NavMenu() {
  const pathname = usePathname();

  return (
    <nav className="space-y-1">
      {menuItems.map((item) => {
        const Icon = item.icon;
        const isActive = pathname.startsWith(item.href);

        return (
          <Link
            key={item.href}
            href={item.href}
            className={cn(
              "flex items-center gap-3 rounded-lg px-3 py-2 text-sm transition-colors",
              isActive
                ? "bg-primary text-primary-foreground"
                : "hover:bg-muted"
            )}
          >
            <Icon className="h-5 w-5" />
            {item.label}
          </Link>
        );
      })}
    </nav>
  );
}
```

**3. 笔记列表** (components/sidebar/nav-notes.tsx):
```typescript
"use client";

import { Button } from "@/components/ui/button";
import { Plus, FileText } from "lucide-react";
import { useNotes, useCreateNote } from "@/features/use-note";
import { useNoteId } from "@/hooks/use-note-id";
import { ScrollArea } from "@/components/ui/scroll-area";

export function NavNotes() {
  const { data } = useNotes(1, 20);
  const { mutate: createNote, isPending } = useCreateNote();
  const { setNoteId } = useNoteId();

  const notes = data?.data || [];

  const handleCreate = () => {
    createNote({
      title: "Untitled Note",
      content: "",
    });
  };

  return (
    <div className="mt-4">
      <div className="flex items-center justify-between px-2 mb-2">
        <h3 className="text-sm font-semibold">Notes</h3>
        <Button
          size="sm"
          variant="ghost"
          onClick={handleCreate}
          disabled={isPending}
        >
          <Plus className="h-4 w-4" />
        </Button>
      </div>

      <ScrollArea className="h-[300px]">
        <div className="space-y-1">
          {notes.map((note: any) => (
            <button
              key={note.id}
              onClick={() => setNoteId(note.id)}
              className="flex items-center gap-2 w-full rounded-lg px-2 py-1.5 text-sm hover:bg-muted"
            >
              <FileText className="h-4 w-4 flex-shrink-0" />
              <span className="truncate">{note.title}</span>
            </button>
          ))}
        </div>
      </ScrollArea>
    </div>
  );
}
```

**4. 用户菜单** (components/sidebar/nav-user.tsx):
```typescript
"use client";

import { authClient } from "@/lib/auth-client";
import { useRouter } from "next/navigation";
import { Button } from "@/components/ui/button";
import { LogOut } from "lucide-react";

export function NavUser() {
  const router = useRouter();

  const handleSignOut = async () => {
    await authClient.signOut();
    router.push("/auth/sign-in");
  };

  return (
    <Button variant="ghost" onClick={handleSignOut} className="w-full justify-start">
      <LogOut className="h-4 w-4 mr-2" />
      Sign Out
    </Button>
  );
}
```

**5. Dashboard 布局** (app/(routes)/(dashboard)/layout.tsx):
```typescript
import { auth } from "@/lib/auth";
import { redirect } from "next/navigation";
import { headers } from "next/headers";
import { AppSidebar } from "@/components/sidebar";

export default async function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const session = await auth.api.getSession({
    headers: await headers(),
  });

  if (!session) {
    redirect("/auth/sign-in");
  }

  return (
    <div className="flex h-screen">
      <AppSidebar />
      <main className="flex-1 overflow-auto">
        {children}
      </main>
    </div>
  );
}
```
```

**预期结果**:
- 响应式侧边栏
- 导航菜单高亮
- 笔记快速访问
- 用户登出功能

---

## 提示词 3.6: 创建笔记对话框 / Create Note Dialog

```
创建笔记编辑对话框系统：

**1. 笔记对话框容器** (components/note-dialog/note-dialog.tsx):
```typescript
"use client";

import { Sheet, SheetContent } from "@/components/ui/sheet";
import { useNoteId } from "@/hooks/use-note-id";
import { NoteView } from "./note-view";

export function NoteDialog() {
  const { noteId, setNoteId } = useNoteId();
  const isOpen = !!noteId;

  const handleClose = () => {
    setNoteId(null);
  };

  return (
    <Sheet open={isOpen} onOpenChange={handleClose}>
      <SheetContent side="right" className="w-full md:w-1/2 p-0">
        {noteId && <NoteView noteId={noteId} />}
      </SheetContent>
    </Sheet>
  );
}
```

**2. 笔记编辑视图** (components/note-dialog/note-view.tsx):
```typescript
"use client";

import { useState, useEffect } from "react";
import { useNote, useUpdateNote, useDeleteNote } from "@/features/use-note";
import { Input } from "@/components/ui/input";
import { Textarea } from "@/components/ui/textarea";
import { Button } from "@/components/ui/button";
import { Trash2 } from "lucide-react";
import { useDebouncedCallback } from "use-debounce";

export function NoteView({ noteId }: { noteId: string }) {
  const { data } = useNote(noteId);
  const { mutate: updateNote } = useUpdateNote();
  const { mutate: deleteNote } = useDeleteNote();

  const note = data?.data;

  const [title, setTitle] = useState(note?.title || "");
  const [content, setContent] = useState(note?.content || "");

  useEffect(() => {
    if (note) {
      setTitle(note.title);
      setContent(note.content);
    }
  }, [note]);

  const debouncedUpdate = useDebouncedCallback((field, value) => {
    updateNote({
      id: noteId,
      json: { [field]: value },
    });
  }, 1000);

  const handleTitleChange = (value: string) => {
    setTitle(value);
    debouncedUpdate("title", value);
  };

  const handleContentChange = (value: string) => {
    setContent(value);
    debouncedUpdate("content", value);
  };

  const handleDelete = () => {
    if (confirm("Are you sure you want to delete this note?")) {
      deleteNote(noteId);
    }
  };

  return (
    <div className="flex flex-col h-full">
      <div className="p-4 border-b flex items-center justify-between">
        <Input
          value={title}
          onChange={(e) => handleTitleChange(e.target.value)}
          placeholder="Note title"
          className="text-lg font-semibold border-none"
        />
        <Button variant="ghost" size="sm" onClick={handleDelete}>
          <Trash2 className="h-4 w-4" />
        </Button>
      </div>

      <div className="flex-1 p-4">
        <Textarea
          value={content}
          onChange={(e) => handleContentChange(e.target.value)}
          placeholder="Start writing..."
          className="h-full resize-none border-none"
        />
      </div>
    </div>
  );
}
```

**3. 在根布局中添加** (app/layout.tsx):
```typescript
import { NoteDialog } from "@/components/note-dialog/note-dialog";

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html>
      <body>
        <Providers>
          {children}
          <NoteDialog />  {/* 全局笔记对话框 */}
        </Providers>
      </body>
    </html>
  );
}
```

依赖:
- use-debounce (用于防抖更新)
```

**预期结果**:
- 笔记侧边弹窗
- 实时自动保存
- URL 状态同步
- 删除确认

---

## ✅ 前端完成检查清单 / Frontend Completion Checklist

完成所有提示词后，验证以下功能：

### 认证流程
- [ ] 用户可以注册并自动登录
- [ ] 用户可以登录
- [ ] 已登录用户访问 /auth/* 自动跳转
- [ ] 未登录用户访问 /home 自动跳转

### 聊天界面
- [ ] 可以发送消息
- [ ] AI 流式响应正常显示
- [ ] 消息历史正确加载
- [ ] 生成限制提示显示

### 侧边栏
- [ ] 导航菜单正常工作
- [ ] 当前页面高亮
- [ ] 笔记列表显示
- [ ] 创建笔记按钮工作

### 笔记系统
- [ ] 点击笔记打开对话框
- [ ] 编辑标题和内容自动保存
- [ ] 删除笔记有确认
- [ ] 关闭对话框后 URL 清空

### 响应式设计
- [ ] 移动端侧边栏可收起
- [ ] 笔记对话框在移动端全屏
- [ ] 聊天界面适配小屏幕

---

## 🔗 相关文档 / Related Documentation

- **前端架构分析**: `analysis/03-frontend-analysis.md`
- **后端API提示词**: `prompts-generated/02-backend-prompts.md`

---

**生成时间**: 2025-11-16
**版本**: v1.0
**下一步**: 集成提示词 (`04-integration-prompts.md`)
