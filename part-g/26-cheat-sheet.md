---
title: "26. Cheat Sheet"
parent: "Part G — Exam Readiness"
nav_order: 26
---

# Chapter 26 — Cheat Sheet
{: .no_toc }

Every must-know fact, by domain. Read it the day before the exam.
{: .fs-5 .fw-300 }

---

## D1 · Agentic Architecture (~27%)

| Topic | Remember |
|:--|:--|
| Agent | Tools in a **loop**. The model proposes, the **app executes and decides** |
| Loop | Continue while `stop_reason == "tool_use"`. Stop on `end_turn`. `max_tokens` = truncated |
| Tool round | Append the assistant content, then **one** user message with all `tool_result`s (`tool_use_id`) |
| Anti-patterns | Parsing text for "done", using an iteration cap as the main stop, checking for text content |
| Workflows | Chaining · routing · parallelization · orchestrator–workers · evaluator–optimizer |
| Decomposition | Sequential (with gates) · parallel fan-out/fan-in · hierarchical · fixed vs adaptive |
| Big review | Per-file passes + **cross-file** pass |
| Multi-agent | Hub-and-spoke. Subagents have **isolated context**. `Task` tool, so `"Task"` goes in `allowed_tools` |
| Parallel subagents | Several Task calls **in one response** |
| Subagent prompt | Goal, scope, output schema, quality bar. Not a step-by-step script |
| Subagent failure | Structured error + partial results → the coordinator decides |
| Enforcement | Gates and hooks for must-never rules. The prompt is for judgment |
| Hooks | `PreToolUse` blocks or changes. `PostToolUse` normalizes or trims |
| Escalate | Explicit request for a human · policy gap · beyond authority · no progress |
| Don't escalate | On sentiment alone · on self-reported low confidence |
| Handoff | Structured: ID, issue, actions taken, reason, recommendation |

## D2 · Tool Design & MCP (~18%)

| Topic | Remember |
|:--|:--|
| Tool | `name` + `description` (**the main routing signal**) + `input_schema` |
| Description | What · when · when not · input format with an example · returns |
| Fix the wrong tool | Descriptions → rename or split → (only then) other measures |
| Schema | required, `enum` + "other", nullable, `additionalProperties: false`, `strict: true` |
| `tool_choice` | `auto` · `any` (must use one) · `tool` (forced) · `none` |
| Tool count | About 4–5 per agent. Large catalog → tool search (`defer_loading`) |
| Errors | `is_error` / `isError` + code, category, retryable, message, suggested action |
| Empty ≠ error | `[]` is a success. A timeout is an error |
| MCP roles | Host → client (1:1) → server. JSON-RPC 2.0. `initialize` first |
| Primitives | Tools = model · resources = app · prompts = user |
| Client features | Sampling (LLM call) · roots (boundaries) · elicitation (ask the user) |
| Transports | stdio (local) · Streamable HTTP (remote, OAuth) |
| Claude Code MCP | `.mcp.json` = project/team · local (default) · user. `${ENV_VAR}` for secrets |
| Build or reuse | Community server for standard tools, custom for team-specific ones |

## D3 · Claude Code (~20%)

| Topic | Remember |
|:--|:--|
| Grep / Glob | Contents / names |
| Edit fails (not unique) | Read + Write |
| Plan mode | Large, multi-file, unfamiliar, several approaches |
| Permission modes | default · acceptEdits · plan · bypassPermissions (sandbox only) |
| Permission rules | allow / ask / deny. **Deny wins** |
| CLAUDE.md levels | Managed · user `~/.claude/` (personal) · project (team) · `CLAUDE.local.md` |
| Modular | `@path` imports · `.claude/rules/*.md` · `paths:` globs |
| Debug memory | `/memory` |
| Locations | `.claude/{commands,skills,agents}` (team) · `~/.claude/...` (personal) |
| Skill frontmatter | `description` · `allowed-tools` · `context: fork` · `argument-hint` · `disable-model-invocation` |
| Hook exit codes | 0 = OK · **2 = block** (stderr → Claude) |
| Sessions | `--continue` · `--resume` · `--fork-session` · `/compact` · `/clear` |
| Stale context | Fresh session + structured summary |
| Refinement | Examples · tests first · interview · batch related issues |
| CI | `claude -p` · `--output-format json` · `--json-schema` · `--allowedTools` · `--max-turns` |
| Review | Independent instance · multi-pass · prior findings → new issues only |

## D4 · Prompts & Structured Output (~20%)

| Topic | Remember |
|:--|:--|
| False positives | Explicit flag / don't-flag **categories**. Not "be conservative" |
| Severity | A rubric with a code example for each level |
| Noisy category | Turn it off temporarily, then refine it |
| Few-shot | 2–4, varied, edge cases, show the reasoning, include "don't flag" |
| Structure | XML tags for instructions / examples / data |
| Evaluation | A test set with expected verdicts, scored in code, **per category** |
| JSON guarantee | Forced tool or structured outputs |
| Schema ≠ meaning | Semantic checks in code (totals, cross-field) |
| Missing value | Nullable, never forced |
| Retry | Doc + failed output + exact error. Useless if the information is absent |
| Error-exposing fields | `calculated_total` vs `stated_total`, `conflict_detected` |
| Confidence | Per field, **calibrated** on labelled data. Sample the high-confidence results |
| Batch API | About 50% cheaper, ≤ 24h, `custom_id`, no tool loop, resubmit the failures only |

## D5 · Context & Reliability (~15%)

| Topic | Remember |
|:--|:--|
| Long conversations | **Case facts** block (IDs, amounts, dates, promises), held in code |
| Lost in the middle | Key information at the start or end, with headers |
| Tool bloat | Trim to the relevant fields |
| Between agents | Structured essentials, not transcripts |
| Long sessions | Scratchpad file · `/compact` · subagents · manifests |
| Caching | Stable prefix first, then `cache_control`. Saves cost and latency only |
| Provenance | Claim + source + excerpt + date, from the subagents onwards |
| Conflicts | Report both with attribution. Established vs contested |
