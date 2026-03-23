# Copilot CLI 长任务工作流指南

> 解决大任务跨 session 执行时的细节丢失、上下文遗忘、难以回溯等问题。

---

## 核心理念

**不要依赖记忆，依赖文件。**

Copilot CLI 的 session 会 compact、会重置，你的大脑也会遗忘。唯一可靠的是**持久化到文件系统中的信息**。顶级工程师的做法是：把工作过程本身当作一个"工程"来管理——有计划、有日志、有检查点、有交接文档。

---

## 工作流全景图

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  Phase 1    │     │  Phase 2    │     │  Phase 3    │
│  规划与分解  │────▶│  执行与记录  │────▶│  交接与恢复  │
│             │     │  (循环)      │     │             │
└─────────────┘     └─────────────┘     └─────────────┘
     │                    │                    │
  plan.md            progress.md          handoff.md
  todos (SQL)        决策日志               下一步指引
                     验证结果快照
```

---

## Phase 1：规划与分解（每个大任务开始时做一次）

### 步骤 1.1：创建任务计划文件

在开始任何实现之前，先让 Copilot 创建 `plan.md`。这是整个任务的"地图"。

**你对 Copilot 说：**
```
帮我做 XXX 任务，先做个计划，不要开始实现。
```

或者使用 Plan Mode（按 Shift+Tab 切换），输入你的需求。

Copilot 会在 session 文件夹中创建 `plan.md`，包含：
- 问题描述与目标
- 分解后的子任务列表
- 依赖关系
- 技术方案选择及理由

### 步骤 1.2：把计划复制到你的项目目录

**关键动作**——session 文件夹会随 session 消失，但你的项目目录不会：

```bash
# 在项目根目录创建一个工作管理目录
mkdir -p .copilot-work

# 把计划文件复制过来（或直接让 Copilot 在这个目录创建）
cp ~/.copilot/session-state/<session-id>/plan.md .copilot-work/plan.md
```

**更好的做法——直接告诉 Copilot 把文件建在项目目录：**
```
把计划文件写到 .copilot-work/plan.md
```

### 步骤 1.3：任务分解原则

每个子任务应该满足 **"一个 session 能完成"** 的粒度：

| 粒度 | 示例 | 适合？ |
|------|------|--------|
| 太大 | "实现整个用户认证系统" | ❌ |
| 合适 | "实现 JWT token 生成和验证模块" | ✅ |
| 合适 | "编写登录 API 端点及单元测试" | ✅ |
| 太小 | "给变量改个名" | ❌ 不需要追踪 |

---

## Phase 2：执行与记录（每个子任务的工作循环）

这是最关键的阶段。每个子任务按以下循环执行：

### 步骤 2.1：开始子任务前 —— 声明当前目标

每次开新 session 或开始新子任务时，先告诉 Copilot 上下文：

```
我正在做一个大任务，计划在 .copilot-work/plan.md 中。
进度记录在 .copilot-work/progress.md 中。
请先阅读这两个文件，然后我们开始做 [具体子任务名]。
```

这一句话能让新 session 的 Copilot **立即恢复上下文**。

### 步骤 2.2：维护进度日志（最重要的习惯）

在 `.copilot-work/progress.md` 中记录每个关键事件。格式如下：

```markdown
# 进度日志

## 2024-01-15 Session 1

### 子任务：实现 JWT 模块
- ✅ 创建了 src/auth/jwt.ts，实现了 sign/verify 函数
- ✅ 使用 RS256 算法（选择理由：公私钥分离更安全）
- ✅ 单元测试通过（12/12）
- ⚠️ 发现 jsonwebtoken 库 v9 有 breaking change，锁定 v8.5.1
- 📁 关键文件：src/auth/jwt.ts, tests/auth/jwt.test.ts

### 子任务：实现登录 API
- ✅ POST /api/login 端点完成
- ✅ 密码使用 bcrypt 哈希，cost factor = 12
- ❌ 限流中间件还没做（移到下一个子任务）
- 📁 关键文件：src/routes/auth.ts, src/middleware/auth.ts

