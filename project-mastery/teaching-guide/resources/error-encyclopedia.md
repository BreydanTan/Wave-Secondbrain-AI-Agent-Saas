# 🆘 错误百科 / Error Encyclopedia

> **Wave AI项目常见错误及解决方案**

---

## 🔴 环境配置错误

### 错误1: Cannot find module 'next'

**错误信息**:
```
Error: Cannot find module 'next'
Require stack:
- /path/to/project/next.config.ts
```

**什么意思**: Node模块未安装

**原因**:
- 刚克隆项目，未运行 `npm install`
- `node_modules` 文件夹被删除
- `package.json` 损坏

**解决方案**:
```bash
# 方案1: 重新安装
npm install

# 方案2: 清除缓存后安装
npm cache clean --force
rm -rf node_modules package-lock.json
npm install

# 方案3: 使用其他包管理器
pnpm install  # 或 yarn install
```

**预防措施**:
- 克隆项目后先运行 `npm install`
- 不要手动删除 `node_modules`

---

### 错误2: Port 3000 is already in use

**错误信息**:
```
Error: listen EADDRINUSE: address already in use :::3000
```

**什么意思**: 3000端口已被占用

**原因**:
- 另一个Next.js项目正在运行
- 其他程序占用3000端口

**解决方案**:
```bash
# 方案1: 使用其他端口
npm run dev -- -p 3001

# 方案2: 查找并结束占用进程
# Mac/Linux:
lsof -i :3000
kill -9 [PID]

# Windows:
netstat -ano | findstr :3000
taskkill /PID [PID] /F

# 方案3: 重启电脑（最简单）
```

---

### 错误3: Invalid environment variables

**错误信息**:
```
Error: Invalid environment variables:
  DATABASE_URL: Required
```

**什么意思**: 缺少必需的环境变量

**原因**:
- `.env.local` 文件不存在
- 环境变量名拼写错误
- 变量值为空

**解决方案**:
```bash
# 1. 检查.env.local是否存在
ls -la | grep .env

# 2. 如果不存在，复制模板
cp .env.example .env.local

# 3. 编辑.env.local，填入所有必需变量
# DATABASE_URL="postgresql://..."
# ANTHROPIC_API_KEY="sk-ant-..."
# 等等

# 4. 重启开发服务器
npm run dev
```

---

## 🟡 数据库错误

### 错误4: Prisma Client未生成

**错误信息**:
```
Error: @prisma/client did not initialize yet
```

**什么意思**: Prisma Client需要重新生成

**原因**:
- 修改了 `schema.prisma`
- 首次运行项目

**解决方案**:
```bash
# 生成Prisma Client
npx prisma generate

# 同时推送schema到数据库
npx prisma db push
```

---

### 错误5: 数据库连接失败

**错误信息**:
```
Error: P1001: Can't reach database server at `localhost:5432`
```

**什么意思**: 无法连接到数据库

**原因**:
- PostgreSQL未启动
- `DATABASE_URL` 配置错误
- 数据库不存在

**解决方案**:
```bash
# 检查DATABASE_URL格式
# 正确格式: postgresql://USER:PASSWORD@HOST:PORT/DATABASE

# 使用SQLite进行本地开发（最简单）
# .env.local中设置：
DATABASE_URL="file:./dev.db"

# 然后运行：
npx prisma db push
```

---

### 错误6: 唯一约束冲突

**错误信息**:
```
Error: Unique constraint failed on the fields: (`email`)
```

**什么意思**: 尝试插入重复的唯一值

**原因**:
- 注册时邮箱已存在
- 数据库有重复数据

**解决方案**:
```typescript
// 前端添加错误处理
try {
  await signUp({ email, password })
} catch (error) {
  if (error.message.includes('Unique constraint')) {
    alert('该邮箱已被注册')
  }
}

// 后端先检查是否存在
const existing = await prisma.user.findUnique({
  where: { email }
})
if (existing) {
  return c.json({ error: '邮箱已存在' }, 400)
}
```

---

## 🟠 API错误

### 错误7: 401 Unauthorized

**错误信息**:
```
POST /api/note 401 (Unauthorized)
```

**什么意思**: 未登录或Session失效

**原因**:
- 用户未登录
- Session过期
- Cookie未发送

**解决方案**:
```typescript
// 前端：检查并处理401
fetch('/api/note').then(res => {
  if (res.status === 401) {
    // 重定向到登录页
    window.location.href = '/sign-in'
  }
})

// 或使用React Query拦截器
```

---

### 错误8: 404 Not Found

**错误信息**:
```
GET /api/nonexistent 404 (Not Found)
```

**什么意思**: API端点不存在

**原因**:
- URL拼写错误
- 路由未注册
- 动态路由参数错误

**解决方案**:
```typescript
// 检查API端点定义
// app/api/[[...route]]/route.ts

// 确保路由已注册
app.get('/note', noteHandler)

// 检查前端调用
fetch('/api/note')  // ✅ 正确
fetch('/api/notes') // ❌ 错误（多了s）
```

---

### 错误9: CORS错误

**错误信息**:
```
Access to fetch has been blocked by CORS policy
```

**什么意思**: 跨域请求被阻止

**原因**:
- 前后端域名不同
- 未配置CORS

