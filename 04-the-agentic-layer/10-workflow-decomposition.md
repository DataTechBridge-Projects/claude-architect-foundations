---
title: Workflow Decomposition
parent: The Agentic Layer
nav_order: 10
duration: "14:56"
---

# Workflow Decomposition
{: .no_toc }

*Video duration: 14:56*

Everything up to this point in the module has assumed a task is already scoped: a well-defined subagent dispatch, a single orchestration loop with a clear stopping point. Real goals rarely arrive that way. "Migrate our billing system to the new provider," "prepare Q3's board deck," "fix the flaky test suite" are goals, not tasks — and turning a goal into something an agent (or a coordinated set of agents) can actually execute is **workflow decomposition**: the deliberate act of breaking a larger objective into discrete, executable steps.

## Why decomposition is a distinct skill from delegation

It's worth separating this from the multi-agent material in the previous two lectures, even though the two are related. Multi-agent coordination is about *how* work gets distributed once you know what the pieces are. Workflow decomposition is about figuring out what the pieces *are* in the first place — and that's a prerequisite step whether the resulting pieces end up running as one agent working through a list, several subagents working in parallel, or a fixed pipeline with no agentic decision-making at any individual step. A poorly decomposed workflow produces bad outcomes regardless of how well the resulting pieces are then coordinated; good coordination cannot rescue a decomposition that split the problem in the wrong place.

## What makes a step a good unit of decomposition

A well-formed step in a decomposed workflow tends to share a few properties, and these are worth internalizing as a checklist rather than a formula, since real tasks will violate one or another of them and require judgment calls.

**A step should have a checkable completion condition.** You should be able to say, concretely, what "done" looks like for this step, independent of the rest of the workflow — "the schema migration script runs against a staging copy without error," not "the database work is finished." Steps without a checkable completion condition are the ones most likely to silently under-deliver, because nothing in the workflow actually verifies they succeeded before the next step builds on top of them.

**A step should have a defined input and a defined output.** What does this step need to start, and what does it produce that the next step (or the final result) consumes? This is the same discipline as the subagent briefing-and-report design from the previous lecture, applied at the workflow level rather than the individual-dispatch level — and for good reason, since a workflow step frequently *is* a subagent dispatch, or could be restructured as one.

**A step should be scoped to a coherent set of tools and context.** If a single step genuinely requires switching between unrelated tool sets or unrelated bodies of context partway through, that's usually a sign the step is actually two steps wearing one label. This connects directly to the short lecture later in this module on splitting when something changes — a shift in required tools, required data, or required reviewer is one of the most reliable signals that a decomposition boundary belongs there.

**A step should be small enough to fail independently and be retried independently.** If a step fails, you want to be able to identify that this specific step failed, understand why, and retry or fix just that step — not discover the failure three steps later as a confusing downstream symptom, and not have to redo unrelated earlier work to retry it.

## Coarse-grained versus fine-grained decomposition

Given those properties, the central tradeoff in workflow decomposition is granularity: how many steps should a given goal be broken into, and how large should each one be?

