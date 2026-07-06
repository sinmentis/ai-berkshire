# AI Berkshire — GitHub Copilot CLI Guide

This repository's `skills/*.md` files are agent-neutral: they describe
research workflows in terms of actions (search the web, launch a background
sub-task, read/write a file) rather than any single coding agent's specific
tool names. This file maps those actions to GitHub Copilot CLI's actual
tools, alongside `CLAUDE.md` (Claude Code) and `AGENTS.md` (Codex).

## Tool mapping

| Skill text says | Use in Copilot CLI |
|---|---|
| "启动后台子任务" / "并行启动 N 个子任务" (launch a background sub-task / launch N in parallel) | The `task` tool. For true parallelism, issue all N calls in the **same response** — one tool call per response runs sequentially. |
| "联网搜索" (search the web) | The `web_search` tool for a synthesized answer, or `web_fetch` to read a specific URL. |
| "通过命令行调用工具" / `python3 tools/...` (run a command-line tool) | The `bash` tool. |
| "读取/写入文件" (read/write a file) | The `view` tool to read, `edit` to modify, `create` for new files. |

No specific model name is pinned here — model selection for sub-tasks is
left to the user's or agent's own configuration, not hardcoded into this
shared repo file.

## Date discipline

Before starting any research, run `date` via the `bash` tool to confirm
today's actual date. Treat that date as the baseline for "latest" data
(prices, market cap, most recent filings) and state the data-cutoff date in
the report header. Never assume the current date from training data — this
is the single most common failure mode observed when running these skills:
training data goes stale and a model will confidently cite outdated market
levels, rates, or index values unless it is explicitly forced to verify
against a live search first.

## Checkout path discipline

Resolve the actual local repository checkout path before calling
`tools/*.py` — do not assume a fixed home-directory path like `~/ai-berkshire/`.
If a session starts outside the repo, locate the real checkout first
(e.g. `find ~ -maxdepth 3 -iname ai-berkshire -type d`).

## Everything else

The "Research Quality Rules" and "Editing Rules" in `AGENTS.md` are already
platform-agnostic and apply as-is to Copilot CLI sessions.
