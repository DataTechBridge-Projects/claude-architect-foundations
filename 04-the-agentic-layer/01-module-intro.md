---
title: Module Intro
parent: The Agentic Layer
nav_order: 1
duration: "3:18"
---

# Module Intro
{: .no_toc }

*Read duration: 3:18*

The Fundamentals module defined an agent precisely: an LLM operating in a loop, choosing actions instead of only producing text, observing what those actions return, and deciding what to do next until the task is finished or some limit is reached. The Agentic Layer is the first of the five core domains in the CCA-F blueprint, and it's where that one-sentence definition gets unpacked into an actual architecture — the mechanics you need to reason about correctly, not just describe.

This module builds in four movements. First, you'll go deep on the orchestration loop itself: model call, tool-use request, tool execution, result returned to the model, repeat. Every agentic system, from a single Claude Code session to a large multi-agent pipeline, is built from this loop running at some scale. Understanding exactly what happens at each step — and where latency, cost, and failure creep in — is the foundation for everything else in this course, because the Tool, Workflow, Prompt, and Context layers each describe ways of shaping what flows through this same loop.

Second, you'll scale the loop up: from one agent making its own decisions to multiple agents coordinating — orchestrators delegating to subagents, agents running in parallel versus in sequence, and the very real overhead and failure modes that appear the moment a system has a second decision-maker in it. Spinning up a subagent is not free, and this module treats it as an architectural decision with real tradeoffs, not a reflexive move you make whenever a task feels complicated.

Third, you'll look at workflow decomposition — how a large, ambiguous goal gets broken into discrete steps an agent, or a team of agents, can actually execute — and how to safeguard that execution with checkpoints, approval gates, and rollback strategies so autonomy doesn't quietly turn into unbounded risk.

Fourth, you'll cover session management: how conversation state, context, and history persist and evolve across a multi-turn interaction, when to summarize or reset that state, and how a session's lifecycle differs from a single loop iteration or a single subagent's lifespan.

By the end of this module, you should be able to look at a described system and identify which part of it is "the agentic layer" — the loop, the delegation structure, the decomposition, or the session boundary — and reason about how a change to one of those pieces ripples into the others. That skill, more than any single definition, is what the exam is testing, and it's the skill the rest of this course assumes you already have.

<!-- prevnext:start -->

---

| [&larr; Previous: The Agentic Layer](./) | [Next: The Orchestration Loop &rarr;](02-the-orchestration-loop) |
|:---|---:|

<!-- prevnext:end -->
