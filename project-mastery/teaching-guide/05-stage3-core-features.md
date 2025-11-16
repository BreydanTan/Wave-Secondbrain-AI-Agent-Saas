# 💾 阶段3：数据库与核心功能 / Stage 3: Database & Core Features

> **目标**: 理解数据模型，掌握CRUD操作
>
> **时长**: 4-5天  |  **难度**: ⭐️⭐️⭐️ 较难

---

## 🎯 学习目标

- ✅ 理解数据库表结构和关系
- ✅ 使用Prisma Studio查看和修改数据
- ✅ 理解笔记和聊天功能的实现
- ✅ 掌握基本的Prisma查询

---

## 🗄️ 数据模型概览

```
User (用户)
  ├── id: string
  ├── email: string
  ├── name: string
  ├── stripeCustomerId: string
  └── 关系:
      ├── notes[] → Note
      ├── conversations[] → Conversation
      └── subscription → Subscription

Note (笔记)
  ├── id: string
  ├── title: string
  ├── content: string
  ├── userId: string (外键)
  └── createdAt: Date

Conversation (对话)
  ├── id: string
  ├── title: string
  ├── userId: string
  └── messages[] → Message

Message (消息)
  ├── id: string
  ├── role: enum (user/assistant)
  ├── content: string
  └── conversationId: string
```

**完整分析**: [数据库分析文档](../analysis/01-database-analysis.md)

---

## 🎯 实践任务

### 任务1：使用Prisma Studio

**步骤**:

1. 启动Prisma Studio:
   ```bash
   npx prisma studio
   ```

2. 浏览器打开: http://localhost:5555

3. 探索数据表：
   - 点击 `User` 表
   - 点击 `Note` 表
   - 观察表之间的关系

**验证清单**:
- [ ] 能看到所有数据表
- [ ] 能手动添加一条Note记录
- [ ] 能看到User和Note的关联

---

### 任务2：理解笔记功能

**代码路径**:
- 前端: `app/(main)/note/page.tsx`
- API: `app/api/[[...route]]/note.ts`
- AI工具: `lib/ai/tools/create-note.ts`

**流程图**:
```
用户点击"创建笔记"
  ↓
前端表单提交
  ↓
POST /api/note
  ↓
Prisma创建记录
  ↓
返回新笔记对象
  ↓
前端显示新笔记
```

**AI协作提示词**:
```
基于project-mastery/analysis/02-backend-analysis.md中的笔记API分析，
帮我理解笔记的完整CRUD流程。
请用简单的语言解释每个步骤。
```

---

### 任务3：实现简单的Prisma查询

**练习**: 在Prisma Studio或代码中尝试：

1. **查询所有笔记**:
   ```typescript
   const notes = await prisma.note.findMany()
   ```

2. **查询单个笔记**:
   ```typescript
   const note = await prisma.note.findUnique({
     where: { id: "note_xxx" }
   })
   ```

3. **创建笔记**:
   ```typescript
   const newNote = await prisma.note.create({
     data: {
       title: "测试笔记",
       content: "这是内容",
       userId: "user_xxx"
     }
   })
   ```

**参考**: [Prisma概念](./02-concept-dictionary.md#prisma)

---

## 📊 阶段评估

- [ ] 我能用Prisma Studio查看数据
- [ ] 我理解User → Note的关系
- [ ] 我能写出基本的Prisma查询
- [ ] 我理解笔记功能的实现流程

**分数**: _____ / 4

---

## ➡️ 下一步

[阶段4：完整功能复制](./06-stage4-full-replication.md)

---

**相关文档**:
- [数据库分析](../analysis/01-database-analysis.md)
- [API文档](../specifications/PROJECT_SPEC_CN.md#6-api接口文档)
