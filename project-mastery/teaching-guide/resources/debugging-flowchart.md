# 🔍 调试决策树 / Debugging Flowchart

> **系统化解决问题的流程图**

---

## 🚨 遇到问题了？从这里开始

```
遇到问题
    │
    ├─ 项目无法启动？ → 跳转到 [启动问题](#启动问题)
    │
    ├─ 页面显示异常？ → 跳转到 [界面问题](#界面问题)
    │
    ├─ API调用失败？ → 跳转到 [API问题](#api问题)
    │
    ├─ 数据库错误？ → 跳转到 [数据库问题](#数据库问题)
    │
    └─ 其他错误？ → 跳转到 [通用排查](#通用排查流程)
```

---

## 🚀 启动问题

```
npm run dev 失败
    │
    ├─ 显示"Cannot find module"？
    │   ├─ YES → 运行 npm install
    │   │        ↓
    │   │      还是失败？
    │   │        ├─ YES → 删除node_modules，重新安装
    │   │        └─ NO  → ✅ 问题解决
    │   └─ NO → 继续
    │
    ├─ 显示"Port already in use"？
    │   ├─ YES → 使用其他端口: npm run dev -- -p 3001
    │   │        或结束占用进程
    │   │        ↓
    │   │      ✅ 问题解决
    │   └─ NO → 继续
    │
    ├─ 显示"Invalid environment variables"？
    │   ├─ YES → 检查.env.local是否存在
    │   │        ↓
    │   │      不存在？
    │   │        ├─ YES → cp .env.example .env.local
    │   │        │        填入必需变量
    │   │        └─ NO  → 检查变量名和值
    │   │        ↓
    │   │      ✅ 问题解决
    │   └─ NO → 继续
    │
    ├─ 显示数据库连接错误？
    │   ├─ YES → 检查DATABASE_URL
    │   │        ↓
    │   │      使用SQLite: DATABASE_URL="file:./dev.db"
    │   │        ↓
    │   │      npx prisma db push
    │   │        ↓
    │   │      ✅ 问题解决
    │   └─ NO → 查看[错误百科](./error-encyclopedia.md)
    │
    └─ 其他错误 → 复制完整错误，使用[调试提示词](../prompts-library/debugging-prompts.md)
```

---

## 🎨 界面问题

```
页面显示异常
    │
    ├─ 页面完全空白？
    │   ├─ 打开浏览器Console (F12)
    │   │    ↓
    │   ├─ 有红色错误？
    │   │    ├─ YES → 阅读错误信息
    │   │    │        ├─ "Hydration failed" → 查看[错误百科#10](./error-encyclopedia.md#错误10-hydration-failed)
    │   │    │        ├─ "Objects are not valid" → 查看[错误百科#11](./error-encyclopedia.md#错误11-objects-are-not-valid-as-a-react-child)
    │   │    │        └─ 其他错误 → Google搜索错误信息
    │   │    └─ NO → 检查Network tab
    │   │             ├─ API请求失败？ → 跳转到[API问题](#api问题)
    │   │             └─ 都成功 → 检查组件是否正确返回JSX
    │   └─ 继续
    │
    ├─ 样式不对？
    │   ├─ 检查Tailwind类名拼写
    │   │    ↓
    │   ├─ 浏览器DevTools查看实际CSS
    │   │    ↓
    │   ├─ Tailwind配置正确？
    │   │    ├─ 检查 tailwind.config.ts
    │   │    └─ 重启开发服务器
    │   │    ↓
    │   └─ ✅ 问题解决 或 使用[调试提示词](../prompts-library/debugging-prompts.md#样式不生效)
    │
    ├─ 组件不显示？
    │   ├─ 添加 console.log 检查组件是否被调用
    │   │    ↓
    │   ├─ 检查条件渲染 (&&, ?)
    │   │    ↓
    │   ├─ 检查数据是否正确传入 (Props)
    │   │    ↓
    │   └─ 使用React DevTools查看组件树
    │
    └─ 交互不work？
        ├─ 检查onClick等事件是否绑定
        ├─ 检查函数是否定义
        ├─ 添加console.log确认函数被调用
        └─ 检查State是否正确更新
```

---

## 🌐 API问题

```
API调用失败
    │
    ├─ 检查Network tab (浏览器DevTools)
    │    ↓
    ├─ 请求发送了吗？
    │    ├─ NO → 检查前端代码
    │    │        ├─ fetch()是否被调用
    │    │        └─ URL是否正确
    │    └─ YES → 继续
    │
    ├─ 状态码是什么？
    │    │
    │    ├─ 401 (Unauthorized)
    │    │    ├─ 用户未登录？
    │    │    │    └─ 重定向到登录页
    │    │    └─ Session过期？
    │    │         └─ 重新登录
    │    │
    │    ├─ 404 (Not Found)
    │    │    ├─ 检查API路径拼写
    │    │    ├─ 检查路由是否注册 (app/api/)
    │    │    └─ 动态路由参数正确？
    │    │
    │    ├─ 400 (Bad Request)
    │    │    ├─ 查看响应body的错误信息
    │    │    ├─ 检查请求参数
    │    │    ├─ 检查请求体格式 (JSON)
    │    │    └─ 后端添加详细日志
    │    │
    │    ├─ 500 (Server Error)
    │    │    ├─ 查看服务器终端日志
    │    │    ├─ 后端添加try-catch
    │    │    ├─ 检查数据库操作
    │    │    └─ 检查环境变量
    │    │
    │    └─ 其他状态码
    │         └─ Google: "HTTP status code XXX meaning"
    │
    └─ 响应慢？
         ├─ 检查数据库查询（添加索引）
         ├─ 使用分页减少数据量
         ├─ 添加缓存
         └─ 优化查询逻辑
```

