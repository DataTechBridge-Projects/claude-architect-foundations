---
title: "Quiz Prep: Claude Code Deep Dive (20 Questions)"
parent: Claude Code Deep Dive
nav_order: 23
---

# Quiz Prep: Claude Code Deep Dive (20 Questions)
{: .no_toc }

Twenty practice questions covering the Claude Code Deep Dive module: what Claude Code fundamentally is, its built-in tool system, onboarding practices, permission modes, the CLAUDE.md hierarchy and precedence rules, exclusion mechanisms, extension mechanisms (slash commands, Skills, subagents, Plan Mode), the hooks system, the Agent SDK, and CI/CD usage. Work through all twenty before checking the Answer Key at the end.

**1.** Which statement best describes what Claude Code adds on top of the underlying model?
A. A fundamentally different, more capable version of the model itself.
B. A harness — a curated tool set, automatic context assembly, and a permission layer — around the same underlying agentic-loop pattern from the Agentic Layer.
C. Nothing; Claude Code is purely a different visual interface with no architectural difference from a raw API call.
D. A replacement for the agentic loop, using a different execution mechanism entirely.

**2.** Why does Claude Code generally require a file to have been read in the current session before it can be edited?
A. To reduce the total number of available tools.
B. To prevent a blind edit made against stale or unverified content, reducing the odds of a confidently wrong change.
C. Because the Edit tool has no other way to determine file encoding.
D. It is a legacy restriction with no remaining design rationale.

**3.** What is the operational difference between `WebFetch` and `WebSearch`?
A. They are two names for the exact same underlying capability.
B. `WebFetch` retrieves content from a specific, already-known URL; `WebSearch` runs a query and returns candidate results to consider fetching next.
C. `WebSearch` can only be used inside Plan Mode.
D. `WebFetch` is deprecated in favor of `WebSearch`.

**4.** A team is introducing Claude Code to a large, unfamiliar production codebase for the first time. Per this module, what's the recommended sequencing?
A. Enable the most permissive permission mode immediately to maximize velocity from day one.
B. Start with small, independently verifiable tasks and a conservative permission mode, earning more autonomy as trust in the specific setup builds.
C. Configure every extension mechanism (Skills, subagents, hooks) before running a single real task.
D. Skip reviewing any auto-generated CLAUDE.md content, since it's assumed accurate.

**5.** In Plan Mode, what is Claude permitted to do before a plan is approved?
A. Nothing at all, including reading files.
B. Read files and run read-only exploratory commands, but make no edits until the plan is approved.
C. Make edits freely as long as they're reverted if the plan is rejected.
D. Only run commands that require no permission check in any mode.

**6.** Which permission mode is appropriate only inside an isolated, disposable environment with no meaningful ability to affect external systems?
A. Default (manual) mode.
B. Accept-edits mode.
C. Bypass-permissions mode.
D. Plan Mode.

**7.** Why does accept-edits mode still require approval for arbitrary shell commands and network access, even though it auto-approves file edits?
A. Because file edits are actually more dangerous than shell commands.
B. Because the mode's design specifically trades away oversight only for the lower-risk category (edits within the working directory) while preserving it for higher-risk, broader-reaching actions.
C. It doesn't — accept-edits auto-approves everything.
D. Because shell commands are technically impossible to auto-approve in any mode.

**8.** What is the single most important fact about how the three-level CLAUDE.md hierarchy (user, project, subdirectory) combines?
A. A higher level always silently overrides a lower one.
B. Only the most specific file's content is ever loaded into context.
C. All discovered levels' content is concatenated into context simultaneously; none of them erase another.
D. Each level requires a separate, manually triggered command to load.

**9.** A user-level CLAUDE.md instruction and a project-level CLAUDE.md instruction genuinely contradict each other. What does this module say actually happens?
A. Claude Code automatically detects the contradiction and silently discards the user-level instruction.
B. Both instructions are present in context; there's no harness-level arbitration, so the responsibility for resolving the contradiction falls on whoever maintains the files, not on the system.
C. The session fails to start until the contradiction is manually resolved.
D. The most recently edited file always wins automatically.

**10.** How does settings precedence (managed, command-line, local, project, user) differ fundamentally from how CLAUDE.md levels combine?
A. It doesn't differ; both work identically.
B. Settings resolve through genuine override — one level wins outright and a conflicting lower level simply doesn't apply — while CLAUDE.md levels concatenate rather than override.
C. Settings never have any precedence order at all.
D. CLAUDE.md levels always override settings.

**11.** Why do subdirectory CLAUDE.md files load differently from root-level ones?
A. They don't — all CLAUDE.md files load identically at session start.
B. Subdirectory files load on demand, only when Claude works with files in that part of the tree, keeping monorepo-scale scoping affordable rather than bloating every session's standing context.
C. Subdirectory files require a paid feature tier.
D. Subdirectory files only apply to test code.

**12.** What is the primary mechanism for excluding a genuinely sensitive file (such as a credentials file) from what Claude Code can read?
A. A strongly worded CLAUDE.md instruction telling Claude never to read it.
B. A permission deny rule scoped to the Read tool for that path, enforced by the harness before any tool result is returned.
C. Renaming the file to something Claude wouldn't recognize.
D. There is no way to exclude any file from being read.

**13.** Which of the following is the strongest-written, most actionable CLAUDE.md constraint, per this module's guidance?
A. "We try to be careful about database changes."
B. "Migrations are one-way. Never edit an already-applied migration file; write a new migration instead."
C. "Be thoughtful when working with data."
D. "Database work requires good judgment."

