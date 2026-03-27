# Harness Engineering × 量化交易策略研发完全指南

> 通过 GitHub Copilot CLI 的 Harness Engineering 方法论，系统化地完成量化交易策略研发工作。

---

## 目录

1. [什么是 Harness Engineering？](#什么是-harness-engineering)
2. [全局画面：量化研发中的应用](#全局画面量化研发中的应用)
3. [第一课：两套文件系统的分工](#第一课两套文件系统的分工)
4. [第二课：项目初始化——搭建 Harness](#第二课项目初始化搭建-harness)
5. [第三课：开始策略研发——用 Harness 驱动 Copilot](#第三课开始策略研发用-harness-驱动-copilot)
6. [第四课：Session 交接——最关键的技能](#第四课session-交接最关键的技能)
7. [第五课：利用 MCP 工具增强研发能力](#第五课利用-mcp-工具增强研发能力)
8. [第六课：五个关键时刻](#第六课五个关键时刻你只需多说一句话)
9. [第七课：实战工作流完整示例](#第七课实战工作流完整示例)
10. [核心原则总结](#核心原则总结)
11. [附录：指令文件完整模板](#附录指令文件完整模板)

---

## 什么是 Harness Engineering？

Harness 的英文原义是"马具/线束"——用来驾驭马匹或连接系统的工具。

在 Copilot CLI 的语境下，**Harness Engineering** 的核心理念是：

> **把 Copilot CLI 当作一个可以被"驾驭"的工程系统，通过 plan.md / progress.md / handoff.md / AGENTS.md 等文件构成一个"线束"（harness），让 AI 在你的控制下持续、可恢复、可追踪地完成复杂工作。**

更具体地说：

- **不要依赖 Copilot 的记忆，依赖文件。** Session 会 compact、会重置，唯一可靠的是持久化到文件系统中的信息。
- **把工作过程本身当作一个"工程"来管理。** 有计划、有日志、有检查点、有交接文档。
- **你是项目经理，Copilot 是你的全栈工程师。** 你通过 Harness 文件指挥它做事。

对于量化交易策略研发，这尤其重要，因为：

- 策略研发是**多阶段**的（研究 → 设计 → 回测 → 优化 → 风控 → 部署）
- 每个阶段都有**大量细节和决策**需要追踪
- 一个策略可能要**跨几天甚至几周**的 session
- 回测结果、参数选择等**数据是核心资产**，丢失代价极高

---

## 全局画面：量化研发中的应用

```
┌─────────────────── Harness Engineering 在量化研发中的应用 ───────────────────┐
│                                                                              │
│  你的角色：策略研究员 + 项目经理                                               │
│  Copilot 的角色：全栈工程师 + 数据分析师 + 研究助手                            │
│  Harness 文件系统：你控制 Copilot 的"缰绳"                                   │
│                                                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐      │
│  │ 1.研究   │→│ 2.设计   │→│ 3.回测   │→│ 4.优化   │→│ 5.部署   │      │
│  │ 市场/数据 │  │ 策略逻辑  │  │ 历史验证  │  │ 参数/风控  │  │ 实盘/监控 │      │
│  └──────────┘  └──────────┘  └──────────┘  └──────────┘  └──────────┘      │
│       │             │             │             │             │               │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │                    Harness 文件系统（缰绳）                          │     │
│  │  plan.md │ progress.md │ handoff.md │ decisions.md │ snapshots/    │     │
│  └─────────────────────────────────────────────────────────────────────┘     │
│       │             │             │             │             │               │
│  ┌─────────────────────────────────────────────────────────────────────┐     │
│  │                    指令文件系统（方向盘）                              │     │
│  │  AGENTS.md │ copilot-instructions.md │ *.instructions.md           │     │
│  └─────────────────────────────────────────────────────────────────────┘     │
└──────────────────────────────────────────────────────────────────────────────┘
```

---

## 第一课：两套文件系统的分工

### A. 指令文件 = 方向盘（控制 Copilot **怎么工作**）

这些文件 Copilot 启动时**自动读取**，定义了它的行为模式。你不需要每次告诉 Copilot "用中文回复"或"跑完代码要测试"——写进指令文件，它就自动遵守。

| 文件 | 位置 | 作用 | 量化场景下写什么 |
|------|------|------|-----------------|
| `.github/copilot-instructions.md` | 项目根目录 | 项目全局指令 | 技术栈、数据源、回测框架、代码规范 |
| `AGENTS.md` | git root | Agent 行为偏好 | 策略研发中的工作纪律 |
| `.github/instructions/*.instructions.md` | 项目根目录 | 模块化指令 | 策略编写规范、回测规范、风控红线 |
| `~/.copilot/copilot-instructions.md` | 用户主目录 | 全局个人偏好 | 中文回复、代码风格、commit 格式 |

> 💡 **关键点：** 仓库级指令优先于用户级指令。Copilot 启动时自动发现并加载这些文件。使用 `/instructions` 命令可以查看当前加载了哪些文件。

### B. Harness 文件 = 缰绳（控制 Copilot **做什么**）

这些文件你在工作过程中**持续更新**，是跨 session 的记忆。session 会消失，但这些文件永远在。

| 文件 | 作用 | 量化场景下写什么 |
|------|------|-----------------|
| `.copilot-work/plan.md` | 总体计划（地图） | 策略的完整研发路线图，子任务分解 |
| `.copilot-work/progress.md` | 进度日志（日记） | 每个研发步骤的结果、发现、数据 |
| `.copilot-work/handoff.md` | 交接文档（接力棒） | 当前在做什么，下一步怎么做 |
| `.copilot-work/decisions.md` | 决策记录（判决书） | 为什么选这个指标/参数/方案 |
| `.copilot-work/snapshots/` | 输出快照（证据库） | 回测结果、参数扫描表、风控报告 |

> 🔑 **核心原则：session 会消失，项目目录不会。任何重要内容都应该落地到项目文件中。**

---

## 第二课：项目初始化——搭建 Harness

当你要开始一个新的量化策略研发项目时，按以下步骤初始化。

### 步骤 2.1：创建项目结构

```bash
mkdir -p my-quant-strategy
cd my-quant-strategy
git init

# 创建所有目录
mkdir -p .github/instructions
mkdir -p .copilot-work/snapshots
mkdir -p src/{strategy,data,backtest,risk,utils}
mkdir -p tests
mkdir -p data
```

最终结构如下：

```
my-quant-strategy/
├── .github/
│   ├── copilot-instructions.md          # 项目全局指令（最重要）
│   └── instructions/
│       ├── strategy.instructions.md     # 策略编写规范
│       └── backtest.instructions.md     # 回测规范
├── .copilot-work/                       # Harness 文件
│   ├── plan.md                          # 策略研发计划
│   ├── progress.md                      # 研发进度日志
│   ├── handoff.md                       # Session 交接
│   ├── decisions.md                     # 决策记录
│   └── snapshots/                       # 关键输出快照
│       ├── backtest-result-v1.md
│       ├── param-scan-20260327.csv
│       └── rb-market-analysis.md
├── AGENTS.md                            # Agent 行为指令
├── src/                                 # 策略代码
│   ├── strategy/                        # 策略逻辑
│   ├── data/                            # 数据获取/处理
│   ├── backtest/                        # 回测引擎
│   ├── risk/                            # 风控模块
│   └── utils/                           # 工具函数
├── tests/                               # 测试代码
├── data/                                # 本地数据缓存
└── requirements.txt                     # Python 依赖
```

### 步骤 2.2：编写核心指令文件

#### `.github/copilot-instructions.md`（量化策略版）

```markdown
# 项目概述

本项目是一个商品期货量化交易策略研发项目。

## 技术栈
- 语言：Python 3.11+
- 回测框架：vnpy / backtrader / 自研
- 数据源：tushare / akshare / 交易所API
- 分析：pandas, numpy, scipy, statsmodels
- 可视化：matplotlib, plotly

## 构建与运行
- `pip install -r requirements.txt` — 安装依赖
- `python -m pytest tests/` — 运行测试
- `python src/backtest/run.py` — 运行回测
- `python src/data/download.py` — 下载数据

## 代码规范
- 策略类统一继承 BaseStrategy
- 信号函数返回 +1(做多), -1(做空), 0(平仓)
- 所有参数通过配置文件/字典传入，不要硬编码
- 回测结果必须包含：年化收益、最大回撤、夏普比率、胜率

## 关键约定
- 数据时间戳统一用 UTC+8
- 资金单位：人民币元
- 持仓计算考虑手续费和滑点
- 所有策略必须有对应的单元测试

## 项目结构
- `src/strategy/` — 策略逻辑实现
- `src/data/` — 数据获取和处理
- `src/backtest/` — 回测引擎
- `src/risk/` — 风控模块
- `src/utils/` — 工具函数
- `tests/` — 测试文件
- `data/` — 本地数据缓存
```

#### `AGENTS.md`（策略研发版）

```markdown
# Agent 指令

## 工作纪律
- 修改策略代码后必须运行回测验证
- 每个研发阶段完成后做 git commit
- 重要发现和决策立即记录到 .copilot-work/progress.md
- 回测结果保存快照到 .copilot-work/snapshots/
- 不确定时先问我，不要猜（特别是关于交易逻辑）

## 量化研发专用规则
- 讨论策略时要基于数据和统计，不要主观臆断
- 参数优化必须做样本外测试，防止过拟合
- 回测结果要用表格呈现：年化收益、最大回撤、夏普比率、卡玛比率、胜率
- 比较多个方案时做对照表
- 风控红线：单笔最大亏损不超过账户2%，最大回撤不超过15%

## 长任务管理
- 计划文件：.copilot-work/plan.md
- 进度日志：.copilot-work/progress.md
- 交接文档：.copilot-work/handoff.md
- 决策日志：.copilot-work/decisions.md
- Session 结束前主动更新交接文档
- 每完成一个子任务自动更新 handoff.md

## Compaction 防丢失规则（必须严格遵守）

### 持续状态同步
你必须在以下时机自动更新 `.copilot-work/handoff.md`，无需我提醒：

1. **每完成一个子任务时** — 立即更新
2. **每做出一个重要决策时** — 立即记录
3. **每次对话超过 10 轮时** — 主动更新一次
4. **每次你感觉对话很长时** — 主动更新一次

### Compaction 后的恢复动作
如果你发现自己的上下文被压缩了，必须**立即**执行：

1. 读取 `.copilot-work/handoff.md` 恢复任务上下文
2. 读取 `.copilot-work/plan.md` 确认总体计划
3. 读取 `.copilot-work/progress.md` 确认进度
4. 向我简要报告当前状态和下一步计划
5. 等我确认后再继续工作
```

#### `.github/instructions/strategy.instructions.md`（可选的模块化指令）

```markdown
---
applyTo: "src/strategy/**"
---

# 策略编写规范

- 所有策略类继承自 `BaseStrategy`
- 信号函数 `generate_signal()` 返回 +1(做多), -1(做空), 0(平仓)
- 参数通过 `__init__(self, params: dict)` 传入
- 必须实现 `backtest_report()` 方法，返回标准化的回测指标
- 不要在策略中硬编码参数，全部走配置
- 每个策略文件必须有对应的 `tests/test_*.py`
```

### 步骤 2.3：初始化 Harness 文件

创建空的 Harness 文件骨架：

```bash
# plan.md — 稍后让 Copilot 帮你填写
echo "# 策略研发计划" > .copilot-work/plan.md

# progress.md — 随工作推进更新
echo "# 进度日志" > .copilot-work/progress.md

# handoff.md — session 交接用
echo "# 任务交接文档" > .copilot-work/handoff.md

# decisions.md — 记录关键决策
echo "# 决策记录" > .copilot-work/decisions.md
```

### 步骤 2.4：首次 Git 提交

```bash
git add -A
git commit -m "chore: 初始化量化策略研发项目结构和 Copilot 指令文件"
```

### 步骤 2.5：验证

启动 `copilot`，输入 `/instructions`，确认所有指令文件被正确加载。

---

## 第三课：开始策略研发——用 Harness 驱动 Copilot

### 阶段一：规划（让 Copilot 生成 plan.md）

**你在 Copilot CLI 中说的第一句话：**

```
我要研发一个商品期货趋势跟踪策略，目标品种是螺纹钢(RB)和热卷(HC)。
先做个研发计划，不要开始实现。把计划写到 .copilot-work/plan.md。
```

Copilot 会生成类似这样的计划：

```markdown
# 螺纹钢/热卷趋势跟踪策略研发计划

## 目标
开发一个趋势跟踪策略，在螺纹钢(RB)和热卷(HC)上实现稳定收益。
目标：年化 > 15%，最大回撤 < 15%，夏普 > 1.0。

## 子任务分解

### Phase 1: 数据准备
- [ ] T1: 获取 RB/HC 历史日线+分钟线数据（2020-2026）
- [ ] T2: 数据清洗和预处理（处理夜盘、节假日、涨跌停）
- [ ] T3: 特征工程（计算技术指标：MA, EMA, ATR, RSI, MACD 等）

### Phase 2: 策略设计
- [ ] T4: 研究 RB/HC 历史走势特征（趋势性、波动率、季节性）
- [ ] T5: 设计趋势识别信号（均线/通道/动量）
- [ ] T6: 设计入场/出场规则（含止损止盈）

### Phase 3: 回测验证
- [ ] T7: 搭建回测引擎（或配置 vnpy/backtrader）
- [ ] T8: 初始回测 + 结果分析
- [ ] T9: 样本内/样本外分割测试（防过拟合）

### Phase 4: 优化与风控
- [ ] T10: 参数敏感性分析（网格搜索 + 热力图）
- [ ] T11: 加入风控规则（止损/仓位管理/最大持仓）
- [ ] T12: 多品种组合回测 + 相关性分析

### Phase 5: 部署准备
- [ ] T13: 实盘接口对接（CTP/vnpy）
- [ ] T14: 模拟盘测试（至少运行 2 周）

## 技术方案
- 数据获取：akshare（免费）+ tushare（备选）
- 回测框架：先用 pandas 自研轻量级框架，复杂场景再用 vnpy
- 策略类型：趋势跟踪（均线+ATR通道）

## 风控约束
- 单笔最大亏损：账户的 2%
- 最大回撤控制：15%
- 最大同向持仓：3 个品种
```

> 💡 **注意：** 计划不需要一次写完美。随着研发深入，你可以让 Copilot 更新 plan.md。

### 阶段二：逐个执行子任务

**关键节奏：每个子任务是一个工作单元。**

#### 开始子任务 T1

你说：

```
请阅读 .copilot-work/plan.md，我们开始做 T1：获取历史数据。
用 akshare 获取螺纹钢主力连续合约 2020年至今的日线数据。
```

Copilot 会：
1. 读取 plan.md 了解全局
2. 编写数据下载脚本
3. 运行并调试
4. 保存数据

完成后你说：

```
T1 完成了，请：
1. 把结果更新到 .copilot-work/progress.md
2. 更新 plan.md 中 T1 的状态为 [x]
3. git commit
```

**progress.md 会被更新成：**

```markdown
# 进度日志

## 2026-03-27 Session 1

### T1: 获取历史数据 ✅
- ✅ 使用 akshare 获取了 RB 主力连续 2020-01-02 至 2026-03-27 的日线数据
- ✅ 共 1,512 个交易日，字段：open/high/low/close/volume/open_interest
- ✅ 数据保存到 data/rb_daily.csv（152KB）
- ⚠️ 发现 akshare 的夜盘数据合并方式与 tushare 不同，已统一为次日归属
- 📁 关键文件：src/data/download.py, data/rb_daily.csv

#### 完成检查清单
- [x] 代码改动已保存
- [x] 数据文件已验证（行数、字段、日期范围）
- [x] 关键发现已记录（夜盘数据归属问题）
- [x] git commit 已提交
```

#### 继续下一个子任务

```
继续做 T2：数据清洗和预处理。
```

重复上述循环。

### 阶段三：遇到重要决策——记录

当你和 Copilot 讨论策略设计方案后做出选择时：

```
把我们刚才的决策记录到 .copilot-work/decisions.md：
选择了双均线+ATR通道的趋势跟踪方案。
```

**decisions.md 会记录：**

```markdown
# 决策记录

## D1: 趋势识别方法选择（2026-03-27）

### 背景
需要为 RB/HC 策略选择趋势识别方法。

### 候选方案
| 方案 | 优点 | 缺点 |
|------|------|------|
| A. MACD 交叉 | 经典方法，参数少 | 延迟较大，震荡市频繁假信号 |
| B. 双均线+ATR通道 | 自适应波动率，延迟较小 | 参数较多（4个） |
| C. 布林带突破 | 简单直观 | 急涨急跌时表现不稳定 |

### 决定
**选择 B — 双均线+ATR通道**

### 理由
1. ATR 能自适应不同波动率环境，RB/HC 波动率变化大
2. 双均线比 MACD 延迟更小（直接使用价格而非 EMA 的 EMA）
3. ATR 通道比布林带对急涨急跌更稳健（不依赖标准差）
4. 参数虽多但物理意义明确（快均线周期、慢均线周期、ATR周期、通道倍数）

### 相关文件
- src/strategy/trend_following.py — 策略实现
- .copilot-work/snapshots/signal-comparison.md — 三种方案的对比分析

---

## D2: 止损方式选择（2026-03-28）

### 决定
**选择 ATR 动态止损（2倍 ATR）**

### 理由
- 固定点位止损在不同波动率下表现不一致
- 百分比止损忽略了品种波动特征
- ATR 止损能自动适应市场状态

---
```

### 阶段四：保存关键输出

当回测跑出结果时：

```
把刚才的回测结果保存到 .copilot-work/snapshots/backtest-v1-20260327.md
```

**快照文件示例：**

```markdown
# 回测结果 v1 — 2026-03-27

## 策略
双均线+ATR通道趋势跟踪

## 参数
- 快均线: 10日, 慢均线: 30日
- ATR周期: 14日, 通道倍数: 2.0
- 止损: 2倍 ATR
- 手续费: 万分之一, 滑点: 1 tick

## 测试区间
- 样本内: 2020-01-02 ~ 2024-06-30
- 样本外: 2024-07-01 ~ 2026-03-27

## 结果

### 样本内
| 指标 | RB | HC | 组合 |
|------|-----|-----|------|
| 年化收益率 | 18.3% | 15.7% | 17.0% |
| 最大回撤 | -12.7% | -14.2% | -10.8% |
| 夏普比率 | 1.42 | 1.18 | 1.55 |
| 卡玛比率 | 1.44 | 1.11 | 1.57 |
| 胜率 | 38.2% | 35.6% | 36.9% |
| 盈亏比 | 2.8 | 2.5 | 2.6 |
| 交易次数 | 127 | 118 | 245 |

### 样本外
| 指标 | RB | HC | 组合 |
|------|-----|-----|------|
| 年化收益率 | 14.1% | 11.3% | 12.7% |
| 最大回撤 | -9.8% | -11.5% | -8.2% |
| 夏普比率 | 1.21 | 0.95 | 1.32 |

## 分析
- 样本外收益比样本内衰减约 25%，在合理范围内
- HC 样本外表现弱于 RB，可能与 2025 年黑色系政策扰动有关
- 组合效果显著：最大回撤从 ~13% 降至 ~10%，夏普从 ~1.3 升至 ~1.5

## 下一步
- 做参数敏感性分析，确认当前参数不在"悬崖边"
- 增加仓位管理模块
```

---

## 第四课：Session 交接——最关键的技能

这是 Harness Engineering 中**最重要的技能**——确保你的工作不会因为 session 中断而丢失。

### 场景 A：当天工作结束

你说：

```
我今天的工作要结束了，请：
1. 更新 .copilot-work/progress.md 记录今天的所有工作
2. 更新 .copilot-work/handoff.md 做交接
3. git commit 所有改动
```

**handoff.md 会被更新成：**

```markdown
# 任务交接文档

## 上次更新：2026-03-27

## 当前状态
正在研发 RB/HC 趋势跟踪策略，已完成 8/14 个子任务（T1-T8）。
目前处于 Phase 3（回测验证）。

## 今天做了什么
- 完成了初始回测（T8），结果达到目标（夏普 > 1.0）
- 发现 HC 在 2025 年表现较弱，可能与政策扰动有关
- 样本外衰减约 25%，在合理范围内

## 正在进行
- T9：样本内/样本外分割测试 — 已完成框架搭建，还没跑完全部组合

## 未解决的问题
1. HC 策略在 2025 年的回撤较大（-11.5%），是否需要加入基本面过滤？
2. 夜盘跳空时 ATR 计算是否需要特殊处理？
3. 参数敏感性分析还没做，不确定当前参数是否鲁棒

## 下一步（按优先级）
1. 完成 T9 的剩余样本外测试
2. 做 T10 参数敏感性分析（重点检查快均线 8-15，慢均线 25-40 的网格）
3. 根据敏感性结果决定是否调整参数

## 关键文件
- src/strategy/trend_following.py — 策略主逻辑
- src/backtest/engine.py — 回测引擎
- src/data/download.py — 数据下载脚本
- data/rb_daily.csv — RB 日线数据
- data/hc_daily.csv — HC 日线数据
- tests/test_strategy.py — 策略单元测试

## 注意事项
- akshare 夜盘数据已统一归属次日（见 D3 决策记录）
- 手续费按万分之一计算，滑点按 1 tick
- 回测起始资金 100 万，每手保证金按交易所标准
```

### 场景 B：第二天恢复工作

启动 Copilot CLI (`copilot` 或 `copilot --continue`)，第一句话：

```
我在做一个量化策略研发项目。请阅读以下文件恢复上下文：
- .copilot-work/plan.md（总体计划）
- .copilot-work/progress.md（进度日志）
- .copilot-work/handoff.md（上次交接）
- .copilot-work/decisions.md（决策记录）
然后告诉我当前状态和下一步。
```

Copilot 会读完文件后回复类似：

> "你已完成 T1-T8，处于 Phase 3。初始回测结果已达标（组合夏普 1.55），样本外衰减约 25%。当前有 3 个未解决问题（HC 政策扰动、ATR 夜盘处理、参数鲁棒性）。下一步是完成 T9 剩余测试，然后做 T10 参数敏感性分析。"

### 场景 C：对话太长，主动 compact

当你感觉对话已经进行了很多轮（通常 15-20 轮），**主动控制节奏**：

```
请更新 .copilot-work/handoff.md，我要手动 compact。
```

等 Copilot 更新完文件后：

```
/compact
```

compact 完成后：

```
请读取 .copilot-work/handoff.md 恢复上下文，继续工作。
```

**这个流程的优势：**
1. 你控制 compaction 时机，不会被突然打断
2. 确保状态文件在 compaction **前**已更新
3. compaction 后立即恢复，无缝衔接

### 场景 D：Session 意外断开

不用慌。如果你一直在维护 progress.md 和 handoff.md：

```bash
# 恢复 session
copilot --continue

# 或直接新开 session，用 Harness 文件恢复
copilot
# 然后说：读文件恢复上下文
```

---

## 第五课：利用 MCP 工具增强研发能力

Copilot CLI 内置了多种 MCP 工具，在量化策略研发中非常有用。

### 5.1 Web 搜索——市场研究

```
帮我搜索"螺纹钢期货 2025年 趋势特征"的最新研究和分析。
```

```
搜索 "commodity futures trend following strategy python" 的最新开源实现。
```

### 5.2 Firecrawl——爬取数据和文档

```
用 Firecrawl 爬取上期所螺纹钢合约的最新交易规则和合约参数。
```

```
帮我从 akshare 的文档页面提取期货数据相关的 API 列表。
```

### 5.3 GitHub 搜索——找开源参考

```
在 GitHub 上搜索 Python 写的商品期货趋势跟踪策略。
```

```
搜索 vnpy 框架中双均线策略的实现示例。
```

### 5.4 Context7——查最新文档

```
查一下 akshare 最新版本中获取期货历史数据的 API 用法。
```

### 5.5 实际使用建议

| 研发阶段 | 推荐使用的工具 | 用途 |
|----------|---------------|------|
| 市场研究 | Web Search, Firecrawl | 了解品种特性、市场结构、政策变化 |
| 数据获取 | Context7, Firecrawl | 查 akshare/tushare API 文档 |
| 策略设计 | GitHub Search, Web Search | 查找开源策略参考、学术论文 |
| 回测验证 | Copilot 自身 | 编写代码、运行回测、分析结果 |
| 参数优化 | Copilot 自身 | 网格搜索、热力图、统计分析 |

---

## 第六课：五个关键时刻——你只需多说一句话

整个 Harness Engineering 的日常使用，归结为 5 个时刻各说一句话：

| # | 时刻 | 你说什么 | 效果 |
|---|------|---------|------|
| 🟢 | 开新 session | "读 .copilot-work/ 下的文件恢复上下文" | 秒速恢复所有记忆 |
| 🔵 | 完成子任务 | "更新 progress.md，标记 plan.md，git commit" | 记录成果，防丢失 |
| 🟡 | 做出决策 | "记录到 decisions.md" | 未来回溯为什么这么做 |
| 🟠 | 看到关键输出 | "保存到 snapshots/" | 回测数据是核心资产 |
| 🔴 | 结束 session | "更新 handoff.md 做交接" | 确保下次无缝衔接 |

**每次只是多说一句话，省下的是反复排查和重做的大量时间。**

---

## 第七课：实战工作流完整示例

假设你从零开始做一个 RB 趋势策略，**一天的完整工作流如下**：

```
=== 早上 9:00 — 开始工作 ===

$ copilot

你: 我要研发 RB 趋势策略，先做计划，写到 .copilot-work/plan.md。
→ Copilot 生成 plan.md，分解为 14 个子任务

你: 开始 T1 — 获取 RB 历史日线数据。
→ Copilot 写代码 → 获取数据 → 验证数据完整性
→ 你: "T1 完成，更新 progress.md，commit。"

你: 继续 T2 — 数据清洗。
→ Copilot 处理缺失值、异常值、夜盘数据
→ 你: "T2 完成，更新 progress.md，commit。"

你: 继续 T3 — 特征工程。
→ Copilot 计算 MA/EMA/ATR/RSI/MACD 等指标
→ 你: "T3 完成，更新 progress.md，commit。"

=== 对话较长了，主动 compact ===

你: 更新 handoff.md，我要 compact。
→ Copilot 更新 handoff.md
→ 你: /compact
→ 你: "读 handoff.md 恢复上下文，继续。"

=== 下午 14:00 — 策略设计 ===

你: 开始 T4 — 分析 RB 历史走势特征。
→ Copilot 做统计分析：趋势持续时间、波动率分布、季节性
→ 你: "保存分析结果到 snapshots/rb-analysis.md。"

你: 开始 T5 — 设计趋势信号。讨论用什么指标组合。
→ 对比了 MACD、双均线+ATR、布林带三种方案
→ 你: "记录决策到 decisions.md：选择双均线+ATR。"
→ 你: "T5 完成，更新 progress.md，commit。"

=== 下午 17:00 — 结束工作 ===

你: 今天结束了，请：
     1. 更新 progress.md 记录今天所有工作
     2. 更新 handoff.md 做交接
     3. commit 所有改动

→ Copilot 更新所有文件并提交
→ 你: /exit

=== 第二天 9:00 — 恢复工作 ===

$ copilot

你: 读 .copilot-work/ 下的 plan/progress/handoff/decisions 文件恢复上下文，
    告诉我当前状态和下一步。

→ Copilot: "你完成了 T1-T5（Phase 1+2 的前半部分）。
    下一步是 T6：设计入场/出场规则。
    你选择了双均线+ATR通道方案（见 D1 决策）。
    有 2 个待解决问题：夜盘 ATR 计算、HC 数据还没获取。"

你: 好的，先处理 HC 数据，再继续 T6。
→ 继续工作...
```

---

## 核心原则总结

### Harness Engineering 的四大原则

1. **📄 文件即记忆**
   - 不依赖 session 上下文，一切持久化到文件
   - session 会消失，项目目录永远在

2. **🗺️ 计划即地图**
   - plan.md 让你和 Copilot 始终知道全局目标和当前位置
   - 子任务粒度控制在"一个 session 能完成"

3. **🤝 交接即保险**
   - handoff.md 是你对抗 compact/session 中断/遗忘的终极武器
   - 主动 compact 比被动等待更安全

4. **📊 数据即资产**（量化专属）
   - 回测结果、参数扫描、风控报告都要用 snapshots/ 保存
   - 这些是策略演进的证据链，也是未来复盘的基础

### 一句话总结

> **把"脑子里的上下文"变成"文件里的上下文"——这就是 Harness Engineering 的全部秘诀。**

---

## 附录：指令文件完整模板

### A. 量化项目一键初始化脚本

```bash
#!/bin/bash
# 使用方法: ./init-quant-project.sh my-strategy-name

PROJECT_NAME=${1:-"my-quant-strategy"}

mkdir -p "$PROJECT_NAME"
cd "$PROJECT_NAME"
git init

# 目录结构
mkdir -p .github/instructions
mkdir -p .copilot-work/snapshots
mkdir -p src/{strategy,data,backtest,risk,utils}
mkdir -p tests
mkdir -p data

# 项目指令文件
cat > .github/copilot-instructions.md << 'EOF'
# 项目概述

本项目是一个商品期货量化交易策略研发项目。

## 技术栈
- 语言：Python 3.11+
- 数据：akshare, pandas, numpy
- 回测：自研 / vnpy / backtrader
- 分析：scipy, statsmodels
- 可视化：matplotlib, plotly

## 命令
- `pip install -r requirements.txt` — 安装依赖
- `python -m pytest tests/` — 运行测试
- `python src/backtest/run.py` — 运行回测

## 代码规范
- 策略类继承 BaseStrategy
- 信号函数返回 +1(多), -1(空), 0(平)
- 参数通过字典/配置传入，不硬编码
- 回测结果包含：年化收益、最大回撤、夏普、胜率

## 约定
- 时间戳: UTC+8
- 资金单位: 人民币元
- 持仓计算考虑手续费和滑点
EOF

# Agent 指令
cat > AGENTS.md << 'EOF'
# Agent 指令

## 工作纪律
- 修改策略代码后运行回测验证
- 每阶段完成后 git commit
- 发现和决策记录到 .copilot-work/progress.md
- 回测结果保存到 .copilot-work/snapshots/
- 不确定时先问我

## 量化规则
- 讨论基于数据和统计，不主观臆断
- 参数优化做样本外测试防过拟合
- 回测结果用表格呈现
- 风控红线：单笔亏损 < 2%，最大回撤 < 15%

## 长任务管理
- 计划: .copilot-work/plan.md
- 进度: .copilot-work/progress.md
- 交接: .copilot-work/handoff.md
- 决策: .copilot-work/decisions.md
- session 结束前主动更新 handoff.md
- 每完成子任务自动更新 handoff.md

## Compaction 恢复
上下文被压缩后立即：
1. 读取 .copilot-work/handoff.md
2. 读取 .copilot-work/plan.md
3. 读取 .copilot-work/progress.md
4. 报告状态，等我确认
EOF

# 策略模块指令
cat > .github/instructions/strategy.instructions.md << 'EOF'
---
applyTo: "src/strategy/**"
---
# 策略编写规范
- 继承 BaseStrategy
- generate_signal() 返回 +1/-1/0
- 参数通过 __init__(params: dict) 传入
- 必须有对应的 tests/test_*.py
EOF

# Harness 文件
echo "# 策略研发计划" > .copilot-work/plan.md
echo "# 进度日志" > .copilot-work/progress.md
echo "# 任务交接文档" > .copilot-work/handoff.md
echo "# 决策记录" > .copilot-work/decisions.md

# .gitignore
cat > .gitignore << 'EOF'
__pycache__/
*.pyc
.venv/
data/*.csv
data/*.pkl
*.egg-info/
.DS_Store
EOF

# requirements.txt
cat > requirements.txt << 'EOF'
akshare>=1.10.0
pandas>=2.0.0
numpy>=1.24.0
scipy>=1.10.0
matplotlib>=3.7.0
plotly>=5.14.0
pytest>=7.3.0
EOF

# 初始提交
git add -A
git commit -m "chore: 初始化量化策略研发项目

- 创建项目目录结构
- 配置 Copilot 指令文件（copilot-instructions.md, AGENTS.md）
- 初始化 Harness 文件（plan/progress/handoff/decisions）
- 添加 Python 依赖配置

Co-authored-by: Copilot <223556219+Copilot@users.noreply.github.com>"

echo ""
echo "✅ 项目 '$PROJECT_NAME' 初始化完成！"
echo ""
echo "下一步："
echo "  cd $PROJECT_NAME"
echo "  copilot"
echo "  # 然后说：我要研发 XXX 策略，先做个计划。"
```

### B. 全局个人偏好模板

在 `~/.copilot/copilot-instructions.md` 中：

```markdown
# 个人偏好

- 使用中文回复
- 代码注释使用英文
- 优先使用已有的库，避免引入新依赖
- git commit message 使用中文描述
- 解释技术决策时给出理由
- 量化相关的数值保留 2 位小数
- 表格展示优先于纯文字描述
```

---

> **最后提醒：Harness Engineering 不是负担，而是加速器。** 每次多说一句"记录一下"、"保存一下"，就能节省未来数小时的重复工作。
