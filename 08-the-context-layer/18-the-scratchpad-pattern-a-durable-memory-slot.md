---
title: "The Scratchpad Pattern: A Durable Memory Slot"
parent: The Context Layer
nav_order: 18
duration: "11:00"
---

# The Scratchpad Pattern: A Durable Memory Slot
{: .no_toc }

*Read duration: 11:00*

The memory tool, covered in the previous lecture, solves the problem of information that needs to survive across entirely separate sessions. This lecture covers a related but narrower pattern that solves a different problem: state that needs to survive *within* a single long or complex task, even as the live context window around it gets compacted, summarized, or handed off to another agent. That pattern is the **scratchpad** — a designated slot where an agent writes down its intermediate reasoning, its current plan, or its findings-so-far, specifically so that this material can be recovered reliably rather than depending on it happening to still be legible somewhere in an increasingly compacted transcript.

## What a scratchpad is for

Picture an agent working through a genuinely multi-step task — a multi-stage debugging investigation, a research task with several sequential sub-questions, a long-running build-and-verify loop. As the task proceeds, the agent accumulates a working understanding: which hypotheses it's already ruled out, what the current plan looks like, which facts it has already confirmed. All of that is exactly the kind of material "Managing the Context" describes as good compaction fodder — verbose, exploratory, prone to become dead weight in the live transcript. But it isn't *worthless*; it's the agent's actual working memory for the task, and losing it outright would mean re-deriving it from scratch, at real cost, if it's needed again later.

The scratchpad resolves this tension by giving that working memory a home outside the ordinary flow of conversation history. Rather than letting the plan and findings live only inside the turn-by-turn transcript — where they're vulnerable to being summarized away, truncated under a token budget, or simply buried under everything that came after — the agent writes them explicitly to a scratchpad: a structured, purpose-built note of "here's what I've established, here's the current plan, here's what's still open." That scratchpad can then be consulted directly, refreshed as the task progresses, and — this is the part that matters most architecturally — handed off cleanly if the task's state needs to move somewhere else: to a compacted continuation of the same session, or to a subagent picking up a piece of the work.

The core value proposition is durability under exactly the conditions that threaten ordinary conversation history. A compaction pass, as described in "Compaction and Long-Conversation Coherence," is a deliberate, judgment-driven summarization of history — but it's still a lossy operation performed by an outside process on the transcript, and it can misjudge what mattered. A scratchpad entry, by contrast, is written by the agent itself, at the moment it knows the material is worth keeping, specifically for the purpose of surviving whatever compaction or handoff comes next. It shifts the responsibility for preserving working state from "hope the compaction pass doesn't lose it" to "the agent deliberately committed it to a durable slot."

## Scratchpad vs. persistent memory: a matter of scope

It's worth being precise about how a scratchpad differs from the memory tool covered in the previous lecture, because they can look similar on the surface — both involve writing state somewhere other than the live conversation. The distinction is scope and lifetime. Persistent memory, as covered there, is built for facts that outlive an entire session or task: a user's standing preference, a project's status across many separate future conversations. A scratchpad is built for state that's scoped to a *single* ongoing task — the plan, the findings, the ruled-out hypotheses for the debugging session happening right now — and its useful life typically ends when that task concludes. A scratchpad entry for a finished task is usually safe to discard entirely; a persistent memory entry, by design, is meant to keep mattering indefinitely. Conceptually, think of the scratchpad as working memory for a task in flight, and persistent memory as long-term memory for facts that outlast any one task.

{: .important }
> **Don't conflate the scratchpad with persistent memory** — a scratchpad is task-scoped working memory that's usually safe to discard once the task ends; persistent memory is meant to keep mattering indefinitely, across entirely separate future sessions.

| | Scratchpad | Persistent memory |
|---|---|---|
| Scope | A single ongoing task | Across entire sessions and tasks |
| Lifetime | Ends when the task concludes | Indefinite, until explicitly updated |
| Analogy | Working memory | Long-term memory |

