---
title: "Claude Code in CI/CD: Non-Interactive Mode and the -p Flag"
parent: Claude Code Deep Dive
nav_order: 20
duration: "11:00"
---

# Claude Code in CI/CD: Non-Interactive Mode and the -p Flag
{: .no_toc }

*Video duration: 11:00*

The Agent SDK lecture covered building custom applications on Claude Code's underlying engine. This lecture covers a simpler, more immediately practical path to the same territory: running Claude Code itself, non-interactively, inside a pipeline — no custom application required, just a single flag away from the interactive tool you already know.

## Interactive vs. Non-Interactive, Mechanically

Ordinary Claude Code use is interactive: a human types a prompt, a conversation unfolds over multiple turns, and permission prompts pause execution waiting for a person to respond. None of that shape fits a CI/CD pipeline, which runs unattended, on a schedule or a trigger, with no human present to approve anything or read a scrolling terminal transcript. **Non-interactive (or "print") mode** — invoked with a specific flag alongside a prompt given directly on the command line — reframes the whole interaction as a single request-and-response: give Claude Code an instruction, let it work (reading, editing, running commands, exactly as it would interactively), and get a final result back, with no back-and-forth and no assumption that anyone is watching it happen in real time.

This is a genuinely different operating mode, not just interactive Claude Code with the human edited out of the loop. Because there's no one to answer a permission prompt, a non-interactive invocation has to be configured up front with everything it needs to actually complete the task without pausing to ask — which tools are pre-approved, and, in some cases, an entire permission-mode setting appropriate for unattended use. An invocation that hits a permission prompt with nobody present to answer it simply stalls (or, in some configurations, aborts the run outright) — which makes deciding, in advance, exactly what this specific automated task is allowed to do without asking, the single most important piece of configuring it correctly.

## What a Pipeline Invocation Looks Like

A representative non-interactive invocation reads like an ordinary shell command: the flag, a prompt describing the task, and a list of tools that are pre-approved for this specific run — for example, letting the run read files, edit them, and execute shell commands, without needing any of those individually confirmed. The output can come back as plain text (fine for a human skimming a CI log afterward) or as structured output — machine-readable JSON carrying the result alongside metadata like cost and session information, which is what a pipeline actually needs if a later step in the same workflow is going to act on the result programmatically rather than just display it.

Non-interactive mode also composes naturally with ordinary shell piping, since it reads from standard input the same way any other command-line tool does — piping a diff, a log file, or a test failure directly into a non-interactive invocation, and having Claude Code process that piped content as part of the task, is a common and natural pattern precisely because non-interactive mode was built to behave like a normal Unix-style command-line tool rather than a bespoke API.

## What This Enables

**Automated code review.** A pipeline step that pipes a pull request's diff into a non-interactive invocation, with an instruction to review specifically for security issues, style violations, or missing test coverage, gives a team a consistent, always-run first pass before human review — not a replacement for human judgment, but a way to make sure a specific, well-defined category of check runs on every single change, every time, without depending on a human reviewer remembering to look for it manually.

**Automated fixes.** Beyond review, a non-interactive invocation can be given permission to actually make changes — fixing lint violations, updating a dependency, applying a mechanical refactor — and either commit the result directly or open it as a proposed change for a human to review, depending on how much autonomy the specific task has earned. This is squarely a permission-mode decision transplanted into a pipeline context: a low-stakes, well-understood automated fix might run with edits pre-approved and commit directly, while a higher-stakes automated change should still land as something a human reviews before it merges, exactly the same oversight-versus-velocity reasoning from the permission-modes lecture, just applied to an unattended context instead of an interactive one.

**Multi-step automated workflows.** Because a non-interactive session can be continued or resumed rather than only ever starting fresh, a pipeline can chain several non-interactive invocations together — one step producing a review, a later step acting on specific findings from that review — carrying context forward across steps rather than starting from zero each time, the same session-control concept the Agent SDK lecture introduced, available here without needing to write any custom application code at all.

## A Reproducibility Consideration

A detail worth flagging for anyone actually configuring a pipeline invocation: an ordinary non-interactive run still picks up whatever standing configuration happens to exist in the environment it runs in — a `CLAUDE.md`, project settings, locally configured integrations — the same as an interactive session would. For a human's own ad hoc use, that's usually exactly the desired behavior. For a CI pipeline specifically, where the whole point is a predictable, identical outcome on every run regardless of which machine happened to execute it, that same automatic pickup can be a liability — a pipeline run's behavior shouldn't depend on incidental configuration that happens to exist on whichever runner executed it that day. Where a pipeline needs a strictly reproducible run, an option to skip the normal auto-discovery of local configuration and rely only on flags passed explicitly to that specific invocation exists precisely to close this gap — trading the convenience of ambient configuration for the guarantee that the same command produces the same behavior everywhere it runs.

## What's Different from Interactive Use, and Why It Matters

The core tradeoff non-interactive mode makes is the same oversight-versus-velocity axis that runs through this entire module, pushed to an extreme: essentially no live oversight, in exchange for the ability to run entirely unattended, on a schedule or a trigger, at whatever scale a pipeline needs. That tradeoff is exactly right for a well-scoped, well-tested automated task — a specific check, a specific class of fix — and exactly wrong for an ambiguous, high-stakes task that genuinely benefits from a human in the loop reacting to what Claude Code discovers along the way. A team's job in adopting non-interactive mode well is drawing that line correctly: which tasks are well-defined and low-risk enough to run unattended, and which still need the interactive product's ability to pause and ask.

This is also why non-interactive mode composes so naturally with everything else this module has covered rather than replacing it: a `CLAUDE.md` still loads (or can be deliberately skipped for a faster, more predictable pipeline run), hooks still fire at the same lifecycle events, and permission rules still govern what's allowed — non-interactive mode changes *who's answering* when a decision point is reached, not whether those decision points exist at all.

**Why it matters for the exam:** Expect questions distinguishing what changes and what doesn't between interactive and non-interactive use, and questions asking you to reason about whether a described automated task is appropriately scoped for unattended execution or actually needs a human in the loop. Also expect a question or two on the concrete CI/CD use cases — automated review and automated fixes — and on why pre-approving specific tools matters more in this mode than in interactive use, where a human can simply answer a prompt as it comes up.
