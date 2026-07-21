---
title: Agents as Wrappers
parent: The Agentic Layer
nav_order: 9
duration: "0:37"
---

# Agents as Wrappers
{: .no_toc }

*Read duration: 0:37*

From an orchestrator's point of view, a subagent is indistinguishable in kind from any other tool: it has a name ("the research agent"), it takes structured input (a task description and scope), and it returns a result. The fact that a whole independent orchestration loop runs underneath that call is invisible to the orchestrator — it never sees the subagent's own tool calls or reasoning, only the report. This is a useful simplification: you can design an orchestrator's tool set as a mix of plain tools and agent-shaped tools without changing how the orchestrator itself reasons about calling them.

Concretely, imagine an orchestrator's tool list includes `read_file`, `run_tests`, and `delegate_to_research_agent`. From the schema the model sees, all three look the same: a name, a set of typed parameters, a description of what it does and when to use it. The model doesn't need a different mental model for "call a function" versus "spin up an entire nested agent that might run for thirty seconds and make a dozen of its own tool calls internally" — it just sees a tool with a contract, calls it, and waits for a result to come back as a tool-result message like any other. The enormous asymmetry in what actually happens underneath — a few milliseconds of file I/O versus a full independent loop with its own model calls, its own tool executions, and its own termination logic — is entirely hidden behind that identical calling convention.

| | `read_file` (plain tool) | `delegate_to_research_agent` (subagent) |
|---|---|---|
| Interface the model sees | Name, parameters, description | Name, parameters, description |
| What happens underneath | Milliseconds of file I/O | A full independent orchestration loop |
| Visibility to the orchestrator | N/A — single deterministic operation | None of the internal loop, only the final report |

This wrapper framing is what makes composition tractable. Because a subagent presents the same interface as a plain tool, you can swap one for the other without touching the orchestrator's reasoning: a `summarize_document` capability might start life as a single well-crafted prompt-and-parse call and later get upgraded into a full subagent that reads the document, cross-checks details against other sources, and drafts a more careful summary — and the orchestrator's tool-calling behavior doesn't need to change at all, only what's on the other side of the name it's calling.

The tradeoff is that "wrapper" cuts off visibility along with complexity. Exactly as covered in the results-polluting-the-context lecture earlier in this module, hiding the subagent's internal loop is precisely what keeps its noisy intermediate exploration out of the orchestrator's context — but it also means that if the subagent's internal reasoning went subtly wrong before it produced its final report, **the orchestrator has no way to inspect that reasoning after the fact**; it can only judge the report it received. Treating a subagent as a wrapper is a deliberate trade of transparency for a clean interface, not a free simplification.

{: .important }
> Wrapping a subagent behind a tool-shaped interface trades transparency for composability: if the subagent's internal reasoning went wrong before it produced its report, the orchestrator has no way to inspect that reasoning after the fact — it can only judge the report it received.

<!-- prevnext:start -->

---

| [&larr; Previous: Subagent Instantiation](08-subagent-instantiation) | [Next: Workflow Decomposition &rarr;](10-workflow-decomposition) |
|:---|---:|

<!-- prevnext:end -->
