---
title: Precedence Rules and Conflict Resolution
parent: Claude Code Deep Dive
nav_order: 7
duration: "10:00"
---

# Precedence Rules and Conflict Resolution
{: .no_toc }

*Video duration: 10:00*

The previous lecture established the single most important fact about the `CLAUDE.md` hierarchy: its three levels concatenate rather than override. That raises an obvious follow-up question this lecture answers directly — if nothing gets silently erased, what actually happens when two loaded instructions genuinely contradict each other? And, just as importantly, is that the same mechanism Claude Code uses everywhere else it layers configuration? It isn't, and the contrast between the two models is one of the more conceptually rich distinctions in this whole module.

## When CLAUDE.md Content Actually Conflicts

Suppose a project-root `CLAUDE.md` says "always use the `Result` wrapper type instead of throwing exceptions," and a subdirectory `CLAUDE.md` in a legacy corner of the same codebase says "this package still uses exceptions; don't introduce the `Result` type here, it isn't supported by the surrounding tooling." Both files load, both are present in context, and — critically — there is no harness-level mechanism that detects this contradiction and picks a winner for you. `CLAUDE.md` content is delivered to the model as context to reason over, not as enforced configuration the way a permission rule is; nothing structurally guarantees compliance with any single instruction, let alone arbitrates between two that disagree.

In practice, a well-behaved model tends to weight instructions that are more specific to the immediate task more heavily than broad, general ones — the subdirectory file's exception-handling guidance, being more narrowly scoped to exactly the code being touched, usually wins out over the root file's general rule in the model's own reasoning. But "usually wins in practice" is a very different guarantee from "is guaranteed to win by the system," and treating the former as the latter is exactly the mistake this lecture exists to head off. If two instructions can genuinely contradict, the correct fix is not to trust the model to resolve it the same way every time — it's to **remove the contradiction at the source**: scope the general instruction more precisely ("use `Result` everywhere except `legacy/`"), or make the exception explicit in the general file itself, so there's nothing left for the model to arbitrate between turn to turn.

This is the practical payoff of understanding the concatenation model correctly: because nothing overrides anything else, the responsibility for resolving conflicts sits with whoever maintains these files, not with the harness. A `CLAUDE.md` hierarchy with unresolved contradictions is a maintenance debt, not a system that will eventually sort itself out.

## A Different Model, Right Next Door: Settings Precedence

`CLAUDE.md` content is not the only thing layered across multiple scopes in Claude Code — settings (permission rules, default modes, and other configuration) are layered too, at levels that closely mirror the `CLAUDE.md` hierarchy: personal settings, project-shared settings, a personal-only local override on top of a project, and, for organizations, a centrally managed policy layer above all of it. But settings resolve through **genuine override precedence**, not concatenation: when two levels specify a conflicting value for the same setting, one wins outright and the other is simply not applied, in a fixed order from most to least authoritative — centrally managed policy first (and unconditionally unbeatable by anything below it, including flags passed on the command line), then command-line arguments for that specific invocation, then a personal local override scoped to one project, then settings shared with the whole team for that project, and finally personal settings that apply everywhere.

| Rank (highest to lowest) | Layer |
|---|---|
| 1 | Centrally managed policy |
| 2 | Command-line arguments |
| 3 | Personal local override (scoped to one project) |
| 4 | Project-shared settings |
| 5 | Personal settings (all projects) |

This is a meaningfully different resolution mechanism from `CLAUDE.md`'s, and conflating the two is a common, exam-relevant error. `CLAUDE.md` disagreements are a *content* problem the model reasons through (imperfectly, and without guarantee); settings disagreements are a *configuration* problem the harness resolves deterministically before the model is ever involved. Put differently: `CLAUDE.md` shapes what Claude *tries* to do; settings determine what Claude Code *permits* it to do — and only one of those two layers offers a hard guarantee about the outcome.

A useful, concrete instance of settings-style override precedence shows up within permission rules themselves: when allow, ask, and deny rules are evaluated for a single action, they resolve in a fixed order — a deny rule wins over a matching ask rule, which in turn wins over a matching allow rule, regardless of how narrowly or broadly each rule is written. A broad deny rule blocking a whole category of command cannot be carved back open by a narrower allow rule aimed at one specific case within it; the deny simply wins. That's a hard guarantee, enforced mechanically, in exactly the way a `CLAUDE.md` instruction to "never run destructive commands" is not — the `CLAUDE.md` instruction shapes intent, but only an actual deny rule guarantees the outcome.

## A Worked Comparison of the Two Models

| | CLAUDE.md content | Settings |
|---|---|---|
| Resolution model | Concatenation — everything loads | Override — one layer wins outright |
| Enforced by | The model's judgment | The harness, deterministically |
| Conflict outcome | Both instructions present; model weighs them | The losing layer simply doesn't apply |

It helps to place the two resolution models side by side against the same underlying scenario. Suppose a personal settings file allows a particular Bash command, while the project's shared settings deny it. Settings resolution answers this immediately and completely: the deny wins, in every session, regardless of what the personal file says, because deny rules from any scope are evaluated before allow rules from any scope — there is no ambiguity left over once the rule is evaluated. Now suppose, instead, a personal `CLAUDE.md` says "always write verbose commit messages" and the project's `CLAUDE.md` says "keep commit messages to one line." Nothing resolves this the way the settings case resolved cleanly — both instructions load, both are genuinely present, and the actual outcome on any given commit depends on how the model weighs them in the moment, which is exactly the uncertainty the settings example doesn't have. Sitting these two cases next to each other is the fastest way to internalize why the same word — "precedence" — means something structurally different depending on which of the two systems it's applied to.

## The General Lesson

Stepping back, the throughline connecting the `CLAUDE.md` hierarchy, settings precedence, and permission-rule evaluation order is a single distinction worth internalizing precisely: **anything delivered to Claude as context is advisory, no matter how many levels it's layered across or how firmly it's worded; anything enforced by Claude Code's harness as configuration is authoritative, resolved deterministically before the model acts.** A team that wants a boundary to be genuinely unbreakable — not just strongly worded — needs to express it as a settings-level deny rule or a hook, not as an emphatic sentence in a `CLAUDE.md` file, however many levels of that hierarchy repeat it. Recognizing which category a given piece of configuration falls into, before reasoning about how a conflict within it resolves, is the actual skill this pair of lectures is building.

**Why it matters for the exam:** Expect at least one scenario question that describes conflicting instructions across `CLAUDE.md` levels and asks what happens — the correct answer engages with concatenation and model judgment, not a clean override. Expect a separate question testing the settings-precedence order directly, and at least one that asks you to sort a mixed list of mechanisms (a `CLAUDE.md` instruction, a deny rule, a hook) by how strong a guarantee each actually provides — that sorting is the highest-leverage single idea in this lecture.

<!-- prevnext:start -->

---

| [&larr; Previous: The Three-Level CLAUDE.md Hierarchy](06-the-three-level-claude-md-hierarchy) | [Next: Subdirectory CLAUDE.md Files and Scoped Instructions &rarr;](08-subdirectory-claude-md-files-and-scoped-instructions) |
|:---|---:|

<!-- prevnext:end -->