---

## 2024-01-16 Session 2
...
```

**什么时候更新进度日志？在以下时刻告诉 Copilot：**

```
把刚才的工作结果更新到 .copilot-work/progress.md
```

更新时机：
1. **完成一个子任务时** —— 记录做了什么、关键文件、验证结果
2. **做出重要决策时** —— 记录选了什么方案、为什么
3. **遇到问题/发现坑时** —— 记录问题和解决方案（或待解决）
4. **准备结束 session 时** —— 记录当前状态和下一步

### 步骤 2.3：保存关键输出快照

当 Copilot 输出了重要的信息（测试结果、错误分析、架构建议等），立即保存：

```
把刚才的测试结果保存到 .copilot-work/snapshots/jwt-test-results.txt
```

```
把刚才的错误分析保存到 .copilot-work/snapshots/login-bug-analysis.md
```

**你需要保存的东西：**
- 测试运行结果（特别是失败的）
- 性能测试/基准测试数据
- 复杂的错误排查过程和结论
- 架构设计讨论的最终结论
- 任何你觉得"之后可能需要回头看"的内容

### 步骤 2.4：验证检查清单

每个子任务完成后，走一遍检查：

```markdown
## 子任务完成检查清单
- [ ] 代码改动已保存
- [ ] 测试已通过（记录结果到 progress.md）
- [ ] 关键决策已记录理由
- [ ] 新增/修改的文件列表已记录
- [ ] 发现的问题/TODO 已记录
- [ ] git commit 已提交（带有清晰的 commit message）
```

**告诉 Copilot：**
```
这个子任务做完了，帮我：
1. 运行测试确认通过
2. 更新 progress.md
3. 更新 plan.md 中这个子任务的状态
4. 做一个 git commit
```

---

## Phase 3：Session 交接（防丢失的关键）

### 步骤 3.1：在 session 即将结束前写交接文档

当你感觉一个 session 已经进行了很长时间（通常 15-20 轮对话后），**主动**做交接：

```
我们可能快到 session 限制了，请帮我写一个交接文档到 
.copilot-work/handoff.md，包含：
1. 当前进行到哪一步
2. 正在做的事情的完整上下文
3. 遇到的未解决问题
4. 下一步具体要做什么
5. 需要注意的坑和关键信息
```

### 步骤 3.2：交接文档模板

```markdown
# 任务交接文档

## 上次更新：2024-01-15

## 当前状态
正在实现用户认证系统，已完成 3/7 个子任务。

## 刚才做了什么
- 完成了登录 API 的基本实现
- 正在调试 token 刷新逻辑，问题出在 refresh token 的过期时间计算

## 未解决的问题
1. refresh token 过期时间使用绝对时间还是滑动窗口？（倾向滑动窗口）
2. 并发刷新时的 race condition 还没处理

## 下一步
1. 修复 token 刷新的过期时间计算（参考 src/auth/refresh.ts:42）
2. 添加并发刷新的互斥锁
3. 编写 token 刷新的集成测试

## 关键文件
- src/auth/jwt.ts — token 生成/验证
- src/auth/refresh.ts — token 刷新（进行中）
- src/routes/auth.ts — 认证路由
- tests/auth/ — 所有认证测试

