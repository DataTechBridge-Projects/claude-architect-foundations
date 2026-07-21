---
title: "Lab: Workflow Strategy Simulator"
parent: The Workflow Layer
nav_order: 10
duration: "1:43"
---

# Lab: Workflow Strategy Simulator
{: .no_toc }

*Video duration: 1:43*

This lab hands you a scenario — a mid-sized team building a Claude-assisted release process that spans drafting release notes, running a pre-merge code review, generating missing tests, and updating a public API reference — and asks you to design the workflow strategy behind it, not just describe what Claude should do.

For each piece of the scenario, you'll decide where it belongs: is it a `CLAUDE.md` entry (stable, project-wide, always relevant), a Skill (a well-defined, occasionally-needed procedure with its own instructions and maybe its own bundled script), a step that should be fully deterministic (no model involved at all), or a step that needs to be wired into CI/CD as an automated, gated pipeline participant. You'll also have to decide sequencing — what has to happen before what — and where a human review gate is non-negotiable versus where an advisory check is enough.

There's no single "correct" architecture here; the simulator scores your reasoning, not just your final placement. Two reasonable engineers can land on different answers for, say, whether test generation should block merge or just flag gaps — what matters is that you can justify the placement using the same criteria this module has been building:

| Placement | Deciding criterion |
|---|---|
| `CLAUDE.md` | Stable and relevant to nearly every session |
| Skill | Well-defined procedure, needed only occasionally |
| Deterministic step | Fully specifiable in advance, independent of input |
| CI/CD gate strictness | Driven by the stakes and reversibility of the output |

Expect the simulator to push back with follow-up "what if" variations once you commit to an initial design, the same way a real exam scenario question often adds a twist after the first answer.

To walk through one slice concretely: release-note drafting is a good candidate for a Skill rather than a `CLAUDE.md` entry, because it's a well-defined procedure (pull merged PR titles, group by category, apply house tone) that only runs at release time, not on every session — bundling it as a Skill keeps that template and its formatting logic out of context the rest of the time. The pre-merge code review, by contrast, is a candidate for CI/CD integration with a mixed gate: advisory for style findings, blocking for anything flagged as a likely security issue, which mirrors the stakes-and-reversibility reasoning from the CI/CD Integration lecture directly. Test generation is interesting precisely because its output is self-validating — a generated test can actually be run — so the simulator will push you to justify why you'd still want a human glance at coverage gaps even when the tests themselves pass cleanly.

Expect the follow-up twists to target exactly the seams this module spent the most time on: what happens if the "occasional" report-generation procedure turns out to be needed on nearly every PR (does it stop being a Skill and become standing `CLAUDE.md` context)? What happens if the documentation-update step starts touching a compliance-relevant page (does the gate that was advisory need to become blocking)? Working through those follow-ups is the real point of the lab — the initial placement is just the opening move.

<!-- prevnext:start -->

---

| [&larr; Previous: Just Another Good, Old Operation](09-just-another-good-old-operation) | [Next: Module Outro &rarr;](11-module-outro) |
|:---|---:|

<!-- prevnext:end -->
