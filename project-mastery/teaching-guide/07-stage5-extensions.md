# 🌟 阶段5：功能扩展 / Stage 5: Feature Extensions

> **目标**: 添加自定义功能，成为项目的主人
>
> **时长**: 持续学习  |  **难度**: ⭐️⭐️⭐️⭐️⭐️ 专家级

---

## 🎯 学习目标

- ✅ 独立分析需求
- ✅ 设计数据模型
- ✅ 实现完整功能（前端+后端+数据库）
- ✅ 测试和调试
- ✅ 部署到生产环境

---

## 💡 扩展功能建议

### 难度：⭐️⭐️ 简单

1. **笔记标签系统**
   - 添加Tag模型
   - 笔记可以添加多个标签
   - 按标签筛选笔记

2. **深色模式**
   - 使用`next-themes`
   - Tailwind暗色变体
   - 保存用户偏好

3. **导出笔记为Markdown**
   - 添加导出按钮
   - 生成.md文件
   - 支持批量导出

---

### 难度：⭐️⭐️⭐️ 中等

4. **笔记分享功能**
   - 生成分享链接
   - 公开笔记页面
   - 访问统计

5. **AI对话历史搜索**
   - 全文搜索消息
   - 按日期筛选
   - 高亮关键词

6. **笔记协作**
   - 多用户编辑
   - 版本历史
   - 评论系统

---

### 难度：⭐️⭐️⭐️⭐️ 困难

7. **语音输入**
   - 集成Web Speech API
   - 实时转文字
   - 发送给AI

8. **图片上传到笔记**
   - 集成Cloudinary/S3
   - 图片压缩
   - Markdown图片语法

9. **AI助手自定义**
   - 用户可以自定义系统提示词
   - 保存多个AI角色
   - 角色切换

---

### 难度：⭐️⭐️⭐️⭐️⭐️ 专家级

10. **知识图谱**
    - 笔记之间的关联
    - 可视化知识网络
    - 智能推荐相关笔记

11. **浏览器插件**
    - Chrome/Firefox扩展
    - 一键保存网页到笔记
    - 快速唤起AI助手

12. **移动APP**
    - React Native
    - 离线支持
    - 推送通知

---

## 🛠️ 功能开发流程

### Step 1: 需求分析

使用AI协作：
```
我想为Wave AI添加"笔记标签"功能。

需求：
- 用户可以为笔记添加多个标签
- 可以按标签筛选笔记
- 标签可以重复使用

请帮我：
1. 分析这个功能需要修改哪些文件
2. 设计数据库模型
3. 列出开发步骤
```

---

### Step 2: 数据库设计

**示例：标签功能**

```prisma
// prisma/schema.prisma

model Tag {
  id        String   @id @default(cuid())
  name      String   @unique
  notes     NoteTag[]
  createdAt DateTime @default(now())
}

model NoteTag {
  noteId String
  tagId  String
  note   Note   @relation(fields: [noteId], references: [id])
  tag    Tag    @relation(fields: [tagId], references: [id])

  @@id([noteId, tagId])
}

// 修改Note模型
model Note {
  // ... 现有字段
  tags NoteTag[]
}
```

**应用迁移**:
```bash
npx prisma db push
```

---

### Step 3: 后端API

**创建标签API**:

```typescript
// app/api/[[...route]]/tag.ts
app.post('/tag', async (c) => {
  const { name } = await c.req.json()
  const session = c.get('session')

  const tag = await prisma.tag.create({
    data: { name }
  })

  return c.json(tag)
})

app.post('/note/:id/tag', async (c) => {
  const noteId = c.req.param('id')
  const { tagId } = await c.req.json()

  await prisma.noteTag.create({
    data: { noteId, tagId }
  })

  return c.json({ success: true })
})
```

---

### Step 4: 前端实现

**标签选择组件**:

```tsx
// components/note/tag-selector.tsx
export function TagSelector({ noteId, onTagAdded }) {
  const [tags, setTags] = useState([])

  // 获取所有标签
  useEffect(() => {
    fetch('/api/tag').then(r => r.json()).then(setTags)
  }, [])

  const handleAddTag = async (tagId) => {
    await fetch(`/api/note/${noteId}/tag`, {
      method: 'POST',
      body: JSON.stringify({ tagId })
    })
    onTagAdded()
  }

  return (
    <div>
      {tags.map(tag => (
        <button key={tag.id} onClick={() => handleAddTag(tag.id)}>
          {tag.name}
        </button>
      ))}
    </div>
  )
}
```

---

### Step 5: 测试

**测试清单**:
- [ ] 创建标签功能正常
- [ ] 为笔记添加标签成功
- [ ] 按标签筛选笔记有效
- [ ] 删除标签不影响笔记
- [ ] 没有性能问题

---

### Step 6: 部署

参考：[部署指南](../specifications/PROJECT_SPEC_CN.md#9-部署指南)

---

## 📚 学习资源

**提示词库**:
- [扩展功能提示词](./prompts-library/extension-prompts.md)
- [调试提示词](./prompts-library/debugging-prompts.md)

**挑战任务**:
- [里程碑挑战](./resources/milestone-challenges.md)

---

## 🎓 从学习者到贡献者

完成这个阶段后，你可以：

1. **向原项目提交PR**
   - Fork项目
   - 在新分支开发功能
   - 提交Pull Request

2. **创建自己的SaaS**
   - 基于Wave AI修改
   - 添加独特功能
   - 部署到自己的域名

3. **帮助他人学习**
   - 写教程博客
   - 录制视频教程
   - 在社区回答问题

---

## 🏆 最终挑战

**创建一个完整的新功能**，包括：

- [ ] 数据库模型设计
- [ ] 后端API实现
- [ ] 前端界面开发
- [ ] 测试通过
- [ ] 部署到生产环境
- [ ] 写使用文档

**完成后，你已经是全栈开发者了！🎉**

---

## ➡️ 持续学习

- 关注技术博客
- 参与开源项目
- 学习新技术（如Rust、Go）
- 构建自己的产品

---

**相关文档**:
- [扩展提示词](./prompts-library/extension-prompts.md)
- [场景提示词](./prompts-library/scenario-prompts.md)

**生成日期**: 2025-11-16