## 注意事项
- jsonwebtoken 锁定 v8.5.1，不要升级到 v9（有 breaking change）
- bcrypt cost factor 用 12，不要改（已做过性能测试，见 snapshots/）
```

### 步骤 3.3：新 session 恢复

新 session 开始时，用一句话恢复全部上下文：

```
我在做一个大任务。请阅读以下文件恢复上下文：
- .copilot-work/plan.md（总体计划）
- .copilot-work/progress.md（进度日志）
- .copilot-work/handoff.md（上次交接文档）
然后继续下一步工作。
```

---

## 文件结构总览

```
你的项目/
├── src/                    # 项目源代码
├── tests/                  # 测试代码
├── .copilot-work/          # 🔑 工作管理目录
│   ├── plan.md             # 总体计划和任务分解
│   ├── progress.md         # 进度日志（按时间记录）
│   ├── handoff.md          # Session 交接文档
│   ├── decisions.md        # 可选：重大决策记录
│   └── snapshots/          # 关键输出快照
│       ├── test-results-01.txt
│       ├── perf-benchmark.md
│       └── bug-analysis-login.md
└── .gitignore              # 把 .copilot-work/ 加进去（如不想提交）
```

---

## 快速参考卡片（日常使用时瞄一眼）

### 🟢 开始新 session 时说

```
请阅读 .copilot-work/ 下的 plan.md、progress.md 和 handoff.md，
恢复上下文后继续工作。
```

### 🔵 完成子任务时说

```
这个子任务完成了，请：
1. 更新 progress.md 记录完成情况
2. 更新 plan.md 标记状态
3. git commit
```

### 🟡 做出重要决定时说

```
把我们刚才的决策记录到 progress.md，包括选了什么和为什么。
```

### 🟠 看到重要输出时说

```
把刚才的 [测试结果/分析/方案] 保存到 
.copilot-work/snapshots/[描述性名字].md
```

### 🔴 准备结束 session 时说

```
请更新 handoff.md 做交接，记录当前状态和下一步。
```

---

## 常见问题

### Q：这套流程会不会太繁琐？

不会。你只需要在 5 个关键时刻各说一句话：
1. 开始 → "读文件恢复上下文"
2. 重要决策 → "记录一下"
3. 子任务完成 → "更新进度"
4. 重要输出 → "保存快照"
5. 结束前 → "写交接"

每次只是多说一句话，省下的是反复排查和重做的大量时间。

### Q：.copilot-work/ 要提交到 git 吗？

看情况：
- **个人项目**：可以提交，方便在不同设备间同步
- **团队项目**：建议加到 `.gitignore`，这是你个人的工作笔记
- **另一个选择**：单独建一个 git 分支或目录来存

### Q：如果忘了做交接就 session 断了怎么办？

别慌。如果你一直在维护 `progress.md`，它就是你的恢复基础。
新 session 中读取 `plan.md` + `progress.md`，大部分上下文都能恢复。
`handoff.md` 是锦上添花——让恢复更快更精准，但不是唯一救命稻草。

### Q：怎么处理特别复杂的调试过程？

对于复杂调试，除了 progress.md 里简要记录外，建议：
```
请把这次调试的完整过程写到 
.copilot-work/snapshots/debug-[问题描述]-[日期].md
包括：现象、排查步骤、尝试过的方案、最终解决方案、根因分析。
```
这样下次遇到类似问题可以直接查阅。

---

## 新项目初始化指南：让 Copilot 自动了解你的项目

> 在项目创建之初就配置好指令文件，Copilot 每次启动都会**自动读取**，无需你反复解释。

### Copilot 自动发现的文件

Copilot CLI 启动时会自动扫描以下位置的文件，**无需手动指定**，所有发现的文件内容会**合并**后注入到上下文中：

| 文件 | 位置 | 作用 | 优先级 |
|------|------|------|--------|
| `.github/copilot-instructions.md` | 项目根目录 | **最核心** — 项目级全局指令 | 仓库级 |
| `.github/instructions/**/*.instructions.md` | 项目根目录 | 按路径/模块拆分的细粒度指令 | 仓库级（模块化） |
| `AGENTS.md` | git root 或 cwd | Agent 行为偏好和工作指令 | 仓库级 |
| `CLAUDE.md` / `GEMINI.md` / `CODEX.md` | git root 或 cwd | 模型特定指令（也会被读取） | 仓库级 |
| `~/.copilot/copilot-instructions.md` | 用户 home 目录 | 全局个人偏好（对所有项目生效） | 用户级 |

> 💡 **仓库级指令优先于用户级指令。** 这意味着团队约定可以覆盖个人偏好。

### 方法一：使用 `/init` 快速初始化（推荐）

```bash
cd /path/to/new-project
copilot
# 进入后输入：
/init
```

`/init` 命令会交互式地引导你创建指令文件，自动生成合理的初始模板。

### 方法二：手动初始化（完全控制）

#### 推荐的项目结构

```
my-project/
├── .github/
│   ├── copilot-instructions.md              # 🔑 项目全局指令（必创建）
│   └── instructions/                         # 可选：模块化指令
│       ├── frontend.instructions.md          # 前端特定规则
│       ├── api.instructions.md               # API 特定规则
│       └── testing.instructions.md           # 测试规范
├── AGENTS.md                                 # 可选：Agent 行为偏好
├── .copilot-work/                            # 长任务管理（本项目的方法论）
│   ├── plan.md
│   ├── progress.md
│   └── handoff.md
└── ...
```

#### 核心文件：`.github/copilot-instructions.md`

这是**最重要的文件**——每次 Copilot 启动都会读取它。应包含以下内容：

```markdown
# 项目概述

