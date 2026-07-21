---
title: Subdirectory CLAUDE.md Files and Scoped Instructions
parent: Claude Code Deep Dive
nav_order: 8
duration: "9:00"
---

# Subdirectory CLAUDE.md Files and Scoped Instructions
{: .no_toc }

*Video duration: 9:00*

The three-level hierarchy lecture introduced subdirectory `CLAUDE.md` files as the mechanism for giving different parts of a large codebase different conventions. This lecture goes further into the specific mechanics that make that mechanism actually work well at scale, because the detail matters: a subdirectory file that behaved exactly like a root file — loaded in full, every session, regardless of relevance — would recreate the bloat problem the Workflow Layer already warned against, just distributed across more files instead of one.

## Loaded Up Front vs. Loaded on Demand

Root-level and ancestor `CLAUDE.md` files (walking up the directory tree from wherever a session starts) are loaded **in full, at launch** — they're part of the standing context every turn in that session pays for, whether or not the current task ever touches the part of the codebase they describe. Subdirectory files further down the tree behave differently: rather than being loaded up front, they're discovered and pulled into context **on demand**, specifically when Claude actually reads or works with files inside that subdirectory. A `frontend/CLAUDE.md` in a monorepo doesn't cost anything in a session that never touches `frontend/` — it simply never loads, because nothing ever gave the harness a reason to look for it there.

This lazy-loading behavior is the entire point of scoping instructions to a subdirectory rather than folding them into the root file. It's the same "load only what's needed, when it's needed" principle the Workflow Layer's Skill Configuration lecture applied to procedural knowledge, applied here to declarative project knowledge instead: a monorepo can carry an arbitrary number of package-specific `CLAUDE.md` files without any of them adding standing cost to sessions that work elsewhere in the tree. This is also precisely why subdirectory scoping is the right tool for monorepo-scale specificity and *not* just a stylistic alternative to putting everything at the root — the two approaches have genuinely different context-cost profiles, not just different organizational aesthetics.

## Practical Patterns for Monorepos

A large, multi-package repository typically settles into a layout where the root `CLAUDE.md` carries only what's genuinely true everywhere — the overall repository layout, cross-cutting tooling (a shared linter, a shared CI pipeline, a company-wide logging convention), and pointers to where package-specific detail lives — while each package or service carries its own file for everything specific to it:

- A `frontend/CLAUDE.md` covering component conventions, the state-management pattern in use, and styling rules specific to that package's framework choice.
- A `services/billing/CLAUDE.md` covering that service's database access pattern, its migration constraints ("migrations are one-way — never edit an applied one"), and any regulatory or compliance context specific to handling payment data.
- A `packages/shared-ui/CLAUDE.md` covering the conventions for a component library other packages consume, distinct from how any single consuming package uses it.

The organizing principle is the same test the Workflow Layer used for what belongs in `CLAUDE.md` at all — "would you tell this to every new engineer working in this specific part of the codebase?" — applied at whatever directory granularity actually matches how the team's ownership and conventions are structured. A monorepo owned by several independent teams, each with a distinct package, naturally produces one subdirectory file per team's area; a monorepo with looser internal boundaries might reasonably use fewer, coarser subdirectory files instead. There's no fixed right number, the same way there's no fixed right number of tools in a tool set — the scoping should match real boundaries in the codebase, not an arbitrary rule about how deep to nest.

## A Practical Failure Mode: Duplication Instead of Scoping

Teams that haven't internalized the on-demand loading behavior sometimes make one of two symmetric mistakes. The first is dumping every package's specifics into the root file "to be safe," recreating the exact bloat problem subdirectory scoping exists to solve — every session pays for every package's conventions, most of them irrelevant to the task at hand. The second, less obvious mistake is **duplicating** the same convention across several subdirectory files instead of stating it once at whatever level is actually common to all of them — for instance, restating a company-wide logging requirement separately in every package's file instead of stating it once at the root, where it belongs because it's genuinely true everywhere. Both mistakes share a root cause: not asking, for each piece of project knowledge, "what is the smallest scope at which this is true and only this true?" before deciding where it lives.

## Scoped Instructions Aren't Only About Packages

Directory-based scoping is the most common use of this mechanism, but the underlying idea — narrower instructions taking effect only in a narrower context — generalizes beyond "one file per package." A directory holding auto-generated code, a `tests/` directory with its own conventions around fixtures and mocking, or a `scripts/` directory of one-off tooling with looser standards than production code can each carry a `CLAUDE.md` calibrated to what actually matters in that specific corner, rather than forcing one root file to awkwardly caveat itself for every exception ("normally X, except in tests, except in scripts, except in generated code..."). Scoping the exception directly to where it applies keeps the root file clean and keeps each scoped file legible on its own.

**Why it matters for the exam:** The high-yield fact to hold onto from this lecture is the loading distinction itself — ancestor files load up front and cost context every turn; subdirectory files load on demand and cost nothing until Claude actually works in that part of the tree. Expect a question that describes a large monorepo and asks where a piece of package-specific knowledge should live, and expect the correct reasoning to hinge on exactly this cost distinction, not just on "more specific things go in more specific files" as a vague intuition.