**Coarse-grained decomposition** uses fewer, larger steps — each one closer to "a whole subgoal" than "a single action." "Research the market," "draft the report," "review the report" is a three-step coarse decomposition of a research project. The advantages are real: fewer handoffs mean less coordination overhead, less risk of losing important context at a boundary, and less orchestration machinery to build and maintain. The disadvantages are also real: a coarse step is closer to an unscoped goal than a scoped task, which pushes more of the actual decomposition burden onto whatever executes that step (an agent's own judgment, in-the-moment, about how to break "research the market" into actual actions) rather than onto the workflow design itself. Coarse steps are also harder to checkpoint meaningfully — if "draft the report" is one step, there's no natural point to catch a problem until the whole draft is done, potentially far along a wrong path.

**Fine-grained decomposition** uses many smaller steps, each closer to a single well-defined action or a small cluster of closely related actions. "List available data sources," "pull pricing data from source A," "pull pricing data from source B," "reconcile pricing data across sources," "compute summary statistics," "draft the pricing section" is a fine-grained decomposition of a piece of that same research project. The advantages: each step is independently checkable and retryable, failures are easy to localize, and checkpoints (covered in depth in the next lecture) are natural to place between steps. The disadvantages: more steps means more handoffs, and every handoff is a place where relevant context has to be explicitly carried forward or it's lost — a fine-grained workflow that doesn't carefully manage what crosses each boundary can end up with each step technically succeeding while the overall result drifts, because no single step had visibility into the whole picture. Fine-grained decomposition also multiplies orchestration overhead in the same way multi-agent fan-out does: more steps generally means more model calls, more coordination logic, and more surface area for something to go wrong procedurally even when each individual step is executed correctly.

| | Coarse-grained | Fine-grained |
|---|---|---|
| Step size | Closer to a whole subgoal | Closer to a single action |
| Handoffs | Fewer, less coordination overhead | More, each one a place to lose context |
| Checkpointing | Hard to place meaningfully | Natural between every step |
| Failure localization | Harder — errors surface late | Easy — failures are isolated to a step |
| Best fit | Low uncertainty, low stakes | High uncertainty, high cost of failure |

## Finding the right grain

There's no universal correct granularity — it's a function of three things: how much uncertainty exists about how to accomplish a given piece of work, how costly a failure partway through would be, and how much benefit there is from checkpointing or reviewing progress at that point.

High uncertainty pushes toward finer granularity: if you genuinely don't know, in advance, exactly how "reconcile pricing data across sources" should be done, isolating it as its own step lets an agent bring real judgment to that specific, bounded problem, and lets you inspect the result before it feeds into the next step. Low uncertainty — a step whose execution is basically mechanical once started — can often be left coarser, or even implemented as a fixed, non-agentic step, since there's little value in agentic decision-making applied to a procedure that doesn't actually vary.

High cost of failure pushes toward finer granularity, specifically because finer steps are more checkpointable: a workflow that touches production data, sends external communications, or spends real money benefits from being broken into steps small enough that a checkpoint can sit right before the risky action, rather than bundling that action into a larger step where a checkpoint would either come too early (before you know enough to approve) or too late (after the risky action already happened).

Low value from intermediate review pushes toward coarser granularity: if there's genuinely nothing useful a checkpoint between two sub-steps would catch — because they're tightly coupled and only make sense evaluated together — merging them avoids paying handoff overhead for a review point nobody would actually use.

## Decomposition and the layers of this course

It's worth explicitly connecting workflow decomposition back to vocabulary from earlier in this module and forward to the Tool and Prompt layers, because exam scenarios tend to blend these. A decomposed step might be executed by a single agent continuing its own loop, by a dispatched subagent (in which case everything from the previous lecture about briefing and reporting applies directly to that step), or by a fixed, non-agentic function call with no model decision-making involved at all — a decomposition doesn't dictate that every step must itself be agentic. Choosing which steps warrant an agent's judgment and which are better served by a deterministic implementation is itself part of decomposition, and it's a recurring theme the Workflow layer of this course develops in much greater depth; this module's treatment is specifically about decomposition as it relates to agent and subagent orchestration, not the full workflow-design toolkit.

## A worked example

Take "migrate our billing system to the new provider" as the original coarse goal. A first pass at decomposition might yield: audit current billing data and identify edge cases, map old-provider fields to new-provider fields, write and test the migration script against a staging copy, get human sign-off on the staging results, run the migration against production during a maintenance window, verify production data post-migration, and decommission the old provider's integration. Each of those seven steps has a checkable completion condition, a defined input and output, a coherent tool/context scope, and can fail independently without corrupting the others.

```mermaid
flowchart LR
    A["Audit billing data"] --> B["Map old to new fields"]
    B --> C["Write and test migration script"]
    C --> D{"Human sign-off"}
    D --> E["Run migration in production"]
    E --> F["Verify production data"]
    F --> G["Decommission old integration"]
``` Notice, too, that the granularity isn't uniform: "map old-provider fields to new-provider fields" might reasonably be one step even though it involves many individual field mappings, because those mappings are tightly coupled and reviewed together, while "run the migration against production" is deliberately isolated as its own single step — small, high-stakes, and exactly the kind of place a human approval checkpoint belongs, which is precisely the subject of the next lecture.

## Why it matters for the exam

Expect the exam to present a goal and either a proposed decomposition or a description of a system that skipped decomposition entirely (an agent handed a large, vague goal directly), and ask you to evaluate whether the granularity is appropriate given the task's uncertainty and stakes, or to identify a step that's actually hiding two unrelated concerns. The reasoning framework here — checkable completion, defined input/output, coherent scope, independent failure — is more exam-relevant than any specific example decomposition, since scenario questions will present situations this course hasn't seen before.

<!-- prevnext:start -->

---

| [&larr; Previous: Agents as Wrappers](09-agents-as-wrappers) | [Next: Splitting When Something Changes &rarr;](11-splitting-when-something-changes) |
|:---|---:|

<!-- prevnext:end -->
