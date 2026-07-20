---
title: Role Play: Mentoring on Workflow Essentials
parent: The Workflow Layer
nav_order: 14
---

# Role Play: Mentoring on Workflow Essentials

**Priya (mentee):** I'm trying to get Claude Code set up properly for our repo. Right now I just paste the same paragraph into every session — "we use the repository pattern, tests run with `npm run test:unit`, never touch `infra/` without approval." It works but it's annoying.

**Devon (mentor):** That paragraph is exactly what `CLAUDE.md` is for. Put it in a file at the repo root, and Claude Code loads it automatically, every session, no pasting required. It's stable, it's true regardless of the task, and it's short enough that paying its context cost every turn is cheap.

**Priya:** Nice. While I'm at it, should I also put our incident postmortems in there? We've had a few payment-related bugs and I want Claude to know the history.

**Devon:** I'd hold off. `CLAUDE.md` gets loaded on *every* session, whether or not that session touches payments. A stack of postmortems is exactly the kind of thing that bloats the file without being relevant most of the time. Better question: when does an agent actually need that history?

**Priya:** Probably only when someone's specifically working in the payments module, or asks for a security-sensitive review there.

**Devon:** Right — so that's either a scoped `CLAUDE.md` nested inside `payments/` if it's truly always-relevant *within* that directory, or, if it's more like "occasionally useful reference material for a specific kind of review," it's actually a candidate for retrieval — pull in the relevant postmortem only when a query touches that area. Don't force everything into the one mechanism just because it's the one you learned first.

**Priya:** Okay, separate question. We also want Claude to generate a compliance audit report — it's got a whole template, several validation rules, a specific format legal signs off on. That's clearly not a `CLAUDE.md` entry, it's too big and only a few tasks need it. Is that a subagent?

**Devon:** Ask yourself first: does generating that report need its own isolated context and its own independent model call, separate from whatever the main agent's already working on? Or does it just need a runbook — a set of detailed instructions, maybe a validation script — that the *current* agent can follow while it's already in the middle of a task?

**Priya:** ...Honestly it's the second one. It's just a detailed procedure. There's no reason it needs a separate context.

**Devon:** Then it's a Skill, not a subagent. That's the distinction people trip on — a Skill loads its instructions straight into the agent that's already working, same context, same thread of reasoning. A subagent is a genuinely separate unit: its own context window, its own model calls, and it only reports back a result. They can look similar from the outside — both are "reach for this to handle a sub-task" — but the mechanism underneath is completely different, and it affects cost, isolation, and how you debug a bad output.

**Priya:** Got it. Last thing — once this report generation is solid, we want it wired into CI so it runs automatically before a release, not just when someone remembers.

**Devon:** Good instinct, and that's a different bar than "it works when I run it." For that to count as real CI/CD integration, you need a defined trigger — say, a release-branch tag — reproducible input, somewhere durable for the output to land, like an artifact or a required check, and an explicit answer to what happens if the report flags something: does it block the release or just warn? And treat the step itself like any other pipeline job — logging, retries, a fallback if it fails. It doesn't get to skip that just because a model's doing the work.

**Priya:** So: `CLAUDE.md` for the standing stuff, Skill for the compliance procedure, and CI integration once we want it enforced rather than optional.

**Devon:** That's the whole module in one sentence.
