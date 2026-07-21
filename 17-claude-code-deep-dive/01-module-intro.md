---
title: Module Intro
parent: Claude Code Deep Dive
nav_order: 1
duration: "2:30"
---

# Module Intro
{: .no_toc }

*Video duration: 2:30*

The Workflow Layer introduced `CLAUDE.md` and Skills as concepts — standing project context that loads automatically, and packaged procedures that load on demand. Both were framed there in the abstract, as workflow-layer mechanisms alongside subagents and CI/CD gates. This module takes the one concrete product that actually implements all of that — Claude Code — and goes deep on how it really works, at the level of detail the exam expects you to reason about rather than just recognize by name.

This module builds in four movements. First, you'll ground Claude Code architecturally: what it fundamentally is (an agentic coding harness built on the same read-plan-act-verify loop from the Agentic Layer, but applied to a specific, opinionated tool set), how its built-in tools for reading, editing, running shell commands, and searching the web map onto the tool-design principles from the Tool Layer, and how a team actually onboards it to an existing codebase and governs what it's allowed to do through permission modes.

Second, you'll go deep on configuration — specifically the full `CLAUDE.md` picture the Workflow Layer only sketched: the three levels at which instructions can live, the precedence and conflict-resolution rules that govern what happens when those levels disagree, scoped subdirectory files for monorepos, mechanisms for excluding files and secrets from what Claude reads, and the concrete patterns real engineering teams actually write into these files.

Third, you'll cover Claude Code's extension mechanisms in the order a team typically adopts them: custom slash commands as the simplest unit of reuse, Skills as packaged procedures (building directly on the Workflow Layer's introduction), custom subagents for isolated delegation, Plan Mode as a safeguard for higher-risk work, and the hooks system for programmatic control over the agent's lifecycle.

Fourth, you'll move from interactive use to production: the Claude Agent SDK for building custom applications on the same agentic loop, and non-interactive (`-p`) mode for running Claude Code inside CI/CD pipelines.

By the end of this module, you should be able to look at a described Claude Code configuration or workflow and identify exactly which mechanism is doing the work, and reason about how changing one — a permission mode, a hook, a `CLAUDE.md` level — ripples into the rest of the system.

<!-- prevnext:start -->

---

| [&larr; Previous: Claude Code Deep Dive](./) | [Next: What Claude Code Is and How It Works &rarr;](02-what-claude-code-is-and-how-it-works) |
|:---|---:|

<!-- prevnext:end -->