## Why the scratchpad matters for continuity, not just tidiness

The practical payoff of a scratchpad shows up at exactly the two moments this lecture's stub description points to: when context gets compacted, and when work gets handed to a different agent.

When a long session crosses the compaction threshold described in "Compaction and Long-Conversation Coherence," a well-maintained scratchpad gives the compaction process something authoritative to preserve or reference, rather than forcing it to reconstruct the task's current state by re-reading and re-summarizing the entire raw transcript. If the scratchpad already contains "current hypothesis: the race condition is in the retry handler; ruled out: connection pooling, timeout misconfiguration; next step: add logging around the retry path," a compaction pass can lean on that structured statement directly instead of trying to extract the same conclusion from twenty turns of exploratory back-and-forth, some of which may no longer be around to re-read by the time compaction runs.

The handoff case is similar in spirit but crosses an agent boundary rather than a time boundary. A task passed from one agent to another — an orchestrator delegating a continuation to a subagent, or a session being picked up by a differently-scoped agent — benefits enormously from a scratchpad that states the task's current status cleanly, rather than requiring the receiving agent to inherit and re-parse the entire history that produced that status. This is the same instinct behind the isolation principle from "Managing the Context" — a subagent performs better when given a purpose-built context for its task rather than an orchestrator's full accumulated history — applied specifically to the problem of *state*, not just instructions: the scratchpad is the artifact that carries "where things stand" across that boundary without dragging the noise that produced it along for the ride.

## Scratchpad hygiene: what belongs there, and keeping it bounded

A scratchpad is only useful if it stays disciplined, and the two failure modes worth naming mirror the two failure modes of context management generally: writing too little, and writing too much.

**Writing too little** defeats the purpose — a scratchpad that only ever records the current instruction, without the ruled-out hypotheses or the reasoning behind the current plan, gives a compaction pass or a receiving subagent no more than the live context already would have. The scratchpad earns its place by capturing exactly the material that's valuable enough to survive but too verbose to keep replaying in full — decisions and their justification, not just their outcomes, echoing the same principle "Managing the Context" applies to summarization generally.

**Writing too much** recreates the problem the scratchpad exists to solve. An agent that treats the scratchpad as a running, unedited transcript of everything it's ever thought during the task turns it into exactly the kind of bloated, diluting artifact this module warns against elsewhere — except now it's a second copy of the problem, sitting alongside the context window rather than instead of it. Good scratchpad hygiene means treating entries the way a good summary treats history: **update in place rather than append indefinitely**, drop findings once they're superseded by a later, more complete finding, and periodically prune anything that turned out not to matter. A scratchpad that's grown to rival the size of the original transcript it was meant to condense has stopped doing its job.

The deciding question for what's worth persisting to the scratchpad versus leaving in live context is the same durability question from the previous lecture, applied at task scope rather than session scope: will a later step in *this same task*, or whoever picks it up next, need this specific conclusion without wanting to re-derive it from scratch? If yes, it belongs on the scratchpad, stated cleanly. If it's exploratory detail that did its job and isn't needed again, it can stay in the ordinary flow of context, subject to compaction like everything else.

**Why it matters for the exam:** Expect the exam to test whether you can distinguish the scratchpad pattern (task-scoped working memory that survives compaction and handoffs within one piece of work) from persistent memory (durable facts that survive across entirely separate sessions), and to recognize scratchpad bloat — an unpruned, ever-growing scratchpad — as its own failure mode, not a safety net. A well-designed scratchpad is judged by the same discipline as a good summary: capture decisions and their justification, not a full transcript.

<!-- prevnext:start -->

---

| [&larr; Previous: The Memory Tool for Persistence](17-the-memory-tool-for-persistence) | [Next: Compaction and Long-Conversation Coherence &rarr;](19-compaction-and-long-conversation-coherence) |
|:---|---:|

<!-- prevnext:end -->
