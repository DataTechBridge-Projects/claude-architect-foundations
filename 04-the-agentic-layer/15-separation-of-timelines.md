---
title: Separation of Timelines
parent: The Agentic Layer
nav_order: 15
duration: "0:38"
---

# Separation of Timelines
{: .no_toc }

*Read duration: 0:38*

This module has quietly introduced several nested timelines, and keeping them separate avoids a lot of confusion. A **loop iteration** is the shortest: one model call, maybe one tool execution. A **subagent's lifespan** covers its own loop, start to report-back. A **task** may span many loop iterations, possibly several subagent dispatches, and completes against a defined condition. A **session** is the longest and least bounded: it can contain many tasks over hours or days. When something goes wrong, ask which timeline it actually happened on before diagnosing why.

| Timeline | Scope | Typical safeguard |
|---|---|---|
| Loop iteration | One model call, maybe one tool execution | Idempotency key on the call |
| Subagent lifespan | Dispatch through report-back | Scoped tools, defined reporting format |
| Task | Many loop iterations, possibly several subagent dispatches | Checkpoint at the task boundary |
| Session | Many tasks over hours or days | Summarization, windowing, or a reset |

Made concrete, a single Claude Code session might run for an entire afternoon and contain a dozen distinct tasks: "fix the failing test," "refactor the auth module," "write release notes." Each of those tasks might itself take fifteen loop iterations to complete, and two or three of those iterations might each dispatch a subagent — say, one to search the codebase for every call site of a function being renamed. That subagent has its own loop iterations nested inside it, invisible to the parent task except for the final report. Four timelines, four different natural units of "how long did this take" and "what state persists here," all active in the same afternoon, all easy to conflate if you're not deliberate about which one you mean.

The diagnostic value of separating them shows up directly when something breaks. If a single tool call returns malformed data, that's a loop-iteration problem — fix the tool or how its result is parsed, and the fix is local. If a subagent's final report is confidently wrong despite the subagent's individual tool calls all succeeding, that's a subagent-lifespan problem — the issue is in how that subagent reasoned across its own loop, not in any single step. If the same kind of task keeps needing manual correction every third or fourth attempt, that's a task-level pattern that a single-iteration bug fix won't touch. And if a session accumulates enough completed tasks that overall response quality degrades late in the day, that's a session-level context management issue — no individual task or subagent did anything wrong; the session's cumulative footprint became the problem.

This layering also explains why the "server" analogy from earlier in this module and the idempotency and checkpointing patterns from the previous lecture apply at different granularities depending on which timeline you're safeguarding: a checkpoint makes sense at the task boundary (resume the task, not the whole session, if it fails), while an idempotency key makes sense at the loop-iteration boundary (don't double-execute one retried tool call). Naming the right timeline first is what lets you reach for the right-sized fix instead of over- or under-correcting.

<!-- prevnext:start -->

---

| [&larr; Previous: Session Management](14-session-management) | [Next: Lab: AI Workforce Allocation Simulator &rarr;](16-lab-ai-workforce-allocation-simulator) |
|:---|---:|

<!-- prevnext:end -->
