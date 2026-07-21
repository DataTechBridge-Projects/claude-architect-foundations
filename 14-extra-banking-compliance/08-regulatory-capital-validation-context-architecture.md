---
title: Regulatory Capital Validation - Context Architecture
parent: "Extra Module: Solution Architecture Patterns - Banking Compliance"
nav_order: 8
duration: "6:40"
---

# Regulatory Capital Validation - Context Architecture
{: .no_toc }

*Video duration: 6:40*

A validation verdict is only as trustworthy as the anchors it's checked against. Get either of the two core reference points wrong — the methodology or the threshold — and every downstream comparison inherits that error while still looking rigorous on its face. Context architecture for capital validation is built around keeping those anchors correct, current, and clearly separated from the specific calculation being examined.

## The Three Things That Have to Be in Context

**Calculation methodology documentation** is the regulatory text and its internal implementation guidance describing how a given capital calculation is supposed to work: exposure classification rules, risk-weight tables, mitigant treatment, and how components aggregate into a final ratio. This is the standard the calculation is being checked against, and it has to be the *current* version — regulatory methodology is revised periodically, sometimes with transition provisions that phase in over several reporting periods, and a validation run against superseded methodology produces a verdict that's confidently wrong in exactly the way this domain can least afford.

**Current regulatory thresholds** are the specific numeric minimums the calculated ratios have to clear — the CET1 minimum, the Tier 1 minimum, any applicable buffers such as the capital conservation buffer or a bank-specific add-on set by its supervisor. These are distinct from methodology in an important way: methodology describes *how to calculate*, thresholds describe *what counts as passing*, and a bank's specific threshold requirements can differ from the generic regulatory minimum based on its own supervisory history. Context has to carry the bank's actual, current applicable thresholds, not just the generic published minimums, or a technically correct calculation could be validated against the wrong bar.

**The specific calculation being validated** is the artifact under examination itself: which exposures were included, how they were classified, what risk weights were applied, what mitigants were recognized, and how the components were aggregated into the reported figure. This is what gets compared against the other two anchors, step by step, as described in the agentic-architecture lecture — and it has to be presented in context with enough structural detail that each step is individually checkable, not just as a final number with no visible working.

## Keeping the Anchors From Drifting Apart

The core context-architecture risk here mirrors the stress-testing use case's cycle-versioning problem, but with a twist specific to validation: methodology and thresholds don't necessarily change on the same schedule as each other, and a calculation being validated might itself have been performed under an earlier methodology version that was current at the time. That means context assembly can't simply grab "the latest version of everything" — it has to determine which methodology version was actually in force when the calculation was performed, and validate against *that* version, unless the specific purpose of the check is to confirm the calculation has been updated to reflect a newer requirement. Getting this wrong in either direction produces a plausible-looking but meaningless verdict: checking a properly-performed historical calculation against today's rules will manufacture false discrepancies, while checking a calculation that should have been updated against outdated rules will miss a real one.

This is why every methodology document and threshold value entering context carries an explicit effective-date range, and why the validation request itself has to specify which reporting period's calculation is under review — that period determines which methodology version is the correct anchor, and the context-assembly layer resolves that mapping deterministically rather than leaving it to inference.

## Scoping Context to a Single Calculation

Unlike stress testing, where context often has to span an entire portfolio's worth of related figures at once, capital validation context is deliberately narrow: a single calculation, its specific relevant methodology provisions, and the applicable thresholds — not the entire regulatory corpus, and not every other calculation the bank has ever validated. This narrowness is a deliberate context-management choice, not a limitation. Pulling in adjacent calculations or unrelated methodology sections doesn't help the comparison and does add noise that can dilute the model's attention on the specific provisions that actually govern the calculation at hand. Related historical validation results for the same calculation type are useful context in a different, clearly labeled role — as background on whether this calculation has a history of discrepancies — but they're kept separate from the current comparison rather than blended into it.

## Architecture Notes

This lecture extends the **Context** layer's provenance discipline from the stress-testing use case into a second dimension: it's not enough to track *which cycle* a document belongs to, the system also has to track *which version was in force when the thing being checked was performed*, and resolve that deterministically rather than defaulting to "latest." Keeping context scoped to a single calculation, with related history clearly labeled rather than blended in, is the same context-isolation principle that shows up across the course wherever noise competes with signal for the model's attention.