**14.** What distinguishes a custom slash command from a Skill?
A. A slash command is a saved prompt invoked explicitly by name with no bundled resources; a Skill can bundle scripts and reference material and, depending on configuration, be discovered automatically based on matching a task's description.
B. Slash commands can only be used by one person; Skills are always team-shared.
C. There is no meaningful difference between the two mechanisms.
D. Skills cannot accept any form of parameterization, while slash commands always can.

**15.** Per this module, what's the key signal that a recurring procedure has crossed the threshold where building a Skill is worth it over continuing ad hoc prompting?
A. The procedure has only ever come up once so far.
B. The same multi-step instruction keeps getting retyped nearly verbatim, has real internal structure, and recurs for more than one occasion or one person.
C. The procedure is expected to change completely every single week.
D. No one besides its author will ever need it again.

**16.** In a Skill's file structure, what is the primary job of its frontmatter description?
A. It has no functional purpose beyond documentation.
B. It's the lightweight discovery signal used to decide whether a given task should trigger loading this Skill's full instructions, playing a role similar to a tool's name and description.
C. It replaces the need for any instructions in the Skill's body.
D. It sets the model's temperature for the session.

**17.** What is the defining property that distinguishes a subagent from a Skill?
A. Subagents are always slower and are therefore never worth using.
B. A subagent runs in its own separate context window with its own model call, returning only a result to the parent; a Skill's content loads into the same context the calling agent is already using.
C. Skills always require their own dedicated model.
D. There is no real difference — the two terms describe the same mechanism.

**18.** A PreToolUse hook exits with code 2. What happens?
A. Nothing; exit code 2 is treated identically to exit code 0.
B. The tool call is blocked before it executes, and the hook's stderr output is fed back as the reason.
C. The action is executed and then silently logged.
D. Claude Code crashes and the session ends immediately.

**19.** Why can't a PostToolUse hook that exits with code 2 undo the action it's reacting to?
A. Exit code 2 is not supported on PostToolUse hooks at all.
B. Because the tool call has already completed by the time a PostToolUse hook runs — it can only react to and object to what already happened, not prevent it retroactively.
C. PostToolUse hooks always run before the tool call, identical to PreToolUse.
D. PostToolUse hooks cannot receive any information about what occurred.

**20.** What capability does the Claude Agent SDK provide beyond a raw call to the underlying model API?
A. Nothing; the SDK is functionally identical to a single completion request.
B. The same agentic loop, built-in tool set, and context-management machinery that powers Claude Code itself, callable from custom application code, including session control across turns.
C. A guarantee that no permission logic is ever needed.
D. A completely different, incompatible model architecture.

## Answer Key

**1. B** — Claude Code is the general agentic-loop pattern instantiated as a specific harness: curated tools, automatic context assembly, and a permission layer wrapped around the same underlying model.

**2. B** — Read-before-edit exists specifically to prevent an edit made against content the model hasn't actually verified as current.

**3. B** — `WebFetch` needs a known URL and retrieves its content; `WebSearch` runs a query to discover candidate URLs worth fetching next.

**4. B** — Onboarding should start narrow and conservative, earning broader autonomy and more extension mechanisms as trust in the specific setup is actually demonstrated.

**5. B** — Plan Mode allows free exploration and reading but blocks all edits until the proposed plan itself is explicitly approved.

**6. C** — Bypass-permissions mode skips nearly all checks and is safe only where the environment itself contains any resulting damage.

**7. B** — Accept-edits deliberately trades oversight away only for the lower-risk category (in-directory file edits and safe filesystem operations), while still gating higher-risk shell and network actions.

**8. C** — All discovered CLAUDE.md files are concatenated into context; the hierarchy is about ordering and specificity, not elimination.

**9. B** — There is no automatic arbitration; both instructions remain present, and resolving a genuine contradiction is the maintainers' responsibility, not the system's.

**10. B** — Settings resolve through genuine, deterministic override (one layer wins outright); CLAUDE.md levels combine additively with no layer erasing another.

**11. B** — Lazy, on-demand loading for subdirectory files is what keeps per-package or per-team specificity affordable at monorepo scale.

**12. B** — A permission deny rule scoped to the Read tool is enforced by the harness itself, unlike a CLAUDE.md instruction, which is only advisory.

**13. B** — It's specific, checkable, and states exactly what must never happen — the actionable-versus-descriptive distinction this module emphasizes.

**14. A** — A slash command is a simple, explicitly-invoked saved prompt; a Skill is a richer package with bundled resources and potential automatic discovery.

**15. B** — Repetition, real internal structure, and recurrence across time or people are the concrete signals that justify the fixed cost of packaging a Skill.

**16. B** — The description is the cheap, always-visible signal used for discovery before a Skill's full instructions are ever loaded.

**17. B** — Genuine context isolation — a separate context window and its own model call — is what makes a subagent a subagent rather than a more elaborate Skill.

**18. B** — Exit code 2 on a blocking-capable event like PreToolUse stops the action outright, with stderr surfaced as the reason.

**19. B** — The action has already completed by the time a PostToolUse hook fires, so exit code 2 there can only register an objection, not reverse what happened.

**20. B** — The Agent SDK exposes Claude Code's actual engine — loop, tools, and context management, including session control — for use inside custom programmatic applications.
