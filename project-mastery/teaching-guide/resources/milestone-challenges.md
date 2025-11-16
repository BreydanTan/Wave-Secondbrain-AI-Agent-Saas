# 🏆 里程碑挑战 / Milestone Challenges

> **通过完成挑战验证你的学习成果**

---

## 🎯 挑战系统说明

每个挑战都有：
- **难度等级**: ⭐️ (1-5星)
- **预计时间**: 完成所需时间
- **知识点**: 涉及的技术概念
- **成功标准**: 如何验证完成

---

## 🌟 挑战1：极简版Wave AI (1天)

**难度**: ⭐️
**目标**: 创建最简单的工作版本

### 要求

创建一个极简的聊天页面：
- ✅ 一个输入框
- ✅ 一个发送按钮
- ✅ 点击按钮显示alert弹窗

### 验证标准

- [ ] 输入框能输入文字
- [ ] 点击按钮显示 alert(输入内容)
- [ ] 页面样式整洁

### 提示

- 不需要连接API
- 不需要数据库
- 重点理解React状态和事件

### 代码骨架

```typescript
'use client'

export default function SimpleChat() {
  const [input, setInput] = useState('')

  const handleSubmit = () => {
    alert(`你说: ${input}`)
  }

  return (
    <div>
      <input value={input} onChange={(e) => setInput(e.target.value)} />
      <button onClick={handleSubmit}>发送</button>
    </div>
  )
}
```

---

## 🌟🌟 挑战2：数据展示 (3天)

**难度**: ⭐️⭐️
**目标**: 从数据库读取并显示数据

### 要求

实现笔记列表页面：
- ✅ 从数据库查询所有笔记
- ✅ 在页面上显示笔记卡片
- ✅ 显示标题、内容、创建时间
- ✅ 实现分页（每页10条）

### 知识点

- Prisma数据库查询
- API路由创建
- 数据获取和显示
- 分页逻辑

### 验证标准

- [ ] 能看到所有笔记
- [ ] 笔记信息显示完整
- [ ] 分页正常工作
- [ ] 样式美观

### 实现步骤

1. **创建API端点** (`app/api/note/route.ts`)
   ```typescript
   export async function GET(request: Request) {
     const notes = await prisma.note.findMany({
       take: 10,
       orderBy: { createdAt: 'desc' }
     })
     return Response.json(notes)
   }
   ```

2. **创建页面组件**
3. **使用fetch获取数据**
4. **渲染笔记列表**

---

## 🌟🌟🌟 挑战3：完整CRUD (1周)

**难度**: ⭐️⭐️⭐️
**目标**: 实现笔记的增删改查

### 要求

完整的笔记管理功能：
- ✅ **Create**: 创建新笔记
- ✅ **Read**: 查看笔记列表和详情
- ✅ **Update**: 编辑笔记
- ✅ **Delete**: 删除笔记

### 知识点

- RESTful API设计
- 表单处理
- 状态管理
- 错误处理
- 用户反馈（toast提示）

### 验证标准

- [ ] 创建笔记成功
- [ ] 查看笔记列表
- [ ] 点击笔记查看详情
- [ ] 编辑保存成功
- [ ] 删除后列表更新
- [ ] 有加载状态提示
- [ ] 有错误处理

### API设计

| 操作 | 方法 | 路径 | 请求体 |
|------|------|------|--------|
| 列表 | GET | `/api/note` | - |
| 详情 | GET | `/api/note/:id` | - |
| 创建 | POST | `/api/note` | `{ title, content }` |
| 更新 | PUT | `/api/note/:id` | `{ title, content }` |
| 删除 | DELETE | `/api/note/:id` | - |

---

## 🌟🌟🌟🌟 挑战4：AI聊天集成 (1-2周)

**难度**: ⭐️⭐️⭐️⭐️
**目标**: 实现真正的AI对话功能

### 要求

创建可用的AI聊天：
- ✅ 多轮对话
- ✅ 流式响应（打字机效果）
- ✅ 聊天历史保存
- ✅ 支持切换AI模型
- ✅ Markdown渲染

### 知识点

- AI SDK使用
- 流式响应处理
- WebSocket/SSE
- 数据库关系（Conversation → Message）
- 长列表优化

### 验证标准

- [ ] 发送消息收到AI回复
- [ ] 回复实时显示（流式）
- [ ] 对话历史保存
- [ ] 切换模型生效
- [ ] Markdown正确渲染
- [ ] 加载状态友好

