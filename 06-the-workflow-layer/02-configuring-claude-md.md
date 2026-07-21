---
title: Configuring CLAUDE.md
parent: The Workflow Layer
nav_order: 2
duration: "14:12"
---

# Configuring CLAUDE.md
{: .no_toc }

*Read duration: 14:12*

A **`CLAUDE.md`** file is a plain-text (usually Markdown) file that Claude Code automatically reads into context at the start of a session, without any user having to paste it in or reference it explicitly. It lives in a project's repository — typically at the root, sometimes nested in subdirectories for more specific scopes — and its entire purpose is to give Claude persistent, standing knowledge of *this* project: its conventions, its constraints, its quirks, and the things a new human contributor would otherwise have to be told in a Slack message or learn by making a mistake first. Once it exists, it applies automatically, every session, without anyone re-explaining it.

That "automatic and persistent" property is what makes `CLAUDE.md` a workflow-layer concept rather than a prompt-layer one. Prompt design (covered in the next module) is about how you phrase a specific request in a specific moment. `CLAUDE.md` is about what's true *regardless* of the specific request — the standing context that should shape every request made against this codebase, whether the task is "fix this bug," "add this feature," or "review this PR." It's infrastructure, not phrasing.

## What CLAUDE.md Actually Is, Architecturally

Mechanically, `CLAUDE.md` content gets folded into the model's context window early, alongside the system prompt and before the specific user turn. Functionally, it behaves like a standing set of project-level instructions that persists across every session started in that directory, without consuming any effort from the user to re-supply. Think of it as the project's onboarding document, except the "new hire" is Claude, it's read on every single session, and it never forgets what's in it (unlike a human onboarding document, which people skim once and then never open again).

This gives `CLAUDE.md` a few defining properties worth naming explicitly, because the exam will test your understanding of each one:

- **Automatic.** No one has to remember to attach it or mention it. If it's present in the project, Claude Code loads it.
- **Persistent across sessions.** It isn't scoped to one conversation; it applies to every session opened against that project, indefinitely, until someone edits the file.
- **Project-scoped.** Its content is specific to the repository (or directory) it lives in — this is local knowledge, not general-purpose model behavior.
- **Human-readable and version-controlled.** Because it's a plain file in the repo, it goes through the same code review, diff, and version-history process as any other file. Changes to project conventions become auditable commits, not tribal knowledge that lives only in someone's head.
- **Cheap to maintain, relative to its value.** It's just a Markdown file. Updating it doesn't require touching model configuration, tool definitions, or infrastructure — someone edits a file and commits it.

## What Belongs in a CLAUDE.md

The strongest mental model for deciding what goes in a `CLAUDE.md` is: *would you tell this to every new engineer who joins the team, regardless of what they're about to work on?* If yes, it's a strong candidate. If it only matters for one specific task, it probably doesn't belong there — it belongs in that task's prompt instead.

Concretely, well-built `CLAUDE.md` files tend to cluster around a few categories:

**Build, test, and run commands.** How to install dependencies, run the test suite, start a dev server, lint the code, or run a single test file. This is exactly the kind of thing that's stable, always relevant, and painful to re-derive every session — the alternative is Claude spending several tool calls rediscovering your `package.json` scripts from scratch every single time.

**Architectural conventions.** Where things live and why. "API routes go in `src/routes`, one file per resource." "We use the repository pattern for all database access — never call the ORM directly from a route handler." "State management is centralized in `src/store`; don't introduce local component state for anything that needs to persist across navigation." These are the load-bearing conventions that keep a codebase coherent, and they're exactly the kind of thing an agent will violate by default if it isn't told, because the "obvious" way to write code in isolation often isn't the way *this* codebase does it.

**Constraints and guardrails.** Things Claude should never do without being asked, or never do at all. "Never modify files under `infra/terraform` without explicit approval." "Do not add new npm dependencies without asking first." "Migrations are one-way — never edit an already-applied migration file; write a new one." These constraints matter enormously for safety and for keeping an agent's autonomy bounded to what's actually appropriate for the codebase.

**Code style and formatting expectations** that aren't already enforced by a linter — naming conventions, comment conventions, preferred patterns over alternatives ("prefer composition over inheritance here," "use the `Result` wrapper type instead of throwing").

**Domain vocabulary and project-specific terminology.** If your team calls a certain kind of object a "ledger entry" rather than a "transaction," or has an internal term for a recurring business concept, defining that vocabulary up front prevents Claude from misreading requests that use it.

**Known gotchas.** "The staging database is shared — don't run destructive migrations against it." "The `legacy/` directory is deprecated and excluded from new work, even though it still builds." These are the landmines an experienced contributor already knows to avoid and a fresh one (human or agent) doesn't.

## How CLAUDE.md Differs from a System Prompt

It's tempting to think of `CLAUDE.md` as "just another system prompt," and the exam will probe exactly this conflation, because the two do overlap in function — both shape model behavior before the user's specific request arrives. But they differ in scope, ownership, and mutability in ways that matter architecturally.

