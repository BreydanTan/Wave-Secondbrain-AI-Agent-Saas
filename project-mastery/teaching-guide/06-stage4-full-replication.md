# 🎨 阶段4：完整功能复制 / Stage 4: Full Feature Replication

> **目标**: 深度理解认证、AI聊天、订阅支付
>
> **时长**: 10-14天  |  **难度**: ⭐️⭐️⭐️⭐️ 困难

---

## 🎯 学习目标

- ✅ 理解BetterAuth认证系统
- ✅ 掌握AI聊天的流式响应
- ✅ 理解AI工具调用机制
- ✅ 掌握Stripe订阅流程

---

## 📚 四大核心模块

### 模块1：用户认证（3-4天）

**关键文件**:
- `lib/auth.ts` - BetterAuth配置
- `app/api/auth/[...all]/route.ts` - 认证API
- `app/(auth)/sign-in/page.tsx` - 登录页面

**学习路径**:

1. **阅读认证分析**: [安全分析文档](../analysis/04-security-analysis.md)

2. **理解Session流程**:
   ```
   用户登录 → 创建Session → 存储到数据库
   每次请求 → 检查Session → 验证用户身份
   ```

3. **AI协作任务**:
   ```
   基于 lib/auth.ts 的配置，
   帮我理解BetterAuth如何：
   1. 注册新用户
   2. 验证密码
   3. 创建Session
   4. 集成Stripe
   ```

**验证任务**:
- [ ] 能成功注册新用户
- [ ] 理解Session存储位置
- [ ] 知道如何检查用户是否登录

---

### 模块2：AI聊天（4-5天）

**关键文件**:
- `app/api/[[...route]]/chat.ts` - 聊天API
- `lib/ai/index.ts` - AI配置
- `app/(main)/chat/[id]/page.tsx` - 聊天界面

**核心概念**:

1. **流式响应**:
   ```typescript
   const result = streamText({
     model: claude,
     messages: [...],
     onChunk: (chunk) => {
       // 实时返回每一块内容
     }
   })
   ```

2. **工具调用**:
   AI自动决定何时调用工具：
   ```
   用户: 帮我创建一条笔记
   AI判断: 需要create-note工具
   AI调用: createNoteTool({ title, content })
   AI回复: ✅ 已创建笔记
   ```

**AI协作任务**:
```
参考 lib/ai/tools/ 文件夹中的工具实现，
解释AI工具调用的完整流程：
1. AI如何决定使用哪个工具
2. 工具参数如何传递
3. 工具结果如何返回给用户
```

**参考**:
- [AI聊天流程](../specifications/PROJECT_SPEC_CN.md#42-ai聊天流程)
- [AI工具分析](../analysis/02-backend-analysis.md#ai工具系统)

---

### 模块3：笔记系统（2-3天）

**已在阶段3学习，这里深入理解**:

- 笔记列表的分页
- 笔记搜索功能
- AI辅助创建笔记

**进阶任务**: 添加"收藏笔记"功能

---

### 模块4：订阅支付（3-4天）

**关键文件**:
- `lib/stripe.ts` - Stripe客户端
- `app/api/[[...route]]/subscription.ts` - 订阅API
- `app/api/webhooks/stripe/route.ts` - Webhook处理

**Stripe流程**:
```
1. 用户点击"升级" → 前端调用 createCheckout API
2. 创建Stripe Checkout Session → 跳转到Stripe支付页
3. 用户完成支付 → Stripe发送Webhook
4. Webhook验证 → 更新数据库订阅状态
5. 用户刷新页面 → 看到已升级
```

**AI协作任务**:
```
基于 app/api/webhooks/stripe/route.ts，
解释Webhook的安全验证流程：
1. 如何验证Webhook签名
2. 如何处理不同的事件类型
3. 如何避免重复处理
```

**参考**: [支付流程](../specifications/PROJECT_SPEC_CN.md#43-stripe订阅流程)

---

## 🏆 阶段挑战

完成以下任一挑战证明你已掌握：

**挑战1**: 添加"笔记分类"功能
- 数据库添加Category模型
- API添加分类端点
- 前端添加分类筛选

**挑战2**: 支持Markdown笔记
- 安装markdown解析库
- 修改笔记显示组件
- 添加预览功能

**挑战3**: AI聊天历史搜索
- 实现消息全文搜索
- 高亮搜索关键词
- 按时间筛选

---

## 📊 阶段评估

- [ ] 我理解用户认证的完整流程
- [ ] 我理解AI流式响应的原理
- [ ] 我能解释AI工具调用机制
- [ ] 我理解Stripe订阅和Webhook

**分数**: _____ / 4

---

## ➡️ 下一步

[阶段5：功能扩展](./07-stage5-extensions.md)

---

**相关文档**:
- [后端API重建提示词](../analysis/prompts-generated/02-backend-prompts.md)
- [前端组件重建提示词](../analysis/prompts-generated/03-frontend-prompts.md)
