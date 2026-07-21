---
title: Hook Lifecycle Events
parent: Claude Code Deep Dive
nav_order: 16
duration: "11:00"
---

# Hook Lifecycle Events
{: .no_toc }

*Read duration: 11:00*

Everything covered so far in this module — permission modes, `CLAUDE.md`, Skills, subagents, Plan Mode — shapes Claude's behavior by changing what it *sees* or what it's *allowed* to do. **Hooks** are a different kind of mechanism entirely: they let you run your own code at specific, fixed points in Claude Code's lifecycle, regardless of what the model decides, giving you a way to observe, log, modify, or block what's happening from outside the model's own reasoning altogether. This lecture covers the lifecycle events a hook can attach to; the next two cover how to actually implement one and where hooks earn their keep in practice.

## Why Hooks Exist Alongside Permissions

It's worth being precise about what a hook adds that permission rules and modes don't already provide, since the two systems can sound similar at a glance. A permission rule answers a yes/no question about a specific tool call — approve it, deny it, or ask. A hook runs **arbitrary code you write**, at a **fixed point in the lifecycle**, and can do anything a script can do: log the event to an external system, inspect the details of what's about to happen and make a more nuanced decision than a static rule could express, inject additional context, or run validation logic against a whole class of actions rather than matching a single pattern. Where a permission rule is a declarative statement ("deny this pattern"), a hook is a full escape hatch into custom logic — the mechanism you reach for when what you need to express is genuinely more than a rule can capture.

## Tool-Call Hooks: Before and After

The two most commonly used lifecycle events bracket every tool call the agentic loop makes:

**Before a tool executes.** A hook attached here runs immediately before Claude Code would carry out a tool call — before a file gets edited, before a shell command runs — and, critically, **before the ordinary permission check even happens**. This is what makes it possible to build custom permission logic that's more sophisticated than a static allow/deny/ask rule: a hook here can inspect exactly what's about to run and decide, based on logic you write, whether it should proceed, be blocked outright, or force a confirmation prompt regardless of what the configured permission mode would otherwise do. This is the mechanism referenced in the permission-modes lecture as the way to build custom logic beyond what static rules alone can express.

{: .important }
> A before-tool-call hook runs *before* the ordinary permission check, not alongside or after it — that's what lets it override, tighten, or bypass what a static allow/deny/ask rule would otherwise decide. Only events attached before an action executes can actually prevent it; after-the-fact events can only react.

**After a tool executes, successfully or not.** A hook attached here runs once a tool call has completed, with access to what it actually did and what it returned. This is the natural point to log what happened (for audit purposes, or simply for a team's own visibility into what an agent has been doing), to react to a specific outcome (a failed test run triggering some downstream action), or to feed additional context back based on the result before the model's next reasoning step.

## Turn-Level and Prompt-Level Events

Bracketing tool calls specifically, another pair of events brackets the conversational turn itself:

**When the user submits a prompt.** A hook here runs before Claude even begins processing what was just typed, which makes it a useful place to inject standing context dynamically (pulling in something computed fresh, rather than baked into a static `CLAUDE.md`) or to validate the prompt itself before the model spends any effort on it.

**When Claude finishes responding for a turn.** A hook here runs once Claude has decided it's done — useful for enforcing that some condition is actually satisfied before the turn is allowed to end (a common real-world use, covered in the real-world-applications lecture, is preventing a turn from ending until tests have actually been run) rather than trusting that the model reliably remembers to check on its own.

## Subagent Lifecycle Events

Because a subagent runs its own independent loop nested inside a session, it has an analogous "finished" moment worth its own hook: a **subagent-stop** event fires when a delegated subagent completes, giving you a point to intervene on or log a subagent's specific output before it's folded back into the parent session's context — useful, for instance, for enforcing that a delegated code-review subagent's output actually follows the expected report format before the orchestrator treats it as done.

## Session-Level Events

At the outermost scope, a pair of events brackets the entire session rather than any single turn or tool call:

**When a session begins (including resuming a previous one).** A hook here is the natural place to load or refresh dynamic context that shouldn't be static — checking out the latest state of something, warming a cache, or announcing to an external system that a session has started, distinct from `CLAUDE.md`'s static, always-the-same content.

**When a session ends.** A hook here is the natural place for cleanup or final logging — recording that a session concluded, tearing down anything set up at session start, or writing a final summary to an external system for later review.

## Context-Management Events

Finally, one event brackets a mechanism unique to how a long session manages its own limits: **before context gets compacted** — the point where accumulated history is about to be summarized to make room for continued work. A hook here is a place to preserve something that would otherwise be at risk of being lost in the summarization (writing full detail out to a file before it gets compressed away in-context, for instance), which matters because compaction is lossy by nature — it trades detail for room, and a hook here is the mechanism for deciding what shouldn't be subject to that trade at all.

| Event | Fires when | Can it block the action? |
|---|---|---|
| Before a tool call | Immediately before a tool executes, before the permission check | Yes — can approve, block, or force confirmation |
| After a tool call | Once a tool call has completed | No — can only react and log |
| Prompt submitted | Before Claude processes a new user message | Can inject context or validate the prompt |
| Turn ends | Once Claude decides a turn is done | Can block the turn from ending |
| Subagent stop | When a delegated subagent completes | Can inspect the subagent's output before it returns to the parent |
| Session start | At the start of a session, including a resume | Can load or refresh dynamic context |
| Session end | When a session ends | Can run cleanup or final logging |
| Before compaction | Just before accumulated history is summarized | Can preserve detail that would otherwise be lost |

## The Pattern Across All of These

Every lifecycle event covered here answers the same underlying question at a different point in the loop: *is there something that must happen here, reliably, regardless of what the model decides to do on its own?* A hook is the mechanism for "regardless of what the model decides" — the model's own judgment, however good, is still just judgment, exercised fresh each time; **a hook fires deterministically, every time its event occurs**, whether or not the model remembers, and that determinism is the entire value proposition.

**Why it matters for the exam:** Expect questions that describe a requirement ("log every shell command that runs," "make sure tests always run before a turn is considered finished," "warn before a risky edit regardless of permission mode") and ask which lifecycle event is the right attachment point. The reliable way to answer these is to first identify *when*, precisely, in the loop the requirement needs to be checked — before or after a tool call, at the start or end of a turn or session, or around compaction — and only then match that timing to the corresponding event.

<!-- prevnext:start -->

---

| [&larr; Previous: Plan Mode: When and Why to Plan First](15-plan-mode-when-and-why-to-plan-first) | [Next: Implementing Hooks: Shell Scripts, Exit Codes, and Gotchas &rarr;](17-implementing-hooks-shell-scripts-exit-codes-and-gotchas) |
|:---|---:|

<!-- prevnext:end -->
