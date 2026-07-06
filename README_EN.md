English | [中文](README.md) | [日本語](README_JA.md)

> 🍴 **This is a fork**: forked from [xbtlin/ai-berkshire](https://github.com/xbtlin/ai-berkshire). Changes in this fork: rewrote all `skills/*.md` into agent-neutral action language (no more hardcoded Claude-Code-only tool calls like `TeamCreate`/`SendMessage`/`TeamDelete`), so the skills run correctly under **GitHub Copilot CLI** too, while keeping full Claude Code / Codex compatibility. Added `COPILOT.md` documenting the Copilot CLI tool mapping. See the design doc under [`docs/superpowers/specs/`](docs/superpowers/specs/) and the implementation plan under [`docs/superpowers/plans/`](docs/superpowers/plans/) for details.

# AI Berkshire — Value Investing Research Framework for the AI Era

> "Price is what you pay, value is what you get." — Warren Buffett

**AI Berkshire** is a collection of investment research skills compatible with Claude Code, Codex, and GitHub Copilot CLI. It systematizes the methodologies of four value investing masters — Buffett, Munger, Duan Yongping, and Li Lu — and delivers professional-grade research through AI Agents.

[Design Philosophy](#design-philosophy) · [Skills Overview](#skills-overview-19-skills) · [Setup Guide](#setup-guide) · [Usage](#usage) · [Reports](#live-research-reports)

---

## Design Philosophy

Ask an AI directly "is this stock worth buying?" and you'll usually get an "on one hand... on the other hand..." balanced analysis that isn't actionable. AI Berkshire is about analysis quality and decision discipline:

- **Four masters in adversarial dialogue, not a single analysis**: Duan Yongping (business essence), Buffett (moat and valuation), Munger (inverse thinking), and Li Lu (long-term certainty) each research independently and challenge each other, rather than being stitched into one bland, balanced report
- **Anti-bias mechanisms**: information-richness grading (A/B/C), inverse failure-scenario planning, a hard veto checklist, and contrarian checks, to prevent the "more data = more certainty" illusion
- **Financial data precision**: all calculations go through [`tools/financial_rigor.py`](tools/financial_rigor.py) (Python `decimal.Decimal`, not `float`), never LLM mental math; key data points are cross-verified from at least 2 independent sources
- **Reproducibility**: the same input produces a consistently structured, consistently deep report, so you can compare across companies and across time
- **Parallel multi-agent research**: team-style skills like `/investment-team` launch multiple independent sub-tasks in parallel rather than splitting one prompt into pieces — each role researches and judges independently, then synthesizes

### Architecture

Three layers:
- **Skill layer**: 19 clear entry points, pick the one for your scenario (deep research / earnings analysis / industry screening / portfolio management / thinking tools)
- **Agent layer**: team-style skills (e.g. `/investment-team`, `/earnings-team`) dispatch multiple independent-perspective sub-tasks in parallel — each searches, judges, and challenges the others before a final synthesis. Lightweight skills skip this layer and go straight to the tools
- **Tool layer**: `tools/financial_rigor.py` and friends handle exact calculation, cross-validation, and report auditing, so every report's data rigor is verifiable

---

## Skills Overview (19 skills)

### 🔬 Deep Research

| Skill | Purpose | Use When |
|-------|------|---------|
| [`/investment-research`](skills/investment-research.md) | Four-masters comprehensive analysis | Full investment research on a public company |
| [`/investment-team`](skills/investment-team.md) | Parallel multi-agent research team | 4 roles research in parallel — fastest, most thorough |
| [`/management-deep-dive`](skills/management-deep-dive.md) | Management deep dive | "Buying a stock is buying people" — when management is the key variable |
| [`/private-company-research`](skills/private-company-research.md) | Private-company deep research | Researching information-scarce private companies like Ant Group, SpaceX |
| [`/deep-company-series`](skills/deep-company-series.md) | 8-part long-form series on one company | Deep long-form series, from reframing to a decision |

### 📊 Earnings Analysis

| Skill | Purpose | Use When |
|-------|------|---------|
| [`/earnings-review`](skills/earnings-review.md) | Earnings deep-read (primary sources) | Read the raw filing, not secondary research — read the 10-K like Buffett does |
| [`/earnings-team`](skills/earnings-team.md) | Earnings-reading team + article publishing | Four masters read the earnings in parallel → editor polish → reader review → publishable article |

### 🏭 Industry Screening

| Skill | Purpose | Use When |
|-------|------|---------|
| [`/industry-research`](skills/industry-research.md) | Full industry-chain scan | Research all the investable opportunities in an industry, sliced by chain segment |
| [`/industry-funnel`](skills/industry-funnel.md) | Industry funnel screening | Whole market → shortlist ≤10 → 3 finalists with deep analysis |
| [`/quality-screen`](skills/quality-screen.md) | Quality screen (7 hard filters) | Quickly exclude non-first-tier companies; works on stocks/industries/indices/themes in batch |
| [`/bottleneck-hunter`](skills/bottleneck-hunter.md) | Supply-chain bottleneck hunter | Start from a megatrend, find physical bottlenecks and arbitrage in the supply chain |
| [`/investment-checklist`](skills/investment-checklist.md) | Buffett pre-buy checklist | Six gates, decide in 10 minutes whether it's worth going deeper |

### 📈 Portfolio Management

| Skill | Purpose | Use When |
|-------|------|---------|
| [`/portfolio-review`](skills/portfolio-review.md) | Portfolio management and optimization | Moving from "researching a company" to "managing a portfolio" — sizing, concentration, rebalancing |
| [`/thesis-tracker`](skills/thesis-tracker.md) | Investment thesis tracking | The discipline system after you buy: keep checking whether the thesis is being falsified |
| [`/thesis-drift`](skills/thesis-drift.md) | Thesis drift detection | Compare two theses/reports, separating fact changes from valuation changes from wording changes |
| [`/news-pulse`](skills/news-pulse.md) | Fast price-move attribution | When a stock moves sharply, figure out "what happened" in 10 minutes |

### 🧠 Thinking Tools

| Skill | Purpose | Use When |
|-------|------|---------|
| [`/dyp-ask`](skills/dyp-ask.md) | Ask Duan Yongping | Think through anything — business, investing, life — the way Duan Yongping would |
| [`/financial-data`](skills/financial-data.md) | Financial-data sourcing and cross-validation spec | Ensures key data comes from 2 independent sources, flags >1% discrepancies |
| [`/wechat-article`](skills/wechat-article.md) | Long-form-to-article rewrite | Author, editor, and reader roles collaborate to turn a research report into a publishable article |

---

## Setup Guide

### 1. Install an AI Client

This repository keeps one canonical, agent-neutral workflow, and provides Claude Code commands, Codex skills, and — via the skills.sh ecosystem — an installable package for any compatible universal agent, including GitHub Copilot CLI. Install whichever client you plan to use.

**Claude Code:**

```bash
npm install -g @anthropic-ai/claude-code
```

**Codex:**

```bash
# macOS / Linux
curl -fsSL https://chatgpt.com/codex/install.sh | sh

# or via npm
npm install -g @openai/codex

# or via Homebrew
brew install --cask codex

# verify
codex --version
```

Windows users can use the official PowerShell installer: `powershell -ExecutionPolicy ByPass -c "irm https://chatgpt.com/codex/install.ps1 | iex"`.

**GitHub Copilot CLI:** no separate client install needed — see "Install Skills" below.

### 2. Install Skills

**Claude Code** (macOS / Linux):

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-claude-commands.sh
```

Windows (PowerShell / Command Prompt):

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-claude-commands.bat
```

These skills call tools frequently; Claude Code prompts for approval on each call by default (a client permission mechanism this repo can't change). If you trust the workflow and are running in a trusted environment, you can skip approval prompts:

```bash
claude --dangerously-skip-permissions
```

Note: this disables Claude Code's tool-approval protection — only use it when you trust the repo, the commands, and the working directory.

**Codex** (macOS / Linux):

```bash
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
./scripts/install-codex-skills.sh

# Optional: install Codex slash prompts for a Claude-Code-like /investment-research experience
./scripts/install-codex-prompts.sh
```

Windows (PowerShell / Command Prompt):

```bat
git clone https://github.com/xbtlin/ai-berkshire.git
cd ai-berkshire
.\scripts\install-codex-skills.bat

REM Optional: install Codex slash prompts
.\scripts\install-codex-prompts.bat
```

**GitHub Copilot CLI:**

`codex-skills/*/SKILL.md` is, despite the directory name, the standard [skills.sh](https://skills.sh) universal skill package format — `npx skills` installs it for any compatible agent, including Copilot CLI:

```bash
npx skills add sinmentis/ai-berkshire@investment-team -g -y
# swap in another skill name (see Skills Overview above), or browse with npx skills find
```

Known limitation: a skill installed mid-session isn't picked up by the current Copilot CLI session — start a new one. You can also `git clone` this repo directly and let Copilot CLI read `skills/*.md` and call the repo's own `tools/*.py` — more complete than the standalone package. See [`COPILOT.md`](COPILOT.md) for details.

**How the three entry points relate**: `skills/*.md` is the agent-neutral canonical workflow source, usable directly by Claude Code, Codex, and Copilot CLI; `codex-skills/*/SKILL.md` are universal skill packages generated from `skills/*.md` by `scripts/sync-codex-skills.py`; `codex-prompts/*.md` are an optional Codex slash-prompt compatibility layer.

### 3. Cost and Model Choice

Deep-research skills run multiple rounds of research, cross-validation, and multi-role synthesis by default, so token usage is higher — in exchange for more complete business, financial, industry, and risk analysis.

When the analysis involves moat, valuation, management, or risk judgment, quality depends more on model capability — don't sacrifice judgment quality just to save on model cost. To control cost, adjust the workflow first: use [`/quality-screen`](skills/quality-screen.md) to quickly exclude companies, [`/news-pulse`](skills/news-pulse.md) for price-move attribution, and only run [`/investment-research`](skills/investment-research.md) or [`/investment-team`](skills/investment-team.md) once a candidate is worth the deeper dive.

---

## Usage

Call directly in Claude Code:

```bash
# Deep research
/investment-research Tencent
/investment-team Meituan
/management-deep-dive "Wang Xing" Meituan
/private-company-research SpaceX
/deep-company-series PDD

# Earnings analysis
/earnings-review Tencent 2025Q4
/earnings-team PDD 2025-annual

# Industry screening
/industry-research "nuclear power"
/industry-funnel "AI compute"
/quality-screen "Hang Seng Index constituents"
/bottleneck-hunter "AI infrastructure"
/investment-checklist Moutai, NVIDIA, Apple

# Portfolio management
/portfolio-review Tencent 30%, Meituan 20%, Moutai 20%, cash 30%
/thesis-tracker PDD
/thesis-drift PDD reports/pdd-thesis-2025Q4.md reports/pdd-thesis-2026Q1.md
/news-pulse Tencent

# Thinking tools
/dyp-ask "Where exactly is PDD's moat?"
/wechat-article Meituan
```

In Codex, install then restart Codex, then describe the task by skill name:

```text
Use investment-research to research Tencent
Use earnings-review to analyze PDD's 2025 annual report
Use industry-funnel to screen AI compute
Use bottleneck-hunter to scan AI infrastructure bottlenecks
Use thesis-drift to compare two PDD investment theses
Use wechat-article to write an investment article about Meituan
```

If you installed Codex slash prompts, restart Codex and search the `/` menu — entries usually appear as `prompts:<name>`:

```text
/prompts:investment-research Tencent
```

In GitHub Copilot CLI, after installing a skill, describe the task in natural language or invoke it by skill name — see [`COPILOT.md`](COPILOT.md) for the exact conventions.

---

## Live Research Reports

Example reports produced with this framework:

| Company | Skill Used | Core Conclusion | Report |
|------|-----------|---------|---------|
| PDD (Pinduoduo) | `/investment-team` | 3.4/5 overall — extremely cheap but 10-year certainty is lacking; moderate position size | [View](reports/拼多多/) |
| Tencent (0700.HK) | `/investment-research` | Social monopoly + excellent capital allocation; 14x forward P/E is reasonably low | [View](reports/腾讯/) |
| 7-company comparison | `/investment-checklist` | Moutai and Tencent pass; NVIDIA, Meituan, Kuaishou pass conditionally; PDD and Pop Mart are gray-zone | [View](reports/多公司对比-checklist-20260408.md) |
| Master investors' holdings tracker | Custom research | Buffett/Li Lu/Duan Yongping's latest 13F holdings + PDD cost-basis analysis | [View](reports/大师持仓追踪-research-20260408.md) |

PRs adding reports you've generated with this framework are welcome.

---

## Roadmap

- [ ] Historical backtesting: AI research vs. actual stock performance
- [ ] Macro economic cycle analysis framework
- [ ] MCP-based real-time data access (Wind/Bloomberg/Yahoo Finance)

---

## Disclaimer

This project is for learning and research purposes only and does not constitute investment advice. Investing involves risk; always do your own due diligence (DYOR).

## License

MIT License
