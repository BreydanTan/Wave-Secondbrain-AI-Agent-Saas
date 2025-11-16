# 📚 Wave AI 学习总览 / Learning Overview

> **目标读者**: 非专业程序员、初学者、想学习全栈AI应用开发的人
>
> **学习方式**: 通过AI协作学习（vibecoder友好）
>
> **最终目标**: 理解、维护、扩展这个AI SaaS项目

---

## 🎯 你将学到什么

完成这个学习路径后，你将能够：

✅ **理解项目** - 知道每个文件的作用，代码如何组织
✅ **运行项目** - 在本地搭建开发环境，看到效果
✅ **修改功能** - 改变样式、添加按钮、修改文本
✅ **扩展功能** - 添加新页面、新API、新数据模型
✅ **与AI协作** - 写出有效的提示词，理解AI的回答
✅ **调试错误** - 看懂错误信息，知道如何修复
✅ **部署上线** - 将项目发布到互联网

---

## ⏱️ 学习时间线

**总时长**: 4-6周
**每日学习**: 2-3小时（灵活调整）
**学习节奏**: 工作日1-2小时，周末3-4小时

### 📅 时间分配建议

| 周次 | 阶段 | 时间投入 | 完成标志 |
|------|------|---------|---------|
| **第1周** | 阶段1 + 阶段2 | 10-15小时 | 项目能运行，理解文件结构 |
| **第2周** | 阶段3 | 10-15小时 | 理解数据库，能查询数据 |
| **第3周** | 阶段4 (前半) | 10-15小时 | 理解认证和笔记功能 |
| **第4周** | 阶段4 (后半) | 10-15小时 | 理解AI集成和支付 |
| **第5-6周** | 阶段5 | 15-20小时 | 添加自定义功能 |

**弹性安排**: 如果某个阶段觉得难，可以多花1-2天巩固

---

## 🎓 五大学习阶段

### 🚀 阶段1：你好世界（第1-2天）

**目标**: 看到第一个成果，建立信心

**你会做什么**:
- 在电脑上运行项目
- 看到浏览器中的AI聊天界面
- 修改一个简单的文本
- 改变一个按钮的颜色

**成功标志**:
- ✅ 浏览器显示 `http://localhost:3000`
- ✅ 你修改的文字出现在页面上
- ✅ 控制台没有红色错误

**需要的工具**:
- 电脑（Windows/Mac/Linux都可以）
- 网络连接
- 2-3GB 磁盘空间

**学习文档**: [`03-stage1-foundation.md`](./03-stage1-foundation.md)

---

### 📂 阶段2：理解结构（第3-5天）

**目标**: 知道文件是怎么组织的，能快速找到代码

**你会学到**:
- `app/` 文件夹是干什么的
- `components/` 里面放什么
- `lib/` 是什么意思
- 为什么有 `.ts` 和 `.tsx` 两种文件

**成功标志**:
- ✅ 能在30秒内找到"登录按钮"的代码
- ✅ 知道要修改样式应该改哪个文件
- ✅ 理解"组件"的概念

**类比理解**:
```
项目就像一个图书馆
├── app/          → 书架（所有页面）
├── components/   → 可复用的段落（可以放到不同书里）
├── lib/          → 工具书（辅助功能）
└── public/       → 公共阅览区（图片、字体）
```

**学习文档**: [`04-stage2-skeleton.md`](./04-stage2-skeleton.md)

---

### 💾 阶段3：数据库连接（第6-10天）

**目标**: 理解数据如何存储和获取

**你会学到**:
- 什么是数据库（用餐厅仓库类比）
- 如何查看数据（像查看仓库里有什么食材）
- 如何添加数据（像往仓库添加食材）
- 表与表之间的关系（像食材分类）

**成功标志**:
- ✅ 能用 Prisma Studio 查看数据
- ✅ 理解 User → Note → Conversation 的关系
- ✅ 能写一个简单的数据库查询

**数据模型概览**:
```
User (用户)
  ├── has many → Note (笔记)
  ├── has many → Conversation (对话)
  └── has one → Subscription (订阅)

Conversation (对话)
  └── has many → Message (消息)
```

**学习文档**: [`05-stage3-core-features.md`](./05-stage3-core-features.md)

---

### 🎨 阶段4：核心功能复制（第11-25天）

**目标**: 重建主要功能，深度理解业务逻辑

这个阶段分为4个子模块：

#### 4.1 用户认证（第11-14天）
- 注册/登录系统
- Session管理
- 路由保护

#### 4.2 笔记系统（第15-18天）
- 创建/编辑/删除笔记
- 笔记列表
- 搜索功能

#### 4.3 AI聊天（第19-22天）
- 流式响应
- AI工具调用
- 多模型切换

#### 4.4 订阅支付（第23-25天）
- Stripe集成
- Webhook处理
- 订阅状态同步