**解决方案**:
```typescript
// Next.js项目通常不会遇到，因为前后端同域
// 如果遇到，在API路由添加CORS头

export async function GET(request: Request) {
  return new Response(JSON.stringify(data), {
    headers: {
      'Access-Control-Allow-Origin': '*',
      'Content-Type': 'application/json',
    },
  })
}
```

---

## 🔵 React/Next.js错误

### 错误10: Hydration failed

**错误信息**:
```
Error: Hydration failed because the initial UI does not match what was rendered on the server
```

**什么意思**: 服务端渲染和客户端渲染不一致

**原因**:
- 使用了 `window` 等浏览器API
- 时间戳或随机数不一致

**解决方案**:
```typescript
// 使用 'use client' 标记客户端组件
'use client'

export function MyComponent() {
  const [mounted, setMounted] = useState(false)

  useEffect(() => {
    setMounted(true)
  }, [])

  if (!mounted) return null

  return <div>{window.innerWidth}</div>
}
```

---

### 错误11: Objects are not valid as a React child

**错误信息**:
```
Error: Objects are not valid as a React child
```

**什么意思**: 尝试直接渲染对象

**原因**:
- 忘记访问对象属性
- 返回对象而不是JSX

**解决方案**:
```typescript
// ❌ 错误
<div>{user}</div>

// ✅ 正确
<div>{user.name}</div>

// ❌ 错误
<div>{new Date()}</div>

// ✅ 正确
<div>{new Date().toLocaleDateString()}</div>
```

---

### 错误12: Cannot read property 'map' of undefined

**错误信息**:
```
TypeError: Cannot read property 'map' of undefined
```

**什么意思**: 尝试遍历undefined/null

**原因**:
- 数据还未加载
- API返回null

**解决方案**:
```typescript
// ❌ 错误
{notes.map(note => ...)}

// ✅ 正确
{notes?.map(note => ...)}
// 或
{(notes || []).map(note => ...)}
// 或
{notes && notes.map(note => ...)}
```

---

## ⚡ AI相关错误

### 错误13: AI API密钥无效

**错误信息**:
```
Error: Invalid API key
```

**什么意思**: AI服务API密钥错误

**原因**:
- 密钥拼写错误
- 密钥过期
- 环境变量未加载

**解决方案**:
```bash
# 1. 检查.env.local
cat .env.local | grep ANTHROPIC_API_KEY

# 2. 确认密钥格式
# Claude: sk-ant-api03-...
# OpenAI: sk-...

# 3. 重启服务器
npm run dev

# 4. 验证密钥是否有效（访问API提供商后台）
```

---

### 错误14: AI工具调用失败

**错误信息**:
```
Error: Tool execution failed: create-note
```

**什么意思**: AI工具执行出错

**原因**:
- 工具参数验证失败
- 工具执行逻辑错误
- 数据库操作失败

**解决方案**:
```typescript
// 检查工具定义
export const createNoteTool = tool({
  description: "...",
  parameters: z.object({
    title: z.string(),
    content: z.string()
  }),
  execute: async ({ title, content }, { user }) => {
    try {
      const note = await prisma.note.create({
        data: { userId: user.id, title, content }
      })
      return { success: true, noteId: note.id }
    } catch (error) {
      console.error('Create note error:', error)
      throw new Error('Failed to create note')
    }
  }
})
```

---

## 💳 Stripe相关错误

### 错误15: Webhook签名验证失败

**错误信息**:
```
Error: No signatures found matching the expected signature for payload
```

**什么意思**: Stripe Webhook签名不匹配

**原因**:
- `STRIPE_WEBHOOK_SECRET` 配置错误
- Webhook密钥与Stripe不一致

**解决方案**:
```bash
# 1. 从Stripe Dashboard获取正确的Webhook密钥
# 2. 更新.env.local
STRIPE_WEBHOOK_SECRET="whsec_..."

# 3. 本地测试使用Stripe CLI
stripe listen --forward-to localhost:3000/api/webhooks/stripe
# 会得到临时密钥：whsec_xxx

# 4. 重启服务器
```

---

## 🛠️ 调试技巧

### 通用调试流程

1. **阅读完整错误信息**
   ```
   不要只看第一行，完整的堆栈信息很重要
   ```

2. **定位错误位置**
   ```
   错误信息通常会指出文件名和行号
   ```

3. **添加console.log**
   ```typescript
   console.log('Before operation:', data)
   // 执行操作
   console.log('After operation:', result)
   ```

4. **使用浏览器DevTools**
   ```
   - Network tab: 检查API请求
   - Console: 查看JavaScript错误
   - React DevTools: 检查组件状态
   ```

5. **查看服务器日志**
   ```
   终端中的输出通常包含详细错误信息
   ```

---

## 📚 获取帮助

如果以上解决方案都不行：

1. **使用调试提示词**: [debugging-prompts.md](../prompts-library/debugging-prompts.md)
2. **查看调试流程图**: [debugging-flowchart.md](./debugging-flowchart.md)
3. **搜索GitHub Issues**: 可能别人遇到过同样的问题
4. **向AI寻求帮助**: 提供完整错误信息和上下文

---

**最后更新**: 2025-11-16
**包含错误数**: 15+
**持续更新**: 遇到新错误会添加到这里
