---
title: Key Distinctions - Claude Code Deep Dive
parent: Claude Code Deep Dive
nav_order: 22
duration: "4:30"
---

# Key Distinctions - Claude Code Deep Dive
{: .no_toc }

*Read duration: 4:30*

A condensed pass through the module's highest-yield either/or distinctions — the kind of pairs the exam likes to test directly against a scenario.

**Advisory versus enforced.** `CLAUDE.md` content, however clearly worded, shapes what Claude *tries* to do — it's context, not configuration. Permission rules, sandboxing, and hooks are enforced by the harness, mechanically, regardless of what the model decides. Any scenario asking "is this guaranteed" turns on this line first.

**CLAUDE.md concatenation versus settings override.** The three-level `CLAUDE.md` hierarchy loads every discovered file simultaneously — nothing is erased, only ordered. Settings (managed, command-line, local, project, user) resolve the opposite way: a genuine, deterministic override, where a higher layer wins outright and a lower one simply doesn't apply.

**Root-loaded versus on-demand.** Ancestor `CLAUDE.md` files load in full at session start, costing context every turn. Subdirectory `CLAUDE.md` files load lazily, only when Claude actually works with files in that part of the tree — the mechanism that makes monorepo-scale scoping affordable.

**Deny, then ask, then allow.** Permission rules resolve in a fixed order regardless of specificity: a broad deny beats a narrow allow every time. A rule set isn't "the most specific rule wins" — it's "the most restrictive category wins."

**Default versus Plan Mode.** Both gate before an edit happens, but at different grain: default mode gates each action one at a time; Plan Mode gates an entire proposed approach up front, before any of it is committed — valuable when the approach is uncertain or the stakes are high, pure overhead when neither is true.

**Accept-edits versus bypass-permissions.** Accept-edits still requires approval for arbitrary shell commands and network access — only file edits and safe filesystem operations run free. Bypass-permissions skips nearly everything, and is appropriate only inside a disposable, isolated environment, never on a machine with real credentials or real production access.

**Permission rules versus sandboxing.** A deny rule stops Claude from *attempting* an action through its own tools. Sandboxing stops an action from *succeeding* at the OS level, even via a subprocess that bypasses Claude Code's tools entirely — complementary layers, not redundant ones.

**Slash command versus Skill.** A slash command is a saved prompt, invoked explicitly by name, with no bundled resources. A Skill is a richer package — bundled scripts, on-demand reference material, and (depending on configuration) automatic discovery based on matching a task's description, not just explicit invocation.

**Skill versus subagent.** A Skill loads its instructions into the *same* context the calling agent is already using. A subagent runs in a genuinely separate context and its own model call, returning only a summary — the sharpest test is whether the sub-task's intermediate noise should reach the parent at all.

**PreToolUse versus PostToolUse.** A hook attached before a tool call can genuinely prevent it (exit code 2 stops the action). A hook attached after a tool call can only react to what already happened — it cannot undo it.

**Interactive versus non-interactive (`-p`) mode.** Interactive sessions assume a human present to answer permission prompts. Non-interactive invocations must be configured in advance with everything they need to complete unattended, since there's no one present to ask.

**Agent SDK versus a raw model API call.** A raw completion gives you one response to one request. The Agent SDK gives you the same loop, tool set, and context management Claude Code itself runs on, callable from your own code.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Outro](21-module-outro) | [Next: Quiz Prep: Claude Code Deep Dive (20 Questions) &rarr;](23-quiz-prep-claude-code-deep-dive-20-questions) |
|:---|---:|

<!-- prevnext:end -->
