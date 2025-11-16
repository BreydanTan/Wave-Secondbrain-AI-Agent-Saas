# 🚀 阶段1：你好世界 / Stage 1: Hello World

> **目标**: 在本地运行项目，看到第一个成果
>
> **时长**: 1-2天
>
> **难度**: ⭐️ 简单

---

## 🎯 学习目标

完成这个阶段后，你将能够：

- ✅ 在电脑上成功运行Wave AI项目
- ✅ 看到浏览器中的AI聊天界面
- ✅ 修改一个简单的文本并看到效果
- ✅ 理解基本的开发流程

---

## 📚 前置知识

**必须具备**：
- 会使用电脑的基本操作
- 有网络连接
- 2-3GB 磁盘空间

**不需要**：
- ❌ 不需要编程经验
- ❌ 不需要理解代码
- ❌ 不需要配置复杂环境

---

## 🛠️ 任务清单

### 任务1：安装必备工具

#### 1.1 安装 Node.js

**目标**: 安装JavaScript运行环境

**步骤**:

1. 访问 https://nodejs.org/
2. 下载 **LTS 版本**（推荐 v20 或 v22）
3. 运行安装程序，一路"下一步"
4. 验证安装：
   ```bash
   node -v
   # 应该显示: v20.x.x 或更高

   npm -v
   # 应该显示: 10.x.x 或更高
   ```

**AI协作提示词**:
```
我在安装Node.js，操作系统是 [Windows/Mac/Linux]。
如果遇到问题，我的错误信息是：[粘贴错误信息]
请帮我解决。
```

**验证清单**:
- [ ] `node -v` 显示版本号
- [ ] `npm -v` 显示版本号
- [ ] 没有错误信息

---

#### 1.2 安装 VS Code

**目标**: 安装代码编辑器

**步骤**:

1. 访问 https://code.visualstudio.com/
2. 下载并安装
3. 安装推荐插件：
   - Prisma（prisma.prisma）
   - Tailwind CSS IntelliSense（bradlc.vscode-tailwindcss）
   - TypeScript and JavaScript Language Features（内置）

**验证清单**:
- [ ] VS Code 能正常打开
- [ ] Prisma 插件已安装
- [ ] Tailwind CSS 插件已安装

---

#### 1.3 安装 Git

**目标**: 安装版本控制工具

**步骤**:

1. 访问 https://git-scm.com/
2. 下载并安装
3. 验证安装：
   ```bash
   git --version
   # 应该显示: git version 2.x.x
   ```

**验证清单**:
- [ ] `git --version` 显示版本号

---

### 任务2：克隆项目

**目标**: 把项目代码下载到本地

**步骤**:

1. 打开终端（Terminal）
   - Windows: 按 `Win + R`，输入 `cmd`
   - Mac: 按 `Cmd + 空格`，输入 `terminal`
   - 或在 VS Code 中按 `` Ctrl + ` ``

2. 进入你想存放项目的文件夹：
   ```bash
   cd Desktop  # 例如桌面
   ```

3. 克隆项目：
   ```bash
   git clone https://github.com/BreydanTan/Wave-Secondbrain-AI-Agent-Saas.git
   cd Wave-Secondbrain-AI-Agent-Saas
   ```

4. 用 VS Code 打开项目：
   ```bash
   code .
   ```

**验证清单**:
- [ ] VS Code 左侧能看到项目文件夹
- [ ] 能看到 `app/`, `components/`, `lib/` 等文件夹
- [ ] 能看到 `package.json` 文件

**常见问题**:

Q: `git` 命令不存在？
A: 重新安装Git，确保勾选"添加到PATH"

Q: 克隆很慢？
A: 网络问题，可以尝试使用代理或下载ZIP

---

### 任务3：安装项目依赖

**目标**: 安装项目需要的所有库

**步骤**:

1. 在项目根目录运行：
   ```bash
   npm install
   ```

2. 等待安装完成（可能需要5-10分钟）

3. 你会看到：
   ```
   added 1234 packages in 5m
   ```

**AI协作提示词**:
```
我在运行 npm install 时遇到错误：
[粘贴错误信息]

我的环境：
- Node版本: [运行 node -v 的结果]
- npm版本: [运行 npm -v 的结果]
- 操作系统: [Windows/Mac/Linux]
```

**验证清单**:
- [ ] `node_modules/` 文件夹已创建
- [ ] 没有红色错误信息
- [ ] 终端显示 "added xxx packages"

**常见问题**:

Q: `EACCES` 权限错误？
A: 不要用管理员权限运行，或运行 `npm cache clean --force`

Q: 网络超时？
A: 切换npm源：`npm config set registry https://registry.npmmirror.com`

---

### 任务4：配置环境变量

**目标**: 设置项目配置

**步骤**:

1. 复制环境变量模板：
   ```bash
   cp .env.example .env.local
   ```

   Windows用户：
   ```bash
   copy .env.example .env.local
   ```

2. 用VS Code打开 `.env.local`

