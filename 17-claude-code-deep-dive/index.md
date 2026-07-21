---
title: Claude Code Deep Dive
nav_order: 18
has_children: true
permalink: /17-claude-code-deep-dive/
---

# Claude Code Deep Dive

Goes deep on Claude Code itself as a product: its architecture and built-in tool system, the full CLAUDE.md hierarchy with precedence and conflict-resolution rules, permission modes and Plan Mode, authoring custom slash commands, Skills, and subagents, the hooks system for programmatic lifecycle control, and production usage through the Claude Agent SDK and non-interactive CI/CD invocation. Where earlier modules covered these ideas conceptually, this module tests whether you can reason precisely about the actual mechanics — which distinction matters for the exam far more than being able to name the features.

**Official docs:** [Claude Code overview](https://code.claude.com/docs/en/overview) · [Hooks](https://code.claude.com/docs/en/hooks) · [Subagents](https://code.claude.com/docs/en/sub-agents)

## Lectures

| # | Lecture | Duration |
|---|---------|----------|
| 1 | [Module Intro](01-module-intro) | 2:30 |
| 2 | [What Claude Code Is and How It Works](02-what-claude-code-is-and-how-it-works) | 14:00 |
| 3 | [The Tool System: Reads, Bash, Edits, and Web Search](03-the-tool-system-reads-bash-edits-and-web-search) | 13:00 |
| 4 | [Starting a Project: Onboarding Claude to a Codebase](04-starting-a-project-onboarding-claude-to-a-codebase) | 12:00 |
| 5 | [Claude Code Permission Modes](05-claude-code-permission-modes) | 11:00 |
| 6 | [The Three-Level CLAUDE.md Hierarchy](06-the-three-level-claude-md-hierarchy) | 13:00 |
| 7 | [Precedence Rules and Conflict Resolution](07-precedence-rules-and-conflict-resolution) | 10:00 |
| 8 | [Subdirectory CLAUDE.md Files and Scoped Instructions](08-subdirectory-claude-md-files-and-scoped-instructions) | 9:00 |
| 9 | [Excluding Files and Content](09-excluding-files-and-content) | 8:00 |
| 10 | [Common CLAUDE.md Patterns for Real Teams](10-common-claude-md-patterns-for-real-teams) | 12:00 |
| 11 | [Creating and Structuring Custom Slash Commands](11-creating-and-structuring-custom-slash-commands) | 12:00 |
| 12 | [What Skills Are and When to Build Them](12-what-skills-are-and-when-to-build-them) | 11:00 |
| 13 | [Authoring Agent Skills](13-authoring-agent-skills) | 13:00 |
| 14 | [Creating and Configuring Custom Subagents](14-creating-and-configuring-custom-subagents) | 12:00 |
| 15 | [Plan Mode: When and Why to Plan First](15-plan-mode-when-and-why-to-plan-first) | 10:00 |
| 16 | [Hook Lifecycle Events](16-hook-lifecycle-events) | 11:00 |
| 17 | [Implementing Hooks: Shell Scripts, Exit Codes, and Gotchas](17-implementing-hooks-shell-scripts-exit-codes-and-gotchas) | 13:00 |
| 18 | [Real-World Hook Applications](18-real-world-hook-applications) | 10:00 |
| 19 | [The Claude Agent SDK: Session Control](19-the-claude-agent-sdk-session-control) | 12:00 |
| 20 | [Claude Code in CI/CD: Non-Interactive Mode and the -p Flag](20-claude-code-in-ci-cd-non-interactive-mode-and-the-p-flag) | 11:00 |
| 21 | [Module Outro](21-module-outro) | 2:00 |
| 22 | [Key Distinctions - Claude Code Deep Dive](22-key-distinctions-claude-code-deep-dive) | 4:30 |
| 23 | [Quiz Prep: Claude Code Deep Dive (20 Questions)](23-quiz-prep-claude-code-deep-dive-20-questions) | — |

<!-- prevnext:start -->

---

| [&larr; Previous: Quiz Prep: MCP Server Building (20 Questions)](../16-mcp-server-building/16-quiz-prep-mcp-server-building-20-questions) | [Next: Module Intro &rarr;](01-module-intro) |
|:---|---:|

<!-- prevnext:end -->
