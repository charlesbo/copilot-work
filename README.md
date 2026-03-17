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

## 一句话总结

> **把"脑子里的上下文"变成"文件里的上下文"——这就是跨 session 工作的全部秘诀。**
