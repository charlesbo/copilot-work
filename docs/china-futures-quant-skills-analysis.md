# 中国期货量化交易 Skills 全景分析与推荐

> 本文档对所有主要 Skills 市场（SkillsMP、AgentKits、agentskills.io、awesome-copilot、GitHub 开源项目）进行全面搜索，梳理与**中国期货量化交易**相关的所有 Skills、插件、MCP 工具，并给出分类分析和推荐方案。

---

## 目录

- [一、直接可用的 Skills 与 MCP 工具](#一直接可用的-skills-与-mcp-工具)
- [二、按量化工作流分析](#二按量化工作流分析)
- [三、推荐方案](#三推荐方案)
- [四、关键发现与市场机会](#四关键发现与市场机会)
- [五、详细工具与 Skill 说明](#五详细工具与-skill-说明)
- [六、参考资源](#六参考资源)

---

## 一、直接可用的 Skills 与 MCP 工具

### 🥇 第一梯队：中国期货市场专属

| 工具 | 类型 | 说明 | 推荐度 |
|------|------|------|--------|
| **QMT-MCP** | MCP Server | 国金 QMT 量化助手，支持策略生成、回测、实盘、风控。基于 XTQuant + FastMCP，支持 A 股和商品期货 | ⭐⭐⭐⭐⭐ |
| **VnPy (VeighNa)** | 框架/可封装 Skill | 中国期货量化交易事实标准。支持 149 家期货公司 CTP 接入，Tick/分钟级回测，AI/ML 集成 | ⭐⭐⭐⭐⭐ |
| **DolphinQuant Strategy-Copilot** | Web 平台 | 零代码 AI 策略生成，已支持上期所（黄金、铜、铝等），自然语言回测 | ⭐⭐⭐⭐ |
| **QuantConnect MCP Server** | MCP Server | 国际标准 MCP 规范，支持多市场回测与 AI Agent 接入 | ⭐⭐⭐⭐ |

### 🥈 第二梯队：通用量化 Skills（可适配期货）

来源：[agiprolabs/claude-trading-skills](https://github.com/agiprolabs/claude-trading-skills)（62 个交易 Skills）

| Skill | 功能 | 期货适配难度 |
|-------|------|-------------|
| **backtrader** | 事件驱动回测引擎 | 低 |
| **correlation-analysis** | 滚动/尾部/制度相关性分析，适合跨品种套利 | 低 |
| **cointegration-analysis** | Engle-Granger/Johansen 协整检验，期货配对交易 | 低 |
| **feature-engineering** | ML 特征构建（价格/成交量/微结构） | 低 |
| **exit-strategies** | 止损/止盈/追踪止损策略模块 | 无需适配 |
| **custom-indicators** | 自定义技术指标（NVT、资金流等） | 低 |
| **portfolio-analytics** | 组合分析与绩效归因 | 低 |
| **risk-management** | 系统性风控规则与仓位管理 | 低 |

### 🥉 第三梯队：辅助平台与 Skills 市场

| 平台 | 说明 | 金融类 Skills |
|------|------|--------------|
| **SkillsMP.com** | 700K+ 开源 Agent Skills 聚合市场 | 财务建模、投资分析、组合管理、量化策略、回测框架等 |
| **AgentKits.net** | 按职业分类的 Skills 目录 | 金融、数据分析、量化交易分类可浏览 |
| **skillsdirectory.org** | 安全审计的 Agent Skills 目录 | 编码、研究、金融、量化交易类 |
| **MCP Market** | MCP 工具与 Skills 聚合 | 数据采集、分析、ML、自动化交易类 |

### 中国本土补充框架

| 框架 | 特点 | 期货支持 |
|------|------|---------|
| **QUANTAXIS** | Python 事件驱动，A 股+期货全品种 | ✅ 直接支持 |
| **RQAlpha** | 米筐开源回测引擎，A 股为主 | ⚠️ 有限支持 |
| **Hikyuu** | C++/Python 混合，高性能 | ⚠️ 需定制 |
| **AkShare / TuShare** | 数据获取 | ✅ 期货行情数据 |

---

## 二、按量化工作流分析

一个完整的期货量化工作流及对应的 Skill/工具覆盖：

```
数据获取 → 策略研发 → 回测验证 → 风控管理 → 实盘交易 → 绩效分析
```

| 环节 | 推荐工具/Skill | 说明 |
|------|---------------|------|
| **行情数据** | QMT-MCP、VnPy (CTP)、TuShare、AkShare | CTP 原生对接，Tick 级实时数据 |
| **策略研发** | DolphinQuant（自然语言）、feature-engineering skill、VnPy AI 模块 | 零代码到专业代码均可覆盖 |
| **回测验证** | backtrader skill、VnPy 回测引擎、QMT-MCP 回测模块 | 多级别回测（Tick/分钟/日线） |
| **信号分析** | correlation-analysis、cointegration-analysis、custom-indicators | 套利/配对/趋势信号 |
| **风控管理** | exit-strategies skill、QMT-MCP 风控模块、VnPy 风控引擎 | 仓位控制/止损止盈/风险敞口 |
| **实盘执行** | VnPy + CTP、QMT-MCP + XTQuant | 直连期货公司柜台 |
| **绩效报告** | DolphinQuant 可视化、QMT-MCP 绩效模块、portfolio-analytics skill | 夏普/最大回撤/年化收益 |

### 覆盖度评估

```
✅ 完全覆盖    ⚠️ 需适配    ❌ 空白

数据获取  ✅  — CTP/QMT/TuShare/AkShare 均可直接使用
策略研发  ✅  — DolphinQuant 零代码 + VnPy/QMT 专业代码
回测验证  ✅  — 多个引擎可选
信号分析  ✅  — trading-skills 多个分析 Skill 可直接用
风控管理  ✅  — exit-strategies + QMT-MCP 风控
实盘执行  ✅  — VnPy CTP / QMT XTQuant
绩效分析  ✅  — 多平台可视化报告

结论：全流程已有工具覆盖，但缺少统一的 Copilot Skill 封装层
```

---

## 三、推荐方案

### 方案 A：零代码快速体验

**适合人群**：新手、产品经理、快速验证想法

```
DolphinQuant Strategy-Copilot
  ↓ 自然语言描述策略（中文）
  ↓ AI 自动生成 + 一键回测
  ↓ 可视化绩效报告（收益率/夏普/最大回撤）
```

| 优点 | 缺点 |
|------|------|
| 中文对话即可，无需编程 | 品种覆盖有限（目前仅上期所） |
| 即时看到回测结果 | 非开源，不可深度定制 |
| 专业绩效报表 | 不支持实盘交易 |

**访问地址**：[dolphinquant.com/strategy-copilot](https://dolphinquant.com/strategy-copilot)

---

### 方案 B：AI + MCP 智能体驱动

**适合人群**：有 Python 基础的开发者、半自动化交易团队

```
QMT-MCP Server + AI Agent (Copilot/ChatGPT/DeepSeek)
  ↓ MCP 协议标准化接口
  ↓ 策略生成 + 回测 + 风控 + 下单
  ↓ 本地部署，全品种支持
```

| 优点 | 缺点 |
|------|------|
| 开源、模块化、可扩展 | 需 Python 3.8+ 基础 |
| 支持实盘下单 | 需要 QMT 柜台账户（国金等） |
| AI 原生集成（ChatGPT/DeepSeek） | 期货品种覆盖取决于 QMT 权限 |
| 多层次风控 | 初期配置较复杂 |

**开源地址**：[github.com/guangxiangdebizi/QMT-MCP](https://github.com/guangxiangdebizi/QMT-MCP)

---

### 方案 C：专业全栈方案 ⭐ 最推荐

**适合人群**：专业量化团队、私募、自营交易

```
VnPy (CTP) + agiprolabs/trading-skills + 自建 MCP Skill
  ↓ VnPy 负责底层（CTP 接入 / 回测引擎 / 实盘执行）
  ↓ trading-skills 提供策略分析模块（协整/相关性/特征工程）
  ↓ 自建 SKILL.md 封装团队工作流
  ↓ Copilot CLI 作为统一交互入口
```

| 优点 | 缺点 |
|------|------|
| 最灵活、最强大 | 初期搭建成本较高 |
| 149 家期货公司全覆盖 | 需要较强 Python/C++ 能力 |
| Tick 级回测 + AI/ML 集成 | VnPy 尚未被封装为标准 Skill |
| 可自定义一切 | 需要自行维护 MCP 层 |
| VnPy 4.0 原生 AI 因子研究 | — |

**核心组件**：
- VnPy 官网：[vnpy.cn](https://www.vnpy.cn)
- 开源代码：[gitee.com/vnpypro/vnpy](https://gitee.com/vnpypro/vnpy)
- Trading Skills：[github.com/agiprolabs/claude-trading-skills](https://github.com/agiprolabs/claude-trading-skills)

---

## 四、关键发现与市场机会

### 现状总结

1. **目前没有专门为"中国期货 CTP"打造的现成 Copilot Skill** —— 这是一个明确的空白市场
2. **QMT-MCP** 是目前最接近的中国本土 MCP 量化方案，但偏 QMT/迅投生态，非 CTP 直连
3. **VnPy** 仍是中国期货 CTP 量化的事实标准（149 家期货公司），但尚未被封装为标准 Agent Skill
4. **agiprolabs/trading-skills** 的 62 个 Skills 偏 Crypto/DeFi，但回测、统计分析类 Skills 可直接复用
5. **SkillsMP** 等市场的金融类 Skills 偏通用（财务建模/投资分析），缺少中国期货特定场景

### 🚀 建议行动：填补空白

将 VnPy 核心功能封装为标准 Agent Skill，建议的 Skill 拆分：

| 建议 Skill 名称 | 封装内容 |
|-----------------|---------|
| `ctp-market-data` | CTP 行情订阅与历史数据获取 |
| `futures-backtest` | VnPy 回测引擎的 Skill 化封装 |
| `futures-strategy` | 常用期货策略模板（CTA 趋势/套利/动量） |
| `futures-risk-control` | 期货风控规则引擎（保证金/仓位/止损） |
| `futures-execution` | CTP 下单执行与订单管理 |
| `futures-analytics` | 绩效分析与可视化报告生成 |

**发布目标**：agentskills.io + awesome-copilot 市场

---

## 五、详细工具与 Skill 说明

### QMT-MCP 详解

QMT-MCP 是基于 FastMCP 和 XTQuant 构建的模块化量化交易助手：

- **策略生成**：AI 自动生成量化策略代码
- **回测引擎**：历史回测，支持逐年/逐月绩效分解
- **实盘执行**：连接 QMT 柜台，支持仿真与实盘下单
- **风控模块**：可配参数的多层风控（防误操作、大额下单保护）
- **部署方式**：Python 3.8+，本地 `.env` 配置即可运行
- **AI 接入**：作为 MCP Server 供 Copilot/ChatGPT/DeepSeek 等 Agent 调用

### VnPy (VeighNa) 详解

VnPy 是中国最大的开源量化交易框架：

- **交易所支持**：上期所(SHFE)、大商所(DCE)、郑商所(CZCE)、中金所(CFFEX)、能源交易中心(INE)
- **期货公司**：支持 149 家国内期货公司 CTP 接入
- **回测能力**：Tick 级和分钟级回测，多进程并行优化
- **AI/ML**：4.0 版本原生支持多因子特征工程和 ML 模型训练
- **技术指标**：内置 TA-Lib（200+ 技术指标）
- **GUI**：PyQt5 图形界面，支持策略监控和手动干预
- **数据补充**：K 线数据补充 API，本地数据存储

### agiprolabs/claude-trading-skills 详解

62 个交易与量化 Skills，以下为期货量化可复用子集：

| Skill | 描述 | 期货应用场景 |
|-------|------|-------------|
| `backtrader` | 事件驱动回测 | CTA 策略回测 |
| `correlation-analysis` | 多资产相关性分析 | 跨品种套利信号 |
| `cointegration-analysis` | 协整检验 | 配对交易/跨期套利 |
| `feature-engineering` | ML 特征构建 | 因子挖掘/信号研究 |
| `exit-strategies` | 出场策略库 | 止损止盈规则 |
| `custom-indicators` | 自定义指标 | 技术分析扩展 |
| `portfolio-analytics` | 组合绩效分析 | 多策略/多品种组合评估 |
| `risk-management` | 风控框架 | 仓位与敞口管理 |

---

## 六、参考资源

### 工具与框架

| 资源 | 链接 |
|------|------|
| VnPy 官网 | [vnpy.cn](https://www.vnpy.cn) |
| VnPy 开源代码 | [gitee.com/vnpypro/vnpy](https://gitee.com/vnpypro/vnpy) |
| QMT-MCP 开源项目 | [github.com/guangxiangdebizi/QMT-MCP](https://github.com/guangxiangdebizi/QMT-MCP) |
| DolphinQuant Strategy-Copilot | [dolphinquant.com/strategy-copilot](https://dolphinquant.com/strategy-copilot) |
| QuantConnect MCP Server | [quantconnect.com/mcp](https://www.quantconnect.com/mcp) |
| Trading Skills（62 个） | [github.com/agiprolabs/claude-trading-skills](https://github.com/agiprolabs/claude-trading-skills) |

### Skills 市场

| 市场 | 链接 |
|------|------|
| SkillsMP（金融分类） | [skillsmp.com/categories/finance-investment](https://skillsmp.com/categories/finance-investment) |
| AgentKits | [agentkits.net](https://www.agentkits.net/) |
| Skills Directory | [skillsdirectory.org](https://www.skillsdirectory.org/) |
| MCP Market | [mcpmarket.com/tools/skills](https://mcpmarket.com/tools/skills) |
| Awesome Copilot 插件中心 | [awesome-copilot.github.com](https://awesome-copilot.github.com) |

### 中国量化资源索引

| 资源 | 链接 |
|------|------|
| 中国量化资源索引 (awesome-quant) | [github.com/thuquant/awesome-quant](https://github.com/thuquant/awesome-quant) |
| 国际量化工具列表 (awesome-quant) | [github.com/wilsonfreitas/awesome-quant](https://github.com/wilsonfreitas/awesome-quant) |
| 回测框架对比 (2026) | [Backtrader vs VnPy vs Qlib](https://dev.to/linou518/backtrader-vs-vnpy-vs-qlib-a-deep-comparison-of-python-quant-backtesting-frameworks-2026-3gjl) |
| QMT-MCP 实操教程 | [bilibili.com (BV1w8LwzvEXK)](https://www.bilibili.com/video/BV1w8LwzvEXK/) |
| AI+MCP 量化智能体教程 | [知乎专栏](https://zhuanlan.zhihu.com/p/1926985137570189694) |

---

*本文档最后更新于 2026 年 4 月。量化交易涉及真实资金风险，请务必在仿真环境充分测试后再用于实盘。*
