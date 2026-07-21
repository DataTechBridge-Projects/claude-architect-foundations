---
title: Excluding Files and Content
parent: Claude Code Deep Dive
nav_order: 9
duration: "8:00"
---

# Excluding Files and Content
{: .no_toc }

*Video duration: 8:00*

Everything covered so far in this module's configuration coverage has been about what Claude Code *should* see and know. This lecture covers the opposite problem: keeping specific files, directories, and content out of Claude's reach entirely — not because they're irrelevant, but because reading or touching them at all is a risk (secrets, credentials), a waste of context on content that adds no value (build output, generated code, vendored dependencies), or a scope boundary a team wants enforced rather than merely requested (another team's part of a monorepo, infrastructure that requires separate review).

## Permission Deny Rules as the Primary Mechanism

Claude Code doesn't have a single dedicated "ignore file" the way some tools use a `.dockerignore` or similar convention for one specific purpose. Instead, exclusion is expressed through the same permission-rule system that governs every other tool: a **deny rule** scoped to a file tool (reading or editing) with a path pattern, written using the same pattern syntax that `.gitignore` files use — bare filenames match at any depth, `**` spans directories, and patterns can be anchored to a project root, a home directory, or the filesystem root depending on how they're written. A rule denying reads of `.env` files, or denying both reads and edits under a `secrets/` directory, keeps that content out of every session governed by the settings file the rule lives in — checked into a project's shared settings, it applies to the whole team; kept in a personal local settings file, it applies only to that individual's sessions.

This matters architecturally in the same way the permission-modes lecture emphasized: a deny rule is enforced by the harness before the model ever gets a result back, not a request phrased to the model and hoped for. Asking Claude, via `CLAUDE.md`, to "never read the `.env` file" is advisory — it shapes what Claude tries, with no structural guarantee behind it. A deny rule on `Read(.env)` is authoritative: the tool call simply doesn't succeed, regardless of what the model intended or was told. Anywhere the stakes of an exclusion are genuinely high — actual credentials, actual secrets, actual regulated data — the deny rule is the correct mechanism, and a `CLAUDE.md` instruction alone is not a substitute for it, however clearly worded.

## Excluding CLAUDE.md Content Specifically

A related but distinct need shows up in large, multi-team monorepos: a project might have dozens of subdirectory `CLAUDE.md` files belonging to teams other than the one a given engineer works on, and while none of that content is sensitive, loading all of it anyway is pure context waste for sessions that will never touch those areas. A setting purpose-built for this lets an engineer (or an organization, at a managed-policy level) exclude specific `CLAUDE.md` files or whole glob patterns of them from loading, independent of the general file-access deny rules used for secrets. This is a context-economy exclusion rather than a security exclusion — the excluded files aren't dangerous to read, they're simply not relevant to this engineer's work, and excluding them keeps every session's standing context focused on what's actually applicable.

## Sandboxing as a Deeper Layer

Permission deny rules stop Claude Code's own tools from accessing excluded content, but they operate at the level of Claude Code's tool calls specifically — they don't reach into an arbitrary subprocess a shell command might spawn that reads a file directly through its own means, bypassing Claude Code's file tools entirely. For exclusions that need to hold even against that scenario — genuinely sensitive credentials in an environment where a compromised or manipulated shell command could otherwise reach them — OS-level **sandboxing** provides a deeper layer, restricting what a Bash-spawned process can actually reach on disk or over the network regardless of which tool or subprocess is doing the reaching. The relationship mirrors the permissions-versus-sandboxing distinction from the tool-system lecture earlier in this module: permission rules stop Claude from *attempting* access through its own tools; sandboxing stops access from *succeeding* even through a path the permission layer doesn't directly govern.

## Why This Matters Beyond Secrets

Exclusion isn't only a security topic. Generated code, build artifacts, vendored third-party dependencies, and large binary or data directories are all things a session gains nothing from reading and loses real context budget on if it does — a session that wanders into a `node_modules/` tree or a `dist/` directory while exploring a codebase is burning context on content that was never meant to be read or reasoned about directly. Excluding these isn't primarily about danger; it's the same context-economy discipline that motivates keeping `CLAUDE.md` files lean and using subdirectory scoping instead of one bloated root file — applied here to raw file access rather than to standing instructions.

**Why it matters for the exam:** The core distinction to hold onto is that exclusion for genuinely sensitive content should be enforced (a deny rule, or sandboxing for a harder guarantee), while exclusion for merely irrelevant content is a context-economy choice that a `CLAUDE.md` instruction or a `claudeMdExcludes`-style setting handles well enough. Expect a scenario question presenting a category of file (an API key, a generated build directory, another team's monorepo package) and asking which exclusion mechanism actually fits — treating every case identically, or reaching for the same mechanism regardless of stakes, is the wrong instinct the question is testing for.