A **system prompt** is typically controlled by whoever is building the overall application or agent harness. It defines the model's role, its general behavioral guardrails, its output format expectations, and often applies across many different projects or contexts that share that harness. It's usually not something an individual project team edits day to day — it's closer to product configuration.

`CLAUDE.md`, by contrast, is scoped to a *project* and owned by the *team working in that project*. It lives in the repository itself, so it's edited by the same people who edit the code, through the same pull-request workflow, and it travels with the codebase — clone the repo, and you get the context along with it. Where a system prompt says "you are a careful, safety-conscious coding assistant," a `CLAUDE.md` says "in *this* repository, tests run with `npm run test:unit`, database access goes through the repository layer, and you must never touch `infra/`."

A useful way to hold the distinction: the system prompt defines who the agent *is*; `CLAUDE.md` defines what the agent needs to know to work competently in *this specific place*. One is about identity and general behavior; the other is about local, project-specific ground truth. Both matter, and a well-designed system typically has both — but conflating them, or trying to cram project-specific conventions into a system prompt meant to generalize across many projects, is an anti-pattern in its own right.

| Aspect | `CLAUDE.md` | System prompt |
|---|---|---|
| Scope | One project or repository | Often spans many projects |
| Owner | The team working in that repo | Whoever builds the agent harness |
| Change process | Edited and reviewed like any other file in the repo | Configured at the application/harness level |
| Defines | What the agent needs to know to work in *this* place | Who the agent *is* and how it generally behaves |

{: .important }
> Don't conflate `CLAUDE.md` with the system prompt. `CLAUDE.md` is project-scoped, team-owned, and version-controlled with the repo; the system prompt defines the agent's identity and general behavior and typically spans many projects. Cramming project-specific conventions into a system prompt — or generic identity and behavior rules into `CLAUDE.md` — is a common exam trap.

## Common Patterns

**Layered CLAUDE.md files.** Claude Code supports `CLAUDE.md` files at multiple directory levels — a root-level file with organization- or repo-wide conventions, and nested files in subdirectories with more specific context (a `frontend/CLAUDE.md` covering component conventions, a `services/billing/CLAUDE.md` covering that service's particular constraints). This mirrors how a large engineering org actually distributes knowledge: some things are true everywhere, some things are true only in one corner of the codebase, and forcing everything into one flat file loses that structure.

**Keeping it living, not archival.** The best `CLAUDE.md` files get edited whenever a convention changes or a new gotcha is discovered, the same way a good README does. A `CLAUDE.md` that hasn't been touched in a year, in an actively changing codebase, is a warning sign — it's likely stale, and **stale project context is worse than no project context**, because it actively misleads rather than simply being absent.

**Treating it as a first-class reviewed artifact.** Changes to `CLAUDE.md` should go through the same review process as code, because a bad instruction in that file has leverage — it silently shapes every future session, not just one.

## Common Anti-Patterns

The most common failure mode is **treating `CLAUDE.md` as a dumping ground.** Once teams discover it works, there's a temptation to paste in the entire architecture decision record, every past incident postmortem, or a full API reference. This defeats the purpose. Every line in `CLAUDE.md` is loaded into context on *every single session*, whether or not that session needs it — so bloating the file has **a real, ongoing cost in context budget** (this is exactly the tension the next lecture explores against retrieval-based approaches). A `CLAUDE.md` should stay tight: durable, high-value, broadly-applicable information only.

A second anti-pattern is **using it for information that changes frequently.** Sprint priorities, who's currently on call, this week's known bug list — these rot almost immediately and turn the file into a source of stale, misleading guidance rather than reliable ground truth. If something changes weekly, it doesn't belong in a file designed to be trusted as stable.

A third is **duplicating what tooling already enforces.** If a linter already enforces a formatting rule, restating it in `CLAUDE.md` is redundant context spend — better to note that the linter exists and should be run, not to re-encode its rules in prose.

Finally, **writing it for humans instead of for an agent that has to act on it.** A good `CLAUDE.md` gives specific, actionable instructions ("run `npm run test:unit` before considering a change complete") rather than vague cultural color ("we care a lot about testing here"). The latter reads fine to a human skimming a README; it gives an agent almost nothing to act on.

**Why it matters for the exam:** Expect scenario questions that describe a piece of project information and ask whether it belongs in `CLAUDE.md`, a system prompt, a Skill, or nowhere persistent at all. The reasoning test is usually: is this stable, is it broadly applicable across tasks in this project, and is it something you'd want loaded automatically every single time — or is it task-specific, volatile, or too large to justify permanent context cost? Getting that sorting right is the core competency this lecture is building toward.

<!-- prevnext:start -->

---

| [&larr; Previous: Module Intro](01-module-intro) | [Next: When to use CLAUDE.md vs. RAG Knowledge? &rarr;](03-when-to-use-claude-md-vs-rag-knowledge) |
|:---|---:|

<!-- prevnext:end -->
