# Design: Agent-neutral skill sources for GitHub Copilot CLI compatibility

Date: 2026-07-06
Status: Draft, pending user review

## Problem

`skills/*.md` is documented as "Claude Code slash-command source" and is written
against Claude Code's specific tool surface: `Task`, `Agent`, `WebSearch`,
`Bash`, `Read`, `Write`, and — for four skills — a persistent multi-agent
"Team" API (`TeamCreate`, `TaskCreate`, `TaskUpdate`, `SendMessage`,
`shutdown_request`, `TeamDelete`).

`scripts/sync-codex-skills.py` generates `codex-skills/*/SKILL.md` from this
source by prepending a generic "Codex adapter note" that asks the executing
agent to mentally translate Claude-only tool names to its own equivalent. The
body text is otherwise copied verbatim.

This repo's `codex-skills/*/SKILL.md` packages are what `npx skills add`
actually installs for any "universal" compatible agent, including GitHub
Copilot CLI (confirmed: the installer labels these packages
`universal: GitHub Copilot, Amp, Antigravity, ...`, symlinked to Claude Code).
But Copilot CLI has no `TeamCreate`/`SendMessage`/`TeamDelete` primitives —
only a single-shot `task` tool that dispatches one sub-agent per call (parallel
when multiple calls are issued in the same turn, with no persistent identity or
inter-agent messaging afterward). Reading the current skill text, a Copilot CLI
session has to invent an entire alternative execution model on the fly, with
no guidance on how "SendMessage to team-lead" or "TeamDelete" should map to
anything it can actually do.

Reference precedent: `mattpocock/skills` and this CLI's own installed
`superpowers` skills achieve true cross-agent portability by writing the
canonical skill text in agent-neutral action language from the start (e.g.
"spin up a background agent... write the findings to a file") rather than
naming specific tool APIs, then leaving per-platform tool-name mapping to a
small satellite reference document. That is a stronger pattern than
"generic adapter note bolted onto Claude-specific text," and is what this
design adopts.

## Goals

- Rewrite `skills/*.md` (all 20 files) so the procedural/orchestration text
  uses agent-neutral action language instead of Claude Code tool names.
- Preserve all domain content unchanged: the four-masters methodology,
  checklists, scoring rubrics, report structure/sections, and all
  `tools/*.py` invocation examples (these are plain shell commands already,
  not agent-specific).
- Regenerate `codex-skills/*/SKILL.md` from the rewritten source so Claude
  Code and Codex users see no loss of capability.
- Add `COPILOT.md` (sibling to `CLAUDE.md` / `AGENTS.md`) documenting
  Copilot CLI-specific tool-name mapping and operating conventions.
- Update `AGENTS.md` to note that `skills/*.md` is now an agent-neutral
  canonical source (not Claude-Code-only) and to reference `COPILOT.md`.
- Validate the result by actually running one rewritten "team" skill
  (`investment-team`) end to end in this Copilot CLI session against a real
  company, in addition to structural checks.

## Non-goals

- No new `copilot-skills/` directory and no new sync script. One canonical
  source, one generated Codex/universal package tree, as today.
- No change to `reports/`, existing report files, or report/file naming
  conventions in `CLAUDE.md`.
- No attempt to preserve upstream-PR-ready commit hygiene beyond normal good
  practice — this work targets the user's own fork
  (`sinmentis/ai-berkshire`, branch `copilot-cli-support`) only. Upstreaming
  to `xbtlin/ai-berkshire` is explicitly out of scope for now.
- No change to `tools/*.py` themselves.

## Transformation rules

Applied uniformly across all 20 `skills/*.md` files. Anything not listed
stays as-is.

