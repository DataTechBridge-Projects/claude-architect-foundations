---
title: Common CLAUDE.md Patterns for Real Teams
parent: Claude Code Deep Dive
nav_order: 10
duration: "12:00"
---

# Common CLAUDE.md Patterns for Real Teams
{: .no_toc }

*Video duration: 12:00*

The Workflow Layer's original `CLAUDE.md` lecture covered *categories* of content that belong in the file — commands, conventions, constraints, vocabulary, gotchas. This lecture is deliberately more concrete: what these categories actually look like written out, in the phrasing that makes them effective for an agent acting on them rather than a human skimming them, since the gap between a vague category and an actually useful instruction is where most real `CLAUDE.md` files fall short in practice.

## The Commands Section

Nearly every effective `CLAUDE.md` opens with, or prominently includes, the concrete commands needed to work in the project — because rediscovering these from scratch every session (parsing a manifest file, guessing at a test runner) is pure waste when they're this cheap to just state:

```markdown
## Commands
- Install: `pnpm install`
- Run dev server: `pnpm dev`
- Run all tests: `pnpm test`
- Run a single test file: `pnpm test path/to/file.spec.ts`
- Lint: `pnpm lint` (must pass before any commit)
- Type-check: `pnpm typecheck`
```

Notice the specificity: not "run the tests" but the exact command, including the single-file variant, which is the one Claude actually needs most often when iterating on a fix. A commands section that only states the top-level command and omits the single-file variant forces Claude to either guess at the flag syntax or run the entire suite for every small change — a small omission with a real, repeated cost.

## Architectural Conventions, Stated as Rules Not Descriptions

The difference between a convention that shapes behavior and one that reads as background color is almost always the difference between a description and a rule:

```markdown
## Architecture
- API routes live in `src/routes/`, one file per resource.
- All database access goes through `src/db/repositories/` — never call the ORM
  directly from a route handler.
- Feature flags are read through `src/flags.ts`, never `process.env` directly.
```

Compare that to the weaker, common alternative: "We try to keep a clean separation between routes and data access." The weaker version is true, sounds reasonable, and **gives an agent almost nothing to act on** — it doesn't say *where* the boundary is enforced or *what specifically* violates it. The stronger version states the rule as something checkable: is this new code calling the ORM directly from a route handler, yes or no.

## Constraints, Written as Explicit Boundaries

The highest-leverage category, and the one worth the most editorial care, is the list of things Claude should never do without explicit sign-off:

```markdown
## Constraints
- Never modify files under `infra/terraform/` without explicit approval in the
  conversation — these changes require a separate infrastructure review.
- Do not add a new npm dependency without asking first.
- Migrations are one-way. Never edit an already-applied migration file; write
  a new migration instead.
- The staging database is shared across teams. Never run a destructive
  migration or a data-deleting script against it.
```

These read almost like a checklist, deliberately — a well-written constraint is specific enough that violating it is unambiguous, which is exactly the property that makes a constraint enforceable in practice rather than just aspirational in tone.

## Domain Vocabulary, Defined Inline

A short glossary prevents an entire category of subtle misunderstanding, where Claude reads a request using project-specific terminology and reasonably (but wrongly) maps it onto a generic meaning:

```markdown
## Vocabulary
- A "ledger entry" is an immutable record of a balance change — never update
  one in place; always write a new entry and let the balance be derived.
- "Cohort" refers to a group of users who signed up in the same billing
  cycle, not an experiment group (see `experiments.md` for that).
```

This category is easy to skip because it feels like documentation rather than instruction, but a misread term early in a task tends to compound — every subsequent decision inherits the wrong mental model.

## Gotchas, as a Living List

The gotchas section is the one most teams undertreat as a one-time write and forget, when it should be the section edited most often, since it exists specifically to capture whatever the team just learned the hard way:

```markdown
## Known Gotchas
- The `legacy/` directory still builds but is fully deprecated — don't extend
  it, even if a task seems to call for it.
- `scripts/seed.ts` is destructive and drops existing local data. Never run it
  against anything but a fresh local database.
- Tests in `payments.spec.ts` are currently flaky under parallel execution —
  a failure there alone isn't necessarily a real regression; re-run in
  isolation before treating it as one.
```

## Pointing Instead of Pasting

One pattern worth calling out on its own is what a mature `CLAUDE.md` does when a piece of relevant knowledge already exists somewhere else — a detailed API reference, a design document, a runbook for an operational procedure. The instinct to paste the whole thing in is understandable and almost always wrong, for the same context-budget reason the Workflow Layer warned about `CLAUDE.md` becoming a dumping ground. The stronger pattern is a short pointer instead:

```markdown
## Further Reference
- Full API contract for the payments service: see `docs/payments-api.md`.
- Incident response steps for a failed deploy: see `runbooks/deploy-rollback.md`.
```

This keeps the always-loaded file lean while still making the deeper material discoverable — Claude can read the referenced file the moment a task actually needs that depth, rather than everyone's session paying for it whether or not it's relevant that day. This is a smaller-scale version of the same "load only what's needed" discipline that motivates subdirectory scoping and Skills elsewhere in this module, applied here to reference material within a single file rather than to whole files or procedures.

## A Realistic Composite Structure

Put together, an effective team `CLAUDE.md` for a mid-sized project tends to land somewhere close to this shape, in this rough order: a one- or two-line project description for orientation, the commands section, architectural conventions, constraints, a short vocabulary section if the domain warrants one, and known gotchas. Sections a project doesn't need (a vocabulary section for a codebase with no unusual domain terms) are simply omitted rather than filled with filler — an empty or padded section costs context for no benefit, the same anti-pattern the Workflow Layer already warned against at the level of the whole file.

## The Pattern Underlying All of These

Every example above shares one property worth naming directly: each is written to be **actionable and checkable**, not merely descriptive. "We care about type safety" is a description; "run `pnpm typecheck` before considering a change complete" is an instruction an agent (or a human) can actually follow and verify compliance with. This is the same distinction the original `CLAUDE.md` lecture drew between writing for humans versus writing for an agent that has to act on the content — this lecture's contribution is making that distinction concrete enough to copy from directly, section by section, rather than leaving it as a principle to apply from scratch.

**Why it matters for the exam:** Expect questions that present two candidate `CLAUDE.md` snippets covering the same underlying fact — one descriptive, one actionable — and ask which is better written, or that ask you to identify which category (commands, conventions, constraints, vocabulary, gotchas) a given real-world example belongs to. The reasoning test underneath both question types is the same: is this specific enough to be checked, and does it belong in this file at all given how broadly and durably it applies.

<!-- prevnext:start -->

---

| [&larr; Previous: Excluding Files and Content](09-excluding-files-and-content) | [Next: Creating and Structuring Custom Slash Commands &rarr;](11-creating-and-structuring-custom-slash-commands) |
|:---|---:|

<!-- prevnext:end -->
