---
title: "Plan Mode: When and Why to Plan First"
parent: Claude Code Deep Dive
nav_order: 15
duration: "10:00"
---

# Plan Mode: When and Why to Plan First
{: .no_toc }

*Video duration: 10:00*

The permission-modes lecture introduced Plan Mode as a variant that shifts approval from "each action, one at a time" to "the whole approach, up front." This lecture is dedicated to that mode specifically, because deciding *when* to reach for it — and how to actually iterate on a plan once one exists — is a distinct skill from just knowing the mode exists.

## What Actually Happens in Plan Mode

Entering Plan Mode changes what Claude is allowed to do without changing what it's allowed to *look at*: it can read files, search the codebase, and run read-only commands to investigate freely, but no edit happens, no matter how confident the model becomes partway through, until a plan is presented and explicitly approved. The output of this phase isn't code — it's a description of an intended approach: which files will change, roughly how, in what order, and why. Only once that description is approved does execution actually begin, typically switching into a more permissive mode for the edit phase itself, since the plan review already served as the approval gate for the *approach* — re-litigating each individual edit at that point would just be redundant oversight over a decision already made.

## The Case For Planning First

The value of this sequencing is easiest to see by contrasting it with what happens without it. In an ordinary default-mode session, uncertainty about the right approach only surfaces edit by edit — Claude tries something, you approve or reject that specific change, and if the overall direction turns out to be wrong, you discover that fact partway through a sequence of edits, some of which may already need to be undone. Plan Mode front-loads that discovery: if the *approach* is wrong, you find out before a single file has changed, when correcting course costs nothing more than saying so and asking for a different plan.

This matters most exactly where uncertainty is highest. A genuinely ambiguous task — "improve error handling in the payment service" — could reasonably be approached several different ways (centralizing error handling in one wrapper, adding validation at each call site, introducing a new error type hierarchy), and those approaches aren't equally good, aren't equally invasive, and aren't equally reversible once underway. Seeing the intended approach stated plainly, before any of it exists as a diff, is exactly where a human's judgment adds the most value relative to its cost — one read of a plan versus reviewing a dozen individual edits that might all need to be reworked if the underlying direction was wrong.

The same logic applies to genuinely high-risk changes, independent of ambiguity: a schema migration, a change to authentication logic, or a refactor touching a widely-depended-on shared module all benefit from a reviewable plan up front, even when the *approach* isn't particularly uncertain, simply because the cost of getting it wrong is high enough that seeing the intended shape of the change before it exists is worth the extra round trip.

## The Case Against Planning First

Plan Mode is not free, and applying it uniformly to every task is a real anti-pattern, not a conservative default with no downside. For a small, well-understood, low-risk task — fixing an obviously wrong string, adding a straightforward test case, a one-line configuration change — there usually isn't a meaningful "approach" question to review at all; the plan would just restate the edit that's about to happen, adding a review step that reviews nothing genuinely uncertain. In that situation, planning first is pure overhead: an extra round trip that delays getting to the actual, uncontroversial change, without buying back any of the risk-mitigation value that justifies the overhead for a harder task.

The practical judgment call, then, mirrors the oversight-versus-velocity tradeoff from the permission-modes lecture directly: reach for Plan Mode when either the *approach* is genuinely uncertain or the *stakes* of getting it wrong are genuinely high, and skip it when neither is true, since the round trip buys nothing in that case beyond friction.

## Iterating on a Plan

A plan, once presented, isn't a binary accept-or-reject proposition — the workflow supports genuine iteration, and using that iteration well is most of what separates a team that finds Plan Mode valuable from one that treats it as a rubber stamp. Reviewing a presented plan, you can:

- **Approve it as written**, moving into execution exactly as proposed.
- **Edit the plan directly** before approving — correcting a detail, reordering steps, or removing part of the proposed scope — so execution proceeds against your corrected version rather than the model's original one.
- **Reject it and ask for a revision**, staying in Plan Mode while explaining what's wrong with the proposed approach, so Claude proposes a new plan informed by that specific feedback rather than starting from scratch or guessing at what you meant.

This loop — propose, review, correct, re-propose — is the actual mechanism by which Plan Mode earns its keep on genuinely ambiguous tasks. A single round of "here's a plan, approved" captures much less value than a round or two of "here's a plan; no, don't touch the public API, keep the change internal to this module; here's a revised plan that respects that" before execution begins. Treating the first proposed plan as something to rubber-stamp rather than something to actually push back on gives up most of the benefit Plan Mode exists to provide.

## Connecting Back to Workflow Safeguarding

Plan Mode is a second, distinct instance of the Agentic Layer's approval-gate concept, sitting at a different point in the loop than the ordinary permission prompts covered in the permission-modes lecture. Where a default-mode permission prompt gates one action at a time, Plan Mode gates an entire intended sequence at once, before any of it is committed — a checkpoint scoped to the *plan* rather than to each individual step within it. Recognizing that these are two genuinely different kinds of gate, suited to different situations, rather than two interchangeable ways of saying "ask before doing," is the core distinction this lecture is building.

**Why it matters for the exam:** Expect a scenario question presenting a task and asking whether Plan Mode is warranted — the correct reasoning weighs ambiguity of approach and stakes of being wrong against the real overhead of an extra review round, rather than treating "more oversight" as automatically better. Also expect a question on what iterating on a plan actually involves, since treating an initial plan as a rubber stamp rather than a genuine draft is a common, testable misunderstanding.