本项目是 [一句话描述]。主要功能包括 [核心功能列表]。
技术栈：[语言/框架/数据库等]。

## 构建与运行命令

- `npm install` — 安装依赖
- `npm run dev` — 启动开发服务器
- `npm run build` — 构建生产版本
- `npm run test` — 运行所有测试
- `npm run lint:fix` — 修复 lint 问题

## 代码风格约定

- 使用 TypeScript strict 模式
- 优先使用函数式组件
- 公开 API 必须添加 JSDoc 注释
- commit message 遵循 Conventional Commits 格式

## 项目结构

- `src/` — 源代码
- `src/components/` — React 组件
- `src/api/` — API 路由
- `src/lib/` — 共享工具函数
- `tests/` — 测试文件

## 关键架构决策

- 认证使用 JWT + refresh token
- 数据库使用 PostgreSQL，ORM 使用 Prisma
- API 遵循 RESTful 设计

## 工作流

- 修改代码后运行 `npm run lint:fix && npm test`
- 从 `main` 分支创建 feature 分支
```

> ⚡ **保持简洁、可操作。** 过长的指令会稀释效果。重点写"Copilot 需要知道什么才能正确工作"。

#### 模块化指令：`.github/instructions/*.instructions.md`

当项目较大时，按模块拆分更清晰。每个文件可以指定只对特定路径生效：

**`.github/instructions/frontend.instructions.md`：**
```markdown
---
applyTo: "src/components/**"
---

# 前端组件规范

- 使用 React 函数式组件 + Hooks
- 样式使用 Tailwind CSS
- 组件文件使用 PascalCase 命名
- 每个组件必须有对应的 .test.tsx 文件
```

**`.github/instructions/api.instructions.md`：**
```markdown
---
applyTo: "src/api/**"
---

# API 开发规范

- 所有端点需要输入验证（使用 zod）
- 错误响应统一使用 { error: string, code: number } 格式
- 需要认证的端点使用 authMiddleware
```

#### Agent 行为文件：`AGENTS.md`

定义 Copilot 作为 Agent 时的行为偏好：

```markdown
# Agent 指令

## 通用规则
- 修改代码后必须运行测试
- 每个子任务完成后做 git commit
- 重要决策记录到 .copilot-work/progress.md
- 不确定时先问我，不要猜

## 长任务管理
- 计划文件：.copilot-work/plan.md
- 进度日志：.copilot-work/progress.md
- 交接文档：.copilot-work/handoff.md
- session 结束前主动更新交接文档
```

#### 全局个人偏好：`~/.copilot/copilot-instructions.md`

这个文件影响你**所有项目**，适合放个人编码习惯：

```markdown
# 个人偏好

- 使用中文回复
- 代码注释使用英文
- 优先使用已有的库，避免引入新依赖
- git commit message 使用 Conventional Commits 格式
- 解释技术决策时给出理由
```

### 新项目初始化检查清单

开始一个新项目时，按以下顺序操作：

```
1. ☐ 创建项目目录，git init
2. ☐ 创建 .github/copilot-instructions.md（项目概述 + 命令 + 规范）
3. ☐ 创建 AGENTS.md（Agent 行为偏好）
4. ☐ 如需模块化规则，创建 .github/instructions/*.instructions.md
5. ☐ 创建 .copilot-work/ 目录（长任务管理）
6. ☐ 确认 ~/.copilot/copilot-instructions.md 有你的全局偏好
7. ☐ git commit 初始文件
8. ☐ 启动 copilot，输入 /instructions 确认所有文件被正确加载
```

> 💡 **验证技巧：** 在 Copilot session 中输入 `/instructions` 可以查看当前加载了哪些指令文件，以及每个文件的启用/禁用状态。

### 一分钟快速初始化脚本

如果你想最快速度搭好骨架：

```bash
mkdir -p .github/instructions .copilot-work/snapshots

cat > .github/copilot-instructions.md << 'EOF'
# 项目概述
本项目是 [TODO: 填写项目描述]。

## 命令
- [TODO: 填写构建/测试/运行命令]

## 代码风格
- [TODO: 填写代码规范]

## 项目结构
- [TODO: 填写目录结构说明]
EOF

cat > AGENTS.md << 'EOF'
# Agent 指令
- 修改代码后运行测试
- 每完成一个子任务做 git commit
- 重要决策记录到 .copilot-work/progress.md
- 不确定时先问，不要猜
EOF

echo "初始化完成！请编辑 .github/copilot-instructions.md 填写项目信息。"
```

---

## Copilot CLI Session 管理完全指南

> 掌握 session 的生命周期——启动、使用、结束、恢复、备份——是高效使用 Copilot CLI 的基础。

### 基础概念

每次运行 `copilot` 命令都会创建或进入一个 **session**。Session 是你和 Copilot 对话的完整上下文，包含：

- 所有对话历史
- Copilot 生成的计划文件
- 工具调用记录
- 上下文压缩的检查点

Session 数据存储在本地：
```
~/.copilot/session-state/{session-id}/
├── events.jsonl      # 完整对话历史
├── workspace.yaml    # 元数据
├── plan.md           # 实施计划（如果创建了的话）
├── checkpoints/      # 上下文压缩历史
└── files/            # 持久化的 session 文件
```

---

### 1. 启动 Session

#### 启动全新 session

```bash
# 最基本的方式——在项目目录下运行
cd /path/to/your/project
copilot
```

首次启动时，Copilot 会询问你是否信任当前目录的文件。选择：
1. **Yes, proceed** — 仅本次 session 信任
2. **Yes, and remember this folder** — 永久信任此目录
3. **No, exit (Esc)** — 退出

#### 带参数启动

```bash
# 指定模型
copilot --model claude-sonnet-4.5

# 直接带一个提示（编程式调用，完成后自动退出）
copilot -p "修复 README 中的拼写错误" --allow-all-tools

# 恢复上一个 session
copilot --continue

# 恢复指定 session
copilot --resume SESSION-ID

# 启用实验性功能（如 autopilot 模式）
copilot --experimental

# 显示欢迎动画
copilot --banner
```

#### 模式切换

进入 session 后，按 **Shift+Tab** 在三种模式间循环：
- **Interactive（交互模式）** — 默认模式，逐步确认
- **Plan（计划模式）** — 先制定计划再执行
- **Autopilot（自动驾驶模式）** — 实验性功能，自主完成任务

---

### 2. 查看 Session 信息

在 session 内部使用以下命令了解当前状态：

```
/session                    # 查看当前 session 概要信息
/session checkpoints        # 查看所有上下文压缩检查点列表
/session checkpoints 1      # 查看第 1 个检查点的详细内容
/session files              # 查看 session 中创建的临时文件
/session plan               # 查看当前的实施计划
/context                    # 可视化展示 token 使用情况
/usage                      # 显示 session 使用统计（请求次数、时长、代码行数等）
```

---

### 3. Session 内上下文管理

#### 自动压缩（Auto-compaction）

当对话接近 **95% token 上限** 时，Copilot 会**自动在后台压缩**历史记录，不会中断你的工作流。这意味着 session 可以"无限"持续。

#### 手动压缩

```
/compact                    # 手动触发上下文压缩（释放 token 空间）
```

压缩会创建一个 **checkpoint**——包含被压缩内容的摘要。你可以通过 `/session checkpoints` 查看。

#### 清除历史（在同一 session 内重新开始）

```
/clear                      # 清除对话历史，相当于在同一 session 内重开
/new                        # 同上，/clear 的别名
```

> ⚠️ `/clear` 会丢弃当前对话历史。如果有重要内容，先保存到文件。

---

### 4. Session 重命名

给 session 起一个有意义的名字，方便以后查找：

```
/rename jwt-auth-feature    # 重命名当前 session
/session rename my-task     # 同上，完整写法
```

---

### 5. 结束 Session

有多种方式结束当前 session：

| 方式 | 操作 | 说明 |
|------|------|------|
| 命令退出 | `/exit` 或 `/quit` | 优雅退出，session 会自动保存 |
| 快捷键退出 | `Ctrl+C` 连按两次 | 第一次取消当前操作，第二次退出 |
| 关闭终端 | `Ctrl+D` | 直接关闭 |

**⚡ 结束前的最佳实践：**

```
# 1. 让 Copilot 总结当前进度
请把本次 session 的工作总结更新到 .copilot-work/progress.md

# 2. 写交接文档（如果任务未完成）
请更新 .copilot-work/handoff.md 做交接

# 3. 提交代码
帮我做一个 git commit，描述本次改动

# 4. 然后退出
/exit
```

---

### 6. 恢复 Session（核心技能）

#### 方法一：快速恢复上一个 session

```bash
copilot --continue
```

这会直接恢复你**最近关闭的本地 session**，包括所有对话历史和上下文。

#### 方法二：从 session 列表中选择

```bash
copilot --resume
```

或者在已有 session 内部：

```
/resume
```

会显示一个 session 列表，用方向键选择后回车恢复。

#### 方法三：恢复指定 session

```bash
copilot --resume SESSION-ID
```

或在 session 内部：

```
/resume SESSION-ID
```

> 💡 **如何找到 Session ID？** 
> - 在 session 内使用 `/session` 查看当前 ID
> - 查看 `~/.copilot/session-state/` 目录下的文件夹名

#### 方法四：从 GitHub 云端恢复（delegate 的任务）

如果你使用了 `/delegate` 将任务发送到 GitHub 云端执行，可以用 `/resume` 查看并恢复这些远程任务。

---

### 7. 备份 Session 内容

#### 方法一：导出为 Markdown 文件

```
/share file ./session-backup.md           # 导出到指定文件
/share file                                # 导出到默认位置
```

这会把整个对话历史导出为一个可读的 Markdown 文件。

#### 方法二：导出为 GitHub Gist

```
/share gist                                # 上传为 GitHub Gist（私密）
```

适合在不同设备间共享 session 内容。

#### 方法三：手动备份 session 目录

```bash
# 查看所有 session
ls -lt ~/.copilot/session-state/

# 备份特定 session
cp -r ~/.copilot/session-state/{session-id} ~/backups/copilot-sessions/

# 备份所有 session
cp -r ~/.copilot/session-state/ ~/backups/copilot-sessions-all/
```

#### 方法四：让 Copilot 帮你保存关键内容到项目目录

这是**最推荐的方式**——把重要内容持久化到你的项目中：

```
# 保存工作总结
把本次 session 的所有关键成果和决策总结到 .copilot-work/progress.md

# 保存特定输出
把刚才的测试结果保存到 .copilot-work/snapshots/test-results-20240115.txt

# 保存交接文档
把完整的工作上下文写到 .copilot-work/handoff.md
```

> 🔑 **核心原则：session 会消失，项目目录不会。** 任何重要内容都应该落地到项目文件中。

---

### 8. Session 管理速查表

```
┌──────────────────────────────────────────────────────────────────┐
│                    Session 生命周期速查                           │
├──────────────────────────────────────────────────────────────────┤
│                                                                  │
│  启动        copilot                  # 新 session               │
│              copilot --continue       # 恢复上一个               │
│              copilot --resume         # 选择恢复                 │
│                                                                  │
│  查看        /session                 # 当前 session 信息        │
│              /context                 # token 使用情况           │
│              /usage                   # 使用统计                 │
│                                                                  │
│  管理        /rename NAME             # 重命名                   │
│              /compact                 # 压缩上下文               │
│              /clear                   # 清除历史                 │
│              /model                   # 切换模型                 │
│                                                                  │
│  备份        /share file PATH         # 导出为文件               │
│              /share gist              # 导出为 Gist              │
│                                                                  │
│  切换        /resume                  # 切换到其他 session       │
│              Shift+Tab                # 切换模式                 │
│                                                                  │
│  结束        /exit                    # 退出                     │
│              Ctrl+C × 2              # 快速退出                 │
│              Ctrl+D                   # 关闭                     │
│                                                                  │
└──────────────────────────────────────────────────────────────────┘
```

---

### 9. 实战场景示例

#### 场景 A：开始一个新的大任务

```bash
# 1. 进入项目目录，启动 Copilot
cd ~/projects/my-app
copilot

# 2. 在 Copilot 中
/rename auth-feature                    # 给 session 命名
# 然后切换到 Plan 模式（Shift+Tab），输入需求
```

#### 场景 B：昨天的任务今天继续

```bash
# 快速恢复
copilot --continue

# 或者从列表中找
copilot --resume
# 在列表中找到 "auth-feature" session
```

#### 场景 C：任务完成，需要归档

```
# 在 session 内
/share file .copilot-work/sessions/auth-feature-session.md
请把最终成果总结到 .copilot-work/progress.md
/exit
```

#### 场景 D：session 意外断开

```bash
# 不用慌，session 自动保存
copilot --continue
# 对话历史完整恢复，继续工作
```

#### 场景 E：session 上下文太长，Copilot 开始"遗忘"

```
# 手动压缩上下文
/compact

# 如果效果不好，保存关键内容后清空重来
请把当前工作状态完整写到 .copilot-work/handoff.md
/clear
# 然后重新加载上下文
请阅读 .copilot-work/handoff.md 恢复上下文
```

---

### 10. 常见问题补充

#### Q：Session 数据会自动清理吗？

目前不会自动清理。所有 session 数据保存在 `~/.copilot/session-state/` 下。如果磁盘空间紧张，可以手动删除不需要的旧 session 目录。

#### Q：`/compact` 和 `/clear` 的区别？

- **`/compact`**：压缩历史但保留摘要，Copilot 仍然"记得"之前的内容要点。会创建 checkpoint。
- **`/clear`**：彻底清除所有对话历史，Copilot 完全从零开始。不可恢复。

#### Q：能同时运行多个 session 吗？

可以。在不同的终端窗口分别运行 `copilot`，每个窗口都是独立的 session。适合同时处理不相关的任务。

#### Q：`--continue` 和 `--resume` 的区别？

- **`--continue`**：直接恢复**最近一个**关闭的 session，无需选择。
- **`--resume`**：显示 session 列表让你**选择**要恢复的 session。也可以 `--resume SESSION-ID` 直接指定。

#### Q：如何防止重要 session 被误删？

1. 养成用 `/rename` 命名 session 的习惯
2. 重要 session 用 `/share file` 导出备份
3. 关键工作内容始终保存到项目目录（`.copilot-work/`）

---

## 一句话总结

> **把"脑子里的上下文"变成"文件里的上下文"——这就是跨 session 工作的全部秘诀。**
