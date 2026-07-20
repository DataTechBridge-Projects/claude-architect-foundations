---
title: Subagent Instantiation
parent: The Agentic Layer
nav_order: 8
duration: "15:36"
---

# Subagent Instantiation
{: .no_toc }

*Video duration: 15:36*

The previous lecture established when splitting work across agents is justified. This one gets into the mechanics: what actually happens when an orchestrator spins up a subagent, what it hands that subagent, what it deliberately withholds, and how the subagent's work makes its way back into the parent's context. These are the details the exam expects you to get precisely right, because "instantiate a subagent" is not one operation — it's a bundle of specific design decisions.

## Fresh context versus shared context

The single most consequential decision in subagent instantiation is whether the subagent starts from a **fresh context** or a **shared context** with its parent.

A subagent instantiated with a fresh context starts with nothing but what the orchestrator explicitly hands it — no visibility into the orchestrator's own conversation history, prior tool calls, or accumulated exploration. This is the default and the far more common pattern, and it's what makes context isolation (the primary benefit from the previous lecture) actually work: if the subagent inherited the parent's full history, all of the parent's accumulated noise would be sitting in the subagent's context too, defeating the purpose of delegating in the first place. Claude Code's subagent feature works this way — a subagent runs its own independent orchestration loop with its own context window, and the only information that crosses the boundary is what's explicitly passed in at dispatch time and what's explicitly returned at completion.

A subagent instantiated with a shared (or partially shared) context has some or all of the parent's history visible to it from the start. This is rarer, and it trades away the isolation benefit for continuity — useful in cases where the subagent genuinely needs the parent's full situational awareness to do its job correctly, and where the cost of re-explaining that situation via an explicit task description would itself be substantial. This pattern is closer to a "second opinion within the same conversation" than a true delegation, and it doesn't get you the parallelism or specialization benefits as cleanly, because the shared context tends to anchor the subagent's reasoning toward whatever the parent already concluded.

The default assumption, and the one the exam will generally expect unless a scenario says otherwise, is fresh context: subagents start clean, and everything they know about the task comes from what the orchestrator deliberately hands them.

## What to hand a subagent

Because a fresh-context subagent starts with nothing, the orchestrator's dispatch has to be a complete, self-contained briefing. Getting this briefing right is most of the skill in subagent design, and it typically has four components.

**A task description.** Not the orchestrator's own framing of the overall goal, but a specific, scoped statement of what this subagent is being asked to do — ideally including what a correct, complete answer looks like, and any constraints on approach. A vague task description ("look into the authentication issue") forces the subagent to guess at scope, which tends to produce either an overly narrow investigation that misses the point or an overly broad one that burns context on irrelevant exploration. A well-scoped description ("determine why login requests from mobile clients are returning 401 after the token refresh; check the token refresh handler and the mobile client's retry logic; report the root cause and the specific file/line where it originates") gives the subagent a bounded, checkable target.

**Relevant context, deliberately curated.** Anything the subagent genuinely needs and cannot cheaply rediscover itself — a specific file path, a prior finding, a constraint from earlier in the conversation — has to be handed over explicitly, since the subagent can't see the parent's history. This is a real design tension: hand over too little and the subagent wastes its own loop rediscovering things the orchestrator already knew; hand over too much and you've defeated the point of a fresh context by pre-loading it with the same noise you were trying to avoid. The discipline here is the same discipline as writing a good prompt in general — include what's load-bearing for the task, omit what isn't, regardless of how naturally it comes to mind while writing the dispatch.

**A defined tool set.** A subagent should generally be given only the tools it needs for its specific task, not the full tool set the orchestrator itself has access to. A subagent tasked with researching an API's documentation doesn't need write access to the file system; a subagent tasked with running a fix doesn't need broad search tools it won't use. This is not merely tidiness — it's a direct safety and blast-radius control (developed further in the workflow safeguarding lecture): the smaller a subagent's tool surface, the smaller the set of things that can go wrong if its judgment is off for this particular task.

**A reporting format.** Because the orchestrator will only see what the subagent chooses to report back, it's worth being explicit about the shape of that report — a structured summary, a specific set of fields, a pass/fail verdict plus supporting detail — rather than leaving it to the subagent's own judgment about what's worth surfacing. An unspecified reporting format tends to produce either report bloat (the subagent, unsure what matters, includes everything) or the opposite failure, an overly terse summary that drops a detail the orchestrator actually needed.

