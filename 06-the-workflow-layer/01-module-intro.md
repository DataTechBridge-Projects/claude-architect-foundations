---
title: Module Intro
parent: The Workflow Layer
nav_order: 1
duration: "2:32"
---

# Module Intro
{: .no_toc }

*Video duration: 2:32*

The Agentic layer answered *who decides what happens next* — one agent, a fixed pipeline, or a coordinated team. The Tool layer answered *what the model can actually do*. This module, the Workflow layer, sits a level deeper than either of those: it governs how the work inside whatever architecture you chose gets configured, organized, and made repeatable across a team, a codebase, and time.

That distinction matters more than it sounds like it should. Agentic architecture is a decision you make once, at design time, about the shape of a system. Workflow design is the ongoing discipline of *feeding* that system — making sure every agent, every session, every run starts from the same shared understanding of conventions, constraints, and available procedures, without someone having to retype that understanding into a prompt every single time. It's the difference between deciding a system will have an agent, and deciding how that agent reliably knows your team indents with two spaces, never touches the `payments/` directory without a human sign-off, and has a ready-made procedure for generating a changelog entry.

This module covers four things that all live at that configuration-and-process layer. First, `CLAUDE.md` files — the persistent, auto-loaded project context that shapes Claude Code's behavior without being restated in every prompt. Second, Skills — packaged, invokable procedures that bundle instructions (and sometimes tools or scripts) for a well-defined sub-task, and how they differ from both a `CLAUDE.md` entry and a full subagent. Third, workflow design itself: sequencing steps, deciding which are deterministic and which are left to agentic judgment, and recognizing which conventions are genuinely local rather than universal. Fourth, CI/CD integration — how Claude-driven steps like code review or test generation stop being ad hoc and become auditable, automated participants in a real delivery pipeline.

By the end of this module, you should be able to look at a piece of context or a repeatable procedure and know exactly where it belongs: in a system prompt, a `CLAUDE.md` file, a Skill, a CI job, or nowhere at all.

<!-- prevnext:start -->

---

| [&larr; Previous: The Workflow Layer](./) | [Next: Configuring CLAUDE.md &rarr;](02-configuring-claude-md) |
|:---|---:|

<!-- prevnext:end -->
