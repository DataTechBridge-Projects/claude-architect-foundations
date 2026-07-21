---
title: "The Claude Agent SDK: Session Control"
parent: Claude Code Deep Dive
nav_order: 19
duration: "12:00"
---

# The Claude Agent SDK: Session Control
{: .no_toc }

*Read duration: 12:00*

Everything covered so far in this module has been about configuring and extending Claude Code as a product — a terminal application (and its editor and desktop surfaces) that a human runs interactively. This lecture and the next mark a shift: from using Claude Code to building on the same underlying engine programmatically, for cases where the goal isn't a human typing into a terminal at all, but a custom application, a script, or a pipeline that needs the same agentic capability.

## What the Agent SDK Actually Provides

The **Claude Agent SDK** exposes the same tools, the same agentic loop, and the same context-management machinery that powers Claude Code itself, packaged for programmatic use rather than interactive use — available as a command-line interface for scripts and pipelines, and as installable packages for building richer applications directly in code. The important distinction to hold onto is what this gives you *beyond* calling the underlying model API directly: **a raw API call to the model gives you a single completion in response to a single request**, and any looping, tool execution, or context management needed to turn that into an actual agent is something you'd otherwise have to build yourself. The Agent SDK gives you that whole harness already built — the read-plan-act-verify loop from earlier in this module, the same built-in tool set (or a scoped-down version of it), and the same context assembly and compaction behavior — and lets you drive it from your own code instead of from a terminal prompt.

This is the same relationship the Agentic Layer draws between "an agent" as a general architectural pattern and Claude Code as one specific, opinionated product built on that pattern, extended one level further: the Agent SDK is the mechanism for building your *own* product on that same pattern, reusing Claude Code's actual engine rather than reimplementing an agentic loop, a tool set, and a context-management strategy from scratch.

## Why Reach for the SDK Instead of the Interactive Product

A team building a custom internal tool — an automated pull-request reviewer, a support-ticket triage agent, a data-pipeline monitor that investigates its own failures — needs agentic behavior embedded inside its own application, with its own interface, its own triggering logic, and its own integration with whatever systems the application already talks to. None of that fits naturally into a human sitting at a terminal typing into Claude Code interactively; it fits a program that starts a session, drives it toward a specific goal, and does something with the result, entirely without a human present for any of it. The Agent SDK is built exactly for this shape of problem: same underlying agentic capability, but invoked from inside a program you control rather than from an interactive terminal session.

## Session Control: Managing State Across Turns Programmatically

The specific capability this lecture's title highlights — **session control** — is the set of mechanisms the SDK provides for managing a conversation's state deliberately, rather than letting it accumulate implicitly the way an interactive session does by default. A few concrete pieces of this matter in practice:

**Starting fresh versus continuing.** Every invocation through the SDK can either start an entirely new session, with no prior history, or continue an existing one — picking up exactly where a previous invocation left off, with its accumulated context intact. This distinction matters architecturally the same way `/clear` matters in interactive use: continuing a session preserves useful accumulated context (what's already been explored, what's already been decided) at the cost of that context continuing to consume space and, eventually, needing compaction; starting fresh discards all of that in exchange for a clean, minimal context appropriate to a genuinely new, unrelated task.

**Addressing a specific session by identity.** A programmatic caller doesn't have the luxury of a human simply remembering "my most recent conversation" — a real application might be juggling many concurrent sessions (one per pull request under review, one per support ticket) and needs to resume the *correct* one deliberately. Session control in the SDK means every session has an addressable identity a calling application can capture and use later, specifically so "continue this one, not that one" is a decision the calling code makes explicitly rather than something left to positional convenience.

**Structured, machine-readable output.** Where an interactive session's natural output is conversational text meant for a human to read, a programmatic caller usually needs something a downstream system can consume reliably — structured output conforming to a schema the calling application defines, rather than prose that has to be parsed after the fact with all the fragility that implies. This is a genuinely different consumption model from interactive use, and the SDK is built with it in mind: a calling application can specify the shape it needs back and get a response that actually conforms to it, rather than reverse-engineering intent from natural-language prose.

**Tool permission handled programmatically.** In interactive use, a permission prompt has an obvious answer: ask the human at the keyboard. A programmatic caller has no human at the keyboard by default, so the SDK provides a way for the calling application itself to supply the approval logic — a callback the application defines that decides, based on whatever criteria the application's own author considers appropriate, whether a given proposed action should proceed. This is the same permission-layer concept from earlier in this module, relocated: **instead of a human approving each action live, the calling application's own code plays that role**, deliberately and in advance, based on rules it was given.

{: .important }
> A programmatic SDK session has no human at the keyboard to answer a permission prompt by default. Supplying the approval callback is the calling application's responsibility — don't assume unattended sessions get safe defaults without that logic being explicitly written.

## Streaming and Turn-by-Turn Visibility

A related capability worth naming directly is the SDK's support for streaming a session's output as it's produced, rather than waiting silently for a final answer and receiving it all at once. For a long-running task — a multi-file investigation, an extended piece of research — a calling application can process events as they arrive: a tool call being made, a partial response being generated, a subagent's own activity nested inside the parent's stream. This matters for exactly the reason it matters in an interactive terminal: visibility into what's happening *during* a long task, not just its eventual conclusion, is often what lets a calling application (or a human watching its output) intervene early rather than discovering a wrong direction only once the whole task has already finished.

## The Throughline: Same Engine, Different Driver

| | Raw model API call | Agent SDK | Interactive Claude Code |
|---|---|---|---|
| What you get | A single completion | Full agentic loop, tools, context management, callable from code | Full agentic loop, driven by a human at a terminal |
| Who approves actions | N/A | A callback the calling application defines | A human, live |
| Output | Raw model text | Structured, machine-readable | Conversational, for a human to read |

It's worth stepping back and being precise about what changes and what doesn't between interactive Claude Code and an SDK-driven application. What doesn't change: the underlying agentic loop, the built-in tool set (or a version of it scoped to what the application needs), the way context is assembled and compacted, and the fundamental read-plan-act-verify cycle from earlier in this module. What does change: who's making the moment-to-moment decisions that a human makes interactively — approving actions, deciding when to continue a session versus start fresh, consuming the output in a form suited to code rather than to a person reading a terminal. The Agent SDK is best understood not as a different product, but as the same engine with its driver's seat opened up to your own code instead of a human's keyboard.

**Why it matters for the exam:** Expect questions distinguishing the Agent SDK from both a raw model API call (which gives you a single completion, not a loop, a tool set, or context management) and from interactive Claude Code (which gives you the same engine but assumes a human at the keyboard making live decisions). Also expect questions specifically probing session control mechanics — starting fresh versus continuing, and why a programmatic caller needs an addressable session identity where a human interactively usually doesn't.

<!-- prevnext:start -->

---

| [&larr; Previous: Real-World Hook Applications](18-real-world-hook-applications) | [Next: Claude Code in CI/CD: Non-Interactive Mode and the -p Flag &rarr;](20-claude-code-in-ci-cd-non-interactive-mode-and-the-p-flag) |
|:---|---:|

<!-- prevnext:end -->