3. **最小配置**（仅为了运行）：
   ```bash
   # 数据库 - 先用本地SQLite测试
   DATABASE_URL="file:./dev.db"

   # 应用URL
   NEXT_PUBLIC_APP_URL="http://localhost:3000"

   # BetterAuth密钥（随机字符串）
   BETTER_AUTH_SECRET="your-random-secret-at-least-32-chars-long"

   # 其他API密钥暂时留空，后续再配置
   ```

**生成随机密钥**:
```bash
node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

**验证清单**:
- [ ] `.env.local` 文件已创建
- [ ] `DATABASE_URL` 已设置
- [ ] `BETTER_AUTH_SECRET` 已设置

**⚠️ 重要提示**:
- `.env.local` 包含敏感信息，不要分享！
- 这个文件不会被Git提交（已在 `.gitignore` 中）

---

### 任务5：初始化数据库

**目标**: 创建本地数据库

**步骤**:

1. 推送数据库Schema：
   ```bash
   npx prisma db push
   ```

2. 你应该看到：
   ```
   ✔ Generated Prisma Client
   ✔ The database is now in sync with your Prisma schema.
   ```

3. （可选）打开Prisma Studio查看数据库：
   ```bash
   npx prisma studio
   ```
   会在浏览器打开 http://localhost:5555

**验证清单**:
- [ ] 命令执行成功，没有错误
- [ ] 项目根目录出现 `dev.db` 文件
- [ ] Prisma Studio 能打开（可选）

**常见问题**:

Q: "Error: P1001 Can't reach database"？
A: 检查 `DATABASE_URL` 是否正确设置

---

### 任务6：启动开发服务器

**目标**: 运行项目，看到界面

**步骤**:

1. 启动开发服务器：
   ```bash
   npm run dev
   ```

2. 等待编译完成，你会看到：
   ```
   ▲ Next.js 15.5.3
   - Local:        http://localhost:3000
   - Network:      http://192.168.x.x:3000

   ✓ Compiled in 3.5s
   ```

3. 打开浏览器，访问：http://localhost:3000

4. 你应该看到 Wave AI 的首页！

**验证清单**:
- [ ] 终端显示 "Compiled successfully"
- [ ] 浏览器能打开 http://localhost:3000
- [ ] 能看到登录或聊天界面
- [ ] 控制台没有红色错误

**常见问题**:

Q: 端口3000已被占用？
A: 改用其他端口：`npm run dev -- -p 3001`

Q: 页面空白？
A: 打开浏览器开发者工具（F12），查看Console错误

Q: 一直编译不成功？
A: 删除 `.next` 文件夹，重新运行 `npm run dev`

---

### 任务7：做第一个修改

**目标**: 修改代码，看到效果

**步骤**:

1. 在VS Code中打开：`app/(main)/page.tsx`

2. 找到类似这样的代码（大约第20行附近）：
   ```tsx
   <h1>Welcome to Wave AI</h1>
   ```

3. 修改文字：
   ```tsx
   <h1>你好，这是我的第一个修改！</h1>
   ```

4. 保存文件（Ctrl+S 或 Cmd+S）

5. 切换到浏览器，页面会**自动刷新**（热重载）

6. 你应该看到修改后的文字！

**AI协作提示词**:
```
我想修改Wave AI首页的标题文字。

我已经打开了 app/(main)/page.tsx 文件。
请帮我：
1. 找到标题所在的位置
2. 告诉我应该修改哪一行
3. 修改成"[你想要的文字]"
```

**验证清单**:
- [ ] 修改后保存文件
- [ ] 浏览器自动刷新
- [ ] 看到修改后的文字
- [ ] 终端没有错误

**进阶挑战**（可选）:
- 修改按钮颜色
- 添加一个新的段落
- 修改页面标题（`<title>` 标签）

---

## 📊 阶段小结

### 自我评估

完成以下检查，确保你真正掌握了：

**环境能力**
- [ ] 我能启动开发服务器（`npm run dev`）
- [ ] 我知道如何查看终端输出
- [ ] 我能打开浏览器开发者工具

**代码理解**
- [ ] 我知道项目的入口文件在哪（`app/page.tsx`）
- [ ] 我理解什么是"热重载"
- [ ] 我能找到某个文字在代码中的位置

**问题解决**
- [ ] 我能识别基本的错误类型（红色 vs 黄色）
- [ ] 我知道去哪里查看错误信息（终端 + 浏览器Console）
- [ ] 我会向AI寻求帮助

**分数**: _____ / 9

- **7分以下**: 继续巩固，重新做一遍任务
- **8-9分**: 优秀！可以进入阶段2

---

## 🎉 恭喜！

你已经完成了第一阶段！你现在能够：

✅ 在本地运行一个真实的全栈项目
✅ 看到代码修改的即时效果
✅ 理解基本的开发流程

**你已经比90%的人走得更远了！**

---

## ➡️ 下一步

准备好进入阶段2了吗？

**前往**: [阶段2：理解结构](./04-stage2-skeleton.md)

在阶段2，你将学习：
- 项目文件是如何组织的
- 每个文件夹的作用
- 如何快速找到任何功能的代码

---

**生成日期**: 2025-11-16
**文档版本**: v1.0
