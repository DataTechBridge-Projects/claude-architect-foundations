---
title: The Three-Level CLAUDE.md Hierarchy
parent: Claude Code Deep Dive
nav_order: 6
duration: "13:00"
---

# The Three-Level CLAUDE.md Hierarchy
{: .no_toc }

*Video duration: 13:00*

The Workflow Layer's "Configuring CLAUDE.md" lecture established what a `CLAUDE.md` file is and what belongs in it, mentioning in passing that Claude Code supports files "at multiple directory levels." This lecture makes that precise: `CLAUDE.md` content can live at three distinct levels of scope, each answering a different question about *who* a given instruction applies to, and understanding all three — and how they combine — is foundational to everything else in this module's coverage of configuration.

## Level One: User Instructions

The broadest-reaching level in terms of *time* (it applies to every project you ever work in) but the narrowest in terms of *audience* (it applies only to you) is a `CLAUDE.md` kept in your personal configuration directory, outside any specific repository. This is where personal preferences that have nothing to do with a particular codebase belong: a preferred commit message style you use regardless of project, a habit of always double-checking database migrations before considering a task done, a general instruction to explain reasoning briefly before making a nontrivial change. None of this is a fact about any specific project's conventions — it's a fact about how *you* want Claude Code to behave, everywhere, every time you use it.

The defining property of this level is that it travels with the person, not the codebase. Clone a fresh copy of a repository on a different machine without your personal configuration, and none of your user-level instructions come along — which is exactly correct, since they were never claims about the project in the first place.

## Level Two: Project Instructions

The middle level, and the one the Workflow Layer's earlier lecture spent most of its time on, is a `CLAUDE.md` committed at a project's root. This is genuinely shared, team-owned content: build and test commands, architectural conventions, constraints, domain vocabulary — the standing knowledge that should be true for anyone working on this codebase, human or agent, regardless of which specific task they're doing today. Because it's committed to the repository, it travels with the *codebase*, not the person: clone the repo anywhere, and this file comes with it, applying identically to every team member's sessions.

This is the level most teams think of when they first hear about `CLAUDE.md`, and for good reason — it's where the highest-leverage, most broadly-applicable project knowledge belongs, and it's the level that gets the most day-to-day maintenance attention as a project evolves.

## Level Three: Subdirectory-Scoped Instructions

The narrowest level in scope (and, often, the most specific in content) is a `CLAUDE.md` placed in a subdirectory rather than at the project root. A large repository is rarely architecturally uniform throughout — a monorepo might contain a frontend package with its own component conventions, a backend service with its own database access patterns, and an infrastructure directory with its own deployment constraints, all coexisting under one root. Forcing every one of those specifics into a single root-level file produces exactly the anti-pattern the Workflow Layer already warned about: a bloated file where most of its content is irrelevant to most of the work happening in any given part of the codebase, paid for in context on every session regardless of which corner of the project that session actually touches.

A subdirectory `CLAUDE.md` solves this by scoping specificity to relevance: a `frontend/CLAUDE.md` covering component and styling conventions doesn't need to be loaded — or even be relevant — for a session working exclusively in `infra/`. This level gets its own full lecture later in this module, since the mechanics of exactly *when* a nested file's content actually enters context (loaded up front versus discovered on demand as Claude reads files in that part of the tree) matter enough to deserve dedicated treatment rather than a summary here.

## How the Three Levels Combine

The single most commonly misunderstood fact about this hierarchy is what "levels" actually means mechanically. It is tempting — and wrong — to assume this works like the settings-file hierarchy covered later in this module, where a higher-precedence layer *overrides* a lower one, so that only one value for a given setting ultimately applies. `CLAUDE.md` files don't work that way. Every discovered file's content is **concatenated** into context — user-level, project-root, and any relevant subdirectory file are all present simultaneously, in the same conversation, none of them silently erasing another.

What *does* vary across levels is **order and specificity**, not survival. Files closer to the root of the filesystem hierarchy load first — user instructions before project instructions — and content nearer to wherever the session is actually working tends to load later or more locally, meaning it's freshest and most specific to the immediate task by the time the model is actually reasoning about what to do. Practically, this means a project-root instruction isn't "more true" than a user-level one in some absolute sense — both are simply present, both shape behavior, and where they genuinely conflict, the model has to resolve that conflict through judgment rather than the harness silently picking one for it. (That failure mode, and how to actually prevent it rather than hope the model resolves it well, is exactly what the next lecture covers.)

This distinction — concatenation with ordering, rather than override — is one of the highest-yield facts in this entire module, precisely because it's so easy to assume the wrong (simpler, override-based) model by analogy to how most layered configuration systems work.

## Why Three Levels, and Not One or Five

It's worth being explicit about why this hierarchy has the shape it does, rather than treating "three levels" as an arbitrary product decision to memorize. Each level answers a genuinely different question:

- **User-level** answers "what do I always want, regardless of project?" — a question about a person's standing preferences.
- **Project-level** answers "what does everyone need to know to work competently on this codebase?" — a question about a team's shared, durable ground truth.
- **Subdirectory-level** answers "what's true only in this specific corner of a larger project?" — a question about local specificity within a bigger shared structure.

These three questions are genuinely different, map onto genuinely different audiences and lifecycles (personal and cross-project, team-owned and codebase-scoped, sub-team-owned and directory-scoped), and collapsing any two of them into one level loses real information. Personal preferences don't belong in a file the whole team reviews and commits; a monorepo's frontend-specific conventions don't belong bloating a root file that every session pays the context cost of loading regardless of relevance; and neither of those things belongs duplicated across every relevant location by hand when a hierarchy can express the scoping directly.

## A Worked Example

Consider an engineer working in `services/billing/` inside a large company monorepo. In a single session there, three files might all be in play: their personal `CLAUDE.md` says to always explain the reasoning behind a nontrivial refactor before making it; the monorepo's root `CLAUDE.md` says all services must use the company's internal logging library and never log raw request bodies; and a `services/billing/CLAUDE.md` says this particular service's database migrations are one-way and must never be edited after being applied. All three are simultaneously true, all three are simultaneously relevant, and none of them is "overriding" any of the others — they're answering three different questions (personal style, company-wide policy, this-service's specific constraint) at three different scopes, exactly as the hierarchy intends.

Notice, too, that none of the three instructions in that example actually contradicts another — they layer cleanly because each was written at the scope it genuinely belongs to. That's not a coincidence built into the example; it's the payoff of using the hierarchy as intended. A team that instead crams company-wide policy into a personal file, or personal habits into the shared root file, loses that clean layering even though the mechanism itself would still concatenate everything correctly — the hierarchy can only pay off if each piece of knowledge actually lives at the scope that matches who it's really true for.

**Why it matters for the exam:** The single fact this exam most wants you to have internalized about this topic is that the three levels *combine additively* rather than override one another — a question that presents a scenario with instructions at two or three levels and asks "which one applies" is very often testing whether you'll wrongly assume an override model. The correct answer is almost always "all of them apply; here's how they interact," not "only the most specific one wins."
