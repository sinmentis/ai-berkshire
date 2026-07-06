中文 | [English](README_EN.md) | [日本語](README_JA.md)

> 🍴 **这是 fork**：本仓库 fork 自 [xbtlin/ai-berkshire](https://github.com/xbtlin/ai-berkshire)。本 fork 的改动：将 `skills/*.md` 全部改写为 agent 中立的动作语言（不再写死 Claude Code 专属的 `TeamCreate`/`SendMessage`/`TeamDelete` 等工具调用），使其能在 **GitHub Copilot CLI** 下也能正确执行，同时保持 Claude Code / Codex 的原有兼容性；新增 `COPILOT.md` 说明 Copilot CLI 的工具映射约定。设计文档见 [`docs/superpowers/specs/`](docs/superpowers/specs/)，实施计划见 [`docs/superpowers/plans/`](docs/superpowers/plans/)。

# AI Berkshire - AI 时代的价值投资研究框架

> "Price is what you pay, value is what you get." — Warren Buffett

**AI Berkshire** 是一套同时兼容 Claude Code、Codex 与 GitHub Copilot CLI 的投资研究 Skill 合集，将巴菲特、芒格、段永平、李录四位价值投资大师的方法论系统化、结构化，通过 AI Agent 实现专业级投资研究。

[设计理念](#设计理念) · [Skills 一览](#skills-一览19个) · [Setup Guide](#setup-guide) · [使用](#使用) · [实战报告](#实战研究报告)

---

## 设计理念

直接问 AI "这只股票值不值得买"，通常会得到一篇"一方面...另一方面..."的平衡分析，没法直接拿来做决策。AI Berkshire 想解决的是分析质量和决策纪律的问题：

- **四大师对抗，而非单一分析**：段永平（生意本质）、巴菲特（护城河与估值）、芒格（逆向思考）、李录（长期确定性）四个视角独立研究、互相挑战，而不是拼接成一份四平八稳的报告
- **反偏见机制**：信息丰富度分级（A/B/C）、逆向失败场景推演、红线一票否决清单、反共识检查，防止"资料多=确定性高"的幻觉
- **金融数据精确性**：所有计算走 [`tools/financial_rigor.py`](tools/financial_rigor.py)（Python `decimal.Decimal` 精确十进制，不用 `float`），不靠 LLM 心算；关键数据至少 2 个独立来源交叉验证
- **可复现**：同样的输入产出结构一致、深度一致的报告，方便跨公司、跨时间对比
- **多 Agent 并行**：`/investment-team` 等团队型 skill 并行启动多个独立子任务同时研究，而不是把一个 prompt 拆成几段——每个角色独立搜索、独立判断，最后综合

### 整体架构

三层设计：
- **Skill 层**：19 个明确入口，按场景选用（深度研究 / 财报分析 / 行业筛选 / 持仓管理 / 思维工具）
- **Agent 层**：团队型 skill（如 `/investment-team`、`/earnings-team`）并行调度多个独立视角子任务，各自搜索、判断、互相挑战，最后综合研判；轻量 skill 不经过这一层，直连工具快进快出
- **工具层**：`tools/financial_rigor.py` 等负责精确计算、交叉验证、报告抽检，保证每份报告的数据严谨性可验证

---

## Skills 一览（19个）

### 🔬 深度研究类

| Skill | 用途 | 适合场景 |
|-------|------|---------|
| [`/investment-research`](skills/investment-research.md) | 四大师综合深度分析 | 对一家上市公司进行全方位投资研究 |
| [`/investment-team`](skills/investment-team.md) | 多 Agent 并行投研团队 | 4 个角色并行研究，最快速、最全面 |
| [`/management-deep-dive`](skills/management-deep-dive.md) | 管理层纵深研究 | "买股票就是买人"——当管理层是核心变量时深挖 |
| [`/private-company-research`](skills/private-company-research.md) | 未上市公司深度研究 | 研究蚂蚁、SpaceX 等信息稀缺的未上市公司 |
| [`/deep-company-series`](skills/deep-company-series.md) | 8 篇长文系列拆一家公司 | 长文深度系列，从认知重置到决策闭环 |

### 📊 财报分析类

| Skill | 用途 | 适合场景 |
|-------|------|---------|
| [`/earnings-review`](skills/earnings-review.md) | 财报精读（一手资料） | 只读原始财报，不依赖二手研报，像巴菲特一样读年报 |
| [`/earnings-team`](skills/earnings-team.md) | 财报精读团队 + 文章发布 | 四大师并行解读财报 → 编辑润色 → 读者评审 → 可发布文章 |

### 🏭 行业筛选类

| Skill | 用途 | 适合场景 |
|-------|------|---------|
| [`/industry-research`](skills/industry-research.md) | 产业链全景扫描 | 研究一个行业的全部投资机会（按产业链环节切片） |
| [`/industry-funnel`](skills/industry-funnel.md) | 行业漏斗筛选 | 全市场 → 粗筛 ≤10 家 → 终选 3 家深度分析 |
| [`/quality-screen`](skills/quality-screen.md) | 去劣筛选（7条硬指标） | 快速排除非一流公司，支持个股/行业/指数/主题批量筛 |
| [`/bottleneck-hunter`](skills/bottleneck-hunter.md) | 供应链瓶颈猎手 | 从超级趋势出发，寻找产业链物理瓶颈和套利机会 |
| [`/investment-checklist`](skills/investment-checklist.md) | 巴菲特买入前 Checklist | 六关快速筛选，10分钟决定是否值得深入 |

### 📈 持仓管理类

| Skill | 用途 | 适合场景 |
|-------|------|---------|
| [`/portfolio-review`](skills/portfolio-review.md) | 组合管理与优化 | 从"研究公司"升级到"管理组合"——仓位、集中度、再平衡 |
| [`/thesis-tracker`](skills/thesis-tracker.md) | 投资论文追踪 | 买入后的纪律系统：持续跟踪论文是否被证伪 |
| [`/thesis-drift`](skills/thesis-drift.md) | 投资论文漂移检测 | 对比两份论文/报告，区分事实变化、估值变化与措辞变化 |
| [`/news-pulse`](skills/news-pulse.md) | 股价异动快速归因 | 股价大涨/大跌时10分钟搞清"发生了什么" |

### 🧠 思维工具类

| Skill | 用途 | 适合场景 |
|-------|------|---------|
| [`/dyp-ask`](skills/dyp-ask.md) | 段永平问答 | 以段永平的方式思考任何问题——商业、投资、人生 |
| [`/financial-data`](skills/financial-data.md) | 财务数据获取与交叉验证规范 | 确保关键数据来自2个独立来源，误差>1%告警 |
| [`/wechat-article`](skills/wechat-article.md) | 长文转公众号文章 | 作者、编辑、读者三个角色协作，把研究报告改写成可发布文章 |

---

## Setup Guide

### 1. 安装 AI 客户端

本仓库保留同一套 agent 中立的 canonical workflow，分别提供 Claude Code commands、Codex skills，以及可通过 skills.sh 生态（含 GitHub Copilot CLI 等通用 agent）直接安装的通用 skill 包。按你使用的客户端安装即可。

**Claude Code：**

```bash
npm install -g @anthropic-ai/claude-code
```

**Codex：**

```bash
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# 或使用 npm
npm install -g @openai/codex

# 或使用 Homebrew
brew install --cask codex

# 验证安装
codex --version
```

Windows 用户可使用官方 PowerShell 安装命令：`powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`。

**GitHub Copilot CLI：** 无需单独安装客户端，见下方"安装 Skills"。

### 2. 安装 Skills

**Claude Code**（macOS / Linux）：

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-claude-commands.sh
```

Windows（PowerShell / Command Prompt）：

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-claude-commands.bat
```

这些 skills 会频繁调用工具，Claude Code 默认会逐次请求授权确认（客户端权限机制，本仓库无法修改）。如果你信任当前 workflow 且在可信环境中运行，可以跳过权限确认：

```bash
claude --dangerously-skip-permissions
```

注意：该模式会关闭 Claude Code 的工具审批保护，只应在你信任仓库、命令和工作目录的情况下使用。

**Codex**（macOS / Linux）：

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-codex-skills.sh

# 可选：安装 Codex slash prompts，获得接近 Claude Code 的 /investment-research 体验
./scripts/install-codex-prompts.sh
```

Windows（PowerShell / Command Prompt）：

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-codex-skills.bat

REM 可选：安装 Codex slash prompts
.\scripts\install-codex-prompts.bat
```

**GitHub Copilot CLI：**

`codex-skills/*/SKILL.md` 虽然目录名叫 codex-skills，实际是 [skills.sh](https://skills.sh) 标准的通用 skill 包格式，`npx skills` 会把它安装给包括 Copilot CLI 在内的所有兼容 agent：

```bash
npx skills add sinmentis/ai-berkshire@investment-team -g -y
# 按需替换成其他 skill 名（见上方 Skills 一览），或用 npx skills find 浏览
```

已知限制：当前会话内新装的 skill 不会立即被 Copilot CLI 识别，需要开一个新会话。也可以直接 `git clone` 本仓库，cd 进目录后让 Copilot CLI 直接读取 `skills/*.md` 并调用仓库自带的 `tools/*.py`——效果上比单独装的 skill 包更完整。详见 [`COPILOT.md`](COPILOT.md)。

**三套入口的关系**：`skills/*.md` 是 agent 中立的 canonical workflow 源文件，Claude Code / Codex / Copilot CLI 均可直接使用；`codex-skills/*/SKILL.md` 是从 `skills/*.md` 生成的通用 skill 包（`scripts/sync-codex-skills.py` 生成）；`codex-prompts/*.md` 是可选的 Codex slash prompt 兼容层。

### 3. 成本与模型选择

深度投研类 Skill 默认会进行多轮研究、交叉验证和多角色综合判断，因此 token 消耗较高，这是为了换取更完整的商业、财务、行业和风险分析。

涉及护城河、估值、管理层和风险交叉判断时，分析质量会更依赖模型能力，不建议为节省成本牺牲关键判断质量。想控制成本时，优先调整 workflow：快速排除公司先用 [`/quality-screen`](skills/quality-screen.md)，股价异动归因用 [`/news-pulse`](skills/news-pulse.md)，只有结果值得深入时再运行 [`/investment-research`](skills/investment-research.md) 或 [`/investment-team`](skills/investment-team.md)。

---

## 使用

在 Claude Code 中直接调用：

```bash
# 深度研究
/investment-research 腾讯
/investment-team 美团
/management-deep-dive 王兴 美团
/private-company-research SpaceX
/deep-company-series 拼多多

# 财报分析
/earnings-review 腾讯 2025Q4
/earnings-team PDD 2025年报

# 行业筛选
/industry-research 核电
/industry-funnel AI算力
/quality-screen 恒生指数成分股
/bottleneck-hunter AI基础设施
/investment-checklist 茅台, 英伟达, 苹果

# 持仓管理
/portfolio-review 腾讯30%, 美团20%, 茅台20%, 现金30%
/thesis-tracker 拼多多
/thesis-drift 拼多多 reports/拼多多-thesis-2025Q4.md reports/拼多多-thesis-2026Q1.md
/news-pulse 腾讯

# 思维工具
/dyp-ask 拼多多的护城河到底在哪里？
/wechat-article 美团
```

在 Codex 中安装后重启 Codex，然后直接按 skill 名称描述任务：

```text
使用 investment-research 研究腾讯
使用 earnings-review 分析 PDD 2025年报
使用 industry-funnel 筛选 AI算力
使用 bottleneck-hunter 扫描 AI基础设施瓶颈
使用 thesis-drift 对比拼多多两份投资论文
使用 wechat-article 写美团投研文章
```

如果安装了 Codex slash prompts，重启后可在 `/` 菜单搜索，入口通常显示为 `prompts:<name>`：

```text
/prompts:investment-research 腾讯
```

在 GitHub Copilot CLI 中，安装对应 skill 后直接用自然语言描述任务，或按 skill 名称调用，具体约定见 [`COPILOT.md`](COPILOT.md)。

---

## 实战研究报告

使用本框架生成的研究报告示例：

| 公司 | 使用 Skill | 核心结论 | 报告链接 |
|------|-----------|---------|---------|
| 拼多多 (PDD) | `/investment-team` | 综合3.4/5，极度便宜但10年确定性不足，适合中等仓位 | [查看报告](reports/拼多多/) |
| 腾讯控股 (0700.HK) | `/investment-research` | 社交垄断+资本配置卓越，14x前瞻PE合理偏低 | [查看报告](reports/腾讯/) |
| 7家公司对比 | `/investment-checklist` | 茅台、腾讯通过；英伟达、美团、快手有条件通过；拼多多、泡泡玛特灰色 | [查看报告](reports/多公司对比-checklist-20260408.md) |
| 大师持仓追踪 | 自定义研究 | 巴菲特/李录/段永平最新13F持仓+PDD成本分析 | [查看报告](reports/大师持仓追踪-research-20260408.md) |

欢迎 PR 提交你用本框架生成的研究报告。

---

## 未来方向

- [ ] 历史回测：AI研报 vs 实际股价表现
- [ ] 宏观经济周期分析框架
- [ ] 基于MCP的实时数据接入（Wind/Bloomberg/Yahoo Finance）

---

## 免责声明

本项目仅供学习和研究目的，不构成任何投资建议。投资有风险，决策需谨慎。请始终做好自己的尽职调查（DYOR）。

## License

MIT License
