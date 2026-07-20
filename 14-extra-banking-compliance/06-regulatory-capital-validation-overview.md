---
title: Regulatory Capital Validation - Overview
parent: Extra Module: Solution Architecture Patterns - Banking Compliance
nav_order: 6
duration: "6:59"
---

# Regulatory Capital Validation - Overview
{: .no_toc }

*Video duration: 6:59*

Regulatory capital — the buffer a bank is required to hold against its risk-weighted assets under frameworks like Basel III, and reported through calculations such as CET1, Tier 1, and total capital ratios — is one of the most consequential numbers a bank produces. It's calculated by combining risk-weighted asset figures across credit, market, and operational risk with a bank's actual capital resources, following a calculation methodology that's dense, detailed, and updated periodically by regulators. Getting that calculation right is not optional in any casual sense: a miscalculated capital ratio can mean a bank believes itself better capitalized than it is, can trigger a regulatory finding if discovered during examination, and in the worst case can mean a bank is operating below a threshold it was legally required to maintain without anyone realizing it.

## The Business Problem

Banks don't calculate regulatory capital with a single formula run once. The calculation methodology spans hundreds of pages of regulatory text, covering how to classify exposures, which risk weights apply to which asset categories, how to treat mitigants like collateral and guarantees, and how various adjustments and floors interact. Internal teams translate that text into calculation engines and spreadsheet-based processes, and — this is the crux of the problem this use case addresses — those internal implementations have to be periodically validated against the letter of the regulation to confirm they still match. Regulations change. Internal systems get modified for other reasons and can drift from the methodology they were originally built to implement. A validation function exists specifically to catch that drift: checking whether a given calculation, as actually performed, follows the current regulatory methodology, and flagging anywhere it doesn't for a human specialist to investigate.

This is a fundamentally different task from stress testing, even though both live in the same regulatory-compliance space. Stress testing is a generative, drafting-heavy exercise built around producing a new artifact — a projection and a narrative — under scenario assumptions that don't yet have a "correct answer" to check against. Capital validation is a verification exercise: there's an existing calculation, an existing regulatory requirement, and the task is to determine whether the former correctly implements the latter. That distinction shapes everything about how the two systems are architected, and it's why this use case gets its own agentic design in the next lecture rather than reusing the stress-testing pattern.

## Why Correctness and Explainability Matter More Than Speed

A capital validation system that's fast but wrong is worse than useless — it's actively dangerous, because a false "this checks out" on a miscalculated ratio removes the very scrutiny that might have caught the error through some other channel. That asymmetry means the system has to be evaluated primarily on precision and explainability, not throughput. A discrepancy the system flags needs an explanation a human specialist can evaluate on its merits: which specific regulatory provision applies, what the current calculation actually did, and precisely where the two diverge. A system that flags "something seems off" without pointing to a specific rule and a specific calculation step isn't useful — it just shifts the entire investigative burden back onto the human, at which point the system has added review overhead rather than removed it.

Explainability also matters because the audience for a flagged discrepancy is rarely the person who built the calculation being checked. It's typically a validation specialist, an internal auditor, or ultimately an examiner — someone who needs to understand *why* the system believes there's a mismatch well enough to independently verify the claim, not someone who's inclined to simply trust an automated flag. A validation system whose reasoning can't be inspected and checked step by step doesn't meet the bar this domain requires, regardless of how accurate it turns out to be in aggregate.

## The High-Level Shape of the Solution

The system this use case describes works from two anchors that always have to be present together: the current regulatory methodology (what the calculation *should* do) and the specific calculation actually being validated (what it *did* do). Claude's role is to compare the two systematically — walking through the calculation's key steps, checking each against the relevant provision of the current methodology, and producing a structured output that either confirms alignment or flags a specific, explained discrepancy for human review. Nothing is auto-corrected and nothing is auto-approved; every flagged discrepancy and every clean pass alike routes to a human validator who makes the actual determination, with the system's output serving as a well-organized starting point for that review rather than a substitute for it.

The remaining lectures in this use case cover how that verification-oriented agentic design differs from a generative drafting agent, what has to be in context for a single validation check to be trustworthy, how retrieval is built to surface the current regulatory text alongside internal interpretations without letting the two get confused with each other, and how the system logs and tracks its own accuracy over time.

## Architecture Notes

Regulatory capital validation is a useful counterpoint to stress testing within the same **Agentic** layer vocabulary: both use cases keep autonomy low and route everything through human review, but where stress testing is a generation task assisted by Claude, this one is fundamentally a verification task, which changes what "good" output looks like at the **Prompt** and **Workflow** layers. The requirement that every flag be explainable back to a specific regulatory provision is also a **Context** layer discipline — the system's usefulness depends entirely on always having the correct, current methodology text anchored alongside the calculation it's checking.
