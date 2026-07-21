---
title: Creating and Configuring Custom Subagents
parent: Claude Code Deep Dive
nav_order: 14
duration: "12:00"
---

# Creating and Configuring Custom Subagents
{: .no_toc }

*Video duration: 12:00*

The Agentic Layer covered subagents as an architectural concept: delegation from an orchestrator to a separately-executing unit of work, with its own context window and its own model call, used when a sub-task's exploratory noise shouldn't pollute the parent's context or when the sub-task genuinely benefits from independent judgment. Claude Code implements exactly that pattern as a concrete, configurable feature, and this lecture covers how a team actually defines and scopes a custom one.

## Why a Custom Subagent, Rather Than the Default

Claude Code can spawn a general-purpose subagent for an exploratory task without any custom configuration at all — hand it a description of what needs investigating, and it runs independently, in its own context, returning only a summary. A **custom** subagent exists for the case where a team keeps spawning the *same kind* of worker, with the *same* instructions, often enough that redefining its behavior from scratch each time is wasted repetition — exactly the same "this keeps getting retyped" signal that justified building a Skill in the earlier lecture, applied here to a delegated unit of work instead of an in-context procedure.

A custom subagent defined once for, say, "review a diff for security issues" or "investigate a failing CI run and summarize the likely cause" becomes a named, reusable delegate: the parent session can hand it a task by name, trusting that its scoped tools, its focused system prompt, and its narrowed context are already configured correctly, rather than having to specify all of that inline every time.

## What a Custom Subagent Definition Contains

A custom subagent is defined similarly to a Skill in spirit — a file with frontmatter and a body — but the frontmatter answers a different set of questions, because a subagent isn't just loading instructions into the current context; it's spinning up an independently-executing unit of work with its own boundaries:

- **A name and a description.** The description plays the same discovery role a Skill's or a tool's description plays: the orchestrating session uses it to decide whether a given task should be delegated to this subagent rather than another one, or handled inline. The same discipline applies — precise and distinguishing beats vague and broad, for exactly the reason the Tool Layer's tool-selection material already established.
- **A system prompt, scoped to this subagent's one job.** Rather than inheriting the parent session's full system prompt and accumulated context, a subagent typically operates with a focused prompt describing exactly what it's meant to do and how — the security-reviewer subagent's prompt describes what to look for and how to report findings; it doesn't need to know anything else about the broader session that spawned it.
- **A restricted tool set.** A subagent can be scoped to only the tools its job actually requires — a subagent meant purely to investigate and report shouldn't necessarily have file-editing tools available at all, the same "narrow the capability to match the actual need" discipline the Tool Layer applies to tool-set design generally, applied here to which tools a *specific delegate* gets rather than which tools exist in a tool set at all.
- **Optionally, a specific model.** Because a subagent runs its own independent model call, it doesn't have to use the same model the parent session is running. A lightweight, well-defined, high-volume task (summarizing log output, say) can be routed to a smaller, faster, cheaper model, while the orchestrating session keeps using a more capable one for the harder judgment calls that actually require it — a cost and latency lever that doesn't exist for Skills, since a Skill executes inline in the calling agent's own model call rather than making a separate one.

## Context Isolation in Practice

The defining property that makes a subagent a subagent, rather than a fancier Skill, is genuine context isolation: a subagent's exploration — every file it reads, every command it runs, every dead end it investigates along the way — happens in its own context window, and only the subagent's final result comes back to the parent session. A parent that delegates "investigate why this test suite is flaky" to a subagent gets back a concise summary of the cause, not a transcript of every log file the subagent had to read and discard to get there. This is precisely the mechanism the Workflow Layer's Skill lecture pointed to as the sharpest line between a Skill and a subagent: a Skill's content lands in the *same* context the parent is already using; a subagent's exploration never does, only its answer does.

That isolation is also what makes a subagent the right choice when a task's intermediate noise would actively hurt the parent session if it accumulated there — a large, exploratory research task that touches dozens of files is exactly the case where delegating protects the parent's context budget for the actual decision-making that follows, rather than spending it on the exploration needed to reach that decision.

## When to Delegate, Concretely

Bringing this back to the Agentic Layer's framing of subagent tradeoffs — delegation isn't free, and spinning one up is a real architectural decision, not a reflexive move for anything that feels complicated. A custom subagent is the right call when a task genuinely needs isolated exploration whose noise shouldn't reach the parent, when the same kind of delegated work recurs often enough to justify defining it once, or when routing to a cheaper or faster model for a well-bounded sub-task is worth the coordination overhead. It's the wrong call for anything that's simple enough to do inline, for a one-off task that will never recur in the same shape, or for a case where the parent session actually *needs* to see the intermediate work rather than just the final answer — delegating in that last case doesn't just add overhead, it actively removes information the parent needed.

## Personal vs. Project Scope

Like slash commands and `CLAUDE.md`, a custom subagent definition can live at more than one scope, and the same reasoning about ownership applies. A subagent defined in a personal configuration directory is available to one person, across every project they work in — a good fit for a personal workflow habit, like a subagent the individual always spins up to double-check their own commits before pushing. A subagent defined inside a project's repository is shared with the whole team the moment it's committed, which is the right home for anything the team collectively relies on — a shared code-review delegate, a shared investigation-and-summarize worker for CI failures — since every team member gets the identical, reviewed definition rather than each person maintaining a slightly different personal version of the same idea.

## A Worked Comparison, Extended

Recall the Workflow Layer's three-way example: a commit-message convention (a `CLAUDE.md` entry), a compliance-report procedure (a Skill), and an overnight batch review of pull requests (a subagent, or in this case, likely several instances of one). A custom subagent for that last case would be defined with a description ("review a single pull request for style and correctness issues, and produce a short summary"), a system prompt describing exactly what "review" means for this team, a tool set limited to reading code and posting a comment (not one that includes broad shell access or the ability to merge anything), and, plausibly, a smaller model, since reviewing one PR at a time is well-bounded, high-volume work rather than a task calling for the deepest available reasoning.

**Why it matters for the exam:** Expect questions asking you to configure or evaluate a described subagent definition — is the tool scope appropriate for its stated job, does the model choice match the task's actual complexity, is context isolation actually buying something here or just adding overhead. The recurring test is whether a described delegation decision is genuinely justified by isolation needs and recurrence, the same judgment call the Agentic Layer introduced and this lecture makes concrete.
