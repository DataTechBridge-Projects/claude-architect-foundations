---
title: Claude Code Permission Modes
parent: Claude Code Deep Dive
nav_order: 5
duration: "11:00"
---

# Claude Code Permission Modes
{: .no_toc }

*Video duration: 11:00*

Every action Claude Code takes that could change something real — editing a file, running a shell command, reaching the network — passes through a permission check before it executes. A **permission mode** is the setting that determines how that check behaves: how often Claude pauses to ask for explicit approval versus how much it's trusted to proceed on its own. This is the workflow-safeguarding concept from the Agentic Layer — checkpoints and approval gates bounding autonomy — made completely concrete: an actual, switchable setting you configure per session or per project, rather than an abstract design principle.

## The Core Tradeoff

Every permission mode sits somewhere on a single axis: oversight versus velocity. More oversight means Claude pauses more often, which slows down a session but bounds the damage any single wrong decision can do, since a human reviews the action before it happens. More velocity means Claude proceeds through longer stretches of work uninterrupted, which is faster and less tedious but means more can go wrong before anyone notices, since approval happens after the fact (if at all) rather than before.

Neither end of that axis is "correct" in the abstract — the right mode is a function of the task's risk profile and how much the operator already trusts Claude Code's behavior on this specific codebase. A greenfield prototype nobody else depends on tolerates far more autonomy than a production billing service; a session run by an engineer who has watched Claude Code work on this exact repository for months tolerates more autonomy than one run by someone using it for the first time. This is precisely the calibration the previous lecture's onboarding advice was building toward: permission mode is the lever that operationalizes "how much has this specific setup earned my trust so far."

## The Modes

**Default (manual) mode.** The starting point and the most conservative setting: Claude can read files freely, but every file edit and every shell command (other than a small built-in set of read-only commands like listing a directory or checking `git status`, which can't mutate anything and so don't need a prompt) pauses for explicit approval before it runs. This is the right default for sensitive work, for a codebase Claude Code hasn't operated in before, and for any session where the cost of an unwanted action is high relative to the friction of approving good ones.

**Plan mode.** A variant of the conservative end that shifts *when* the approval happens rather than removing it: Claude explores the codebase, reasons about an approach, and writes up a plan — but makes no edits at all until that plan itself is approved. Where default mode approves actions one at a time as they come up, plan mode approves an entire intended sequence of actions up front, then executes with far less interruption once that plan is accepted. This is valuable specifically when the *approach* is what's uncertain — a genuinely ambiguous task, or a change spanning enough files that reviewing edits one at a time loses the forest for the trees — and it gets a full lecture of its own later in this module because of how directly useful (and how easy to under-use) it is.

**Accept-edits mode.** A meaningfully looser setting: file edits and common, low-risk filesystem operations (creating a directory, moving a file, and similar operations scoped to the working directory) proceed without a prompt, while everything else — arbitrary shell commands, network access — still requires approval. This trades a real amount of oversight for a real amount of velocity, and the intended usage pattern reflects that tradeoff explicitly: instead of approving each edit inline, you review the accumulated changes afterward, typically with a diff tool, the same way you'd review a colleague's pull request rather than watching them type it live. It's a good fit for iterating on code you're actively reviewing anyway, where the diff review *is* the safeguard, just moved to a different point in the workflow.

**Bypass-permissions mode.** The far end of the velocity axis: essentially every check is skipped, and actions execute immediately with no prompts at all. This mode is explicitly *not* meant for use on a developer's primary machine against a real, connected environment — its intended use is an isolated, disposable environment (a container or VM with no meaningful ability to cause external damage) where the blast radius of any single wrong action is contained by the environment itself rather than by the permission layer. Reaching for this mode outside that context — on a laptop with real credentials, a real production database connection, or real customer data reachable from the working directory — trades away the entire safety model for convenience, which is a bad trade in almost every realistic scenario. Even in this mode, a small set of protected paths (a project's own git internals and Claude Code's own configuration directory, among others) and a few especially destructive command patterns still prompt, as a last-resort circuit breaker against catastrophic model error rather than a meaningful safety boundary on their own.

| Mode | Oversight level | What proceeds without a prompt | Best fit |
|---|---|---|---|
| Default (manual) | Highest | Built-in read-only commands only | Sensitive work, unfamiliar codebase |
| Plan mode | High, shifted to plan review | Exploration and planning; no edits | Ambiguous approach or high-stakes change |
| Accept-edits | Medium | File edits, safe filesystem operations | Actively-reviewed iteration |
| Bypass-permissions | Lowest | Nearly everything | Isolated, disposable environments only |

## Switching Modes, and Why the Choice Belongs to a Human

Permission mode is not something Claude decides for itself mid-task — it's set by the person running the session (interactively, by cycling through modes, or as a persistent default in project or personal configuration) or, for organizations, by policy that individual settings can't override. That's a deliberate design choice, not an incidental one: if the model that's being supervised could also decide how closely it's supervised, the entire premise of a permission layer as an external safeguard collapses. The oversight mechanism has to sit outside the thing being overseen to actually function as oversight — the same principle that shows up in the Agentic Layer's treatment of checkpoints and approval gates generally, and one of the more conceptually important, easy-to-miss details in this whole topic.

## Connecting Back to Workflow Safeguarding

The Agentic Layer framed workflow safeguarding around checkpoints, approval gates, and rollback strategies as general mechanisms for keeping autonomy bounded. Permission modes are the most direct, most literal instantiation of "approval gates" this course covers: a configurable point in the loop where execution pauses until a human signs off, with the mode setting controlling how many gates exist and how much passes through them without a pause. Rollback strategies show up elsewhere in Claude Code (session history, the ability to review and revert changes after the fact) but the gate itself — the thing that decides whether an action even happens before a human weighs in — is exactly what permission modes are.

**Why it matters for the exam:** Expect scenario questions that describe a task's risk profile and a team's familiarity with a codebase, then ask which permission mode fits. The reliable reasoning path is the oversight-versus-velocity axis: how costly is an unwanted action here, and how much has this specific setup already earned autonomy. Also expect at least one question probing *why* mode selection belongs to a human rather than the model — that's a conceptually loaded distinction the exam likes to isolate on its own.

<!-- prevnext:start -->

---

| [&larr; Previous: Starting a Project: Onboarding Claude to a Codebase](04-starting-a-project-onboarding-claude-to-a-codebase) | [Next: The Three-Level CLAUDE.md Hierarchy &rarr;](06-the-three-level-claude-md-hierarchy) |
|:---|---:|

<!-- prevnext:end -->