## What to withhold

Just as important as what to hand over is what to deliberately leave out, and this is where a lot of naive subagent designs go wrong by defaulting to "just give it everything, to be safe."

**The orchestrator's full history.** As established above, this is the entire point of fresh-context instantiation — withholding it is not a limitation to work around, it's the mechanism that makes delegation valuable in the first place.

**Tools irrelevant to the subtask.** Beyond the blast-radius argument above, an oversized tool set also degrades the subagent's own decision quality: every additional tool definition is additional context competing for the model's attention at every step of the subagent's loop, and an irrelevant tool is a pure cost with no offsetting benefit — it can only ever be a distraction or, worse, get called inappropriately.

**Authority beyond the task's scope.** A subagent dispatched to investigate a bug shouldn't, by default, also have unrestricted authority to deploy a fix, unless that's explicitly part of what it was asked to do — scope creep in a subagent's authority is exactly the kind of unbounded-autonomy risk the workflow safeguarding lecture addresses directly.

**Irrelevant sibling subagent activity.** In a parallel-dispatch scenario, a subagent generally shouldn't see what its sibling subagents are doing in real time — that would reintroduce a shared-context coupling the parallel pattern was specifically designed to avoid, and it invites subagents to second-guess or duplicate each other's work instead of executing their own scoped task independently.

## How subagent results fold back into the parent's context

Once a subagent's own loop terminates — by its own judgment, or by hitting a limit imposed on it — its final report is what crosses back into the orchestrator. Mechanically, this is exactly the tool-result step of the orchestration loop: from the orchestrator's perspective, "the subagent finished" looks identical to any other tool execution completing, and the subagent's report is appended to the orchestrator's context as a tool result, exactly like a file read or an API call's output would be.

This has two important consequences. First, everything discussed in the "Results Polluting the Context" aside applies here with extra force: a subagent's report is often the *distilled* output of a much larger internal exploration, and if that report isn't actually distilled — if the subagent dumps its full internal reasoning or raw findings back to the parent instead of a summary — the orchestrator's context absorbs a large chunk of exactly the noise the subagent was meant to contain. Designing a good subagent means designing a good report format at least as much as it means designing a good task description.

Second, because the orchestrator only sees the report, not the subagent's process, the orchestrator's ability to sanity-check the subagent's work is limited to whatever the report itself reveals. This is why well-designed subagent reports typically separate a conclusion from its supporting evidence (not just "the bug is in the token refresh handler" but the specific evidence for that claim) — it gives the orchestrator, or a human reviewing the orchestrator's own output, something to verify the claim against, rather than an assertion that has to be trusted at face value purely because it came back from an agent that "did the work."

## A worked example

Consider an orchestrator handling "prepare a competitive analysis of three named products." A reasonable instantiation: dispatch three subagents in parallel, one per product, each handed a fresh context containing only that product's name, a fixed list of dimensions to research (pricing, core features, target market), read-only web-search and fetch tools, and an explicit instruction to report findings as a short structured summary per dimension with source links, not as a full transcript of everything found. Each subagent runs its own loop, does its own exploration — which might involve dozens of searches and page reads — and reports back only the distilled summary. The orchestrator, receiving three structured summaries rather than three sprawling research transcripts, then has a clean, comparably-shaped context from which to write the final comparison. Notice how much of the design work here is in the briefing (what's handed over) and the reporting contract (what comes back), not in the subagent's internal loop itself — that loop is just the standard orchestration loop from earlier in this module, running three times independently.

## Why it matters for the exam

Expect the exam to describe a subagent dispatch — sometimes a well-designed one, sometimes a flawed one — and ask you to identify what's missing (an underspecified task description, an overbroad tool grant, no defined reporting format) or what's correctly designed. The fresh-context-by-default assumption, the four components of a good briefing, and the principle that a subagent's report re-enters the parent exactly as a tool result would, are the specific, testable mechanics to have solid before moving on to workflow decomposition and safeguarding, both of which build directly on how subagents are instantiated and scoped.
