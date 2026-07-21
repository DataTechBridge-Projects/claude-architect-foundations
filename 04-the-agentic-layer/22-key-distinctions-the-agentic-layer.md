---
title: Key Distinctions - The Agentic Layer
parent: The Agentic Layer
nav_order: 22
duration: "3:16"
---

# Key Distinctions - The Agentic Layer
{: .no_toc }

*Video duration: 3:16*

A condensed pass through the module's highest-yield either/or distinctions — the kind of pairs the exam likes to test directly against a scenario.

**Agentic versus non-agentic.** Agentic means the model's own decision, at each step, shapes what it observes next (the closed loop: decide, act, observe, decide again). A single model call, or a fixed pipeline that calls the model once as one step among predetermined ones, is not agentic — regardless of how sophisticated the output looks.

**Structured tool use versus prose-described actions.** Structured tool calls are schema-validated, machine-parseable, and composable as the tool set grows. Prose-described intent requires fragile secondary parsing, has no validation boundary, and degrades as more actions are added. The Claude API's native tool-use content blocks are structured by design.

**Single agent versus multi-agent.** Multi-agent coordination is justified by context isolation, genuine parallelism, specialization, or independent review — not by task difficulty alone. It always adds latency, cost, and coordination overhead that a single well-managed agent doesn't pay.

**Orchestrator versus subagent.** The orchestrator holds the overall task and decides on decomposition and delegation; it sees only what a subagent explicitly reports back, never the subagent's internal loop. From the orchestrator's side, a subagent is just another structured tool call.

**Fresh context versus shared context.** Fresh-context subagents (the default) start with nothing but an explicit briefing — this is what makes context isolation actually work. Shared-context subagents trade that isolation for continuity, and are the exception, not the rule.

**Parallel versus sequential coordination.** Parallel suits independent subtasks and saves wall-clock time at the cost of result reconciliation. Sequential suits genuine data dependencies and is simpler to reason about, at the cost of stacked latency.

**Coarse-grained versus fine-grained decomposition.** Coarse steps mean fewer handoffs but push more implicit decomposition burden onto whatever executes each step. Fine-grained steps are independently checkable and checkpointable but multiply coordination overhead. Match granularity to a step's uncertainty and cost of failure.

**Checkpoint versus approval gate.** A checkpoint is any pause for inspection or recovery; an approval gate is specifically a human go/no-go decision, reserved for irreversible or high-blast-radius actions rather than applied uniformly.

**Reversible versus compensable versus irreversible actions.** Reversible actions can simply be undone. Compensable actions need a defined counteracting step. Irreversible actions need a gate *before* they happen, because nothing can fix them after.

**Loop iteration versus task versus session.** These are nested, distinct timelines — a symptom belongs to whichever one it actually occurred on, and misattributing it (calling a session-management problem a loop problem, or vice versa) leads to the wrong fix.