**成功标志**:
- ✅ 主要功能正常工作
- ✅ 能解释每个功能的实现原理
- ✅ 遇到bug知道如何调试

**学习文档**: [`06-stage4-full-replication.md`](./06-stage4-full-replication.md)

---

### 🌟 阶段5：功能扩展（第26天+）

**目标**: 添加自己的功能，从学习者变成创造者

**你可以做什么**:
- 添加"收藏笔记"功能
- 添加"分享对话"功能
- 集成新的AI模型
- 添加数据导出功能
- 自定义AI系统提示词

**成功标志**:
- ✅ 成功添加一个自定义功能
- ✅ 功能经过测试，没有bug
- ✅ 代码遵循项目的风格

**学习文档**: [`07-stage5-extensions.md`](./07-stage5-extensions.md)

---

## 🧰 学习工具箱

### 必备工具

1. **代码编辑器**: VS Code（推荐）
   - 下载: https://code.visualstudio.com/
   - 必装插件: Prisma, Tailwind CSS IntelliSense

2. **Node.js**: v18 或更高
   - 下载: https://nodejs.org/
   - 验证: 运行 `node -v`

3. **Git**: 版本控制
   - 下载: https://git-scm.com/
   - 验证: 运行 `git --version`

4. **数据库工具**: Prisma Studio（项目自带）
   - 运行: `npm run prisma:studio`

### AI协作伙伴

你会频繁使用AI来学习和开发，推荐：
- **Claude** (Anthropic) - 擅长解释代码
- **ChatGPT** (OpenAI) - 擅长生成代码
- **Cursor** - AI代码编辑器

**AI协作技巧**:
1. **具体化**: "帮我在 app/page.tsx:45 添加一个按钮"
2. **带上下文**: "基于 project-mastery/analysis/03-frontend-analysis.md 中的组件结构..."
3. **引用文档**: "参考 teaching-guide/02-concept-dictionary.md#API端点..."
4. **分步验证**: 每个步骤都要验证是否成功

---

## 📖 学习资源导航

### 1. 理解项目

| 想了解 | 阅读文档 |
|--------|---------|
| 项目是做什么的 | [`specifications/PROJECT_SPEC_CN.md#1-项目概述`](../specifications/PROJECT_SPEC_CN.md) |
| 技术栈是什么 | [`analysis/00-project-overview.md`](../analysis/00-project-overview.md) |
| 数据库结构 | [`analysis/01-database-analysis.md`](../analysis/01-database-analysis.md) |
| API接口列表 | [`analysis/02-backend-analysis.md`](../analysis/02-backend-analysis.md) |

### 2. 动手实践

| 想做什么 | 使用资源 |
|---------|---------|
| 从零开始搭建 | [`analysis/prompts-generated/01-foundation-prompts.md`](../analysis/prompts-generated/01-foundation-prompts.md) |
| 添加新功能 | [`prompts-library/extension-prompts.md`](./prompts-library/extension-prompts.md) |
| 调试错误 | [`resources/error-encyclopedia.md`](./resources/error-encyclopedia.md) |
| 完成挑战 | [`resources/milestone-challenges.md`](./resources/milestone-challenges.md) |

### 3. 概念学习

| 不理解的术语 | 查找位置 |
|-------------|---------|
| 技术术语 | [`02-concept-dictionary.md`](./02-concept-dictionary.md) |
| 架构概念 | [`01-architecture-visuals.md`](./01-architecture-visuals.md) |
| 流程图 | [`specifications/PROJECT_SPEC_CN.md#4-核心业务流程`](../specifications/PROJECT_SPEC_CN.md) |

---

## 🎮 学习模式

### 模式1：跟随者模式（推荐初学者）

**特点**: 严格按照教程步骤
**适合**: 从未做过全栈开发的人
**优点**: 不会迷路，稳步前进
**缺点**: 可能感觉机械

**如何使用**:
1. 从阶段1开始，一步一步做
2. 不跳过任何验证步骤
3. 遇到不懂的先记录，继续往下
4. 每个阶段结束做自我评估

### 模式2：探索者模式（推荐有基础的人）

**特点**: 自由探索，按需学习
**适合**: 有过编程经验的人
**优点**: 学习自己感兴趣的部分
**缺点**: 可能遗漏重要概念

**如何使用**:
1. 先运行项目（阶段1必做）
2. 选择感兴趣的功能深入学习
3. 遇到不懂的查阅概念词典
4. 尝试添加功能时参考扩展提示词

### 模式3：挑战者模式（推荐有经验的开发者）

**特点**: 通过完成挑战学习
**适合**: 想快速掌握项目的开发者
**优点**: 高效，成就感强
**缺点**: 需要较强的自学能力

**如何使用**:
1. 快速浏览项目结构和规格文档
2. 直接跳到 [`resources/milestone-challenges.md`](./resources/milestone-challenges.md)
3. 从挑战1开始，逐个完成
4. 遇到障碍时查阅分析文档