| Claude-Code-specific text | Replace with |
|---|---|
| `TeamCreate` + `TaskCreate` (N tasks) + parallel `Task` launch + `SendMessage` reports + `shutdown_request` + `TeamDelete` | "Assemble a virtual team of N roles. Launch all N as independent background sub-tasks in parallel (a single dispatch, not a persistent conversation). Each sub-task researches independently and returns its findings directly as its result — there is no separate messaging step and nothing to explicitly tear down afterward. Once all N results are back, synthesize as the coordinating role." |
| "使用 Agent 工具启动后台 Agent" / "使用 Task 工具启动后台 Agent" | "启动一个/多个后台子任务（sub-task）" — i.e., "launch one or more background sub-tasks" |
| `WebSearch` | "search the web" / 联网搜索 |
| `Bash` (as a named tool) | "run a shell/command-line command" / 通过命令行运行 |
| `Read` / `Write` (as named tools) | "read the file" / "write the file" |
| Fixed home-directory paths (e.g. `~/ai-berkshire/...`) when instructing tool calls | "locate the actual repository checkout path first" (already present in the Codex adapter note; carry the same caution into the rewritten body where it's instructive, not just in the generated wrapper) |

Explicitly unaffected: `python3 tools/financial_rigor.py ...` and other
`tools/*.py` invocations (already agent-neutral shell commands), all
Chinese-language domain/methodology content, report section structures,
scoring tables, and file-naming conventions from `CLAUDE.md`.

## File tiers

| Tier | Files | Why |
|---|---|---|
| Heavy rewrite | `investment-team.md`, `news-pulse.md`, `private-company-research.md` | Use the full persistent Team + SendMessage + TeamDelete ceremony; needs real procedural rewrite, not just word substitution |
| Medium rewrite | `earnings-team.md` | Uses single-shot "Agent 工具" background launch (no persistent messaging) — structure already close to portable, mostly terminology swap |
| Light rewrite | Remaining 16 files | Scattered `Bash`/`WebSearch`/`Task 工具`/`Read`/`Write` mentions; mechanical terminology swap, no structural change |

## New file: `COPILOT.md`

Sibling to `CLAUDE.md` / `AGENTS.md`, covering:

- Project layout pointer (same as `AGENTS.md`).
- Tool-name mapping table for GitHub Copilot CLI: parallel background
  research → `task` tool (issue multiple calls in one turn for true
  parallelism); web search → `web_search` / `web_fetch`; shell/python
  commands → `bash`; file read/write → `view` / `edit` / `create`.
  (No specific model name is pinned here — model choice is left to the
  user's/agent's own configuration, not hardcoded into a shared repo file.)
- Date discipline: before starting any research, run `date` via the `bash`
  tool to confirm today's date; treat it as the baseline for "latest," and
  never assume the current date from training data (same rule already in
  `AGENTS.md`, restated for Copilot CLI explicitly since it's the exact
  failure mode observed in practice).
- Checkout-path discipline: resolve the actual local checkout path before
  calling `tools/*.py`; do not assume a fixed home-directory path.
- Cross-reference to `AGENTS.md`'s "Research Quality Rules" and "Editing
  Rules" sections, which remain platform-agnostic and apply as-is.

## `AGENTS.md` update

Small addition, not a rewrite:
- Note that `skills/*.md` is the agent-neutral canonical source (used to
  generate Codex packages and consumed directly by any tool-capable coding
  agent, including GitHub Copilot CLI), not Claude-Code-specific.
- Add `COPILOT.md` to the list of per-platform behavior docs alongside
  `CLAUDE.md` and this file.

## Validation plan

1. Structural: `grep` across all rewritten `skills/*.md` for
   `TeamCreate|TaskCreate|TaskUpdate|SendMessage|shutdown_request|TeamDelete|WebSearch|Task 工具|Agent 工具` —
   expect zero matches.
2. Regeneration consistency: `python3 scripts/sync-codex-skills.py` then
   `python3 scripts/sync-codex-skills.py --check` — expect a clean check
   with no stale files.
3. Live test: in this Copilot CLI session, follow the rewritten
   `skills/investment-team.md` against one real, well-known public company,
   using this CLI's own `task` tool for the parallel role dispatch. Confirm
   the four roles actually run in parallel from the rewritten instructions
   alone (no manual reinterpretation beyond what the new text says) and that
   a coherent final report comes out the other end.

## Open questions / assumptions

- Assumes `tools/financial_rigor.py` and `tools/report_audit.py` run
  correctly under whatever Python is available in the working environment;
  not modifying these scripts.
- Assumes report file conventions in `CLAUDE.md` remain valid for output
  produced via Copilot CLI (no changes proposed there).
