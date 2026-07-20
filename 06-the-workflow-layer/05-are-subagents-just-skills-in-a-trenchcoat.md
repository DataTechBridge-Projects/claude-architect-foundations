---
title: Are Subagents Just... Skills In a Trenchcoat?
parent: The Workflow Layer
nav_order: 5
duration: "1:00"
---

# Are Subagents Just... Skills In a Trenchcoat?
{: .no_toc }

*Video duration: 1:00*

They can look alike from a distance — both are "a reusable capability the main agent reaches for to handle a sub-task" — but the mechanism underneath is fundamentally different, and the exam cares about the mechanism. A **subagent** is instantiated with its own context window and makes its own independent model calls; it reasons separately, can run tool calls the parent never sees in detail, and reports back only a result (or summary) that the parent then incorporates. A **Skill** never leaves the calling agent's context: its instructions (and any bundled tools) are loaded directly into the *current* agent's existing context, and the same agent, same model call sequence, continues acting — just now with a runbook in hand. Same surface goal, "get help with a sub-task," completely different cost, isolation, and failure-mode profile underneath.

A worked example makes the trenchcoat metaphor collapse quickly. Take "review this pull request." As a Skill, that's a set of instructions — house review checklist, which categories of finding are blocking, how to format the comment — loaded into the *current* agent's context the moment it recognizes a PR-review task, and then the same agent, in the same conversation, walks through the diff using its existing tools. As a subagent, "review this PR" instead spins up an entirely separate agent with its own fresh context: it reads the diff independently, forms its own judgment uncontaminated by whatever the parent agent has already reasoned about the change, and hands back only a summary of findings. The parent never sees the subagent's intermediate reasoning or false starts — only the conclusion.

That difference has real consequences, not just architectural trivia. A Skill is cheap: no new model call is spun up, no context window duplicated, and the parent's own accumulated understanding of the task carries straight through. But that cheapness is also a limit — a Skill can't give you a genuinely independent second opinion, because it's reasoning inside the same context that's already been shaped by everything that came before it in the session. A subagent costs more (a separate model call, its own token spend, coordination overhead to hand off the task and receive the result) but buys isolation: a fresh perspective free of the parent's accumulated assumptions, and a context boundary that keeps a noisy or exploratory sub-task from polluting the parent's own reasoning trace.

This is the same distinction the Skill Configuration lecture flags as the detail candidates most often miss: reach for a Skill when a well-defined procedure just needs to augment the current agent's capability for one step; reach for a subagent when the sub-task genuinely needs isolated reasoning, its own verification, or a fresh set of eyes the parent's own context can't provide.
