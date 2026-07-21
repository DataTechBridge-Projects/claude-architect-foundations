---
title: Creating and Structuring Custom Slash Commands
parent: Claude Code Deep Dive
nav_order: 11
duration: "12:00"
---

# Creating and Structuring Custom Slash Commands
{: .no_toc }

*Read duration: 12:00*

A **custom slash command** is the simplest unit of reusable instruction in Claude Code: a saved prompt, invoked by typing `/` followed by its name, that expands into a full instruction the moment you invoke it rather than requiring you to type (or remember) that instruction from scratch every time. If you've ever caught yourself pasting nearly the same paragraph into a session for the third time this week — "review this diff for security issues, checking specifically for X, Y, and Z" — a slash command is the mechanism that turns that repeated paste into a single short invocation.

## What a Slash Command Actually Is

Mechanically, a custom slash command is a plain Markdown file whose content *is* the prompt. There's no separate configuration language to learn — the body of the file is exactly what gets sent as your instruction once the command is invoked, the same way you'd type it directly into the conversation. A file might contain something as simple as:

```markdown
Review the currently staged changes for security issues. Check specifically
for: SQL built through string concatenation, secrets or credentials in the
diff, and any new dependency added without a corresponding lockfile update.
Report findings as a short bulleted list; don't rewrite the code yourself.
```

Saved as a command named, say, `security-review`, typing `/security-review` sends exactly this instruction, verbatim, without you retyping a word of it.

## Frontmatter: Metadata About the Command Itself

Beyond the prompt body, a slash command file can carry YAML frontmatter describing the command rather than instructing Claude — most usefully a short **description** (surfaced when you're browsing available commands, so you can tell what a command does without opening the file) and, where relevant, a restriction on which tools the command is permitted to use while it runs. A command meant purely to produce a written review, for instance, can be scoped so it can't accidentally invoke file-editing tools at all, which is a useful safety property distinct from — but complementary to — the permission-mode and deny-rule mechanisms covered earlier in this module: scoping *at the command level* means the restriction travels with the command itself, applying identically no matter what broader permission mode the session happens to be running under.

## Arguments: Making a Command Reusable Across Inputs

A command that only ever does exactly one fixed thing is useful, but a command that can take a parameter is more useful still. Slash commands support this through an arguments placeholder in the body — text typed after the command name on invocation gets substituted into the prompt wherever the placeholder appears. A command for drafting a changelog entry from a ticket number, for instance, might be written so that invoking it with a specific ticket ID substitutes that ID directly into the instruction, without needing a separate command per ticket or requiring you to write the full instruction out by hand each time. This is what turns a slash command from "a saved fixed prompt" into "a small, parameterized procedure" — closer in spirit to calling a function with an argument than to expanding a static macro, even though the underlying mechanism is still just text substitution into a prompt.

## Namespacing with Subdirectories

As a team accumulates more than a handful of commands, a flat list of names starts to strain — two different domains might both want a command called `review`, or a team might want to group commands by the part of the workflow they support. Organizing command files into subdirectories addresses this: a command file nested under a subdirectory is invoked with that structure reflected in its name (a command for release-related tasks grouped under a `release` namespace, invoked as something like `/release:changelog`, distinct from an unrelated `/review` command sitting elsewhere). This is a purely organizational mechanism — it doesn't change what a command does, only how a growing set of them stays navigable and how naming collisions get avoided as the set grows, the same discipline good tool-set design already teaches in the Tool Layer: a small set of ambiguous names is more error-prone than a larger, clearly organized one.

## Personal vs. Project Scope, and Team Distribution

Slash commands, like `CLAUDE.md` files, come in more than one scope. A command saved in your personal configuration directory is available to you, in every project, regardless of what any specific repository contains — the right home for a workflow habit that's about how *you* like to work, not about any particular codebase (a personal command for drafting a stand-up update in your preferred format, say). A command saved inside a project's own repository, by contrast, is **shared with the whole team through version control** the instant it's committed: anyone who clones the repository gets that command available immediately, with no separate installation step, exactly the same distribution mechanism that makes a project-level `CLAUDE.md` team-owned rather than personal.

This project-scoped distribution is the practically important part of "structuring" a slash command well for a team, more than the file mechanics themselves. A well-designed team command library treats each command the way a team would treat a shared script: committed, reviewed like any other change to the repository, and named clearly enough that a new team member browsing available commands understands what each one does without having to open the file and read its full prompt.

## Writing Commands That Stay Useful

A command that isn't clearly named or clearly described tends to rot the same way a stale `CLAUDE.md` entry does — except worse, because a command that no one remembers exists or trusts might as well not exist, and the effort of writing it was wasted. Two habits keep a team's command library actually useful rather than accumulating unused files. First, name commands by what they do, not by who wrote them or when — `security-review` invites use in a way `jt-review-v2` never will, and **a command's name is effectively its whole discovery signal**, since there's no separate description shown unless someone thinks to look. Second, prune commands the same way the Workflow Layer recommended pruning Skills: a command describing a workflow the team no longer follows is worse than no command, because it actively invites someone to invoke a stale procedure by name, trusting that a saved command reflects current practice.

## What Belongs in a Slash Command vs. Elsewhere

| | Slash command | Skill |
|---|---|---|
| Invocation | Explicit, by name only | Explicit, or automatically discovered by description match |
| Bundled resources | None — just a prompt | Scripts, reference material, examples |
| Right fit when | The reusable unit is just a prompt | The procedure needs supporting files or auto-discovery |

Slash commands sit at the simple end of Claude Code's extension mechanisms, and recognizing when a slash command is the right tool — rather than reaching for a fuller Skill, the subject of the next two lectures — matters for the same reason picking the right granularity of tool matters in the Tool Layer. A slash command is the right fit when the reusable unit is genuinely just **a prompt**: no bundled scripts, no reference material to load conditionally, no need for the agent to discover and invoke it automatically based on matching a task to a description — you invoke it explicitly, by name, because you already know that's what you want to happen right now. The moment a reusable procedure needs bundled supporting files, on-demand reference material, or automatic discovery based on task content rather than an explicit invocation, it has outgrown a slash command and belongs in a Skill instead — which is exactly where this module goes next.

{: .important }
> Don't conflate a slash command with a Skill. A slash command is only ever an explicitly-invoked, static prompt — it cannot bundle scripts or reference material, and it never triggers automatically. The moment a procedure needs either of those, it belongs in a Skill instead.

**Why it matters for the exam:** Expect questions distinguishing a slash command from a Skill along exactly this line — explicit, name-based invocation of a plain prompt versus a richer, potentially auto-discovered package with bundled resources — and questions about scope (personal versus project) that mirror the same personal-versus-team-owned distinction tested elsewhere in this module for `CLAUDE.md` files. The through-line across all of these is the same: does this reusable unit travel with a person or with a codebase, and is it invoked explicitly or discovered automatically.

<!-- prevnext:start -->

---

| [&larr; Previous: Common CLAUDE.md Patterns for Real Teams](10-common-claude-md-patterns-for-real-teams) | [Next: What Skills Are and When to Build Them &rarr;](12-what-skills-are-and-when-to-build-them) |
|:---|---:|

<!-- prevnext:end -->