### 核心代码

```typescript
// 使用AI SDK
import { useChat } from '@ai-sdk/react'

export function Chat() {
  const { messages, input, handleSubmit, isLoading } = useChat({
    api: '/api/chat'
  })

  return (
    <div>
      {messages.map(m => (
        <div key={m.id}>{m.content}</div>
      ))}
      <form onSubmit={handleSubmit}>
        <input value={input} />
      </form>
    </div>
  )
}
```

---

## 🌟🌟🌟🌟⭐️ 挑战5：完整功能上线 (2-3周)

**难度**: ⭐️⭐️⭐️⭐️⭐️
**目标**: 部署一个完整可用的应用

### 要求

实现并部署完整应用：
- ✅ 用户注册/登录
- ✅ AI聊天
- ✅ 笔记管理
- ✅ AI工具调用（创建笔记、搜索）
- ✅ 订阅支付（可选）
- ✅ 部署到Vercel
- ✅ 配置自定义域名（可选）

### 知识点

- 完整的全栈开发
- 认证系统
- 支付集成
- 部署流程
- 环境变量管理
- 域名配置

### 验证标准

- [ ] 用户能注册并登录
- [ ] 登录后能使用所有功能
- [ ] AI聊天完全可用
- [ ] 笔记CRUD正常
- [ ] AI能调用工具创建笔记
- [ ] 应用在线可访问
- [ ] 性能良好（3秒内加载）
- [ ] 移动端体验良好

### 部署检查清单

```
部署前：
[ ] 所有功能本地测试通过
[ ] 环境变量准备好
[ ] 数据库创建（使用Vercel Postgres或Supabase）
[ ] Stripe配置完成（如需要）

部署步骤：
[ ] 连接GitHub到Vercel
[ ] 配置环境变量
[ ] 触发部署
[ ] 数据库迁移

部署后：
[ ] 测试所有功能
[ ] 检查Console无错误
[ ] 验证AI API调用
[ ] 测试支付流程（如有）
```

---

## 🎖️ 专家级挑战

### 挑战A：性能优化

**目标**: 将首屏加载时间优化到1秒内

- [ ] 使用Next.js Image优化图片
- [ ] 实现代码分割
- [ ] 添加Redis缓存
- [ ] 优化数据库查询
- [ ] 使用CDN加速静态资源

### 挑战B：添加新AI模型

**目标**: 集成一个新的AI模型（如Gemini或Grok）

- [ ] 配置AI Provider
- [ ] 添加到模型列表
- [ ] 前端支持选择
- [ ] 测试工具调用兼容性

### 挑战C：实现知识图谱

**目标**: 笔记之间的智能关联

- [ ] 添加笔记关联表
- [ ] AI自动识别相关笔记
- [ ] 可视化笔记关系网络
- [ ] 智能推荐相关笔记

### 挑战D：浏览器插件

**目标**: Chrome扩展一键保存网页到笔记

- [ ] 创建Chrome Extension项目
- [ ] 实现网页内容提取
- [ ] 调用Wave AI API保存
- [ ] 发布到Chrome Web Store

---

## 📊 挑战记录卡

复制这个模板记录你的进度：

```markdown
# 我的挑战记录

## 挑战1：极简版Wave AI
- 开始日期: 2025-11-XX
- 完成日期: 2025-11-XX
- 用时: X小时
- 难点: [描述遇到的困难]
- 学到: [学到的新知识]
- 截图: [添加成果截图]

## 挑战2：数据展示
- 开始日期:
- 完成日期:
- 用时:
- 难点:
- 学到:
- 截图:

[继续记录...]
```

---

## 🎯 挑战建议

### 新手（0-3个月经验）

建议顺序：挑战1 → 挑战2 → 挑战3

### 有一定基础（3-6个月）

直接从挑战2或3开始

### 有经验（6个月+）

挑战4 → 挑战5 → 专家级挑战

---

## 🏅 完成奖励

完成每个挑战后：

1. **在GitHub创建仓库展示作品**
2. **写博客记录学习过程**
3. **分享到社交媒体**
4. **帮助其他学习者**

完成全部5个主要挑战后，你已经是合格的全栈开发者了！🎉

---

**相关文档**:
- [场景提示词](../prompts-library/scenario-prompts.md)
- [扩展提示词](../prompts-library/extension-prompts.md)
- [自我评估清单](./self-assessment-checklists.md)

**最后更新**: 2025-11-16
