---
title: Regulatory Capital Validation - Agentic Architecture
parent: "Extra Module: Solution Architecture Patterns - Banking Compliance"
nav_order: 7
duration: "8:26"
---

# Regulatory Capital Validation - Agentic Architecture
{: .no_toc }

*Video duration: 8:26*

Every other use case in this course, including stress testing earlier in this module, asks an agent to *produce* something — a draft, a thesis, a plan. Regulatory capital validation asks something different: it asks Claude to *check* something that already exists. That distinction sounds subtle, but it changes the agentic architecture from top to bottom, because a checking task has a different failure mode, a different notion of success, and a different relationship between model output and human trust than a generative one does.

## Verification Is a Different Cognitive Posture Than Generation

A generative agent's job is to produce a plausible, well-supported artifact and let a human evaluate whether it's good enough. Its natural failure mode is producing something fluent but wrong — the review gate exists to catch fluency masquerading as correctness. A verification agent's job is structurally inverted: it's given an existing artifact (a calculation) and an existing standard (the regulatory methodology) and has to determine, as rigorously as possible, whether the former satisfies the latter. Its natural failure mode isn't producing something implausible-sounding — it's failing to notice a genuine discrepancy (a false negative) or raising a discrepancy that isn't actually one (a false positive), both of which look identical to a generative hallucination from the outside but come from a different place: not "invented content" but "faulty comparison."

That difference matters architecturally because it changes what the agent's core loop actually does. Rather than an open-ended reasoning process converging on a novel output, the agent's loop here is closer to a structured audit: decompose the calculation into its constituent steps (exposure classification, risk-weight assignment, mitigant treatment, aggregation, ratio computation), retrieve the specific regulatory provision governing each step, compare the calculation's actual treatment against that provision, and record a verdict — match, mismatch, or insufficient information to determine — for each step individually rather than for the calculation as a whole. Working at the level of individual steps rather than a single holistic judgment is itself an agentic design decision: it keeps each comparison narrow enough to be checkable by a human reviewer, and it means a single ambiguous step doesn't force an all-or-nothing verdict on an otherwise-clean calculation.

## Why This Agent's Role Differs From a Drafting Agent

A drafting agent, like the one in the stress-testing use case, is judged by how much high-quality output it produces relative to the human effort of writing it from scratch — its value is in compression. A verification agent's value proposition is entirely different: it's judged by how reliably it surfaces the discrepancies that matter and how rarely it either misses one or cries wolf on one that isn't real. That means the tuning target during development is different too. A drafting agent gets iterated toward better prose and better synthesis. A verification agent gets iterated toward calibration — adjusting how conservatively it flags ambiguous cases, refining how it decomposes calculations into checkable steps, and specifically tracking (as the observability lecture in this use case covers) its own false-positive and false-negative rates over time, because those two numbers are the actual measure of whether the system is doing its job.

This also means the agent should be explicitly instructed and structured to resist a specific temptation generative models have: filling gaps with plausible-sounding inference. When a calculation's documentation doesn't clearly show how a particular exposure was classified, a drafting agent's instinct — trained into it by everything else it does — is to make a reasonable guess and move forward smoothly. A verification agent has to do the opposite: when the evidence needed to confirm or deny a match isn't available, the correct output is an explicit "insufficient information to verify this step," not a confident guess in either direction. Architecting for that means the agent's output schema has to include that third state as a first-class option alongside "match" and "mismatch" — not a workaround, but a designed part of what a complete verdict looks like in this domain.

## The Review Boundary

As with stress testing, nothing here is auto-resolved. But the review boundary sits in a different place than it does for a generative task. In the stress-testing use case, a human reviews a draft before it's used. Here, a human reviews the agent's *verdict* — including, importantly, its clean passes, not only its flagged discrepancies, at least on a sampled basis. That matters because a verification system's most dangerous failure mode is a false negative: a real discrepancy the system waves through as a match. A review process that only spot-checks flagged items and never samples clean passes has no way to catch that failure mode at all, which is why periodic sampled review of "no discrepancy found" verdicts belongs in the workflow design alongside full review of every flagged item.

The agent's authority, in other words, is bounded not just by "don't finalize anything" — the same boundary stress testing has — but by a second, more specific boundary: don't let a verdict of "clean" escape scrutiny just because it's the less exciting outcome. Both boundaries exist because the cost of a missed capital-calculation error is at least as severe as the cost of a bad stress-test submission, and in some respects worse, since it can mean the bank has been operating with a mistaken understanding of its own capital position.

## Architecture Notes

This use case shows the **Agentic** layer's vocabulary applying to a task shape the course hasn't emphasized elsewhere: low autonomy here isn't just about capping what the agent can finalize, it's about designing a structured, step-by-step verification loop whose natural failure modes (false positive, false negative, unwarranted inference) are different from a generative agent's. The decomposition into individually checkable steps is also a **Workflow** layer decision that keeps human review scoped and fast, while the requirement to sample clean verdicts, not just flagged ones, is a **Context** and **Workflow** discipline aimed squarely at catching the false-negative failure mode that matters most in this domain.