---

## 🔄 学习检查点

### 每日检查清单

**开始学习前**:
- [ ] 设定今天的学习目标（1-2个小任务）
- [ ] 准备好AI助手（Claude/ChatGPT）
- [ ] 打开VS Code和浏览器

**学习过程中**:
- [ ] 每完成一个小任务就验证结果
- [ ] 遇到错误先尝试自己解决5分钟
- [ ] 5分钟解决不了，向AI寻求帮助
- [ ] 记录学到的新概念

**结束学习后**:
- [ ] 提交代码到Git（`git commit -m "完成XXX"`）
- [ ] 更新学习日志（今天学了什么）
- [ ] 预览明天要学的内容

### 阶段检查点

每个阶段结束时，使用对应的自我评估清单：
- [`resources/self-assessment-checklists.md#阶段1`](./resources/self-assessment-checklists.md)
- [`resources/self-assessment-checklists.md#阶段2`](./resources/self-assessment-checklists.md)
- ...

**分数标准**:
- **7分以下**: 继续巩固当前阶段，不要急着前进
- **8-10分**: 可以进入下一阶段
- **11-12分**: 优秀！考虑尝试挑战任务

---

## 🌟 学习激励系统

### 成就徽章

完成以下里程碑可以给自己"颁发"徽章：

🏅 **Hello World** - 成功运行项目
🏅 **文件侦探** - 30秒内找到任意功能的代码位置
🏅 **数据大师** - 完成第一个CRUD操作
🏅 **AI驯服师** - 成功集成一个AI功能
🏅 **支付专家** - 理解Stripe订阅流程
🏅 **全栈战士** - 独立添加一个完整功能（前端+后端+数据库）
🏅 **部署达人** - 将项目部署到Vercel
🏅 **贡献者** - 向原项目提交PR

### 社区支持

**遇到困难？寻求帮助**:
1. 查看 [`resources/error-encyclopedia.md`](./resources/error-encyclopedia.md)
2. 使用调试流程图 [`resources/debugging-flowchart.md`](./resources/debugging-flowchart.md)
3. 向AI提问（参考 [`prompts-library/debugging-prompts.md`](./prompts-library/debugging-prompts.md)）
4. 搜索 GitHub Issues
5. 在 Discord/Telegram 社区提问

**中文资源**:
- 掘金: 搜索"Next.js 15"、"Prisma"等关键词
- SegmentFault: 技术问答
- 知乎: "Next.js教程"、"全栈开发"
- B站: 视频教程

---

## 📝 学习日志模板

建议创建一个 `learning-log.md` 文件记录学习过程：

```markdown
# 我的Wave AI学习日志

## 第1天 - 2025-11-XX

### 今天的目标
- [ ] 安装Node.js和VS Code
- [ ] 克隆项目
- [ ] 运行开发服务器

### 完成情况
- [x] 安装Node.js ✅
- [x] 克隆项目 ✅
- [ ] 运行开发服务器 ❌（遇到错误）

### 遇到的问题
1. 运行 `npm install` 时报错 "Cannot find module..."
   - 解决: 运行 `npm cache clean --force` 后重试

### 学到的新概念
- npm: Node包管理器
- package.json: 项目依赖配置文件

### 明天计划
- 解决开发服务器启动问题
- 阅读项目结构文档
```

---

## 🎯 最终目标验证

完成所有阶段后，你应该能够：

### 知识维度
- [ ] 解释Next.js的App Router是如何工作的
- [ ] 画出用户登录的完整流程图
- [ ] 说明Prisma如何与数据库交互
- [ ] 解释AI工具调用的原理

### 技能维度
- [ ] 独立添加一个新的数据模型
- [ ] 创建一个新的API端点
- [ ] 设计和实现一个新页面
- [ ] 集成一个第三方API

### 实战维度
- [ ] 修复3个以上的bug
- [ ] 优化至少1个性能问题
- [ ] 成功部署到生产环境
- [ ] 向他人解释这个项目

---

## 🚀 开始你的学习之旅

准备好了吗？选择你的学习模式，然后开始吧！

**第一步**: 前往 [**阶段1：你好世界**](./03-stage1-foundation.md)

**祝你学习愉快！记住，编程学习最重要的是：**
1. **动手实践** - 不要只看不做
2. **小步快跑** - 每天进步一点点
3. **拥抱错误** - 错误是最好的老师
4. **寻求帮助** - 遇到困难及时提问
5. **享受过程** - 编程很有趣！

---

**生成日期**: 2025-11-16
**文档版本**: v1.0
**相关文档**:
- 项目规格: [`../specifications/PROJECT_SPEC_CN.md`](../specifications/PROJECT_SPEC_CN.md)
- 项目分析: [`../analysis/00-project-overview.md`](../analysis/00-project-overview.md)