---

## 🗄️ 数据库问题

```
数据库错误
    │
    ├─ "Prisma Client未初始化"？
    │    ├─ 运行 npx prisma generate
    │    └─ 重启开发服务器
    │         ↓
    │       ✅ 问题解决
    │
    ├─ "Can't reach database"？
    │    ├─ 检查DATABASE_URL
    │    ├─ PostgreSQL是否运行？
    │    └─ 改用SQLite测试
    │         DATABASE_URL="file:./dev.db"
    │         ↓
    │       npx prisma db push
    │         ↓
    │       ✅ 问题解决
    │
    ├─ "Unique constraint failed"？
    │    ├─ 检查是否有重复数据
    │    ├─ 前端添加检查
    │    └─ 后端返回友好错误
    │         ↓
    │       ✅ 问题解决
    │
    ├─ "Foreign key constraint"？
    │    ├─ 检查关联数据是否存在
    │    ├─ 先创建父记录再创建子记录
    │    └─ 检查onDelete行为
    │         ↓
    │       ✅ 问题解决
    │
    └─ 查询返回null/undefined？
         ├─ 确认数据库中有数据
         │    └─ 使用 npx prisma studio 查看
         ├─ 检查查询条件 (where)
         ├─ 检查include关系
         └─ 添加console.log查看查询结果
```

---

## 🔧 通用排查流程

当你不知道问题出在哪里时：

### 第1步：确认基础环境

```
[ ] 开发服务器是否运行？
    → npm run dev

[ ] .env.local配置是否正确？
    → cat .env.local

[ ] 数据库是否连接？
    → npx prisma studio

[ ] Node/npm版本正确？
    → node -v (应该 >= 18)
    → npm -v
```

### 第2步：定位问题层次

```
问题在哪一层？

[ ] 前端（浏览器）
    → 查看 Browser Console
    → 查看 Network tab
    → 使用 React DevTools

[ ] 后端（服务器）
    → 查看终端日志
    → 添加 console.log
    → 检查API处理器代码

[ ] 数据库
    → 使用 Prisma Studio 查看数据
    → 检查Schema定义
    → 查看查询日志
```

### 第3步：逐层排查

```
前端排查：
  ├─ 请求是否发送？ → Network tab
  ├─ 参数是否正确？ → Console.log
  ├─ 响应如何处理？ → 检查then/catch
  └─ State是否更新？ → React DevTools

后端排查：
  ├─ 请求是否收到？ → 终端日志
  ├─ 参数是否正确？ → Console.log
  ├─ 业务逻辑有错？ → 添加断点
  └─ 响应是否返回？ → Console.log

数据库排查：
  ├─ Schema正确？ → schema.prisma
  ├─ 数据存在？ → Prisma Studio
  ├─ 查询正确？ → Console.log查询
  └─ 关系正确？ → 检查外键
```

### 第4步：寻求帮助

```
准备以下信息：

1. 完整错误信息
   └─ 截图或复制全部错误堆栈

2. 相关代码
   └─ 前后10行代码

3. 已尝试的方法
   └─ 列出你试过的所有方法

4. 环境信息
   ├─ Node版本
   ├─ 操作系统
   └─ 浏览器

5. 复现步骤
   └─ 如何触发这个错误

使用[调试提示词](../prompts-library/debugging-prompts.md)向AI求助
```

---

## 💡 调试技巧

### 1. Console.log大法

```typescript
// 在关键位置添加日志
console.log('✅ 1. Before fetch')
const response = await fetch('/api/note')
console.log('✅ 2. Response:', response.status)
const data = await response.json()
console.log('✅ 3. Data:', data)
```

### 2. 二分法排查

```
把代码注释掉一半
↓
还有问题？
  ├─ YES → 问题在这一半，继续二分
  └─ NO  → 问题在另一半
```

### 3. 对比工作的代码

```
找一个类似的、工作正常的功能
↓
对比两者的区别
↓
找出差异点
```

### 4. 从简单开始

```
先实现最简单的版本
↓
能工作吗？
  ├─ YES → 逐步添加复杂度
  └─ NO  → 说明基础有问题
```

---

## 📚 相关资源

- [错误百科](./error-encyclopedia.md) - 常见错误及解决方案
- [调试提示词](../prompts-library/debugging-prompts.md) - 向AI求助的模板
- [自我评估清单](./self-assessment-checklists.md) - 检查你的理解程度

---

**最后更新**: 2025-11-16
**提示**: 保存这个流程图，遇到问题时按图索骥！
