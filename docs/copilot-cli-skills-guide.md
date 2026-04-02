# GitHub Copilot CLI Skills 完全指南

> 本文档详细介绍 GitHub Copilot CLI 的 Skills（技能）系统，包括基本概念、使用方法、插件市场扩展以及当前最流行的 Skills 分类汇总。

---

## 目录

- [一、Skills 基本概念](#一skills-基本概念)
- [二、快速上手：分步教程](#二快速上手分步教程)
- [三、插件市场（Plugin Marketplace）](#三插件市场plugin-marketplace)
- [四、可扩展的 Skills 分类大全](#四可扩展的-skills-分类大全)
- [五、当前最流行的 Skills 与插件](#五当前最流行的-skills-与插件)
- [六、自定义 Skill 开发指南](#六自定义-skill-开发指南)
- [七、安全注意事项](#七安全注意事项)
- [八、参考链接](#八参考链接)

---

## 一、Skills 基本概念

### 什么是 Skills？

Skills 是 Copilot CLI 的**可复用增强能力模块**。每个 Skill 封装了特定领域的专业工作流程，可以让 Copilot 在对话中自动加载并执行专业任务。

### 核心特性

| 特性 | 说明 |
|------|------|
| **按需加载** | 仅在匹配到用户意图时才加载，不浪费上下文窗口 |
| **跨平台兼容** | 遵循 agentskills.io 开放标准，可在 Copilot CLI、VS Code、Claude Code 等工具间通用 |
| **模块化结构** | 每个 Skill 是一个包含 `SKILL.md` 的文件夹，可含脚本、模板、配置 |
| **多级发现** | 项目级 `.github/skills/`、用户级 `~/.copilot/skills/`、或通过插件安装 |

### Skill 的组成结构

```
my-skill/
├── SKILL.md          # 核心指令文件（YAML frontmatter + Markdown 说明）
├── scripts/          # 可选：自动化脚本
├── templates/        # 可选：代码/文档模板
└── config/           # 可选：配置文件
```

**SKILL.md 示例：**

```yaml
---
name: csv-analysis
description: 分析 CSV 数据并生成可视化报告
tools: [bash, python]
---

# CSV 数据分析 Skill

当用户请求分析 CSV 文件时，执行以下步骤：
1. 读取并解析 CSV 文件结构
2. 生成数据统计摘要
3. 创建可视化图表
4. 输出分析报告
```

---

## 二、快速上手：分步教程

### 第一步：查看当前可用的 Skills

在 Copilot CLI 交互界面中输入：

```
/skills
```

这会列出所有已安装和可用的 Skills，包括内置 Skills 和通过插件安装的 Skills。

### 第二步：通过自然语言触发 Skill

**你不需要记住任何特殊命令**，直接用自然语言描述需求即可：

```
# 示例 1：触发 CLI 自动化工具构建
你说: "帮我为这个 GUI 应用构建一个命令行自动化工具"
→ 自动触发 cli-anything skill

# 示例 2：触发测试运行
你说: "运行这个项目的测试套件并更新测试报告"
→ 自动触发 test skill

# 示例 3：触发代码验证
你说: "验证这个 CLI 工具是否符合规范标准"
→ 自动触发 validate skill
```

### 第三步：跟随 Skill 引导完成任务

每个 Skill 都有自己的标准化流程。例如 `cli-anything` skill 的执行流程：

```
1. 📖 读取 HARNESS.md 规范文件
2. 🔍 分析目标应用的功能和界面
3. 🏗️ 按照规范生成 CLI 工具代码
4. 🧪 编写并运行测试
5. 📋 输出验证报告
```

### 第四步：管理已安装的 Skills

```bash
/skills              # 查看所有 Skills
/plugin              # 管理插件（含 Skills）
/plugin install xxx  # 安装新插件
```

---

## 三、插件市场（Plugin Marketplace）

### 什么是插件市场？

插件市场是集中发现、安装和更新 Skills 及插件的平台。Copilot CLI 支持**多个插件市场同时使用**，包括官方和社区维护的市场。

### 主要插件市场一览

| 市场名称 | 类型 | 特点 | 访问方式 |
|----------|------|------|----------|
| **copilot-plugins** | 🏛️ 官方 | GitHub 官方默认市场，质量有保障 | 内置默认，无需额外配置 |
| **awesome-copilot** | 🌐 社区 | 社区驱动，数百款插件，有专属网站和搜索引擎 | [awesome-copilot.github.com](https://awesome-copilot.github.com) |
| **agentskills.io** | 📐 标准 | 开放标准平台，跨工具兼容（Copilot/Claude/Codex） | [agentskills.io](https://agentskills.io) |
| **claudeforge-marketplace** | 🔄 跨平台 | 共享开放标准，与 Claude Code 生态互通 | 通过 `/plugin marketplace add` 添加 |
| **VS Code 扩展市场** | 🖥️ IDE | 通过 VS Code Copilot 自定义界面安装 Agent Plugin | VS Code 内置 |

### 如何使用插件市场

#### 浏览市场中的插件

```bash
# 浏览官方市场
/plugin marketplace browse copilot-plugins

# 浏览社区市场
/plugin marketplace browse awesome-copilot
```

#### 从市场安装插件

```bash
# 安装指定插件
/plugin install <plugin-name>

# 从指定市场安装
/plugin install <plugin-name>@<marketplace-name>

# 示例：安装 Azure Skills 插件
/plugin marketplace add microsoft/azure-skills
/plugin install azure@azure-skills
```

#### 添加自定义市场

```bash
# 添加第三方或私有插件市场
/plugin marketplace add <marketplace-url>
```

#### 安装后重新加载

```bash
# 如果插件包含 MCP 服务器，需要重新加载
/mcp reload
```

---

## 四、可扩展的 Skills 分类大全

### 🔧 开发工具类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **cli-anything** | 为任意 GUI 应用构建完整的 CLI 自动化工具 |
| **scaffolding** | 项目脚手架生成（React、Rails、Next.js 等） |
| **docker-for-copilot** | Docker 容器化辅助、Dockerfile 生成、漏洞分析 |
| **github-actions** | GitHub Actions 工作流管理、CI/CD 集成 |

### 🧪 测试与质量类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **test** | 运行测试套件并更新测试报告（TEST.md） |
| **test-case-generator** | 自动生成单元测试和集成测试脚手架 |
| **validate** | 验证工具/代码是否符合规范标准 |
| **code-analysis** | 静态分析、代码检查、安全审计 |

### 📝 文档与报告类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **readme-generator** | 自动生成项目 README 文档 |
| **commit-summarizer** | 分析 Git 历史，自动生成 Changelog |
| **csv-analysis** | CSV 数据分析与可视化报告生成 |

### ☁️ 云服务与基础设施类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **azure-skills** | Azure 云资源管理与部署 |
| **aws-skills** | AWS 服务集成与管理 |
| **power-platform** | Microsoft Power Platform（Power Pages、Power Apps、Dataverse） |

### 🔍 搜索与知识类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **perplexityai** | 实时编码问题搜索与引用式回答 |
| **stack-overflow** | 集成 Stack Overflow 问答库 |
| **repository-review** | 全面的代码库分析与建议 |

### 🏗️ 工程管理类

| Skill / 插件 | 功能说明 |
|--------------|----------|
| **list** | 列出所有已安装/生成的 CLI 工具 |
| **refine** | 改进已有 CLI 工具，扩展功能覆盖 |
| **devops-workflow** | DevOps 流水线管理与监控 |

---

## 五、当前最流行的 Skills 与插件

### 🏆 Top 10 最受欢迎插件排行

| 排名 | 插件名称 | 类别 | 亮点 |
|------|---------|------|------|
| 1 | **Docker for Copilot** | 开发工具 | 容器化全流程，集成 Docker Scout 漏洞扫描 |
| 2 | **PerplexityAI** | 搜索知识 | 实时搜索引擎驱动的编码问答 |
| 3 | **Stack Overflow** | 搜索知识 | 社区验证的技术解答直达终端 |
| 4 | **Azure Skills** | 云服务 | 微软官方出品，Power Platform 深度集成 |
| 5 | **cli-anything** | 开发工具 | GUI→CLI 自动化工具生成，附带完整测试框架 |
| 6 | **GitHub Actions Integration** | 工程管理 | 终端内管理工作流、触发部署、监控构建 |
| 7 | **Test Automation Bundle** | 测试质量 | 一键测试运行 + 代码审查 + CI 报告集成 |
| 8 | **README Generator** | 文档报告 | 智能分析项目结构，自动生成专业文档 |
| 9 | **Code Analysis Bundle** | 测试质量 | 静态分析 + Lint + 安全审计一体化 |
| 10 | **Commit Summarizer** | 文档报告 | Git 历史分析，自动生成结构化 Changelog |

### 🌟 内置 Skills 速查

Copilot CLI 默认附带的 Skills（无需额外安装）：

| Skill | 命令触发词示例 | 功能 |
|-------|---------------|------|
| `cli-anything` | "构建 CLI 工具" | 为 GUI 应用创建命令行自动化工具 |
| `list` | "列出所有工具" | 查看已安装/生成的工具列表 |
| `refine` | "改进这个工具" | 扩展和优化现有 CLI 工具 |
| `test` | "运行测试" | 执行测试套件并更新报告 |
| `validate` | "验证工具规范" | 检查是否符合 HARNESS.md 标准 |

---

## 六、自定义 Skill 开发指南

### 创建你自己的 Skill

#### 1. 创建 Skill 目录

```bash
mkdir -p .github/skills/my-custom-skill
```

#### 2. 编写 SKILL.md

```yaml
---
name: my-custom-skill
description: 我的自定义 Skill 描述
tools: [bash, python]
---

# My Custom Skill

## 触发条件
当用户请求执行 XXX 任务时触发此 Skill。

## 执行步骤
1. 步骤一：...
2. 步骤二：...
3. 步骤三：...
```

#### 3. 添加可选的脚本和模板

```bash
.github/skills/my-custom-skill/
├── SKILL.md
├── scripts/
│   └── analyze.sh
└── templates/
    └── report-template.md
```

#### 4. 测试你的 Skill

启动 Copilot CLI，用自然语言触发你的 Skill，验证其行为是否符合预期。

### 发布 Skill 到市场

1. 将 Skill 打包为 Plugin 格式
2. 提交到 `agentskills.io`、`awesome-copilot` 或 GitHub 仓库
3. 通过 Marketplace API 发布

---

## 七、安全注意事项

> ⚠️ **重要提醒**

- **审查代码**：插件可以包含脚本和 Hooks，安装前务必审查代码内容
- **信任来源**：优先使用官方市场（`copilot-plugins`）的插件
- **权限控制**：注意插件请求的工具和路径权限
- **社区插件**：从社区市场安装时，检查发布者信誉和代码仓库活跃度
- **私有市场**：企业用户可搭建私有插件市场，确保内部合规

---

## 八、参考链接

| 资源 | 链接 |
|------|------|
| GitHub Copilot CLI 官方文档 | [docs.github.com/copilot](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli) |
| 插件查找与安装指南 | [docs.github.com/.../plugins-finding-installing](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing) |
| Awesome Copilot 社区插件中心 | [awesome-copilot.github.com](https://awesome-copilot.github.com) |
| Agent Skills 完全指南 | [chris-ayers.com](https://chris-ayers.com/posts/agent-skills-plugins-marketplace/) |
| VS Code Agent 插件文档 | [code.visualstudio.com/.../agent-plugins](https://code.visualstudio.com/docs/copilot/customization/agent-plugins) |
| 创建 Agent 插件教程 | [kenmuse.com](https://www.kenmuse.com/blog/creating-agent-plugins-for-vs-code-and-copilot-cli/) |
| Copilot CLI 扩展 Cookbook | [htek.dev](https://htek.dev/articles/copilot-cli-extensions-cookbook-examples) |
| 插件系统与 Skills 架构 | [deepwiki.com](https://deepwiki.com/github/copilot-cli/5.5-plugin-system-and-skills) |

---

*本文档最后更新于 2026 年 4 月。随着 Copilot CLI 生态的快速迭代，建议定期查阅官方文档获取最新信息。*
