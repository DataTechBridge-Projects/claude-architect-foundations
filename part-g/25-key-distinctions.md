---
title: "25. Key Distinctions & Traps"
parent: "Part G — Exam Readiness"
nav_order: 25
---

# Chapter 25 — Key Distinctions & Traps
{: .no_toc }

Look-alike concepts the exam uses to build its tempting wrong answers.
{: .fs-5 .fw-300 }

<details open markdown="block">
<summary>Contents</summary>
{: .text-delta }
1. TOC
{:toc}
</details>

---

## Agents, Tools & Control

| A | vs | B | The difference |
|:--|:--:|:--|:--|
| Workflow | ↔ | Agent | Code controls the path ↔ the model controls the path |
| Prompt instruction | ↔ | Gate / hook | Probabilistic ↔ **deterministic** |
| `end_turn` | ↔ | `max_tokens` | Finished ↔ **truncated** |
| Iteration cap | ↔ | `stop_reason` | Safety net ↔ **the real stop signal** |
| Coordinator | ↔ | Subagent | Plans and synthesizes ↔ does one scoped task in a **fresh context** |
| Fork | ↔ | Resume | Branch to try something different ↔ continue the same line |
| Escalation criteria | ↔ | Sentiment / confidence | Reliable ↔ **unreliable** triggers |
| Transient error | ↔ | Business-rule error | Retry with backoff ↔ **don't retry**, explain or escalate |
| Empty result | ↔ | Failed lookup | Valid "none found" ↔ **`is_error`** |

## Tools & MCP

| A | vs | B | The difference |
|:--|:--:|:--|:--|
| `tool_choice: auto` | ↔ | `any` | May reply in text ↔ must call **some** tool |
| `any` | ↔ | forced `tool` | Claude picks the tool ↔ **you** pick it |
| Tool | ↔ | Resource | Model-controlled action ↔ **app-controlled read-only data** |
| Resource | ↔ | Prompt (MCP) | App-controlled data ↔ **user-selected template** |
| stdio | ↔ | Streamable HTTP | Local subprocess, one client ↔ remote, many clients, OAuth |
| Sampling | ↔ | Roots | The server asks for an LLM call ↔ the client sets file boundaries |
| Community server | ↔ | Custom server | Standard integrations ↔ team-specific systems |
| Tool annotations | ↔ | Server enforcement | Hints ↔ **real security** |

## Claude Code

| A | vs | B | The difference |
|:--|:--:|:--|:--|
| Grep | ↔ | Glob | File **contents** ↔ file **names / paths** |
| Edit | ↔ | Write | Unique-match change ↔ whole file (the fallback) |
| Plan mode | ↔ | Direct execution | Large, unclear, many files ↔ small, clear |
| User `~/.claude/CLAUDE.md` | ↔ | Project `CLAUDE.md` | Only you ↔ **the whole team** |
| Subdirectory CLAUDE.md | ↔ | `.claude/rules` + `paths:` | One folder ↔ files **spread across** folders |
| CLAUDE.md | ↔ | Skill | Always loaded ↔ **on demand** |
| Skill | ↔ | Hook | Guidance ↔ **guarantee** |
| Skill | ↔ | Subagent | A workflow in the main context (or `fork`) ↔ a separate worker with its own tools |
| `.claude/…` | ↔ | `~/.claude/…` | Shared through git ↔ personal |
| Author-session review | ↔ | Independent review | Shared blind spots ↔ **fresh eyes** |

## Prompts, Output & Context

| A | vs | B | The difference |
|:--|:--:|:--|:--|
| "Be conservative" | ↔ | Explicit criteria | Vague ↔ **categorical flag / don't-flag** |
| Schema-valid | ↔ | Correct | Syntax ↔ **meaning** (check in code) |
| Retry with the error | ↔ | Retry is useless | Fixable format ↔ **the information isn't in the source** |
| Required field | ↔ | Nullable field | Forces a guess ↔ allows an honest `null` |
| Batch API | ↔ | Synchronous | 50% cheaper, ≤ 24h, no tool loop ↔ instant, blocking checks |
| Summary | ↔ | Case facts | Lossy ↔ **exact**, held in code |
| Aggregate accuracy | ↔ | Per-category accuracy | Can hide failures ↔ shows them |
| Prompt caching | ↔ | Better accuracy | Cost and latency ↔ (no effect on quality) |
| Conflicting sources | ↔ | Picking one | Report both with attribution ↔ ❌ hides the disagreement |

---

## 🧭 Universal Answer Heuristics

1. **Must never happen** → code (gate, hook, deny rule), never only a prompt.
2. **Wrong tool chosen** → improve the descriptions first.
3. **Missing coverage** → look upstream (decomposition), not downstream.
4. **Subagent lacks information** → pass it explicitly. There is no shared memory.
5. **Failure** → a structured error with context. Never silence, never a generic message, never a total abort.
6. **Too complex** → the simplest architecture that meets the requirement.
7. **Shared with the team** → project-level files committed to git, with secrets in environment variables.
8. **Quality claim** → prove it with a test set, broken down per category.
